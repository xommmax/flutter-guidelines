# Flutter Guideline

Hi! this document contains recommendations indicating different policies, procedures or standard on how development
should be conducted.

General coding guideline can be found, in the project here **analysis_options.yaml** file.

## Project code structure

The project code should be structured with a mix of **folder-by-feature** and  **folder-by-type**.

Meaning at the root level you have folders defining features. For example:

```
|-- common/
|-- authentication/
|-- settings/
```

Common/Core feature contains classes, functions or widgets that by default should be available in any features.

So the idea is to have features of your application divided by top level folders.

Think of every top level folders as a separate dart/flutter package, dependencies or access boundary of your
application.

So if you add something to the common folder, you have to ask yourself, will 99.% features of our application need that
?

If you think of it as a dependency, the common dependency is always declared in your pubspec.yaml.

So before adding a class, function or a widget to the common folder, ask yourself:

* Is it something that every feature need in there pubspec.yaml, or it’s more something that’s optional ?
* If the answer is YES than add it to common, if the answer is NO than extract it to another folder.

Inside each feature folder you have folders defined by type.

Here are the folder types that would be part of a feature folder.

```
|-- ui/
|-- cubits/
|-- business_objects/
|-- exceptions/
|-- use_cases
|-- services/
|-- repositories/
|-- data_sources/
|-- dtos/
```

If there's more than one file related to a single file, you can group them in a folder, for example generated code for a
class in a file. For example:

```
|-- authentication/
|---- cubits/
|------ login/
|-------- login_cubit.dart
|-------- login_cubit.freezed.dart
|-------- login_cubit.g.dart
```

## User Interface (UI)

Code that is related to the user's device interface, for example: UI PageBuilders, UI Screens, UI View, UI Components.

### Screen

A screen is a user interface component that fill the whole device display and is the container of user interface view or
component (Button, Checkboxes, Images and ect).

Also, a screen is a specific application navigation destination.

Guideline:

- Screens filename and classes should be suffixed with **Screen**.
- Screens classes should only interact with cubit classes.
- Screen build method should be divided by private widgets that separate its **update regions**/**use case**. For example:

```dart
class LoginScreen extends StatelessWidget {
  //...fields and constructor...
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        _LoginLogo,
        _LoginForm,
        _LoginActions,
        _LoginFooter
      ],
    );
  }
}

// extracted by use case.
class _LoginLogo extends StatelessWidget {}

// extracted by update regions.
class _LoginForm extends StatefullWidget {}

// extracted by update regions.
class _LoginActions extends StatelesssWidget {}

// extracted by update regions.
class _LoginFooter extends StatelessWidget {}
```

- After extracting to private widgets. If the screen file number of lines is greater than 400, 
  you should move the private widgets to a part file. Part file naming convention for screen: 
  ${name of the screen file}_components.dart. For example:

login_screen.dart

```dart
part 'login_screen_components.dart';

class LoginScreen extends StatelessWidget {
  //...fields and constructor...
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        _LoginLogo,
        _LoginForm,
        _LoginActions,
        _LoginFooter
      ],
    );
  }
}

// Is kept here because it's does not break the 400 max line rule.
class _LoginLogo extends StatelessWidget {}
```

login_screen_components.dart

```dart
part of 'login_screen.dart';

/// [LoginScreen]'s fields.
class _LoginForm extends StatelessWidget {}

// ************************ Footer ************************

/// [LoginScreen]'s footer.
class _LoginFooter extends StatelessWidget {}

// ************************* ACTIONS *********************************

/// [LoginScreen]'s actions.
class _LoginActions extends StatelessWidget {}
```

- Do not pass the cubit to screen constructors, instead access them using BlocBuilder, BlocListener or BlocConsumer.
  For example:

```dart
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LoginCubit, LoginCubitState>(
      // Here cubit is not specified either.
      builder: (BuildContext context, LoginCubitState state) {},
    );
  }
}
```

- Use private widget classes instead of functions to build subtrees, for example:

```dart
class SomeWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //...
  }
}

class _SubTree1 extends StatelessWidget {}

class _SubTree2 extends StatelessWidget {}

class _SubTree3 extends StatelessWidget {}

class _SubTree4 extends StatelessWidget {}
```

- A screen action/event callback should be part of the screen class as a method.
  * Action/Event callback method should start with **on** prefix.
  * Name of an action/event should match its use case.
  For example:

```dart
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      childreen: <Widget>[
        Button1(onClick: _openRegisterUser),
        Button2(onClick: _openLogin),
        Field(onTextChanged: _onUserNameTextChanged),
      ],
    );
  }
  
  void _openRegisterUser() {}
  
  void _openLogin() {}
  
  void _onUserNameTextChanged(String newText) {}
}
```

### UI View

A view is a user interface component that does not fill the whole device display and is the container of user interface view or
component (Button, Checkboxes, Images and ect).

Also, a view is not an application navigation destination.

Guideline:

- View filename and classes should be suffixed with **View**.
- View classes should only interact with cubit classes.
- Screen build method should be divided by private widgets that separate its **update regions**/**use case**. For example:


