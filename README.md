# Sprout Android

Sprout Android is a native Java game that turns plant discoveries into
collectible creatures. Take a plant photo, identify its species, generate a
sprite, and keep it in a garden. Botanical taxonomy determines the creature's
move pool for turn-based battles against a bot.

The native project retains its original Plantemon name in its folders,
packages, and app label. Start in [Plantemon/](Plantemon/) on the `infosys`
default branch. This repository also contains independent
[Expo client](plantemon-app/README.md) and [API](plantemon-server/README.md)
ports with separate setup and persistence.

![Sprout project poster](docs/Infosys_Poster.png)

## What is built

- Camera and gallery capture with Plant.id species identification and botanical
  metadata, including care needs, toxicity, and taxonomy.
- A two-step sprite workflow that crafts a description with Gemma and renders
  it with Flux. Cached sprites avoid repeat generation for the same name.
- A six-pot garden, plant information pages, and local JSON persistence.
- Four taxonomy-based move pools, speed-ordered turns, accuracy checks, damage,
  and three heal charges per player per battle.
- A manual-name fallback with a cropped photograph when identification fails.

## Stack and architecture

| Technology | Role in the native game |
|---|---|
| Java and Android SDK | Activities connect the camera, garden, information, and battle screens to the game model. |
| AndroidX and Material components | Screen layout, lifecycle callbacks, and automatic garden saving when the app moves to the background. |
| OkHttp and `org.json` | Background requests to Plant.id and NVIDIA, plus parsing of provider responses. |
| Plant.id, NVIDIA Gemma, and Flux | Species identification, sprite description, and image generation are separate steps with photo fallbacks. |
| Internal JSON and PNG files | The garden and sprites persist on the device without an account or shared backend. |
| Gradle and JUnit | The Android build and local battle-logic tests use the checked-in Gradle project. |

The game model separates `Plant` and `Player` data from actions and the battle
state machine. `BattleController` lets the same turn loop accept human input
and bot decisions. `PlantFactory` constructs plants from scans and saved data.
These boundaries make the repository a concrete study of object-oriented game
logic alongside Android lifecycle and network integration.

The [native technical reference](docs/native-android.md) covers the source
layout, scan pipeline, taxonomy mapping, battle rules, persistence, and build
configuration. The Expo port uses a separate TypeScript model and cloud API;
its behavior must be checked independently.

## Local setup

Use Android Studio, the Android SDK for `compileSdk = 36` and
`compileSdkMinor = 1`, and a Gradle JDK compatible with Android Gradle Plugin
9.1.0 and the Gradle 9.4.0 wrapper. The app requires Android 10 or later.

```bash
git clone --branch infosys https://github.com/Neonat/sprout-android.git
cd sprout-android/Plantemon
```

Open `Plantemon/` in Android Studio. Configure the SDK and Gradle JDK, then
create an uncommitted `Plantemon/local.properties` with these provider keys:

```properties
PLANT_API_KEY=your_plant_id_key
NVIDIA_API_KEY=your_nvidia_key
FLUX_API_KEY=your_nvidia_key
```

Sync Gradle and run the `app` module on a device or emulator. Keys become
`BuildConfig` fields in the native app. The app compiles without keys, but
provider calls cannot succeed. The separate API port keeps provider keys on
the server instead.

## Tests and current limits

The native battle tests are in
[BattleLogicTest.java](Plantemon/app/src/test/java/com/g4ng/logic/BattleLogicTest.java).
The [Android workflow](.github/workflows/android.yml) runs `./gradlew build`
from `Plantemon/` on `infosys`. The workflow still selects JDK 11; review that
runtime against the current plugin and wrapper requirements before relying on
CI. Local logic tests do not validate camera behavior, device rendering, or
live provider responses.

The native bot heals below 40% HP when it has charges, then chooses a random
move otherwise. At battle end, `BattleHandler` restores both gardens and heal
charges before the UI reads the results. The HP bars therefore show restored
values during the final display. `SwitchAction` remains an inactive
placeholder. The Expo port restores HP through a separate
`restoreAfterBattle()` call.

## Screenshots

<details>
<summary>Home, camera, garden, plant information, and battle screens</summary>

![Home screen](docs/HomeScreen.PNG)
![Camera](docs/camera.png)
![Garden](docs/garden.png)
![Plant information](docs/infoscreen.png)
![Adventure](docs/adventure.png)

</details>

## Project context and related work

Sprout is a collaborative project about plant diversity, tourism, and
collectible creatures. This repository is a fork of
[gxy812/katsu-an-g4ng](https://github.com/gxy812/katsu-an-g4ng), with the upstream
team's history and subsequent local development retained.

The [public project summary](https://www.linkedin.com/in/nathaniel-sim/details/experience/)
records best-project awards in SUTD 50.001 Information Systems and Programming
and 50.003 Elements of Software Construction, a S$2,000 Baby Shark Fund grant,
and the Dell Book Prize with a S$1,500 cash award. These recognize Sprout and
its project teams across development.

The [web game](https://github.com/Neonat/sprout-web) is a separate React,
Express, and Firestore implementation. The private
[studio](https://github.com/Neonat/sprout-studio) holds sprite generation and
review tooling. The private
[web prototype](https://github.com/Neonat/sprout-web-prototype) preserves the
earlier Next.js experiment. The private
[Android placeholder](https://github.com/Neonat/sprout-android-placeholder)
points here and contains no implementation.

## License

The upstream [MIT license](LICENSE) and its copyright notice remain in place.
