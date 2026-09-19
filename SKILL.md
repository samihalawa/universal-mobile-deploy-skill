---
name: universal-mobile-deploy-skill
description: Analyze a real project, choose the correct mobile deployment path, install the right native dependencies, wire mobile release services like RevenueCat when needed, and ship with Expo/EAS, Fastlane, Capacitor, or native tooling without forcing the wrong stack.
---

# Universal Mobile Deploy Skill

Use this skill when the user wants a project analyzed and then deployed or prepared for deployment to iOS and/or Android.

This skill is for real project execution, not generic advice. Start by proving what the project actually is, then choose the correct deployment path.

When a project can remain Expo-first without breaking required native capabilities, prefer Expo and EAS over dropping into custom native automation.

## Primary Objective

Take an arbitrary project and get it onto the right mobile deployment path with the least-fragile workflow:

- `Expo managed` or `Expo prebuild` projects should usually use `EAS Build` and `EAS Submit`
- `bare React Native` projects should usually use native iOS/Android tooling plus `Fastlane`
- `Capacitor` projects should usually use Capacitor sync/open plus native platform builds, optionally with Fastlane for release automation
- `native iOS`, `native Android`, or mixed native repos should use their native build chain and Fastlane where appropriate
- subscription and in-app purchase apps should also validate `RevenueCat`, store products, entitlements, and SDK wiring before release
- mature Expo repos may use `EAS` for building and submission while still keeping `Fastlane` for screenshots, custom iOS release steps, simulator builds, or post-build automation

Do not force Fastlane onto a project just because mobile deployment was mentioned.

## Hard Rules

- Analyze the actual project before naming a deployment tool.
- Verify whether deployment logic already exists before adding a new one.
- Prefer the repo's current stack and deployment path over inventing a new architecture.
- If the repo is web-only and has no native target, do not pretend it is ready for App Store or Play Store deployment.
- Use the root `.env` as the canonical runtime and deployment config source when the repo follows that convention.
- Avoid hardcoded bundle IDs, Apple IDs, team IDs, package names, keystore paths, or store metadata when the repo already defines them.
- Do not recommend Fastlane as the default answer for Expo-managed apps.
- If the project is Expo-based and can stay Expo-based, prefer Expo/EAS first.
- Do not bolt RevenueCat on blindly; first verify whether the app really sells subscriptions, consumables, or entitlements.
- Do not recommend `expo prebuild` unless native customization or the existing repo state actually requires it.
- Do not generate duplicate deployment systems when the repo already has one that works.
- Verify commands, credentials wiring, and platform prerequisites before claiming the pipeline is ready.
- Verify external mobile services such as RevenueCat, push, crash reporting, and deep-link config when the release flow depends on them.

## Phase 1: Project Detection

Inspect the real repo first. At minimum check:

