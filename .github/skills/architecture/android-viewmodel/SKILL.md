---
name: android-viewmodel
description: Best practices for implementing Android ViewModels using Kotlin 2.3+ Explicit Backing Fields, StateFlow for UI state, and SharedFlow for one-off events.
---

# Android ViewModel & State Management

## Instructions

Use `ViewModel` to own UI state and business logic. It should survive configuration changes and expose immutable APIs.

> **Kotlin 2.3+**
>
> Prefer **Explicit Backing Fields** instead of the traditional `_state`/`state` backing property pattern.
>
> - Kotlin 2.3: Experimental (`-Xexplicit-backing-fields`)
> - Kotlin 2.4+: Stable (recommended)
>
> Avoid creating duplicate backing properties unless targeting older Kotlin versions.

---

## 1. UI State (StateFlow)

### Purpose

Represents the persistent UI state.

Examples:

- Loading
- Success(data)
- Error(message)
- Empty

### Declaration

Use an explicit backing field.

```kotlin
val uiState: StateFlow<UiState>
    field = MutableStateFlow(UiState.Loading)
```

The backing field is mutable (`MutableStateFlow`) while consumers only see `StateFlow`.

### Updating State

Prefer `update { }` for atomic updates.

```kotlin
_uiState.update { old ->
    old.copy(isLoading = false)
}
```

With explicit backing fields:

```kotlin
uiState.update {
    it.copy(isLoading = false)
}
```

You may also assign directly when appropriate.

```kotlin
uiState.value = UiState.Success(data)
```

---

## 2. One-Off Events (SharedFlow)

### Purpose

Use for transient events that should not survive recomposition or configuration changes.

Examples:

- Navigate
- Show Snackbar
- Show Toast
- Open Dialog
- Request Permissions

### Declaration

```kotlin
val uiEvent: SharedFlow<UiEvent>
    field = MutableSharedFlow(
        replay = 0,
        extraBufferCapacity = 1
    )
```

`replay = 0` ensures events are **not replayed** after recreation.

Using `extraBufferCapacity = 1` is recommended for UI events sent with `tryEmit()`.

### Sending Events

Suspend:

```kotlin
uiEvent.emit(UiEvent.NavigateBack)
```

Non-suspending:

```kotlin
uiEvent.tryEmit(UiEvent.ShowSnackbar("Saved"))
```

---

## 3. Collecting in Compose

### StateFlow

Always collect lifecycle-aware.

```kotlin
val state by viewModel.uiState.collectAsStateWithLifecycle()
```

### SharedFlow

Collect inside a single `LaunchedEffect`.

```kotlin
LaunchedEffect(Unit) {
    viewModel.uiEvent.collect { event ->
        when (event) {
            is UiEvent.NavigateBack -> { /* ... */ }
            is UiEvent.ShowSnackbar -> { /* ... */ }
        }
    }
}
```

Do **not** collect `SharedFlow` with `collectAsState()`.

---

## 4. Collecting in XML/View System

Use lifecycle-aware collection.

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect {
            render(it)
        }
    }
}
```

Likewise for events.

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiEvent.collect(::handleEvent)
    }
}
```

---

## 5. Coroutine Scope

Launch all ViewModel work in `viewModelScope`.

```kotlin
viewModelScope.launch {
    repository.refresh()
}
```

Business logic should preferably be delegated to UseCases or Repositories.

---

## 6. State Design

Represent the entire screen with a single immutable state object.

```kotlin
data class UiState(
    val isLoading: Boolean = false,
    val items: List<Item> = emptyList(),
    val error: String? = null
)
```

Avoid exposing multiple unrelated `StateFlow`s for one screen unless they truly have different lifecycles.

---

## 7. Best Practices

Expose immutable APIs (`StateFlow`, `SharedFlow`)

Prefer Explicit Backing Fields (Kotlin 2.4+)

Use immutable UI state

Use `update {}` when modifying state

Keep one-off events in `SharedFlow`

Keep business logic out of Composables

Use `collectAsStateWithLifecycle()`

Use `repeatOnLifecycle()` in Views

Launch work in `viewModelScope`

Do not expose `MutableStateFlow`

Do not use `StateFlow` for navigation events

Do not use `SharedFlow` for persistent screen state

Do not keep duplicate `_state` properties when Explicit Backing Fields are available
