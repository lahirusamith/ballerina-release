---
layout: ballerina-left-nav-release-notes
title: Swan Lake Beta3
permalink: /downloads/swan-lake-release-notes/swan-lake-beta3/
active: swan-lake-beta3
redirect_from: 
    - /downloads/swan-lake-release-notes/swan-lake-beta3
    - /downloads/swan-lake-release-notes/
    - /downloads/swan-lake-release-notes
---
### Overview of Ballerina Swan Lake Beta3

<em>This is the third beta release leading up to the Ballerina Swan Lake GA release.</em> 

It introduces the new language features planned for the Swan Lake GA release and includes improvements and bug fixes done to the compiler, runtime, standard library, and developer tooling after the Swan Lake Beta2 release.

### Updating Ballerina

If you are already using Ballerina, you can use the [Update Tool](/learn/tooling-guide/cli-tools/update-tool/) to directly update to Ballerina Swan Lake Beta3 as follows. 

To do this, first, execute the command below to get the update tool updated to its latest version. 

> `bal update`

If you are using an **Update Tool version below 0.8.14**, execute the `ballerina update` command to update it. Next, execute the command below to update to Swan Lake <VERSION>.

> `bal dist pull slbeta3`

### Installing Ballerina

If you have not installed Ballerina, then download the [installers](/downloads/#swanlake) to install.

### Language Updates

#### New Features

##### Introduction of the `!is` Operator

The `!is` operator has been introduced to check if a value does not belong to a given type. This is the negation of the `is` operator.

```ballerina
import ballerina/io;
 
public function main() {
   int|string? x = 10;
 
   if x !is () {
       io:println("int or string value: ", x);
   }
}
```

##### Inferring Types for Numeric Literals in Additive and Multiplicative Expressions

The type for numeric literals in additive and multiplicative expressions is now inferred from the contextually-expected type.

When the contextually-expected type for an additive or multiplicative expression is `float`, the type of a literal used as a sub expression is inferred to be `float`. Similarly, if the contextually-expected type is `decimal`, the type of the literal is inferred to be `decimal`.

```ballerina
float a = 10 + 3.0 + 5.0;
float b = 5 / 10.2;
decimal c = 10.0 * 3;
decimal d = 10 + 5 - 3.0;
```

##### Isolated Inference

The compiler now infers `isolated` for service objects, class definitions, variables, and functions in scenarios where if all of them explicitly specify an `isolated` qualifier, they would meet the requirements for isolated functions and isolated objects.

The following service and its methods are now inferred to be isolated.
```ballerina
import ballerina/http;
 
int defaultIncrement = 10;
 
service / on new http:Listener(8080) {
   private int value = 0;
 
   resource function get value() returns int {
       int value;
       lock {
           value = self.value;
       }
 
       lock {
           return value + defaultIncrement;
       }
   }
 
   resource function post increment(int i) {
       lock {
           self.value += i;
       }
   }
}
```

The compiler does not infer `isolated` for any constructs that are exposed outside the module.

##### Type Narrowing in the `where` Clause of a Query Expression/Action

The `where` clause in a query now narrows the types similar to `if` statements.

```ballerina
import ballerina/io;
 
public function main() returns error? {
   int?[] v = [1, 2, (), 3];
   int total = 0;
   check from int? i in v
       where i is int
       do {
           // Type of `i` is narrowed to `int`.
           total += i;
       };
   io:println(total); // Prints 6.
}
```

#### Improvements

##### Enum Declarations with Duplicate Members

Enum declarations can have duplicate members.

For example, the following declarations where both `LiftStatus` and `TrailStatus` have the same `OPEN` and `CLOSED` members are now allowed.
```ballerina
enum LiftStatus {
   OPEN,
   CLOSED = "0",
   HOLD
}
 
enum TrailStatus {
   OPEN,
   CLOSED = "0"
}
```

However, it is an error if the same enum declaration has duplicate members. Similarly, it is also an error if different enums initialize the same member with different values.

##### `string:Char` as the Static Type of String Member Access

The static type of the member access operation on a value of type `string` has been updated to be `string:Char` instead of `string`.

```ballerina
public function main() {
   string str = "text";
 
   // Can now be assigned to a variable of type `string:Char`.
   string:Char firstChar = str[0];
}
```

##### Directly Calling Function-typed Fields of an Object

Fields of an object that are of a function type can now be directly called via an object value using the method call syntax.

```ballerina
class Engine {
   boolean started = false;
  
   function 'start() {
       self.started = true;
   }
}
 
class Car {
   Engine engine;
 
   function () 'start;
 
   function init() {
       self.engine = new;
       // Delegate `car.start` to `engine.start`.
       self.'start = self.engine.'start;
   }
}
 
public function main() {
   Car car = new;
   car.'start(); // Call the function via the object field.
}
```

##### Tuple to JSON Compatibility

A tuple value whose members are JSON compatible can now be used in a context that expects a JSON value.

```ballerina
[string, int, boolean...] a = ["text1", 1, true, false];
// Now allowed.
json b = a;
```

##### Error Return in the `init` Method of a Service Declaration

Previously, the `init` method of a service declaration could not have a return type containing an error. That restriction has been removed with this release.
If the `init` method of a service declaration returns an error value, it will result in the module initialization failing.

```ballerina
import ballerina/http;
 
service / on new http:Listener(8080) {
   function init() returns error? {
       // Return an error for demonstration.
       // This will result in module initialization failing.
       return error("Service init failure!");
   }
}
```

##### Using `check` in Object Field Initializers

`check` can now be used in the initializer of an object field if the class or object constructor expression has an `init` method with a compatible return type (i.e., the error type that the expression could evaluate to is a subtype of the return type of the `init` method).

If the expression used with `check` results in an error value, the `init` method will return the error resulting in either the `new` expression returning an error or the object constructor expression resulting in an error.

```ballerina
import ballerina/io;
 
int? value = ();
 
class NumberGenerator {
   int lowerLimit;
 
   // `check` can be used in the field initializer
   // since the `init` method's return type allows `error`.
   int upperLimit = check value.ensureType();
 
   function init(int lowerLimit) returns error? {
       self.lowerLimit = lowerLimit;
   }
}
 
public function main() {
   NumberGenerator|error x = new (0);
 
   if x is error {
       io:println(x);
   }
}
```

##### Wildcard Binding Pattern Support in Variable Declarations

The wildcard binding pattern can now be used in a variable declaration with a value that belongs to type `any`.

```ballerina
import ballerina/io;
 
float _ = 3.14;
var _ = io:println("hello");
```

Using the wildcard binding pattern when the value is an error will result in a compilation error.

```ballerina
var _ = error("custom error"); // Compilation error.
```

##### Relaxed Static Type Requirements for `==` and `!=`

Previously, `==` and `!=` were allowed only when both operands were of static types that are subtypes of `anydata`. This has now been relaxed to allow `==` and `!=` if the static type of at least one operand is a subtype of `anydata`.
```ballerina
error? x = ();
 
// Now allowed.
if x == () {
 
}
``` 

##### Relaxed Query Expression Keywords

Identifiers that are keywords in a query expression context (`where`, `join`, `order`, `by`, `equals`, `ascending`, `descending`, `limit`, `outer`, and `select`) can now be used as ordinary identifiers outside query contexts. i.e., they are no longer required to be quoted identifiers.

```ballerina
// Now allowed. No longer required to use a quoted identifier (`'limit`).
int limit = 5;
// Now allowed. No longer required to use a quoted identifier (`string:'join()`).
string b = string:join(" ", "hello", "world!"); 
``` 

#### Bug Fixes

- In a stream type `stream<T, C>`; the completion type `C` should always include nil if it is a bounded stream. A bug where this was not validated for stream implementors has been fixed.

```ballerina
class StreamImplementor {
   public isolated function next() returns record {|int value;|}|error? {
       return;
   }
}
 
stream<int, error> stm = new (new StreamImplementor()); // Will now result in an error.
```

- Resource methods are no longer added to the type via object type inclusions. This was previously added even though resource methods do not affect typing.

```ballerina
service class Foo {
   resource function get f1() returns string {
       return "foo";
   }
 
   function f2() returns int => 42;
}
 
// It is no longer required to implement the `get f1` resource method.
service class Bar {
   *Foo;
 
   function f2() returns int => 36;
}
```

- A bug in string unescaping of unicode codepoints > `0xFFFF` has been fixed.

```ballerina
import ballerina/io;
 
public function main() {
  string str = "Hello world! \u{1F600}";
  io:println(str);
}
```

The above code snippet which previously printed `Hello world!  ὠ0` will now print `Hello world! 😀`.

- A bug in escaping `NumericEscape` has been fixed.

```ballerina
import ballerina/io;
 
public function main() {
  string str = "\\u{61}pple";
  io:println(str);
}
```

This code snippet, which previously printed `\u0061pple` will now print `\u{61}pple`.

- A bug that resulted in `NumericEscape` in the template string not being interpreted literally has been fixed.

```ballerina
import ballerina/io;
 
public function main() {
  string str = string `\u{61}pple`;
  io:println(str);
}
```

The code snippet above, which previously printed `\u0061pple` will now print `\u{61}pple`.

- A bug that resulted in self-referencing not being detected when referenced via a `let` expression or a constant reference expression has been fixed.

The following will now result in errors.

```ballerina
const int INTEGER = INTEGER; // Compilation error.

public function main() {
    string s = let string[] arr = [s] in arr[0]; // Compilation error.
}
```

- Type narrowing is now reset on any assignment to a narrowed variable. Previously, type narrowing was not reset if the static type of the new value was a subtype of the narrowed type. This was a deviation from the specification.

```ballerina
public function main() {
    int|string v = 0;

    if v is int {
        int x = v; // Valid.

        // Now, the type is reset to `int|string` even though `1` belongs to `int`.
        v += 1;

        int y = v; // Invalid, will now result in an error.
    }
}
```

To view all bug fixes, see the [GitHub milestone for Swan Lake Beta3](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+is%3Aclosed+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22+label%3AType%2FBug+label%3ATeam%2FCompilerFE).

### Runtime Updates

#### Improvements

##### Improved Configurable Variables to Support XML Types Through TOML Syntax

The `configurable` feature is improved to support variables with XML types through the TOML syntax.

For example, if the XML-typed configurable variables are defined in the following way,

``` ballerina
configurable xml xmlVar = ?;
```
the values can be provided in the `Config.toml` file as follows.


```toml
xmlVar = "<book><name>Sherlock Holmes</name></book>"
```

##### Improved Configurable Variables to Support Additional Fields and Rest Fields for Records

The `configurable` feature is improved to support additional fields and rest fields in record variables through the TOML syntax.

For example, if a configurable variable with open record type is defined in the following way,

```ballerina
type Person record {
};

configurable Person person = ?;
```

the values can be provided in the `Config.toml` file as follows.


```toml
[person]
intVal = 22
floatVal = 22.33
stringVal = "abc"
arrVal = [1,2,3]
mapVal.a = "a"
mapVal.b = 123
```

The additional fields that are created from the TOML values will have the following types.

* TOML Integer - `int`
* TOML Float - `float`
* TOML String - `string`
* TOML Boolean - `boolean`
* TOML Table - `map<anydata>`
* TOML Table array - `map<anydata>[]`

Similarly, if a configurable variable with a record type that contains a rest field is defined in the following way,

```ballerina
public type Numbers record {|
   int ...;
|};

configurable Numbers numbers = ?;
```

the values can be provided in the `Config.toml` file as follows.


```toml
num1 = 11
num2 = 26
```

##### Improved the Printed Error Stacktrace to Include the Cause

The error stack trace has been improved to include the error cause locations. Stack frames of the wrapped error causes are also added to the stack trace.

E.g.,
For the following example,
```ballerina
public function main() {
    panic bar();
}
function bar() returns error {
    return error("a", y());
}
function y() returns error {
    return x();
}
function x() returns error {
    return error("b");
}
```

the expected stack trace will be as follows.

```
error: a
at cause_location.0:bar(main.bal:6)
cause_location.0:main(main.bal:2)
cause: b
at cause_location.0:x(main.bal:14)
cause_location.0:y(main.bal:10)
... 2 more
```
##### New Runtime Java APIs

###### Invoke the Ballerina Object Method Asynchronously
New JAVA Runtime APIs are introduced to execute the Ballerina object method from Java. The object method caller can decide 
whether to execute the object method sequentially or concurrently using the appropriate API.

If the caller can ensure that the given object and object method is isolated and no data race is possible for the mutable 
state with given arguments, they can use the `invokeMethodAsyncConcurrently` method, or otherwise, the `invokeMethodAsyncSequentially` method.

```java
boolean isIsolated = object.getType().isIsolated(methodName);
if (isIsolated) {
    BFuture bFuture = env.getRuntime().invokeMethodAsyncConcurrently(object, methodName, strandName, metadata, 
                                    callback, properties, returnType, args);
} else {
    BFuture bFuture = env.getRuntime().invokeMethodAsyncSequentially(object, methodName, strandName, metadata,
                                    callback, properties, returnType, args);
}
```

The previous `invokeMethodAsync` methods are deprecated.

######  API to Retrieve the Isolated Ballerina object or object method 

The two new APIs below are introduced to the `ObjectType`.
```java
    boolean isIsolated();

    boolean isIsolated(String methodName);
```

##### Removed the Package Version from the Runtime

The fully-qualified package version has been removed from the runtime and will only have the major version. Therefore, when you provide 
the version to the Ballerina runtime Java API like creating runtime values, you need to provide only the package runtime version. The stack traces will contain only the major package versions.


#### Bug Fixes

- The completion type of a stream is now considered in runtime assignability checks.

```ballerina
stream<int, error?> stm = new (new StreamImplementor());
 
// Evaluated to true in SL Beta2, evaluates to false now.
boolean streamCheck = stm is stream<int>;
```

To view bug fixes, see the [GitHub milestone for Swan Lake Beta3](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+is%3Aclosed+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22+label%3AType%2FBug+label%3ATeam%2FjBallerina).

### Standard Library Updates

#### New Features

##### Crypto Package
- Improved the hash APIs for cryptographic salt

##### GraphQL Package
- Added field alias support for GraphQL documents
- Added variable support in GraphQL requests
- Added mutation support for GraphQL services
- Added typename introspection
- Added input objects support
- Added block string support

##### gRPC Package
- Added declarative auth configurations
- Added timestamp, duration, and struct type support
- Added OAuth2 JWT bearer grant type support for client
- Introduced the support for a directory with PROTO files as the input (`--input` flag) for the gRPC command
- Introduced the support for external import paths in the gRPC command using the `--proto_path` flag
- Added the PROTO file name as a suffix for the `ROOT_DESCRIPTOR` constant and `getDescriptorMap` function to fix the re-declared symbol issue when multiple stub files are put into the same module. If you are going to regenerate the stub files with the Swan Lake Beta3 release, you need to change the service annotation like below.
```ballerina
@grpc:ServiceDescriptor {
   descriptor: ROOT_DESCRIPTOR_<PROTO_FILE_NAME>,
   descMap: getDescriptorMap<Proto_File_Name>()
}
service "HelloWorld" on new grpc:Listener(9090) {
```

##### HTTP Package
- Enabled HTTP trace and access log support
- Added HATEOAS link support
- Introduced the `http:Cache` annotation to the resource signature
- Introduced support for the service-specific media-type subtype prefix
- Introduced the introspection resource method to get the generated OpenAPI document of the service
- Added OAuth2 JWT bearer grant type support for client

##### JWT Package
- Added HMAC signature support for JWT
  
##### Log Package
- Added observability span context values to the log messages when observability is enabled.

##### OAuth2 Package
- Added JWT bearer grant type support
    
##### SQL Package
- Added support for `queryRow()` in the database connectors. This method allows retrieving a single row as a record, or a single value from the database.
```ballerina
record{} queryResult = sqlClient->queryRow(`SELECT * FROM ExTable where row_id = 1`);
int count = sqlClient->queryRow(`SELECT COUNT(*) FROM ExTable`);
```

##### WebSocket Package
- Added OAuth2 JWT bearer grant type support for client
- Introduced retrying for the WebSocket client
- Introduced the header annotation and query param binding support

#### Improvements

##### GraphQL Package
- Validate the `maxQueryDepth` at runtime as opposed to validating it at compile time

##### HTTP Package
- Added support for the `map<json>` as query parameter type
- Added support for nilable client data binding types

##### WebSocket Package
- Made the WebSocket caller isolated
- Introduced a write timeout for the WebSocket client

##### SQL Package
- Improved the throughput performance with asynchronous database queries.
- Introduced new array out parameter types in call procedures.
- Changed the return type of the SQL query API to include the completion type as nil in the stream. The SQL query code below demonstrates this change.
    
    **Previous Syntax**
    ```ballerina
    stream<RowType, error> resultStream = sqlClient->query(``);
    ```
    **New Syntax**
    ```ballerina
    stream<RowType, error?> resultStream = sqlClient->query(``);
    ```
- Improved Error Types in SQL module with the introduction of typed errors for data manipulation under `sql:ApplicationError`.

##### IO Package
- Changed the `io:readin` function input parameter to optional. In the previous API, it was required to pass a value to be printed before reading the user input as a string. Remove it due to the breaking change and made it optional. It is not recommended to pass a value to print it in the console.

#### Bug Fixes

To view bug fixes, see the [GitHub milestone for Swan Lake Beta3](https://github.com/ballerina-platform/ballerina-standard-library/issues?q=is%3Aclosed+is%3Aissue+milestone%3A%22Swan+Lake+Beta3%22+label%3AType%2FBug).

### Code to Cloud Updates

#### Improvements
- Added a flag to disable docker image generation
- Added the secret generation support for http clients
- Improved diagnostics for failed value retrievals
- Added support for multiple volumes

#### Bug Fixes

To view bug fixes, see the [GitHub milestone for Swan Lake Beta3](https://github.com/ballerina-platform/module-ballerina-c2c/issues?q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22).

### Dependency Management 

With Swan Lake Beta3, the way how dependencies of a package are managed has been changed. The new implementation will ensure the following. 

 - The new build will ensure that you will get updates of the dependent packages (direct or transitive) automatically
 - Introducing the `--sticky` flag in case you want to lock the dependency versions for subsequent builds

The following changes have been introduced.
- Introduced the `Dependencies.toml` version 2. Going forward, this will be automatically managed by the `build` command and you should not modify it. If you already have a `Dependencies.toml` file, it will be automatically updated to the new version.
- Packages in the local repository need to be configured in the `Ballerina.toml` file. The format is as follows.
    ```
    // To be finalized
    ```
    
### Developer Tools Updates

#### New Features

##### Language Server

- Added a code action to pull modules from the Ballerina Central
- Added the completion extension API for TOML configuration files
- Added completion support for the `Ballerina.toml` file
- Added the offline build configuration option
Clients can set the `ls.compilation.online` system property to`true` or `false` in order to run the language server's compilations online or offline. By default, the compilations are running offline. 

##### Ballerina OpenAPI Tool
- Introduced a new command-line option to generate all the record fields that are not specifically mentioned as 
  `nullable:false` in the OpenAPI schema property as nullable to reduce the type conversion errors in the OpenAPI to
  Ballerina command
  >`bal openapi -i <openapi-contract-file>  --nullable`
- Introduced a new command-line option to add user-required license or copyright headers for the generated Ballerina
  files via OpenAPI to Ballerina command
  >`bal openapi -i <openapi-contract-file> --license <license-file> `
- Introduced a new command-line option to generate the JSON file via the Ballerina to OpenAPI command
  >`bal openapi -i <service-file> --json`
- Added support to generate a boilerplate of test functions for each remote function implemented within a
  client connector  

##### Debugger
- Added debugger expression evaluation support for the following types:
    - error constructor expressions
    - explicit new expressions
    - XML attribute access expressions
    - annotation access expressions
    - range expressions
    - trap expressions
    - function, object method and action invocations with rest arguments
- Introduced log points support

#### Improvements
##### Ballerina OpenAPI Tool
###### Ballerina OpenAPI client and schema generation improvements for the OpenAPI to Ballerina command
- Added support to generate suitable client connector authentication mechanisms by mapping the security schemes
  given in the OpenAPI specification (OAS)
- Added support to generate API documentation for the client init method, remote functions, and records
- Added support to facilitate users to set common client configurations when initializing the connector 
- Added support to generate records for nested referenced schemas in the OpenAPI specification 
- Improved the OpenAPI tool to select the `https` server URL when multiple URLs are given in the OpenAPI specification
 
###### The Ballerina to OpenAPI command improvements
- Added support for language server extension 
- Improved the response status code map to `202` when the resource function does not have the `return` type
- Improved mapping different status code responses in the resource function
- Enhanced generating an OpenAPI schema with Ballerina `typeInclusion` scenarios
- Added resource function API documentation mapping to the OAS description and summary
- Improved the resource function request payload mapping with the OAS `requestBody`
- Added the resource signature `http:Cache` annotation mapping to response headers
- Improved reference resolving for accessing a separate module data type to map OAS object schemas
- Improved nullable record field mapping to an OAS schema property
 
#### Bug Fixes

To view bug fixes, see the GitHub milestone for Swan Lake Beta2 of the repositories below.

- [Language Server](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+is%3Aclosed+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22+label%3AType%2FBug+label%3ATeam%2FLanguageServer)
- [Update Tool](https://github.com/ballerina-platform/ballerina-update-tool/issues?q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+project%3Aballerina-platform%2F32)
- [OpenAPI](https://github.com/ballerina-platform/ballerina-openapi/issues?q=is%3Aissue+is%3Aclosed+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22+label%3AType%2FBug)
- [Debugger](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+label%3AType%2FBug+label%3AArea%2FDebugger+milestone%3A%22Ballerina+Swan+Lake+-+Beta3%22+is%3Aclosed)
- [Test Framework](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+label%3ATeam%2FTestFramework+milestone%3A%22Ballerina+Swan+Lake+-+Beta2%22+label%3AType%2FBug+)

#### Ballerina Packages Updates

### Breaking Changes
