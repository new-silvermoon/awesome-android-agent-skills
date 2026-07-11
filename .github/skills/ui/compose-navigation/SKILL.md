---
name: navigation3
description: Implement navigation in Jetpack Compose using Navigation 3. Use when asked to build state-driven navigation, manage back stacks, scope ViewModels, implement adaptive layouts, or migrate from Navigation Compose.
---

# Navigation 3

## Overview

Implement state-driven navigation in Jetpack Compose using Navigation 3. Unlike Navigation Compose, Navigation 3 models navigation as application state rather than through a `NavController`. This skill covers navigation keys, back stack management, ViewModel scoping, entry decorators, adaptive layouts, deep links, animations, state restoration, and testing.

## Setup

Add the Navigation 3 dependencies:

```kotlin
// build.gradle.kts
dependencies {

    implementation("androidx.navigation3:navigation3-runtime:1.0.0-alpha08")
    implementation("androidx.navigation3:navigation3-ui:1.0.0-alpha08")

    // Lifecycle integration
    implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.9.2")

    // Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.8.1")
}

// Enable serialization plugin
plugins {
    kotlin("plugin.serialization") version "2.2.0"
}
```

---

## Core Concepts

### 1. Define Navigation Keys

Navigation destinations are represented by immutable serializable objects.

```kotlin
import kotlinx.serialization.Serializable

@Serializable
data object Home

@Serializable
data class Profile(
    val userId: String
)

@Serializable
data class Product(
    val productId: String,
    val showReviews: Boolean = false
)

@Serializable
data object Settings
```

Keys become your navigation state.

---

### 2. Create the Back Stack

Navigation 3 replaces `NavController` with a mutable state back stack.

```kotlin
@Composable
fun MyApp() {

    val backStack = remember {
        mutableStateListOf<Any>(Home)
    }

    AppNavDisplay(backStack)
}
```

---

### 3. Create NavDisplay

```kotlin
@Composable
fun AppNavDisplay(
    backStack: SnapshotStateList<Any>
) {

    NavDisplay(
        backStack = backStack,
        onBack = {
            if (backStack.size > 1) {
                backStack.removeLast()
            }
        }
    ) { key ->

        when (key) {

            Home ->
                HomeScreen(
                    onProfileClick = {
                        backStack += Profile(it)
                    }
                )

            is Profile ->
                ProfileScreen(
                    userId = key.userId
                )

            is Product ->
                ProductScreen(
                    productId = key.productId,
                    showReviews = key.showReviews
                )

            Settings ->
                SettingsScreen()
        }
    }
}
```

---

## Navigation Patterns

### Navigate Forward

```kotlin
backStack += Profile("user123")
```

### Navigate Back

```kotlin
backStack.removeLast()
```

### Replace Current Screen

```kotlin
backStack[backStack.lastIndex] = Home
```

### Clear Back Stack

```kotlin
backStack.clear()
backStack += Home
```

### Pop To Root

```kotlin
while (backStack.size > 1) {
    backStack.removeLast()
}
```

---

## Argument Handling

### Retrieve Arguments

Arguments already exist inside the navigation key.

```kotlin
when (val key = currentKey) {

    is Profile -> {
        ProfileScreen(
            userId = key.userId
        )
    }

    is Product -> {
        ProductScreen(
            productId = key.productId
        )
    }
}
```

### Pass IDs, Not Objects

```kotlin
// CORRECT
backStack += Profile(user.id)

// Fetch object inside ViewModel
class ProfileViewModel(
    savedStateHandle: SavedStateHandle,
    repository: UserRepository
) : ViewModel() {

    val profile = savedStateHandle.toRoute<Profile>()

    val user =
        repository.getUser(profile.userId)
}
```

```kotlin
// INCORRECT

backStack += User(...)
backStack += ProductRepository(...)
backStack += ProductViewModel(...)
```

---

## ViewModel Integration

Navigation 3 scopes ViewModels using entry decorators.

```kotlin
NavDisplay(

    backStack = backStack,

    entryDecorators = listOf(

        rememberSceneSetupNavEntryDecorator(),

        rememberSavedStateNavEntryDecorator(),

        rememberViewModelStoreNavEntryDecorator()
    )

) { key ->

    // destinations
}
```

### ViewModel Example

```kotlin
class ProfileViewModel(
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    val profile =
        savedStateHandle.toRoute<Profile>()
}
```

---

## Entry Decorators

Navigation 3 uses decorators to attach lifecycle functionality.

### Scene Setup

```kotlin
rememberSceneSetupNavEntryDecorator()
```

Creates the navigation scene for each entry.

### Saved State

```kotlin
rememberSavedStateNavEntryDecorator()
```

Automatically restores destination state after process recreation.

### ViewModel Store

```kotlin
rememberViewModelStoreNavEntryDecorator()
```

Scopes ViewModels to each navigation entry.

---

## Adaptive Navigation

Navigation 3 integrates with Material Adaptive layouts.

```kotlin
NavDisplay(

    backStack = backStack,

    sceneStrategy = rememberListDetailSceneStrategy()

)
```

Use adaptive scene strategies to automatically switch between:

- Single pane (phones)
- Two pane (tablets)
- Foldables

---

## Deep Links

Deep links should resolve into navigation keys.

```kotlin
fun handleDeepLink(uri: Uri) {

    val userId =
        uri.lastPathSegment ?: return

    backStack += Profile(userId)
}
```

Avoid manually constructing route strings.

---

## Animations

Navigation transitions are defined using scene transitions.

```kotlin
NavDisplay(

    backStack = backStack,

    transitionSpec = {

        fadeIn() togetherWith fadeOut()

    }
)
```

Navigation 3 animation APIs may evolve while in alpha.

---

## State Restoration

Navigation keys are serializable and automatically restored.

```kotlin
val backStack = rememberSaveable(
    saver = navBackStackSaver()
) {
    mutableStateListOf(Home)
}
```

Always ensure keys are serializable.

---

## Testing

Navigation becomes simple because it is state-driven.

### Example

```kotlin
@Test
fun navigateToProfile() {

    val backStack =
        mutableStateListOf<Any>(Home)

    backStack += Profile("123")

    assertEquals(
        Profile("123"),
        backStack.last()
    )
}
```

Compose UI tests can verify screen rendering by inspecting the current back stack.

---

## Migration from Navigation Compose

| Navigation Compose | Navigation 3 |
|-------------------|--------------|
| `NavController` | Mutable back stack |
| `NavHost` | `NavDisplay` |
| `navigate()` | `backStack += Key` |
| `popBackStack()` | `removeLast()` |
| String routes | Serializable keys |
| `composable()` | `when(key)` |
| Navigation graph | State-driven destinations |

---

## Recommended Project Structure

```
navigation/

    AppNavigation.kt

    NavigationKeys.kt

    NavigationDisplay.kt

feature/

    home/

    profile/

    settings/
```

---

## Critical Rules

### DO

- Use immutable serializable keys
- Keep navigation state inside Compose
- Pass IDs instead of complex objects
- Scope ViewModels using entry decorators
- Use `rememberSaveable` for state restoration
- Model navigation as observable application state

### DON'T

- Use string routes
- Pass repositories or ViewModels through navigation
- Mutate navigation keys
- Store business objects in the back stack
- Recreate the back stack on recomposition
- Mix Navigation Compose APIs with Navigation 3 APIs

---

## References

- Android Navigation 3 documentation
- Navigation 3 samples
- Lifecycle ViewModel Navigation 3 documentation
- Material 3 Adaptive Navigation documentation
