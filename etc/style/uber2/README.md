# Uber Protobuf Style Guide V2

This is the second version of the Uber Protobuf Style Guide.

  * [Package Naming](#package-naming)
  * [Package Versioning](#package-versioning)
  * [Directory Structure](#directory-structure)
  * [File Structure](#file-structure)
  * [Syntax](#syntax)
  * [File Options](#file-options)
  * [Imports](#imports)
  * [Enums](#enums)

See the [uber](uber) directory for an example of all concepts explained in this Style Guide.

Note that for documentation purposes, there are commenting requirements, however these are
separated into sections towards the end of this document so that we concentrate on the
Protobuf definition structure first. Please make sure you take careful note of these requirements.

## Package Naming

Some conventions:

- A **package name** as the full package name, i.e. `uber.trip.v1`.
- A **package sub-name** as a part of a package name, ie `uber`, `trip`, or `v1`.
- A **package version** is the last package sub-name that specifies the version,
  i.e. `v1`, `v1beta1`, or `v2`.

Package sub-names should be short and descriptive, and can use abbreviations if necesary.
Package sub-names should only include characters in the range `[a-z0-9]`, i.e always lowercase
and with only letter and digit characters. If names get too long or have underscores, the
generated stubs in certain languages are less than idiomatic.

As illustrative examples, the following are not acceptable package names.

```proto
// Note that specifying multiple packages is not valid Protobuf, however
// we do this here for brevity.

// The package sub-name credit_card_analysis is not short, and contains underscores.
package uber.finance.credit_card_analysis.v1;
// The package sub-name creditcardanalysisprocessing is longer than desired.
package uber.finance.creditcardanalysisprocessing.v1;
```

The following are acceptable package names.

```proto
// Each package sub-name is short and to the point.
package uber.trip.v1;
// Grouping by finance and then payment is acceptable.
package uber.finance.payment.v1;
// Ccap is for credit card analysis processing.
package uber.finance.ccap.v1;
```

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## Package Versioning

The last package sub-name should be a major version of the package, or the major version
followed by the beta version, specified as `vMAJOR` or `vMAJORbetaBETA`, where `MAJOR` and `BETA`
are both greater than 0. The following are examples of acceptable package names.

```proto
package uber.trip.v1beta1;
package uber.trip.v1beta2;
package uber.trip.v1;
package uber.trip.v2beta1;
package uber.trip.v2;
package something.v2;
```

As illustrative examples, the following are not acceptable package names.

```proto
// No version.
package uber.trip;
// Major version is not greater than 0.
package uber.trip.v0;
// Beta version is not greater than 0.
package uber.trip.v1beta0;
```

Packages with only a major version are considered **stable** packages, and packages with a major
and beta version are considered **beta** packages.

Breaking changes should never be made in stable packages, and stable packages should never depend
on beta packages. Both wire-incompatible and source-code-incompatible changes are considered
breaking changes. The following are the list of changes currently understood to be breaking.

- Deleting or renaming a package.
- Deleting or renaming an enum, enum value, message, message field, service, or service method.
- Changing the type of a message field.
- Changing the tag of a message field.
- Changing the label of a message field, i.e. optional, repeated, required.
- Moving a message field currently in a oneof out of the oneof.
- Moving a message field currently not in a oneof into the oneof.
- Changing the function signature of a method.
- Changing the stream value of a method request or response.

Beta packages should be used with extreme caution, and are not recommended.

Instead of making a breaking change, rely on deprecation of types.

```proto
// Note that all enums, messages, services, and service methods require
// sentence comments, and each service must be in a separate file, as
// outlined below, however we omit this here for brevity.

enum Foo {
  option deprecated = true;
  FOO_INVALID = 0;
  FOO_ONE = 1;
}

enum Bar {
  BAR_INVALID = 0;
  BAR_ONE = 1 [deprecated = true];
  BAR_TWO = 2;
}

message Baz {
  option deprecated = true;
  int64 one = 1;
}

message Bat {
  int64 one = 1 [deprecated = true];
  int64 two = 2;
}

service BamAPI {
  option deprecated = true;
  rpc Hello(HelloRequest) returns (HelloResponse) {}
}

service BanAPI {
  rpc (GoodbyeRequest) returns (GoodbyeResponse) {
    option deprecated = true;
  }
}
```

If you really want to make a breaking change, or just want to clean up a package, make a new
version of the package by incrementing the major version and copy your definitions as
necessary. For example, copy `foo.bar.v1` to `foo.bar.v2`, and do any cleanups required.
This is not a breaking change as `foo.bar.v2` is a new package. Of course, you are responsible
for the migration of your callers.

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## Directory Structure

Files should be stored in a directory structure that matches their package sub-names. All files
in a given directory should be in the same package.

The following is an example of this in practice.

```
.
└── uber
    ├── finance
    │   ├── ccap
    │   │   └── v1
    │   │       └── ccap.proto // package uber.finance.ccap.v1
    │   └── payment
    │       ├── v1
    │       │   └── payment.proto // package uber.payment.v1
    │       └── v1beta1
    │           └── payment.proto // package uber.payment.v1beta1
    └── trip
        ├── v1
        │   ├── trip_api.proto // package uber.trip.v1
        │   └── trip.proto // package uber.trip.v1
        └── v2
            ├── trip_api.proto // package uber.trip.v2
            └── trip.proto // pacakge uber.trip.v2
```

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## File Structure

All files should be ordered in the following manner.

   1. License Header (if applicable)
   2. Syntax
   3. Package
   4. File options (alphabetized)
   5. Imports (alphabetized)
   6. Everything Else

Protobuf definitions should go into one of two types of files: **Service files** or
**Supporting files**.

A service file contains exactly one service, and it's corresponding request and response messages.
This file is named after the service, substituting `PascalCase` for `lower_snake_case`. The service
should be the first element in the file, with requests and responses sorted to match the order
of the declared service methods.

A supporting file contains everything else, i.e. enums, and messages that are not request or
response messages. These files have no enforced naming structure or otherwise, however the general
recommendation is that if you have less than 15 definitions, they should all go in a file named
after the last non-version package sub-name. For example, for a package `uber.trip.v1` with less
than 15 non-service-related definitions, you would have a single supporting file
`uber/trip/v1/trip.proto`. While there are arguments for and against the single file
recommendation, this provides the easiest mechanism to normalize file structure across a repository
of Protobuf files while making it simple for users to grok a Protobuf package without having to
change between multiple files, each requiring many imports.

The following is an example of a supporting file with two definitions. Note that it is merely
coincidence that the file is named `trip.proto` and it contains a definition `Trip` - there is
no need to name files by a type or types contained within them, other than for services.

```proto
syntax = "proto3";

package uber.trip.v1;

option csharp_namespace = "Uber.Trip.V1";
option go_package = "tripv1";
option java_multiple_files = true;
option java_outer_classname = "TripProto";
option java_package = "com.uber.trip.v1";
option objc_class_prefix = "UTX";
option php_namespace = "Uber\\Trip\\V1";

import "uber/user/v1/user.proto";
import "google/protobuf/timestamp.proto";

// A trip taken by a rider.
message Trip {
  string id = 1;
  user.v1.User user = 2;
  google.protobuf.Timestamp start_time = 3;
  google.protobuf.Timestamp end_time = 4;
  repeated Waypoint waypoints = 5;
}

// A given waypoint.
message Waypoint {
  // In the real world, addresses would be normalized into
  // a PostalAddress message or such, but for brevity we simplify
  // this to a freeform string.
  string postal_address = 1;
  // The nickname of this waypoint, if any.
  string nickname = 2;
}
```

The following is an example of a service file named `uber/trip/v1/trip_api.proto` showing a service
`TripAPI` with two service methods, and requests and responses ordered by method. Note that
request and response messages do not require comments.

```proto
syntax = "proto3";

package uber.trip.v1;

option csharp_namespace = "Uber.Trip.V1";
option go_package = "tripv1";
option java_multiple_files = true;
option java_outer_classname = "TripApiProto";
option java_package = "com.uber.trip.v1";
option objc_class_prefix = "UTX";
option php_namespace = "Uber\\Trip\\V1";

import "uber/trip/v1/trip.proto";
import "google/protobuf/timestamp.proto";

// Handles interaction with trips.
service TripAPI {
  // Get the trip specified by the ID.
  rpc GetTrip(GetTripRequest) returns (GetTripResponse);
  // List the trips for the given user before a given time.
  //
  // If the start index is beyond the end of the available number
  // of trips, an empty list of trips will be returned.
  // If the start index plus the size is beyond the available number
  // of trips, only the number of available trips will be returned.
  rpc ListUserTrips(ListUserTripsRequest) returns (ListUserTripsResponse);
}

message GetTripRequest {
  string id = 1;
}

message GetTripResponse {
  Trip trip = 1;
}

message ListUserTripsRequest {
  string user_id = 1;
  google.protobuf.Timestamp before_time = 2;
  // The start index for pagination.
  uint64 start = 3;
  // The maximum number of trips to return.
  uint64 max_size = 4;
}

message ListUserTripsResponse {
  repeated Trip trips = 1;
  // True if more trips are available.
  bool next = 2;
}
```

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## Syntax

The syntax for Protobuf files should always be `proto3`. It is acceptable to import `proto2` files
for legacy purposes, but new definitions should conform to the newer `proto3` standard.

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## File Options

File options should be alphabetized. All files should specify a given set of file options that
largely conform to the [Google Cloud APIs File Structure](https://cloud.google.com/apis/design/file_structure).
Note that `prototool create` and `prototool format --fix` automate this for you, and this can be done
as part of your generation pipeline, so there is no need to conform to this manually.

The following are the required file options for a given package `uber.trip.v1` for a file named
`trip_api.proto`.

```proto
syntax = "proto3";

package uber.trip.v1;

// The csharp_namespace should be the package name with each package sub-name capitalized.
option csharp_namespace = "Uber.Trip.V1";
// The go_package should be the last non-version package sub-name concatenated with the
// package version.
//
// Of special note: For the V1 Style Guide, this was the last package sub-name concatenated
// with "pb", so for a package "uber.trip", it would be "trippb", but since the V2 Style Guide
// requires package versions, we have changed "pb" to be the package version.
option go_package = "tripv1";
// The java_multiple_files option should always be true.
option java_multiple_files = true;
// The java_outer_classname should be the PascalCased file name, removing the "." for the
// extension.
option java_outer_classname = "TripApiProto";
// The java_package should be "com." plus the package name.
option java_package = "com.uber.trip.v1";
// The objc_class_prefix should be the uppercase first letter of each package sub-name,
// not including the package-version, with the following rules:
//   - If the resulting abbreviation is 2 characters, add "X".
//   - If the resulting abbreviation is 1 character, add "XX".
//   - If the resulting abbreviation is "GBP", change it to "GPX". "GBP" is reserved
//     by Google for the Protocol Buffers implementation.
option objc_class_prefix = "UTX";
// The php_namespace is the same as the csharp_namespace, with "\\" substituted for ".".
option php_namespace = "Uber\\Trip\\V1";
```

While it is unlikely that a given organization will use all of these file options for their
generated stubs, this provides a universal mechanism for specifying these options that matches
the Google Cloud APIs File Structure, and all of these file options are built in.

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## Imports

Imports should be alphabetized. Imports should all start from the same base directory for
a given repository, usually the root of the repository. For local imports, this should match
the package name, so if you have a file `uber/trips/v1/trips.proto` with package `uber.trips.v1`,
you should import it as `uber/trips/v1/trips.proto`. For external imports, this should generally
also be the root of the repository. For example, if importing [googleapis](https://github.com/googleapis/googleapis)
definitions, you would import `google/logging/v2/logging.proto`, not `logging/v2/logging.proto`,
`v2/logging.proto`, and such.

Note that the
[Well-Known Types](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf)
should be used whenever possible, and imported starting with `google/protobuf`, for example
`google/protobuf/timestamp.proto`. Prototool provides all of these out of the box for you.

```
.
└── google
    └── protobuf
        ├── any.proto
        ├── api.proto
        ├── compiler
        │   └── plugin.proto
        ├── descriptor.proto
        ├── duration.proto
        ├── empty.proto
        ├── field_mask.proto
        ├── source_context.proto
        ├── struct.proto
        ├── timestamp.proto
        ├── type.proto
        └── wrappers.proto
```

These are available for browsing at
[github.com/protocolbuffers/protobuf/src/google/protobuf](https://github.com/protocolbuffers/protobuf/tree/master/src/google/protobuf)
and are also includeed in the `include` directory of each [Protobuf Releases ZIP
file](https://github.com/protocolbuffers/protobuf/releases).

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**

## Enums

Enums should always be `PascalCase`. Enum values should be `UPPER_SNAKE_CASE`.

The enum option `allow_aliases` should never be used.

Enum values have strict naming requirements.

  1. All enum values must have the name of the enum prefixed to all values as `UPPER_SNAKE_CASE`.

For example, for an enum `TripType`, all values must be prefixed with `TRIP_TYPE_`.

This is due to Protobuf enums using C++ scoping rules. This results in it not being
possible to have two enums with the same value. For example, the following is not valid
Protobuf, regardless of file structure.

```proto
syntax = "proto3";

package uber.trip.v1;

enum Foo {
  CAR = 0;
}

enum Bar {
  // Invalid! There is already a CAR enum value in uber.trip.v1.
  CAR = 0;
}
```

Compiling this file will result in the following errors from `protoc`.

```
uber/trip/v1/trip.proto:10:3:"CAR" is already defined in "uber.trip.v1".
uber/trip/v1/trip.proto:10:3:Note that enum values use C++ scoping rules, meaning that enum values
are siblings of their type, not children of it.  Therefore, "CAR" must be unique within
"uber.trip.v1", not just within "Bar".
```

  2. All enum values must have a 0 `INVALID` value.

For example, for an enum `TripType`, there must be `TRIP_TYPE_INVALID = 0;`.


  3. The invalid value carries no semantic meaning, and if a value can be purposefully
  unset, i.e. you think a value should be purposefully null over the wire, then
  there should be a `UNSET` value as the 1 value.

For example, for an enum `TripType`, you may add a value `TRIP_TYPE_UNSET = 1;`.

**[⬆ Back to top](#uber-protobuf-style-guide-v2)**
