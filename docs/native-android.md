# Native Android technical reference

This reference describes the Java application in [Plantemon/](../Plantemon/).
Start with the [project overview](../README.md) for setup and related ports.

## Source layout

All paths below are relative to `Plantemon/app/src/main/java/com/g4ng/`.

| Package | Main responsibilities |
|---|---|
| `ui/` | `Plantemon` initializes data and restores the garden. `GameState` holds the live player. Activities render home, scan, garden, information, and battle screens. |
| `model/` | `Plant`, `Player`, and the `BattleState` enum describe creatures, gardens, healing, and turn phases. |
| `logic/` | Actions, controllers, `PlantFactory`, and `BattleHandler` implement construction and battle rules. |
| `database/` | `MoveBase` and `TaxonomyMoveMapBase` load JSON definitions. `PlantJsonHandler` reads and writes the garden. |
| `service/` | `PlantApiService` identifies plants. `SpriteGeneratorService` handles description, rendering, and caching. `AiSpriteGenerator` is the older single-step caller. |

## Scan pipeline

1. `ScanActivity` receives a camera image, gallery image, or test image.
2. `PlantApiService.fetchPlantDetails()` calls Plant.id v3 and parses the name,
   taxonomy, and botanical metadata.
3. `SpriteGeneratorService.generate()` checks the cache for that plant name.
4. On a cache miss, NVIDIA Gemma-3-27b-it uses the photo and name to write a
   sprite description. Flux.2-klein-4b renders the description.
5. The app stores the sprite and constructs a plant with moves derived from
   taxonomy. The garden is then saved locally.

When identification fails, a dialog accepts a manual plant name and uses the
photo as a cropped 192 by 192 sprite. A sprite-generation failure also falls
back to the photograph. The fallback plant has Tackle, Vine Whip, Leaf Shield,
and Solar Blast.

| Service | Endpoint |
|---|---|
| Plant.id | `https://api.plant.id/v3/identification` |
| NVIDIA chat | `https://integrate.api.nvidia.com/v1/chat/completions` |
| NVIDIA Flux | `https://ai.api.nvidia.com/v1/genai/black-forest-labs/flux.2-klein-4b` |

Provider requests run on background threads. Model names and endpoints above
are the values in the checked-in source; current service availability requires
separate validation.

## Taxonomy and moves

`TaxonomyMoveMapBase` selects a move pool using the Plant.id taxonomy.

| Group | Criterion | Pool ID |
|---|---|---|
| Mosses and nonvascular plants | `phylum != Tracheophyta` | `0` |
| Ferns | Class is Polypodiopsida, Lycopodiopsida, or Equisetopsida | `1` |
| Conifers | Class is Pinopsida, Cycadopsida, Ginkgoopsida, or Gnetopsida | `2` |
| Flowering plants | Remaining vascular plants | `3` |

The factory shuffles the pool and assigns four moves. Definitions are in
[moves.json](../Plantemon/app/src/main/res/raw/moves.json) and
[taxonomy.json](../Plantemon/app/src/main/res/raw/taxonomy.json).

## Battle rules

`BattleHandler` moves through `P1_MOVE`, `P2_MOVE`, `PROCESSING`, and `END`.
Human input supplies one action. `BotController` supplies the opponent action.
The faster plant acts first, and a fainted defender cannot retaliate.

`Move` checks accuracy and computes damage with random variation between 0.7
and 1.1. Damage is clamped at zero. The defender action contributes its defense
value to the calculation.

`HealAction.calculateHealAmount()` returns `floor(maxHP * 0.2) + 10`.
`execute()` caps HP at 100, consumes one of the player's three charges, and
provides a defense value of 15 for that turn. The bot chooses healing below
40% HP while charges remain. Otherwise, the bot chooses a random move.

At `END`, `handleEnd()` adds the winner to the turn log and calls
`restoreGarden()` for both players. That restores HP and resets heal charges
before `BattleActivity` reads the models for its result display. The final HP
bars show restored plants. `SwitchAction` is not active.

The separate [Expo game model](../plantemon-app/src/game/battle.ts) delays
restoration until `restoreAfterBattle()`. Native tests explicitly cover the
native restoration behavior; matching some test scenarios does not establish
complete parity between ports.

## Design patterns in the source

| Pattern | Implementation |
|---|---|
| Singleton | `GameState`, `MoveBase`, and `TaxonomyMoveMapBase`. |
| Strategy | `BattleController`, `HumanController`, and `BotController` share a turn loop. |
| Factory | `PlantFactory` constructs a plant from API data, a scan, or saved data. |
| State machine | `BattleHandler` advances the `BattleState` enum. |
| Template method | `Base.read()` loads records and delegates insertion to subclasses. |
| Callback | `SpriteCallback` reports asynchronous sprite generation success or failure. |
| Copy constructor | `Plant(Plant other)` copies opponent plant data. |
| Lifecycle observer | `ProcessLifecycleOwner` triggers garden saving when the app backgrounds. |

## Persistence and permissions

`PlantJsonHandler` saves the garden to `getFilesDir()/TEMP.json`.
`Plantemon.onCreate()` restores that data at startup. The lifecycle observer
saves it when the app moves to the background.

Sprites are PNG files under `getExternalFilesDir("Sprites")`. The generation
cache uses `getFilesDir()/sprite_cache/sprite_<name>.png`.

| Manifest permission | Use |
|---|---|
| `CAMERA` | Plant photo capture, with a runtime permission request. |
| `INTERNET` | Provider calls. |
| `ACCESS_COARSE_LOCATION` | Declared, but not actively used. |

## Build configuration

The [app Gradle file](../Plantemon/app/build.gradle.kts),
[version catalog](../Plantemon/gradle/libs.versions.toml), and
[wrapper configuration](../Plantemon/gradle/wrapper/gradle-wrapper.properties)
are the source of truth.

| Setting | Value |
|---|---|
| Application ID | `com.g4ng.plantemon` |
| Minimum SDK | `29` |
| Target SDK | `35` |
| Compile SDK | `36`, minor `1` |
| Android Gradle Plugin | `9.1.0` |
| Gradle wrapper | `9.4.0` |
| Java source compatibility | `11` |

AppCompat, ConstraintLayout, Material components, AndroidX Activity, lifecycle
process support, and OkHttp are declared through the version catalog. JUnit
covers local logic; AndroidX test and Espresso dependencies are available for
instrumented tests. Java source compatibility is separate from the JDK needed
to run Gradle.

`PLANT_API_KEY`, `NVIDIA_API_KEY`, and `FLUX_API_KEY` are read from
`Plantemon/local.properties` into `BuildConfig`. The file is ignored by Git.
