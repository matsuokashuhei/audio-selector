# Spec: Corresponding output device suggestion

## Why
When a user selects an audio input device that can also serve as an output device
(e.g. a USB headset, Bluetooth headphones), they almost always want the same
physical device for output too. Today they must find and select it again from the
output list by number, even though it is the same device. Suggesting the
corresponding output device saves a keystroke and reduces mismatched
input/output pairs.

## Users
Users of `audio-selector` in the interactive flow who select a device that has
both input and output capability.

## Requirements
1. After the user selects an input device, the tool looks for an output device
   with the same CoreAudio device UID in the output device list.
2. Matching is by UID equality only — never by name. Two devices with the same
   name but different UIDs are not considered the same device.
3. When a corresponding output device exists:
   a. The output selection screen is shown as normal (the list is not skipped).
   b. The corresponding output device is placed first in the list (position 1).
   c. The corresponding output device is marked with `(matches input)` /
      `(入力と同じ)`.
   d. Pressing Enter (empty input) accepts the corresponding output device —
      this overrides the existing "Enter keeps current" behaviour for the
      output screen when a match exists.
   e. The user may type a different number to select a different output device
      (override).
4. When no corresponding output device exists, the output selection screen
   behaves exactly as today (current device first, Enter keeps current).
5. When the corresponding output device is also the current output device, show
   only the `(current)` marker (the match is redundant).
6. The confirmation screen is always shown afterward, per the existing 確定
   definition (explicit OK only).
7. The feature is always enabled; there is no flag to disable it.
8. `--built-in` mode is unaffected.
9. The suggestion applies regardless of how the input device was selected (typed
   number or Enter to keep current).

## Non-goals
- Matching by device name or fuzzy matching.
- Suggesting an input device from an output selection (input is selected first;
  there is no prior selection to match against).
- Changing the confirmation flow or the 確定 / キャンセル semantics.
- Affecting `--built-in` non-interactive mode.
- An opt-out flag.

## Edge cases & tradeoffs
- **Matched device == current device**: Show only `(current)`. The match info is
  redundant since the user already has the device as output. Avoids a noisy
  `(current) (matches input)` line.
- **Enter semantics change**: When a match exists, Enter no longer means "keep
  current" on the output screen — it means "accept the matched device." This is
  a behaviour change from today, but it is the whole point of the suggestion.
  The user can still keep the current device by typing its number. The current
  device keeps its `(current)` marker so the user can find it.
- **Multiple output devices with the same UID**: Cannot happen in CoreAudio
  (UIDs are unique). If it did, `first(withUID:)` takes the first.
- **No match**: Existing behaviour, unchanged.
- **Prompt text**: The prompt on the output screen must reflect the match. When
  a match exists, the hint changes from "Enter=keep current" to indicate that
  Enter accepts the matched device (exact wording is an implementation detail).
  When no match, the prompt is unchanged.
- **Chosen tradeoffs**: UID-only over name-matching (avoids false positives on
  duplicate names — the tests already have devices sharing the name "USB
  Audio"). Show-and-wait over skip-screen (preserves override ability and
  visibility). Always-on over a flag (override-by-typing makes it
  non-intrusive).

## Open questions
- Exact prompt wording when a match exists (implementation detail, to be
  decided during implementation).
- Whether help text should mention the matching behaviour (likely yes, for
  completeness).

## Acceptance criteria
- [ ] When the selected input device has a corresponding output device (same
      UID), the output screen shows that device first, marked `(matches input)`
      / `(入力と同じ)`.
- [ ] Pressing Enter on the output screen with a match present selects the
      matched device, even when it differs from the current output.
- [ ] Typing a different number on the output screen selects that device instead
      of the matched one.
- [ ] When no output device shares the input device's UID, the output screen
      behaves exactly as before.
- [ ] When the matched device is also the current output device, only
      `(current)` is shown.
- [ ] The confirmation screen still appears after output selection in all
      cases.
- [ ] `--built-in` mode is unchanged.
- [ ] Existing tests pass; new tests cover the match, no-match, override, and
      matched==current cases.
- [ ] `CONTEXT.md` contains the "対応する音声出力デバイス" term.
