Moshi-sealed
============

## 🚧 WIP 🚧

Reflective and code gen implementations for serializing Kotlin sealed classes via Moshi polymorphic adapters.

Simple add `sealed:{typeLabel}` where `{typeLabel}` is the value of the type label that should be
used for a Moshi `PolymorphicJsonAdapterFactory`. Annotate subtypes with `@TypeLabel` to indicate 
their type label value. One `object` can be used as an unknown default value via `@DefaultObject`, 
or the sealed type can be annotated with `@DefaultNull` to indicate that `null` should be the default.

```kotlin
@JsonClass(generateAdapter = true, generator = "sealed:type")
sealed class Message {

  @TypeLabel("success")
  @JsonClass(generateAdapter = true)
  data class Success(val value: String) : Message()

  @TypeLabel("error")
  @JsonClass(generateAdapter = true)
  data class Error(val error_logs: Map<String, Any>) : Message()

  @DefaultObject
  object Unknown : Message()
}
```

### Installation

Moshi-sealed can be used via reflection or code generation.

#### Code gen

Code gen works via annotation processor, and only requires adding the kapt dependency:

```gradle
kapt "io.sweers.moshisealed:moshi-sealed-codegen:{version}"
```

No runtime configuration is needed, code gen will generate `JsonAdapter`s in a way that Moshi understands
natively.

#### Reflection

Reflection works via `MoshiSealedJsonAdapterFactory`. Just add this in moshi before 
`KotlinJsonAdapterFactory` in your Moshi instance construction.

```kotlin
val moshi = Moshi.Builder().add(MoshiSealedJsonAdapterFactory()).add(KotlinJsonAdapterFactory()).build()
```

Gradle dependency:

```gradle
implementation "io.sweers.moshisealed:moshi-sealed-reflect:{version}"
```

#### Using both

For faster build times on IDE debug builds, you can use both reflection and code gen in tandem. This
affords the benefit of avoiding annotation processing overhead during builds while continuing to use 
it for CI and production builds. Code gen _should_ be the preferred option for runtime performance 
and binary size, as the reflect artifact uses `kotlin-reflect`.

Kotlin modules:

```gradle
dependencies {
  if (properties.containsKey('android.injected.invoked.from.ide')) {
    implementation 'io.sweers.moshisealed:moshi-sealed-reflect:<version>'
  } else {
    implementation 'io.sweers.moshisealed:moshi-sealed:<version>'
    kapt 'io.sweers.moshisealed:moshi-sealed-codegen:<version>'
  }
}
```

Java modules:

```gradle
dependencies {
  if (properties.containsKey('android.injected.invoked.from.ide')) {
    implementation 'io.sweers.moshisealed:moshi-sealed-reflect:<version>'
  } else {
    implementation 'io.sweers.moshisealed:moshi-sealed:<version>'
    annotationProcessor 'io.sweers.moshisealed:moshi-sealed-codegen:<version>'
  }
}
```

License
-------

    Copyright (C) 2019 Zac Sweers

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.