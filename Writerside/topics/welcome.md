# Welcome

Apollo Execution is a code-first GraphQL execution library.

Features:

* Generates a GraphQL schema from your Kotlin code: write Kotlin, get a typesafe API.
* Doesn't use reflection. Use it on the JVM and enjoy ultra-fast start times. Or use it on Kotlin native. Apollo Execution is KMP-ready!
* Supports custom scalars, subscriptions, persisted queries and everything in the current [GraphQL draft](https://spec.graphql.org/draft/).
* Integration with Ktor and Spring boot

Under the hood, Apollo Execution uses [KSP](https://kotlinlang.org/docs/ksp-overview.html) to generate GraphQL resolvers and types information from your Kotlin code.

## Getting started

### Gradle configuration

Add the `com.apollographql.execution` Gradle plugin and dependencies to your build script:

```kotlin
// build.gradle.kts
plugins {
  // Kotlin and KSP are required
  id("org.jetbrains.kotlin.jvm").version(kotlinVersion)
  id("com.google.devtools.ksp").version(kspVersion)
  // Add the Apollo Execution plugin
  id("com.apollographql.execution").version("%latest_version%")
}

dependencies {
  // Add the runtime dependency
  implementation("com.apollographql.execution:apollo-execution-runtime:%latest_version%")
}

// Configure codegen
apolloExecution {
  service("service") {
    packageName = "com.example"
  }
}
```

### Defining your root query

Then write your root query in a `Query.kt` file:

```kotlin
// Define your root query class 
// @GraphQLQuery is the entry point for KSP processing
@GraphQLQuery
class Query {
  // Public functions become GraphQL fields 
  // Kdoc comments become GraphQL descriptions
  // Kotlin parameters become GraphQL arguments
  /**
   * Greeting for name
   */
  fun hello(name: String): String {
    return "Hello $name"
  }
}
```

Run the codegen:

```shell
./gradlew kspKotlin
```

Apollo Execution generates a schema in `graphql/schema.graphqls`:

```
type Query {
  """
   Greeting for name
  """
  hello(name: ID!): ID!
}
```

It is recommended to commit this file in source control so you can track changes made to your schema.

### Executing your query

The codegen generates a `com.example.ServiceExecutableSchemaBuilder` class that is the entry point to execute GraphQL requests:

```kotlin
val executableSchema = ServiceExecutableSchemaBuilder()
  .build()
```

You can then create a GraphQL request:
```kotlin
val request = GraphQLRequest.Builder()
  .document("{ hello(name: \"sample\") }")
  .build()
```

And execute it:
```kotlin
val response = executableSchema.execute(
  request,
  ExecutionContext.Empty
)

println(response.data)
// {hello=Hello sample}
```

For more details, see the [Using variables](variables.md), [Execution contexts](execution-context.md) and other documentation pages

## 3rd party bindings

The Apollo Execution execution algorithm are network agnostic but for convenience, they comes with bindings for:

* Ktor ([documentation](ktor.md))
* Spring ([documentation](spring.md))

See the respective documentation for how to configure a server with an `ExecutableSchema`