### UI Components

## Cubits

[Cubits](https://pub.dev/packages/flutter_bloc) are classes that contains business logic for your UI, cubit is bound to
a state, that represent the UI state.

Guideline Recommendations:

- Only cover business logic needed for the UI.
- Return Futures for public actions.
- Only interact with high level classes, meaning: Use cases.
- Return final actions result to caller. For example:

 ```dart
 Future<String?> bookAppointment(BookingData booking);

Future<bool> login(String username, String password);
 ```

- Only use **Repositories**, **Services** interfaces not concrete implementations.
- Suffix your class with Cubit, example LoginCubit.
- Cubit class files should be under the cubits folder of the feature.

## Cubit's State.

A cubit state class represent the state of ui bounded to it a given time.

Guideline Recommendations:

- Use a single class for state or define a base class and define subclass for each state, you can use the dart
  meta [sealed](https://api.flutter.dev/flutter/meta/sealed-constant.html) annotation to have you with that.
- Final actions result should not be part of the state. for example:

```dart
class LoginState {
  bool logginSuceeded // avoid this.
}
```

- Keep the class in the same file were the Cubit class is defined.

## Use Cases

Classes that used for making a single business operation/goal/job/task in your domain.

Guidelines:
- UseCase name should always have a **prefix**, which should be a verb, that describes the job that this class is doing. Ex: `class GetCurrentUserUseCase`, `class SignInUseCase`;
- UseCase name should always have a **suffix** `UseCase`;
- UseCase class should always have a public function `call()`, where return type is the result of executing the use case;
- UseCase should have access only to `Repositories`, `Services` or any other `high level coordinators`;
- UseCase **shouldn't** have access to `DataSource` or `Cubit` or interact with dtos or any other low level object;
- UseCase should be used in `Cubit` or in other `UseCases`.

Examples:
```dart
class GetCurrentUserUseCase {
	final UserRepository _repository;
	const GetCurrentUserUseCase(this._repository);
	Future<User?> call() async {
		await _repository.getCurrentUser();
	}
}
```

```dart
class AuthenticateMemberUseCase {
  /// Create a [AuthenticateMemberUseCase].
  const AuthenticateMemberUseCase(
    /* constructor arguments */
  );

  /* ... fields ...*/

  /// Execute the use case.
  Future<TegTask<void>> call(MemberAuthenticationCredentials credentials) {
    return runTaskSafelyAsync<void>(() async {
      final bool isEmailValid = credentials.email.isEmail;

      if (!isEmailValid) {
        throw const TegEmailInvalidException();
      }
      
      final bool isConnected = await _hostDeviceInternetService.isConnected();

      if (!isConnected) {
        throw const TegInternetUnavailableException();
      }

      final String? deviceId = await _hostDeviceInfoRepository.getDeviceId();

      if (deviceId == null) {
        throw const TegDeviceIdUnavailableException();
      }

      final PublicPrivateKeyPair keyPair = await _keyGenerator.generate();

      final Member member = await _authService.signIn(
        email: credentials.email,
        password: credentials.password,
        deviceId: deviceId,
        publicKey: keyPair.publicKey,
      );

      await _updateCurrentMemberUseCase(
        member: member,
        memberPrivateKey: keyPair.privateKey,
        deviceId: deviceId,
      );

      await _saveMemberAuthenticationCredentialsUseCase(credentials);

      final TegTask<List<Account>> memberAccountsTask = await _getMemberAccountsUseCase();

      if (memberAccountsTask.failed) {
        throw memberAccountsTask.exception;
      }

      await _updateMemberCurrentAccountUseCase(
        member: member,
        deviceId: deviceId,
        memberPrivateKey: keyPair.privateKey,
        account: memberAccountsTask.result.first,
      );
    });
  }
}
```

## Repositories

Classes that provide access to data using **only** **CRUD** operations for a specific feature or a scope of a feature,
for example:

```dart
abstract class BookingRepository {
  Futute<Booking> getBookingById(String id);

  Future<List<Booking>> getBookings();

  Future<void> deleteBookingById(String id);

  Future<void> saveBooking(Booking booking);
}
```

Guideline Recommendations:

- Suffix class name with Repository.
- Only define repositories for features or scope of features and name them so.
- Use repositories to provide access to data and manipulate data.
- Repositories should coordinating access between data sources or decide which data source to use for example by feature
  flag. Example:

```dart
class DefaultBookingRepository implements BookingRepository {
  final LocalBookingDataSource local;
  final RemoteBookingDataSource remote;

  Future<Booking?> getBookingById(String id) async {
    Booking? savedBooking = await local.getBookingById(id);

    if (savedBooking == null) {
      Booking? remoteBooking = await remote.getBookingById(id);

      if (remoteBooking != null) {
        await local.saveBooking(remoteBooking);
      }
      return remoteBooking;
    }
    return savedBooking;
  }

//...other operations...
}
```

- Define interfaces for repository api and concrete implementation by type, for example:

Interface

 ```dart
abstract class BookingRepository {}
```

Implementations

```dart
class DefaultBookingRepository implements BookingRepository {}

class AnonymousBookingRepository implements BookingRepository {}

class PremiumBookingRepository implements BookingRepository {}
```

- Do not use tools, platform related libraries nor perform network logic in repositories, encapsulate that in data
  sources.
- Only use data sources inside repositories.
- Only receive as input primitive or business object and output business objects.
- Implementations of the **Repository API** should be named with the **repository class** name as suffix.
- Avoid naming repositories with **Impl**  suffix, name them using **Default** prefix + **Base repositories name**.
- Return Asynchronous type for public API (Future or Stream).
- Implementation of the repository should not change their public API (return type, method arguments).
- Repository class files should be under the repositories folder of the feature.

## Services

Classes that provide access to functionalities for a specific feature or a scope of a feature, for example:

```dart
abstract class AuthenticationService {
  Future<void> authenticate(String username, String password);

//...other functionalities...
}

abstract class AppointmentService {
  Future<void> register(Appointment appointment);

//...other functionalities...
}
```

Guideline Recommendations.

- Suffix class name with **Service**.
- Only define services for features or scope of features and name them so.
- Use services to provide functionalities required to implement a business logic.
- Use data sources to access to data required to implement a functionality business logic.
- Do not use tools, platform related libraries nor perform network logic in services, implement it in data sources.
- Only use data sources inside services.
- Only receive as input primitive or business object and output business objects.
- Implementations of the **Service API** should be named with the **service class** name as suffix.
- Avoid naming services with **Impl**  suffix, name them using **Default** prefix + **Base service name**.
- Return Asynchronous type for public API (Future or Stream).
- Implementation of the service should not change their public API (return type, method arguments).
- Service class files should be under the services folder of the feature.

## Data sources

Classes that implement access to data located locally or remotely. Example

```dart
abstract class LocalBookingDataSource {
  Futture<void> saveBooking(Booking booking);

//...other functionalities...
}

abstract class RemoteBookingDataSource {
  Future<booking> saveBooking();
}
```

Guideline Recommendations:

- Keep the base type of data sources to local and remote.
- Name implementation after library or tool used and the data source name as suffix, for example:

```dart
class SQLiteBookingDataSource implements LocalBookingDataSource {
  //...............Implementation................
}

class RestApiBookingDataSource implements RemoteBookingDataSource {
  //...............Implementation................
}

class MemoryBookingDataSource implements LocalBookingDataSource {
  //...............Implementation................
}
```

- Handle library or tool errors in data source and convert them to app custom exception.
- Only receive as input primitive or business object and output business objects or primitive.
- Implementations of the **Data Source API** should be named with the **Data Source class** name as suffix.
- Return Asynchronous type for public API (Future or Stream).
- Implementation of the data sources should not change their public API (return type, method arguments).
- Data sources class files should be under the datasources folder of the feature.

## Data Transfer Objects - DTO

Data transfer objects - used to transfer data from one system to another. Example

```dart
class ApiRequest {
  //...fields...
}

class ApiResponse {
  //...fields...
}
```

```dart
class SQLiteTableDefinition {
  //...fields...
}
```

Guidelines:

- Use DTO to represent database tables.
- Use DTO to represent api request and response.
- Use DTO to implement library required data. For example to use a persistence library your data need to extend an
  object or add more fields, create a DTO instead.
- Implement mappers in the DTO class, for example.

```dart
class ApiRequest {
  //...fields...

  factory ApiRequest.fromBO(BOObject bo) {
    //...mapper code...
  }

  BOObject toBO() {
    //...mapper code...
  }
}
```

- Do not implement saving or fetching logic in DTO, implement it in data sources.
- DTO class files should be under the dtos folder of the feature.

## Business Object

Business Object are data classes representation of a part of the business or an item within it, they are used to execute
business logic independently of platform, library or tools.

A business object may represent, for example, **a person, place, event, business process, or concept** and  **invoice, a
product, a transaction or even details of a person**.

Guideline:

- Business Object classes name should be suffixed with BO.
- Business Object classes should only contains data fields or method to format data fields.
- Business Object classes should implement equals and hash-code methods, fromJson and toJson.
- Business Object classes fields type should only be primitive or other business object.
- Business Object class files should be under the business_objects folder of the feature.

## Localization

For localization, we use bd_l10n tool:

```yaml
project-type: flutter
features:
  - name: 'Common Localization' # name of the feature is also used as name of the generated file and class.
    translation-dir: translation/common # where your localized messages are stored.
    translation-template: common_en.arb # which file to use as template.
    output-dir: lib/localizations/ # directory were the generated code will be generated.
```

When creating a new feature, for example Authentication, you would add the following settings:

```yaml
project-type: flutter

features: # list of features used in the project, a project must have at least one feature.  
  - name: 'Common Localization' # name of the feature is also used as name of the generated file and classes.  
    translation-dir: translation/common # where your localized messages are stored.  
    translation-template: en.arb # which file to use as template, this file should be in the translation-dir  
    output-dir: lib/localizations/ # Were the generated localization classes written.  

  - name: 'Authentication Localization' # name of the feature is also used as name of the generated file and classes.  
    translation-dir: translation/authentication # where your localized messages are stored.  
    translation-template: en.arb # which file to use as template, this file should be in the translation-dir  
    output-dir: lib/localizations/ # Were the generated localization classes written.
```

## How to structure assets

The same as for Project code structure.

First the feature name than the assets type and the files.

Example:

```
|-- authentication/
|---- svg/
|------ password_hidden_icon.svg
|------ forget_password_icon.svg
|---- png/
|------ background.png
```
