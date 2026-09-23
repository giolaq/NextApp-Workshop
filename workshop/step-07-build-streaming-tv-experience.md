# Step 7: Build a streaming TV experience with an AI prompt

In this capstone, you'll use an AI coding assistant to transform the workshop Hello World into a small streaming-style TV app. The result is inspired by the [React Native Multi-TV App Sample](https://github.com/AmazonAppDev/react-native-multi-tv-app-sample), while deliberately keeping the implementation small enough to understand during the workshop.

Instead of copying the reference app's complete navigation architecture, you'll build one end-to-end experience:

```text
Browse screen → Full-screen player → Browse screen
```

The exercise brings together the workshop's main ideas: shared React Native code, platform-specific files, D-pad focus, 10-foot UI design, native dependencies, and cross-platform validation.

## 7.1 Explore the reference experience

Open the [React Native Multi-TV App Sample](https://github.com/AmazonAppDev/react-native-multi-tv-app-sample) and look for these TV patterns:

- A large hero area that gives the focused item context
- Horizontal content browsing designed for a remote control
- Strong, visible focus feedback
- Selection that leads into full-screen video playback
- Shared application code with platform-specific playback implementations

Use the sample to understand the intended experience, not as code to copy. The workshop version uses only four static content items, one shelf, and a simple browse/player state machine.

## 7.2 Review the implementation prompt

The [`workshop/prompts`](./prompts/) directory contains [`streaming-tv-prompt.txt`](./prompts/streaming-tv-prompt.txt). Read it before giving it to your coding assistant:

```bash
cat workshop/prompts/streaming-tv-prompt.txt
```

The prompt acts as an implementation specification. It defines:

- The browse screen, hero, shelf, and content data
- TV-safe spacing, typography, and focus behaviour
- The shared video-player interface
- Vega, Expo TV, and web playback implementations
- Accessibility and test requirements
- The commands and interactions that must be validated

Notice the constraints as well as the requested features. The assistant should not add a remote catalog, a full navigation framework, adaptive HLS/DASH playback, captions, seeking, or media-session integration during this exercise.

## 7.3 Give the prompt to your coding assistant

Start your coding assistant from the repository root so it can resolve every path in the prompt. Ask it:

```text
Read workshop/prompts/streaming-tv-prompt.txt and implement it in this
repository.
Follow the validation section and report all changes, checks, build artifacts,
and remaining warnings.
```

If your assistant cannot read repository files directly, copy and paste the contents of `workshop/prompts/streaming-tv-prompt.txt` into the conversation.

The prompt instructs the assistant to preserve unrelated work and stay on the current branch. Let it inspect the existing code before it edits anything, because the new experience should reuse the monorepo, scaling utilities, and platform-resolution setup you used in the earlier steps.

## 7.4 Review the generated architecture

Most of the feature should live in `packages/shared`. Review the assistant's changes and identify these responsibilities:

```text
packages/shared/src/
├── components/
│   ├── Hero.tsx
│   ├── ContentCard.tsx
│   └── player/
│       ├── PlayerView.tsx
│       ├── VideoPlayer.tsx
│       ├── VideoPlayer.kepler.tsx
│       └── VideoPlayer.web.tsx
├── data/
│   └── content.ts
├── screens/
│   └── HomeScreen.tsx
└── theme/
    └── safeZones.ts
```

Check that the implementation follows these boundaries:

- `HomeScreen` owns the small browse/player state machine.
- Shared browse components contain the common layout and focus behaviour.
- `VideoPlayer.tsx` uses `react-native-video` for Expo TV targets.
- `VideoPlayer.kepler.tsx` uses the Vega W3C `VideoPlayer` and `KeplerVideoSurfaceView` in URL mode with clear MP4 content, following the repository's Vega SDK 0.22 guidance.
- `VideoPlayer.web.tsx` provides a simple browser fallback.
- Platform-specific dependencies are added only to the workspace that needs them.
- `packages/vega` contains the W3C media dependency, required Babel configuration, and media service declarations in `manifest.toml`.

This is the same platform-resolution pattern you used for `HeaderLogo`, Lottie, and the movie list, now applied to a more substantial native feature.

## 7.5 Validate the code

The coding assistant should run the complete validation list from the prompt. You can also run the core checks yourself:

```bash
# Shared package tests and type checking
yarn workspace @multitv/shared test --runInBand
yarn workspace @multitv/shared tsc --noEmit

# Vega tests, lint, type checking, and debug build
WATCHMAN_DISABLE=1 yarn workspace @multitv/vega test --runInBand
yarn workspace @multitv/vega lint
yarn workspace @multitv/vega tsc --noEmit
yarn vega:build

# Expo TV lint and type checking
yarn workspace @multitv/expotv lint
yarn workspace @multitv/expotv tsc --noEmit

# Check for whitespace errors
git diff --check
```

If your environment cannot run one of these checks, make sure the assistant reports the exact command, error, and remaining manual action rather than treating the check as passed.

## 7.6 Test the TV interaction

Build and launch the app on the Vega Virtual Device:

```bash
vega virtual-device start
yarn vega:vvd:mseries  # or yarn vega:vvd:intel
```

Use the remote or keyboard to verify:

1. The browse screen opens with the first Featured card focused.
2. Moving right changes the hero image, title, and description immediately.
3. The focused card scales up and has a visible orange focus treatment.
4. Pressing Select opens the chosen video in the full-screen player.
5. Play/Pause changes playback state.
6. The controls hide after a short period and return after interaction.
7. Exit or Back stops playback and returns to a usable position in the shelf.

Run the web target as a quick second-platform check:

```bash
yarn expotv:web
```

The layout and browse behaviour should remain shared, while the platform-specific player file changes automatically.

## What you've learned

- **Prompt as specification**: A useful coding prompt defines scope, architecture, interaction details, tests, and validation—not just the desired appearance.
- **10-foot UI**: TV interfaces need readable typography, safe margins, clear hierarchy, and unmistakable focus feedback.
- **Shared experience, native playback**: Browse logic can be shared while video playback uses the implementation appropriate to each platform.
- **Focus-driven presentation**: Remote focus is application state that can update the hero before the user selects anything.
- **Verification matters**: Generated code is not complete until its tests, builds, platform behaviour, and remaining warnings have been checked.

---

**Workshop complete:** You have built a shared multi-platform TV experience and measured its scrolling performance with ADBT.
