# UXMA-Bindings

Eases the development of [MVVM] based software projects running on the [JVM] by providing additional constructs and simplifying wrappers around [RxJava].

## Features

* Clean core library independent of any UI
* Android specific extending library
* Tested and well documented

## Installation

Maven

```xml
<dependency>
    <groupId>com.madesign</groupId>
    <artifactId>mabindings-[core|android]</artifactId>
    <version>0.0.1</version>
</dependency>
```

## Introduction

The basic idea of MaBindings is to provide methods to define reactive properties which are bindable.

### Properties

The idea of properties is driven by two basic interface, namely `ReadableProperty<T>` and `WriteableProperty<T>`. The first one defines the reading methods `get()` and `observe()`, while the second one defines the writing method `set(T)`. Methods to read and write the current state, but also to observe changes in time. We use [RxJava] and its `Observable<T>` construct to represent the stream of value changes.

All existing property implementations inherit from the abstract class `Property<T>`, which extends both interfaces.

### Binder

By extending the property notion by a reactive component using the `Observable<T>`, we can define _bindings_ between properties, which describe the flow of values.

### Validation

## An Example

A simple example of how you would use **MaBindings** in an Android application, by defining your business logic inside a _ViewModel_ and binding the properties and commands to the corresponding UI elements.

**ViewModel.java**
```java
Property<String> email = BackingProperty.create();
Property<String> password = BackingProperty.create();

Property<String> name = ComputedProperty.create(firstName, lastName,
	(first, last) -> first + " " + last);

Validation<String> emailValid = Validation.create(email,
    Validators.email());
Validation<String> passwordValid = Validation.create(password,
    Validators.required());

Event<String> loggedIn = Event.create();
    
Command login = RelayCommand.create(() -> {
	// perform the actual login ...
    if (login(email.get(), password.get())) {
        loggedIn.fire(this, email.get());
    }
}, Validation.collect(emailValid, passwordValid));
```

**Activity.java**
```java
AndroidBinder binder = new AndroidBinder();
ViewModel viewModel = new ViewModel();

@Override
protected void onCreate(Bundle bundleInstance) {
    super.onCreate(bundleInstance);
    
    EditText emailField = (EditText)findViewById(R.id.email_field);
    EditText passwordField = (EditText)findViewById(R.id.password_field);
    Button loginButton = (Button)findViewById(R.id.login_button);
    
    binder.bind(viewModel.email, EditTextProperties.text(emailField),
        BindingType.TWO_WAY);
    binder.bind(viewModel.password, EditTextProperties.text(passwordField),
        BindingType.TWO_WAY);
    binder.bind(viewModel.login, loginButton);

    binder.bind(viewModel.loggedIn, (sender, email) -> {
        Toast.makeText(this, "User " + email + " logged in!", Toast.LENGTH_LONG);
    });
}

@Override
protected void onDestroy() {
	binder.unbindAll();
	super.onDestroy();
}
```

As you can see the `Activity` merely defines the bindings between the `ViewModel` properties and the View properties in a declarative fashion. The real logic happens within the `ViewModel`.

## License

More to come

[MVVM]: http://en.wikipedia.org/wiki/Model_View_ViewModel
[JVM]: http://en.wikipedia.org/wiki/Java_virtual_machine
[RxJava]: https://github.com/Netflix/RxJava
