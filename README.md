# Biocoin Android Coding Style

The Biocoin Android Coding Style is based on the [official Android Kotlin Style Guide](https://developer.android.com/kotlin/style-guide), 
and [Kotlin Language Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html).
Please read the official Style and Convention Guides from Android & Kotlin carefully first. 

We will use [Ktlint, a Kotlin linter with built in formatter](https://ktlint.github.io/) for enforcing the Biocoin Android Coding Style.


# Classes

### Constructors

Initialize the properties of a class via primary constructor parameters instead of using an `init` block.

#### Do:

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```
#### Don't:

```kotlin
class Person(firstName: String, lastName: String, age: Int) {
    val firstName: String
    val lastName: String
    var age: Int

    init {
        this.firstName = firstName
        this.lastName = lastName
        this.age = age
    }

    // ...
}
```

For formatting these primary constructors, follow the Kotlin coding conventions:

>Classes with longer headers should be formatted so that each primary constructor argument is in a separate line with indentation. Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces should be located on the same line as the parenthesis.
>
>For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    // ...
}
```

### Lambdas

Follow the coding conventions:

>In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters from the body. Whenever possible, a lambda should be passed outside of parentheses.
>
>```kotlin
>list.filter { it > 10 }.map { element -> element * 2 }
>```
>In lambdas which are short and not nested, it's recommended to use the `it` convention instead of declaring the parameter explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.

To add to this, lambda parameters (or those in destructured declarations) which are unused should be replaced with an underscore, unless leaving the parameter significantly improves readability. Android Studio will nag you about this anyway:

```kotlin
 .subscribe(
	{ bitmap -> view.displayImage(bitmap) },
	{ _ -> view.setUiState(UiState.FAILURE) })
```

### Lambda Returns
Adding a `return` to a Lambda in Kotlin is optional, and multiline Lambdas will just return the last line. In complex multi-line Lambdas, strongly consider adding a `return@map/flatMap/whatever` statement for the sake of clarity, as the return may not be particularly obvious at first glance.

### Companion Objects
Companion objects, such as those for `Fragment` `newInstance` methods, should be defined at the bottom of a class declaration;

```kotlin
class MyFragment : Fragment() {

    init {
        Injector.INSTANCE.inject(this)
    }

    @Inject
    lateinit var dataManager: DataManager

    fun doSomething() {
        // ...
    }

    companion object {
        const val BUNDLE_VALUE_ONE = "bundle_value_one"

        fun newInstance(value1: String, value2: String): MyFragment {
            // ...
        }
    }
}
```
As a sidenote regarding `val` properties in `companion objects`; accessing them from Java requires accessing the companion object too, which is ugly and not idiomatic:

```java
String key = MyFragment.Companion.BUNDLE_VALUE_ONE;
```

Delcaring the property as a `const val` allows you to access a property as if it was static from Java code:

```java
String key = MyFragment.BUNDLE_VALUE_ONE;
```
This also inlines any access to the `val`. There is a caveat though: this only works with primitives and Strings. For more information, check out [this](https://blog.egorand.me/where-do-i-put-my-constants-in-kotlin/) excellent article regarding constants in Kotlin.

# Functions

### Unit (void) Functions

If a function returns Unit (eg `void` in Java), the return type should be omitted:

```kotlin
fun foo() { // ": Unit" is omitted here
	// ...
}
```

The exception is an empty or stub method, in which case convert the method to an expression body:

```kotlin
fun foo() = Unit
```
With that being said, if stubbing functions prefer using Kotlin's inline `TODO()` function instead to make it obvious that a function is yet to be implemented:

```kotlin
fun foo() {
    TODO("This is yet to be implemented")
}
```
### Expression Bodies

Functions whose bodies are single line should be converted to expression bodies where possible, unless that function returns `Unit`. One exception is passthrough methods, where the caller function is delegating to another class' method. 

Whilst expression bodies allow the omission of the return type, strongly consider keeping it for the sake of readability unless the function quite obviously returns a primitive type such as a `Boolean` or `String`. Whilst it's trivial to work out the return type using an IDE, omitting these types makes code review painful and more error prone.

### Functions vs Properties

You'll often find that the `Convert Java file to Kotlin` intention in Android Studio/IntelliJ will convert some methods into properties. This might be a bit jarring coming from a Java background. However, properties are the idiomatic way of doing many things in Kotlin (no getters/setters, for instance). Follow JetBrain's simple guidelines to see whether a property is appropriate:

>In some cases functions with no arguments might be interchangeable with read-only properties. Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.
>
>Prefer a property over a function when the underlying algorithm:
>
>* does not throw
>* has a O(1) complexity
>* is cheap to calculate (or caсhed on the first run)
>* returns the same result over invocations

### Extension Functions

As you would with creating Java util classes, create an extension package and make separate files for each type:

- extensions
  - ContextExtensions.kt
  - ViewExtensions.kt
  - ...

Extension functions are fun, but don't go overboard. Try not to hide mountains of complexity behind clever extensions.

# Flow Control

### When Statements

To keep `when` statements clean, do not call more than one function from a condition. Prefer to move these functions into a single enclosing function:

#### Do
```kotlin
when (aValue) {
    1 -> doSomethingForCaseOne()
    2 -> doSomethingForCaseTwo()
    3 -> doSomethingForCaseThree()
}

fun doSomethingForCaseTwo() {
    foo()
    bar()
}
```

#### Don't

```kotlin
when (aValue) {
    1 -> doSomethingForCaseOne()
    2 -> {
        foo()
        bar()
    }
    3 -> doSomethingForCaseThree()
}
```
Similarly, because `when` doesn't fall through, separate cases using commas if you wish for multiple cases to be handled the same way:

#### Do
```kotlin
when (aValue) {
    1, 3 -> doSomethingForCaseOneAndThree()
    2 -> doSomethingForCaseTwo()
}
```
#### Don't

```kotlin
when (aValue) {
    1 -> doSomethingForCaseOneAndThree()
    2 -> doSomethingForCaseTwo()
    3 -> doSomethingForCaseOneAndThree()
}
```



# XML

## Naming 

### Non layout files

*Prefixes:*

- `bg_` prefix for all background or foreground resources
- `ic_` prefix for all icons
- `color_` prefix for color resources

*Suffixes:*

- `_selector` :suffix for selectors
- `_ripple` :suffix for ripple variants resources (only for projects with minSdk < 21)
- suffixes for states : Prefixes & suffixes can be combined.

### Layout files

*Prefixes:*

- `activity_` for activity layout
- `fragment_` for fragment layout
- `layout_` for include layouts
- `item_` for recycler view items
- `view_` for custom view

*Common*

All resource names after prefixes described above should also contain feature name.

Examples:

- `activity_` + `restaurants_map.xml`
- `layout_` + `restaurant_header.xml`

```
<string name="restaurant_header_title">Restaurant</string>
<string name="restaurant_header_action_book">Book table</string>
```


### Structure
XML tags should be ordered as follows: 'xmlns' first, then id, then `layout_width` and `layout_height` then alphabetically. 

Add a space between the closing slash and the final attribute. E.g. ```android:textSize="10dp" />```


### Id names
Layout resource ids should use the following naming convention where possible:<br/>
```<layout name>_<object type>_<object name>```<br/>
E.g.
```
home_listview_hotels
hotel_item_imageview_star_rating
```

### Example
Given a layout called profile.xml:
```
<?xml  version="1.0"  encoding="utf-8"?> 
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="match_parent" 
    android:layout_height="match_parent"
    android:orientation="vertical" > 

    <!-- Avatar icon -->
    <ImageView
        android:id="@+id/profile_imageview_avatar" 
        android:layout_width="wrap_content" 
        android:layout_height="wrap_content" />
</LinearLayout> 
```



