# bdk-kotlin

⚠️⚠️ The code in this repository has been moved to [bdk-ffi](https://github.com/bitcoindevkit/bdk-ffi), and development is continuing there. This repository is now considered an archive, and will not accept new pull requests or issues. ⚠️⚠️

<br/>
<br/>
<br/>

This project builds .jar and .aar packages for the `jvm` and `android` platforms that provide 
[Kotlin] language bindings for the [`bdk`] library. The Kotlin language bindings are created by the 
[`bdk-ffi`] project which is included as a git submodule of this repository.

## How to Use

To use the Kotlin language bindings for [`bdk`] in your `jvm` or `android` project add the 
following to your gradle dependencies:
```groovy
repositories {
    mavenCentral()
}

dependencies {
  
  // for jvm
  implementation 'org.bitcoindevkit:bdk-jvm:<version>'
  // OR for android
  implementation 'org.bitcoindevkit:bdk-android:<version>'
  
}
```

You may then import and use the `org.bitcoindevkit` library in your Kotlin code. For example:

```kotlin
import org.bitcoindevkit.*

// ...

val externalDescriptor = "wpkh([c258d2e4/84h/1h/0h]tpubDDYkZojQFQjht8Tm4jsS3iuEmKjTiEGjG6KnuFNKKJb5A6ZUCUZKdvLdSDWofKi4ToRCwb9poe1XdqfUnP4jaJjCB2Zwv11ZLgSbnZSNecE/0/*)"
val internalDescriptor = "wpkh([c258d2e4/84h/1h/0h]tpubDDYkZojQFQjht8Tm4jsS3iuEmKjTiEGjG6KnuFNKKJb5A6ZUCUZKdvLdSDWofKi4ToRCwb9poe1XdqfUnP4jaJjCB2Zwv11ZLgSbnZSNecE/1/*)"

val databaseConfig = DatabaseConfig.Memory

val blockchainConfig =
  BlockchainConfig.Electrum(
    ElectrumConfig("ssl://electrum.blockstream.info:60002", null, 5u, null, 10u)
  )
val wallet = Wallet(externalDescriptor, internalDescriptor, Network.TESTNET, databaseConfig, blockchainConfig)
val newAddress = wallet.getNewAddress()
```

### Example Projects

#### `bdk-android`
* [Devkit Wallet](https://github.com/thunderbiscuit/devkit-wallet)  
* [Padawan Wallet](https://github.com/thunderbiscuit/padawan-wallet)

#### `bdk-jvm`
* [Tatooine Faucet](https://github.com/thunderbiscuit/tatooine)

### How to build
_Note that Kotlin version `1.6.10` or later is required to build the library._

1. Clone this repository and initialize and update its [`bdk-ffi`] submodule.
```shell
git clone https://github.com/bitcoindevkit/bdk-kotlin
git submodule update --init
```
2. Follow the "General" bdk-ffi ["Getting Started (Developer)"] instructions.
3. If building on MacOS install required intel and m1 jvm targets
```sh
rustup target add x86_64-apple-darwin aarch64-apple-darwin
```
4. Install required targets
 ```sh
 rustup target add x86_64-linux-android aarch64-linux-android armv7-linux-androideabi
 ```
5. Install Android SDK and Build-Tools for API level 30+
6. Setup `$ANDROID_SDK_ROOT` and `$ANDROID_NDK_ROOT` path variables (which are required by the 
   build tool), for example (NDK major version 21 is required):
 ```shell
 export ANDROID_SDK_ROOT=~/Android/Sdk
 export ANDROID_NDK_ROOT=$ANDROID_SDK_ROOT/ndk/21.<NDK_VERSION>
 ```
7. Build kotlin bindings
 ```sh
 # build JVM library
 cd bdk-jvm
 ./gradlew buildJvmLib
 
 # build Android library
 cd bdk-android
 ./gradlew buildAndroidLib
 ```
8. Start android emulator (must be x86_64) and run tests
```sh
./gradlew connectedAndroidTest 
```

## How to publish

### Publish to your local maven repo
```shell
# bdk-jvm
cd bdk-jvm
./gradlew publishToMavenLocal --exclude-task signMavenPublication

# bdk-android
cd bdk-android
./gradlew publishToMavenLocal --exclude-task signMavenPublication
```

Note that the commands assume you don't need the local libraries to be signed. If you do wish to sign them, simply set your `~/.gradle/gradle.properties` signing key values like so:
```properties
signing.gnupg.keyName=<YOUR_GNUPG_ID>
signing.gnupg.passphrase=<YOUR_GNUPG_PASSPHRASE>
```

and use the `publishToMavenLocal` task without excluding the signing task:
```shell
./gradlew publishToMavenLocal
```

## Verifying Signatures
Both libraries and all their corresponding artifacts are signed with a PGP key you can find in the
root of this repository. To verify the signatures follow the below steps:

1. Import the PGP key in your keyring.
```shell
# Navigate to the root of the repository and import the ./PGP-BDK-BINDINGS.asc public key
gpg --import ./PGP-BDK-BINDINGS.asc
    
# Alternatively, you can import the key directly from a public key server
gpg --keyserver keyserver.ubuntu.com --receive-key 2768C43E8803C6A3
    
# Verify that the correct key was imported
gpg --list-keys
# You should see the below output
pub   ed25519 2022-08-31 [SC]
    88AD93AC4589FD090FF3B8D12768C43E8803C6A3
uid           [ unknown] bitcoindevkit-bindings <bindings@bitcoindevkit.org>
sub   cv25519 2022-08-31 [E]
```

2. Download the binary artifacts and corresponding signature files.
- from [bdk-jvm]
   - `bdk-jvm-<version>.jar`
   - `bdk-jvm-<version>.jar.asc`
- from [bdk-android]
   - `bdk-android-<version>.aar`
   - `bdk-android-<version>.aar.asc`

3. Verify the signatures.
```shell
gpg --verify bdk-jvm-<version>.jar.asc 
gpg --verify bdk-android-<version>.aar.asc

# you should see a "Good signature" result
gpg: Good signature from "bitcoindevkit-bindings <bindings@bitcoindevkit.org>" [unknown]
```

### PGP Metadata
Full key ID: `88AD 93AC 4589 FD09 0FF3 B8D1 2768 C43E 8803 C6A3`  
Fingerprint: `2768C43E8803C6A3`  
Name: `bitcoindevkit-bindings`  
Email: `bindings@bitcoindevkit.org`  

[Kotlin]: https://kotlinlang.org/
[Android Studio]: https://developer.android.com/studio/
[`bdk`]: https://github.com/bitcoindevkit/bdk
[`bdk-ffi`]: https://github.com/bitcoindevkit/bdk-ffi
["Getting Started (Developer)"]: https://github.com/bitcoindevkit/bdk-ffi#getting-started-developer
[Gradle Nexus Publish Plugin]: https://github.com/gradle-nexus/publish-plugin
[bdk-jvm]: https://search.maven.org/artifact/org.bitcoindevkit/bdk-jvm/0.9.0/jar
[bdk-android]: https://search.maven.org/artifact/org.bitcoindevkit/bdk-android/0.9.0/aar