- `package.json`
- lockfile: `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, or `bun.lock*`
- `app.json`, `app.config.js`, `app.config.ts`
- `eas.json`
- `capacitor.config.ts`, `capacitor.config.js`, or `capacitor.config.json`
- `ios/` and `android/`
- `fastlane/`
- `RevenueCat` or purchases-related SDKs in `package.json`, native files, or env
- `Gemfile`
- `Podfile`
- `build.gradle*`, `settings.gradle*`, `gradle.properties`
- Xcode project/workspace files
- CI config files that already build or submit mobile apps
- product/paywall config, purchase hooks, subscription screens, entitlement checks, and backend webhook handling when monetization exists

Classify the project into one of these buckets:

### 1. Expo Managed

Signals:

- `expo` dependency is present
- `app.json` or `app.config.*` exists
- `ios/` and `android/` are absent or clearly generated artifacts
- `eas.json` may already exist

Default path:

- `EAS Build` for cloud builds
- `EAS Submit` for store/TestFlight submission
- `EAS Update` only when the runtime/update strategy is explicitly safe
- only add Fastlane if the repo already uses it for metadata, screenshots, or post-build release automation

### 2. Expo Prebuild / Bare Expo

Signals:

- `expo` dependency exists
- `ios/` and `android/` exist and are actively used
- `expo prebuild` is already part of the workflow or native modules require it

Default path:

- native iOS/Android verification
- `EAS Build` if already established
- Fastlane may be appropriate for store release automation after native build outputs exist

### 3. Bare React Native

Signals:

- `react-native` dependency exists
- `ios/` and `android/` are first-class project directories
- no Expo-managed workflow controls the build

Default path:

- native dependency validation first
- CocoaPods + Android SDK/Gradle validation
- Fastlane for beta/release automation is usually appropriate

### 4. Capacitor

Signals:

- `@capacitor/core` dependency
- `capacitor.config.*`
- iOS and/or Android shells created with Capacitor

Default path:

- web app build first
- `npx cap sync`
- open/build native shells
- Fastlane can automate TestFlight/App Store or Play Console submission after native projects exist

### 5. Native iOS / Android

Signals:

- Xcode project/workspace and/or Android Gradle project without Expo or Capacitor ownership

Default path:

- native build verification
- Fastlane for signing, beta, screenshots, metadata, and production submission

## Phase 2: Existing Deployment Audit

Before adding anything, inspect whether the project already has:

- `fastlane/Fastfile`, `Appfile`, `Matchfile`, `Deliverfile`, `Snapfile`
- `eas.json`
- GitHub Actions, CircleCI, Bitrise, Xcode Cloud, Codemagic, or custom CI steps
- existing lanes like `beta`, `release`, `internal`, `submit`, `screenshots`
- existing signing assets, keystore references, or App Store Connect API key wiring
- existing Play Console or App Store metadata folders
- existing RevenueCat SDK usage, API keys, entitlement names, product IDs, offerings, or webhook consumers
- existing Expo services such as EAS Update, EAS Submit, notification setup, or runtime versioning
- custom EAS lifecycle hooks such as pre-install or on-success scripts for native setup, patching, symbol upload, or release notifications

If a working deployment path already exists, prefer repairing or completing it instead of replacing it.

## Phase 3: Choose The Correct Path

### Path A: Expo Managed / Expo First

Choose this when the project is clearly Expo-managed.

Core actions:

- validate Expo config, bundle/package identifiers, runtime version, and app ownership
- inspect `eas.json` and existing profiles
- install only missing Expo-native dependencies with `expo install`
- use `eas build` for iOS and Android builds
- use `eas submit` for store submission when credentials and store setup are ready
- validate whether `eas update` is part of the release strategy and whether runtime versioning is safe
- if the app uses subscriptions or purchases, verify Expo-compatible RevenueCat setup before shipping

Do not:

- add native Fastlane lanes as the primary workflow unless the project already depends on them
- run `expo prebuild` just because native directories are absent

Allow for a hybrid Expo path when the repo already proves it:

- `EAS Build` and `EAS Submit` can remain the primary delivery mechanism
- `Fastlane` can still be the right secondary tool for screenshots, metadata sync, simulator artifacts, custom plist cleanup, build-number shaping, or bespoke release lanes
- prefer the existing split of responsibilities instead of collapsing everything into one tool

Use `expo prebuild` only if:

- the app needs native modules unsupported in pure managed workflow
- the repo already uses prebuild
- the user explicitly wants native ownership
- or store/release requirements cannot be satisfied cleanly in managed Expo

### Path B: Bare React Native + Fastlane

Choose this when native iOS/Android directories are first-class and Expo does not control the build.

Core actions:

- verify Node, package manager, Ruby strategy, CocoaPods, Xcode CLI tools, Java, and Android SDK
- install JS dependencies with the repo's package manager
- install iOS pods if needed
- verify package name, bundle ID, versioning, signing config, and scheme/flavor names
- wire Fastlane lanes to the real project configuration
- use Match or the repo's existing signing strategy when appropriate
- use `pilot`, `deliver`, `supply`, or equivalent only after the build lane works

Prefer:

- existing `Gemfile` + `bundle exec fastlane` if the repo already uses Bundler
- Homebrew Fastlane only when there is no repo-managed Ruby toolchain
- RevenueCat SDK and native purchase dependencies already chosen by the repo, instead of swapping monetization stacks

### Path C: Capacitor

Choose this when the app is fundamentally web-based with Capacitor native shells.

Core actions:

- verify the web build command and output directory
- verify `capacitor.config.*`
- build the web app first
- run `npx cap sync`
- inspect native shells for signing and version alignment
- use native builds for verification
- use Fastlane only where it adds value for release automation after native projects are real
- if the app sells subscriptions, verify the Capacitor-side purchase bridge and RevenueCat/native store plugin wiring

Do not:

- treat Capacitor like Expo
- skip the web-build-to-native-sync step

### Path D: Native Only

Choose this when the repo is already native.

Core actions:

- verify schemes, targets, bundle IDs, flavors, signing, and store metadata
- repair or author Fastlane config against the real native project
- validate beta and production release flows separately
- validate monetization SDKs such as RevenueCat or native store billing where applicable

## Phase 3.5: RevenueCat And Mobile Service Setup

Run this phase only when the app actually sells subscriptions, consumables, trials, or entitlement-gated features.

At minimum inspect:

- purchase or paywall screens
- subscription state hooks and entitlement checks
- product identifiers used in app code
- `react-native-purchases`, `purchases-capacitor`, `react-native-iap`, StoreKit, Play Billing, or similar dependencies
- backend webhooks or entitlement sync consumers
- environment variables for RevenueCat public SDK keys and server/webhook secrets

Required checks:

- confirm the app uses the same product IDs in code, App Store Connect, Google Play Console, and RevenueCat
- confirm entitlement names in code match RevenueCat exactly
- confirm offerings are defined and that the app requests the intended offering
- confirm iOS and Android public SDK keys are wired from the correct environment source
- confirm restore-purchases, login/identify, logout, and anonymous-user behavior are intentional
- confirm the app does not gate paid features on stale local state alone
- confirm webhook handling if the backend relies on subscription status server-side
- for Expo-based apps, confirm the RevenueCat integration path stays Expo-compatible before forcing prebuild or bare native changes
- confirm whether the backend also implements RevenueCat webhook ingestion, OAuth connection, subscriber sync, product mapping, or MCP setup flows, and keep app-side assumptions aligned with that backend contract

Default implementation guidance:

- `Expo / React Native`: prefer the official RevenueCat SDK path already compatible with the repo
- `Capacitor`: prefer the RevenueCat Capacitor plugin when the project is already Capacitor-based
- `native iOS / Android`: use the platform-native RevenueCat SDK only when the repo is already native

Testing guidance:

- distinguish UI-only purchase testing from real store sandbox testing
- in Expo preview environments, confirm whether the app is running in a mocked or preview purchase mode instead of assuming real transactions are available
- for real iOS or Android sandbox subscription testing, prefer a development build over plain Expo Go-style preview shells
- if the team uses RevenueCat Test Store or equivalent non-store testing, verify which behaviors it covers and which still require App Store Connect or Play Console sandbox validation
- verify restore purchases, identify/log in transitions, entitlement refresh, and paywall state after app relaunch

Do not:

- mix unrelated purchase stacks unless there is a migration plan
- add RevenueCat without aligning store products and entitlement names first
- claim subscription release readiness without testing purchase state resolution and restore flow

## Phase 4: Native Dependency Checklist

Always validate the prerequisites that match the chosen stack:

### Shared

- package manager and lockfile consistency
- root `.env` and any required deployment variables
- Apple Developer / App Store Connect credentials path
- Google Play service account or existing Play Console auth path
- RevenueCat keys, webhook secrets, entitlement names, and product ID mapping when monetization exists
- release services such as EAS Update, push, crash reporting, and deep-link domains when those are part of production rollout

### iOS

- Xcode and Command Line Tools
- Ruby/Fastlane strategy
- CocoaPods when applicable
- correct scheme/workspace/project
- signing identities, provisioning, team ID, bundle ID

### Android

- Java version
- Android SDK path
- Gradle wrapper
- keystore and signing config
- applicationId / package name

### Expo / EAS

- Expo account ownership
- EAS project linkage
- build profiles
- whether profiles cleanly separate development client, internal preview, and production
- credentials status for iOS and Android
- runtime version and update policy
- EAS Update channel or branch strategy when over-the-air updates are used
- Expo notifications, config plugins, and native-capability requirements when enabled
- any EAS lifecycle hooks that install native tool dependencies, patch submodules, upload symbols, or send release notifications

### Capacitor

- web output path correctness
- installed Capacitor platforms
- sync status after dependency or config changes
- plugin health for purchases, push, auth, and deep links if those features exist

### RevenueCat / Purchases

- product IDs match store setup
- offerings and entitlements match app code
- SDK keys are correct per platform
- sandbox/test account strategy exists
- restore flow and subscription state refresh are testable
- backend webhook path is configured if the server consumes subscription lifecycle events

## Phase 5: Verification Before Deploy

Before claiming readiness or shipping:

- prove the selected path matches the real project structure
- verify builds or at least the exact preflight commands for the selected stack
- verify credentials wiring without exposing secrets
- verify store identifiers and version/build increments
- verify the app can actually produce the required artifact: `.ipa`, `.aab`, `.apk`, or EAS build artifact
- verify the submission command matches the selected distribution target: internal, TestFlight, beta, production
- verify RevenueCat or purchase flows with the correct sandbox/test account path when monetization exists
- verify EAS Update policy separately from binary release when Expo uses over-the-air updates
- verify whether build hooks, symbol upload, and post-build notifications are part of the release contract rather than optional extras

## Phase 6: Deployment Execution

Choose the smallest correct end-to-end path:

- internal/beta testing first if the release has not been validated
- production submission only after the build and signing paths are proven
- use existing store metadata/screenshots if already present
- if screenshots or metadata are missing, state that gap explicitly instead of pretending release is complete
- if purchases exist, verify product availability and entitlement resolution before store submission
- if Expo uses EAS Update, separate binary rollout from OTA rollout and verify both intentionally

## Failure Patterns To Avoid

These are the common mistakes this skill exists to prevent:

- treating every mobile deployment request as “set up Fastlane”
- treating every Expo app as “just run EAS” without checking runtime versioning, config plugins, native modules, and update policy
- ignoring existing `eas.json`, `fastlane/`, or `capacitor.config.*`
- adding a second deployment system without checking the first one
- assuming a React project should become React Native
- assuming a web app should become Capacitor when no mobile target was requested
- adding RevenueCat without checking whether purchases already exist or whether the product catalog is aligned
- using hardcoded IDs, team names, or secret values copied from another project
- claiming a deploy path is complete without proving the project can build

## Expected Outcome

When this skill is used well, the agent should be able to say, with proof:

- what stack the project actually uses
- why Expo/EAS, Fastlane, Capacitor, or native tooling is the right path here
- which existing deployment assets were reused
- which release services such as RevenueCat or EAS Update were verified or repaired
- whether the project uses a hybrid Expo plus Fastlane release model and how responsibilities are split
- which dependencies or credentials were missing and how they were wired
- what build/submission path was verified
- what remains blocked, if anything

## 2026-09-19 — Merged release laws (absorbed from the app-submission, app-store-autopilot and project-lane skills)

These were previously split across several overlapping deploy skills. They live here now so there is one owner for mobile release.

### What "verified" means
Live user surface only: the running app on a real device; ASC `reviewSubmissions` showing a fresh `submittedDate` + `WAITING_FOR_REVIEW` + the correct build attached; Play track reads showing the versionCode; IAP packages actually loading on device. A green build, an API 201, a dashboard setting or a successful upload is **not** shipping. Build `VALID` != approval; uploaded != submitted; TestFlight-ready != review-healthy.

### Human-only gates (everything else is automatable)
First-time app creation and signing; the Paid Apps Agreement; App Privacy publish; Play's "Send changes for review" click; attaching the **first** consumable/subscription to a version in ASC web.

### Apple
- Mint ASC JWTs with the raw 64-byte `R||S` ES256 signature. A DER signature returns `401 NOT_AUTHORIZED` and looks exactly like a revoked key.
- Guideline 3.1.2(c): subscription title, period and localized price **plus** functional EULA and privacy links on the purchase surface itself. ASC metadata being compliant is not enough.
- A distinct **Restore Purchases** control on *every* IAP surface; auto-restore on launch does not count (3.1.1).
- First consumable/subscription must be attached on the version page in ASC web; standalone submission returns 409.
- Cancelling a blocking submission flips the version and **drops the attached build**; re-attach and wait for the state flip before adding the review item (else 409 `ENTITY_STATE_INVALID`). Use relationship `appStoreVersion` (the name `appStoreVersionForReview` 409s).
- Metadata-only blockers: fix in ASC and resubmit the **same** valid build; never rebuild for metadata.
- Approval != public listing: `appAvailabilityV2` must exist with territories enabled; verify the public storefront.
- Age rating declarations live at `/v1/appInfos/{appInfoId}/ageRatingDeclaration` — **not** on `/v1/apps/{id}` or `/v1/appStoreVersions/{id}` (both 404 `PATH_ERROR`). Read the `platform` attribute before treating any version as blocking state: a `MAC_OS` draft never blocks iOS.
- Reviewers use iPhone 17 Pro Max and iPad Air 11-inch (M3); test iPad explicitly. Frozen state >~24h = wedged: cancel and resubmit.

### Google Play
- The privacy policy must name the app, the developer and the legal entity exactly as the store listing does — a mismatch is rejected as "Invalid Privacy Policy".
- `assetlinks.json` must carry the **Play App Signing** SHA-256 (`generatedApks.certificateSha256Hash`); the upload key never signs installed builds.
- After a rejection: commit edits with `changesNotSentForReview=true`, promote the fixed release, then the human "Send changes for review" click queues it. Trust the tracks API over the rejection email's versionCode.
- First-publish draft apps accept only `status: draft` via the edits API; Data safety, IARC, Ads and Sign-in are Console-only; a greyed IARC "Next" is one unanswered radio pair.
- Google Play references must not appear in an iOS build (Guideline 2.3.10): keep every store mention platform-conditional, and gate cross-store download buttons on the native platform rather than the browser.

### Build systems
- Never git-track `ios/` or `android/` in a CNG/Expo repo — a single tracked file makes every EAS build skip `expo prebuild` (no Podfile, `spawn pod ENOENT`). Capacitor repos are the opposite: those directories are tracked and must stay synced with `npx cap sync`.
- Do not pin an Xcode 26 image for Expo SDK 57: `expo-modules-jsi` uses `weak let`, rejected by Swift 6. Use the SDK-aligned image (`sdk-57`).
- Extension targets (share/widget) need the remote-credential profile.
- `npx expo install --fix` before local builds; never run two local EAS builds concurrently.
- If `eas submit` is opaque, upload the existing artifact with `xcrun altool --upload-app` — do not rebuild first.
- One native release per session/day for Capacitor lanes; never run two fastlane lanes concurrently (shared `dist/` and the Play edit API produce "This edit has expired").
- A documented local lane can be broken by the OS toolchain: on macOS 26 system Ruby 2.6 reports `universal-darwin26` while the Xcode 26.5 SDK ships only `universal-darwin25` headers, so `bundle exec fastlane` fails — **and pnpm still exits 0**. Read the log, not the exit code; use the standalone homebrew fastlane as the substitute.

### Regression guards that saved real releases
- Never change a shipped response shape to carry extra data: if installed clients map over an array, add a sibling endpoint instead of wrapping the payload.
- Verify a "dead" flag before deleting it — a flag can have one live caller in an unrelated path.
- Remove source-guard residue (comments naming deleted identifiers) or the test suite stays red.
- When several agents share one worktree, commit with an isolated index (`GIT_INDEX_FILE` + `read-tree`/`write-tree`/`commit-tree`/`update-ref`) and repair the real index afterwards; a plain `git commit` can resurrect another agent's deleted work from a stale shared index.

### Closure format
One line per layer, no narrative: `WEB: <hash> live | iOS: <build> WAITING_FOR_REVIEW|skipped(<why>) | Android: <versionCode> <track>|skipped(<why>)`, plus the SHIPPED/EVIDENCE line. Unresolved work is `CHECKPOINT`/`BLOCKED` with the exact count — never padded into a report.

## Related specialist skills (do not duplicate)

- Coolify / DNS / DB / auto-deploy: `coolify-megawebs-deploy-domain-db-autodeploy-skill`
- Whole-project analytics/SEO/growth umbrella: `complete-project-deploy-analytics-growth-skill`
- Store listing media: `app-store-media-generator-skill`
- Preferred ASC CLI when Helm is installed: `helm-asc` (command -v helm-asc, else /opt/homebrew/bin/helm-asc, else Helm.app Helpers). Query IDs before mutating; --agent on obvious workflows.
- Device verification: `ios-automation` / `android-automation` (Midscene; never background, one command at a time).
