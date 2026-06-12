# KMP Payment Architecture (Kotlin Multiplatform)

## Instructions

When building or extending a **payment feature** in a **Kotlin Multiplatform (KMP)** project that targets both Android and iOS, follow this architecture. It combines **Clean Architecture** layers with **Koin** scoped DI and a **custom MVI Reducer** pattern — all inside `commonMain` with zero platform-specific imports.

---

### 1. Module Structure

Organise each payment sub-feature (`bankOtp`, `napas`, `payx`, etc.) with strict layer separation:

```
payment/
├── <feature>/
│   ├── core/            # Platform bridge API interfaces (composeApi/)
│   ├── data/
│   │   ├── di/          # Koin data-layer modules
│   │   ├── model/       # Network / persistence DTOs
│   │   └── repositoryImpl/
│   ├── domain/
│   │   ├── entities/    # Pure Kotlin domain models
│   │   ├── repositories/ # Repository interfaces
│   │   └── useCases/
│   └── presentation/
│       ├── model/       # Presentation params / UI models
│       └── viewModel/
│           ├── reducer/ # State / Action / Effect definitions
│           └── <Feature>VM.kt
├── base/                # Shared core across all sub-features
│   ├── core/composeApi/ # INavigator, ILogger, IRouteApi, etc.
│   ├── di/
│   └── presentation/viewModel/reducer/Reducer.kt
└── di/
    ├── PaymentKoinModule.kt       # Root lazy module
    ├── PaymentViewModelKoinModule.kt
    └── PaymentKoinScope.kt        # Scoped session management
```

**Rules:**
- `domain/` must contain **only pure Kotlin** — no `android.*`, no `kotlinx.coroutines.android`, no Compose imports.
- `data/` and `presentation/` may use Koin, kotlinx.coroutines, and Compose Multiplatform APIs.
- Platform-specific behaviour lives behind interfaces in `base/core/composeApi/` (e.g. `INavigator`, `ISmsApi`).

---

### 2. Dependency Injection with Koin (not Hilt)

Use **Koin** — not Hilt — because Hilt requires the Android framework and cannot compile in `commonMain`.

#### Root lazy module

```kotlin
@OptIn(KoinExperimentalAPI::class)
fun paymentModule(environment: String) = lazyModule {
    includes(
        payXCoreModule(),
        payXDataModule(),
        paymentUseCaseModule(),
        paymentPresentationModule(environment)
    )
}
```

- Use `lazyModule { }` at the root to defer initialisation until first use.
- Sub-modules use `module { }` with `factory { }`, `single { }`, or `scopedOf()`.

#### Scoped sessions

Payment flows are session-scoped (one scope per checkout). Manage lifecycle explicitly:

```kotlin
// Start a scoped session before navigating into payment
PaymentKoinScope.start(requestPayXData)

// Inside DI, retrieve the active scope
val scope = PaymentKoinScope.getScope(PaymentKoinScope.PAYX)

// When the flow ends, close the scope to release all scoped instances
PaymentKoinScope.end(checkoutId)
```

Scopes are linked: a `payx` scope is linked to a `payment_base` scope so base dependencies are visible:

```kotlin
val baseScope = koin.getOrCreateScope("payment_base-$id", named(PAYMENT_BASE))
val scope     = koin.getOrCreateScope(id, named(PAYX))
scope.linkTo(baseScope)
```

---

### 3. MVI with Custom Reducer

Each ViewModel drives state via an immutable **Reducer** — there is no `viewModel.update {}` shorthand.

#### Reducer interface (base)

```kotlin
interface Reducer<State : Reducer.State, Action : Reducer.Action, Effect : Reducer.Effect> {
    interface State
    interface Action
    interface Effect

    fun reduce(prev: State, action: Action): Pair<State, Effect?>
}
```

#### Implementing a Reducer

```kotlin
class FeatureReducer :
    Reducer<FeatureReducer.State, FeatureReducer.Action, FeatureReducer.Effect> {

    @Immutable
    data class State(
        val isLoading: Boolean = false,
        val result: String = ""
    ) : Reducer.State

    @Immutable
    sealed class Action : Reducer.Action {
        data object StartLoad : Action()
        data class SetResult(val value: String) : Action()
    }

    @Immutable
    class Effect : Reducer.Effect  // extend if one-off side-effects are needed

    override fun reduce(prev: State, action: Action): Pair<State, Effect?> = when (action) {
        is Action.StartLoad    -> prev.copy(isLoading = true) to null
        is Action.SetResult    -> prev.copy(isLoading = false, result = action.value) to null
    }
}
```

#### ViewModel base

```kotlin
class FeatureVM(
    private val navigator: INavigator,      // platform bridge — injected via Koin
    private val repository: IFeatureRepository,
) : ViewModel<FeatureReducer.State, FeatureReducer.Action, FeatureReducer.Effect>(
    FeatureReducer.State(),
    FeatureReducer()
) {
    fun loadData() {
        viewModelScope.launch(Dispatchers.IO) {
            dispatch(FeatureReducer.Action.StartLoad)
            val result = repository.fetch()
            dispatch(FeatureReducer.Action.SetResult(result))
        }
    }
}
```

- Call `dispatch(action)` — **never** mutate `_uiState` directly.
- Use `Dispatchers.IO` for all repository calls; `Dispatchers.Main` is the default for state updates.
- Expose one-off events as `Flow`/`SharedFlow` on the ViewModel, **not** as `Effect` on the Reducer (effects are reserved for synchronous reducer side-effects).

---

### 4. Platform Bridge Interfaces

All platform-specific capabilities are injected via interfaces defined in `base/core/composeApi/`. Never call platform APIs directly from `commonMain`.

| Interface | Responsibility |
|-----------|---------------|
| `INavigator` | Push/pop/dismiss screens |
| `ILogger` | Logging (maps to platform loggers) |
| `IRouteApi` | Deep-link / external URL opening |
| `ISmsApi` | SMS/OTP listening (Flow-based) |

Example usage in a ViewModel:

```kotlin
fun getDOtp() {
    viewModelScope.launch {
        routeApi.openUrl(params.otpURL)   // platform opens the bank's D-OTP URL
    }
}

fun listenSmsOtp(maxLength: Int) {
    CoroutineScope(Dispatchers.IO).launch {
        smsApi.listenOTP(maxLength)
            .catch { e -> logger.i("OTP listen error: ${e.message}") }
            .collect { otp ->
                if (otp.length == maxLength) onChangeOtp(otp)
            }
    }
}
```

---

### 5. Checklist

- [ ] `domain/` layer has **no** `android.*` or platform imports.
- [ ] All ViewModels extend the shared `ViewModel<State, Action, Effect>` base class.
- [ ] State is mutated **only** through `dispatch(action)`.
- [ ] State, Action, and Effect classes are annotated with `@Immutable`.
- [ ] Koin modules use `lazyModule` at root; sub-modules use `factory`/`single`/`scopedOf`.
- [ ] Payment session scope is explicitly `start()`-ed and `end()`-ed around the checkout flow.
- [ ] Platform capabilities are injected via `INavigator`, `ILogger`, `IRouteApi`, or `ISmsApi` — never called directly.
- [ ] `viewModelScope.launch(Dispatchers.IO)` is used for all suspend repository calls.
