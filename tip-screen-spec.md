# Interactive Tip-Screen Prop — Project Spec

**Owner:** Joseph Bath (Growth)
**Context:** Golden Goat Café takeover · Gumloop 3.0 launch
**Status:** Built (v1), ready for device setup
**Deliverable:** `tip-screen.html` (single self-contained file)

---

## 1. Overview

An interactive recreation of the Square checkout tip screen, built to run fullscreen on an iPad at the Golden Goat Café takeover. It exists purely as a visual prop: customers tap it, it reacts like a real terminal, and we capture the moment on video. It does not process payments, connect to Square, or collect any data.

The screen faithfully mirrors the Square "Add a Tip" layout — near-black background, hatched blue preset buttons, and the Custom Tip / No Tip row — so it reads as authentic on camera. Copy and amounts are isolated in one config block so the same prop can be reskinned for the discount mechanic (e.g. preset buttons becoming discount tiers instead of tip amounts).

## 2. Goals & non-goals

**Goals**
- Look indistinguishable from a real Square tip screen on camera.
- React instantly and satisfyingly to a tap (press feedback → processing → confirmation).
- Run locked-down on an iPad so the public can use it unsupervised.
- Let a barista reset the flow between customers/takes without breaking the illusion.
- Be trivially editable to swap tip copy for discount copy.

**Non-goals**
- No real payment processing, Square integration, or card reading.
- No data capture, analytics, or backend.
- No native app / App Store / TestFlight distribution.

## 3. Platform decision

Standalone fullscreen HTML page, **not** a TestFlight/native app.

| | HTML page (chosen) | TestFlight app (rejected) |
|---|---|---|
| Setup | Host a file, add to Home Screen | Apple Dev account, Xcode build, provisioning, Beta review |
| Iteration | Edit + refresh | New build per change |
| Cost | Free | $99/yr |
| Kiosk lockdown | Guided Access (built into iOS) | Build it yourself |

The two iOS features that make the HTML route behave like a kiosk:
- **Add to Home Screen** — with the right meta tags, it launches fullscreen with no Safari address bar.
- **Guided Access** — triple-click the side button to lock the iPad to this one screen so customers can't swipe out or leave the page.

## 4. Target environment

- **Device:** iPad (any modern model), Safari.
- **Orientation:** Designed for landscape (matches the reference photo); flexes to portrait.
- **Hosting:** Static host — Netlify Drop, Vercel, or GitHub Pages. Once added to the Home Screen, it runs without the address bar; keep the iPad on Wi-Fi for first load.
- **Offline:** After first load the page is light enough to stay cached for the session. For a guaranteed-offline shoot, the file can also be opened locally.

## 5. Functional requirements

1. Display a tip screen with title, subtotal, three preset amounts, Custom Tip, No Tip, Cancel, and a language label.
2. Tapping a preset or a custom amount runs a short processing animation, then a green checkmark and a thank-you message, then auto-returns to the start screen.
3. **No Tip** runs the same confirmation flow without an amount.
4. **Custom Tip** opens a working numeric keypad; the entered amount flows into the confirmation.
5. **Cancel** returns to the start screen.
6. A barista can reset to the clean start screen at any time via a hidden control.
7. The screen auto-resets after a period of inactivity.

## 6. Screens & flows

- **Tip screen (default):** title "Add a Tip", subtotal, three preset buttons ($1 / $2 / $3), Custom Tip + No Tip row, globe + "English" footer.
- **Custom keypad:** numeric entry with backspace, a running `$0.00` display, Back, and Add Tip.
- **Confirmation overlay:** processing spinner (~1.1s) → animated green checkmark + "Thank you!" + amount line (~2.2s) → auto-reset.

Flow summary:
```
Tip screen ──tap $1/$2/$3──────────► Processing ──► Checkmark ──► (auto) ► Tip screen
           ──tap Custom Tip──► Keypad ──Add Tip──► Processing ──► Checkmark ──► Tip screen
           ──tap Cancel──────────────────────────────────────────────────► Tip screen
Any state  ──long-press top-left corner / 60s idle──────────────────────► Tip screen
```

