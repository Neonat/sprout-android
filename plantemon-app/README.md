# Plantemon Expo client

A separate React Native port of the [Java Android game](../README.md).
The app has sign-in, a plant garden, camera and image capture, plant information,
and a local turn-based battle screen. Expo supplies Android, iOS, and web
entry points from one TypeScript project. Those targets are not a statement
that store builds or browser journeys have been validated.

## Stack and boundaries

| Technology | Role |
|---|---|
| Expo and React Native | Cross-platform screens, camera access, image selection, and development tooling. |
| Expo Router | File-based navigation under `app/`, with separate auth and game routes. |
| Clerk | Account flows and bearer tokens for the API. SecureStore caches tokens on native targets. |
| TypeScript game model | Ports plants, players, moves, taxonomy pools, and battle logic into `src/game/`. |
| AsyncStorage | Caches the profile and garden response on the device. |
| Plantemon API | Owns provider credentials, cloud garden storage, and sprite URLs. |

The native Java game stores its garden locally and calls providers directly.
This port uses the [API server](../plantemon-server/README.md) for those
operations. The two implementations have separate configuration and behavior.

## Local setup

From the repository root:

```bash
cd plantemon-app
npm install
cp .env.example .env.local
```

Set `EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY` and `EXPO_PUBLIC_API_URL` in
`.env.local`. The Clerk project must match the API server's credentials.
Use a server address reachable by the selected device or emulator. A physical
device cannot reach your computer through its own `localhost`.

```bash
npm start
```

Use the Expo terminal shortcuts for iOS, Android, or web. iOS simulation needs
Xcode; Android emulation needs Android Studio. Native hardware can use a
compatible Expo Go installation. The API and its backing services must be
configured before account and scan flows can complete.

## Checks

Run these commands inside `plantemon-app/`:

```bash
npm test
npm run typecheck
```

Jest covers the TypeScript game model with scenarios adapted from
[BattleLogicTest.java](../Plantemon/app/src/test/java/com/g4ng/logic/BattleLogicTest.java).
Those cases cover shared rules, not complete parity with the native app.
The port restores HP through `restoreAfterBattle()`; the native handler
restores gardens before the result display.

## Source map

| Path | Contents |
|---|---|
| `app/(auth)/` | Sign-in and sign-up screens. |
| `app/(app)/` | Home, garden, scan, plant information, and battle screens. |
| `src/game/` | Plant construction, moves, and local battle state. |
| `src/lib/api.ts` | Authenticated calls to the API. |
| `src/lib/storage.ts` | Local profile and garden cache. |
| `assets/data/` | Move and taxonomy definitions copied from Android. |

## Current limits

- The battle screen uses the first garden plant and a clone of that plant as
  the opponent.
- The server defaults the username to `Trainer`; there is no username editor.
- The native manual-name fallback for failed identification is not ported.
- App icons and launch artwork retain Expo defaults.
- Store distribution, live provider behavior, and complete device journeys
  require separate verification.

Project context, team provenance, and the repository license are in the
[root README](../README.md).
