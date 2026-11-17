# Surprise Gifts — implementation

## Flow

The `--surprise-gifts-demo` launch argument selects a simulated marketplace purchase flow. A purchase records the selected collectible locally and opens the recipient picker. The native transfer confirmation adds an unchecked surprise option. Confirmation inserts a native gift service message into the selected chat's local history.

Unchecked inserts the opened marker; checked inserts the sealed marker. Unwrapping updates that local message to the opened state. Account-scoped state is used so switching accounts does not share demo purchases.

## Native integration

Paths below are relative to the repository root.

| Source | Responsibility |
| --- | --- |
| `submodules/TelegramUI/Components/Gifts/GiftViewScreen/Sources/GiftViewScreen.swift` | Demo marketplace purchase and recipient selection |
| `submodules/TelegramUI/Components/Gifts/GiftViewScreen/Sources/GiftTransferAlertController.swift` | Surprise checkbox and simulated transfer confirmation |
| `submodules/TelegramCore/Sources/TelegramEngine/Messages/LocalWidgetMessages.swift` | Native local service-message insertion and reveal |
| `submodules/TelegramCore/Sources/TelegramEngine/Messages/TelegramEngineMessages.swift` | Engine methods exposed to the UI |
| `submodules/TelegramUI/Components/Chat/ChatMessageGiftBubbleContentNode/Sources/ChatMessageGiftBubbleContentNode.swift` | Card layout, variants, reveal timing and persistence |
| `submodules/TelegramUI/Components/Gifts/GiftAnimationComponent/Sources/GiftCompositionComponent.swift` | Collectible composition and model reels |
| `submodules/TelegramUI/Components/Gifts/GiftSlotsComponent/` | Attribute and number text reels shared with the upgrade screen |

`GiftDemoScreen` also contains an earlier standalone prototype and its store. The current interaction uses the native chat gift message, rather than that separate screen.

## Card and animation

- The logical card width is 220 pt, with the native 224 pt outer plate and 216 pt inner backdrop. Outer and inner corner radii are 24 and 20 pt.
- The sealed card uses the green-star gift sticker and a black inner surface at 12% opacity over the chat's service-background plate. It does not show the collectible's pattern.
- The cycling composition uses a 120 pt model and chat-specific backdrop geometry. The regular upgrade screen keeps its own geometry defaults.
- The composition remains attached as the message becomes opened. Deceleration promotes the selected reel host without replacing its model node or restarting playback.
- Cycling numbers are cosmetic. The final number and attributes come from the chosen collectible.
- Reduce Motion bypasses the animated reveal.

## What is local

Gift messages use `Namespaces.Message.Local` without the outgoing-pending flag. Insertion and reveal do not send a message to the recipient. Marketplace browsing and loading collectible assets still use the normal Telegram client and its network connection; this is not an offline account harness.

The demo does not purchase a real inventory item, debit a wallet, transfer ownership, synchronize the reveal to another device or notify a recipient. Production would require corresponding server behavior and a review of forwarding, deletion, chat-list previews and history reconciliation for local demo messages.

## Build and reproduction

Base checkout: `24638bc69bd083981b178dd1e7f921eebbf7fc46`.

Apply `changes.patch` to the pinned base using the instructions in README.md, then use the upstream compilation guide. The patch includes new source files and was checked against a pristine base index with `git apply --cached --check`. A separate clean-checkout build has not been run.

The project pins Xcode in `versions.json`. The local build used Xcode 27.2 with the version override and the installed Metal toolchain. Set up your own API identity and build configuration as described in the root guide. Launch with `--surprise-gifts-demo`.

## Verification

The full Bazel simulator target built successfully, and the fresh app bundle was installed and launched while preserving the account's data container. `git diff --check` passed for the code changes.

The earlier blank-model handover was confirmed in recorded frames. The retained-composition fix, subsequent background layering correction and translucent surface still need a fresh frame-level verification. Build success does not establish visual correctness. Light/dark themes, cell reuse, chat re-entry, Reduce Motion and the shared ordinary upgrade flow remain relevant checks.

## Demo media

A public implementation recording has not been prepared. Three selected stills from the supplied design reference are included in `screenshots/` and labelled as concept frames in the README. The full reference video and development recordings are not bundled. The intended implementation demonstration is: choose collectible → select recipient → enable surprise → confirm → sealed chat card → unwrap → settled collectible.