## 7. Visual spec

Tuned to match the Square reference photo.

| Token | Value | Use |
|---|---|---|
| `--bg` | `#0a0a0a` | Screen background (near-black) |
| `--blue` | `#1c9dd7` | Preset tip buttons |
| `--blue-press` | `#1684b6` | Preset pressed state |
| `--blue-alt` | `#34a9cf` | Custom Tip / No Tip row |
| `--accent` | `#2bc0ff` | Cancel link |
| `--text` | `#ffffff` | Title, button labels |
| `--muted` | `#8a8a8e` | Subtotal, language label |

- **Typeface:** system stack (`-apple-system` → San Francisco on iPad), which reads very close to Square's font.
- **Texture:** subtle diagonal hatch on the preset buttons via a repeating linear gradient, matching the photo. Buttons dip down slightly and darken on touch.

## 8. Barista reset

- **Hidden long-press corner:** an invisible 90×90px zone in the **top-left**. Press and hold ~1.2s from any screen (including mid-confirmation or in the keypad) to snap back to the clean start. A stray tap does nothing. Invisible to customers.
- **Idle auto-reset:** returns to the start screen after **60s** of no touches, as a backstop for when a customer wanders off mid-flow.

## 9. Configuration

All in the `CONFIG` block at the top of the script; visible button text is in the HTML near the top of the file.

| Key | Default | Meaning |
|---|---|---|
| `subtotal` | `"2.70"` | Amount shown under the title |
| `thankYouMsg` | `"Thank you!"` | Confirmation headline |
| `processingMs` | `1100` | Processing spinner duration |
| `showCheckMs` | `2200` | How long the checkmark holds before auto-reset |
| `longPressMs` | `1200` | Hold time on the corner to reset |
| `idleResetMs` | `60000` | Inactivity reset (set `0` to disable) |

To change the preset amounts or labels, edit the button text and the `data-amount` attributes in the HTML.

## 10. Device setup (run-of-show)

1. Host `tip-screen.html` (Netlify Drop is fastest) and open the URL in Safari on the iPad.
2. Share → **Add to Home Screen**; launch from the new icon (fullscreen, no address bar).
3. iPad settings: lock orientation, set **Auto-Lock to Never**, turn **brightness up**, enable **Do Not Disturb**.
4. Settings → Accessibility → **Guided Access** on, set a passcode.
5. Triple-click the side button to start Guided Access and lock to the screen.
6. To reset between customers, long-press the top-left corner ~1.2s. To exit at end of day, triple-click and enter the passcode.

## 11. Filming notes

- Wipe the glass between takes (the prop is glossy and shows fingerprints).
- Mind glare from café lighting / windows; position the iPad to avoid reflections.
- The top-left reset corner keeps your hand out of frame and off the buttons between takes.
- Consider a stand or mount that matches the Square register silhouette for authenticity.

## 12. Discount-mechanic variant (planned)

For the Gumloop discount angle, reskin without touching the logic:
- Retitle "Add a Tip" → e.g. "Your discount".
- Swap preset labels to discount tiers and relabel the confirmation copy.
- Optionally adjust the secondary row (e.g. "Custom" → a branded option).

Keep this as a second saved copy of the file so the authentic tip version stays available for footage.

## 13. Risks & edge cases

- **Wi-Fi drop:** mitigate by adding to Home Screen ahead of time / loading once before the event; or run the file locally for a fully offline shoot.
- **Accidental reset:** unlikely given the long-press requirement, but the corner is intentionally placed in dead space away from buttons.
- **Screen sleep mid-event:** prevented by Auto-Lock = Never.
- **Customer escapes the page:** prevented by Guided Access.

## 14. File inventory

- `tip-screen.html` — the prop (HTML/CSS/JS, no dependencies, no build step).
- `tip-screen-spec.md` — this spec.
