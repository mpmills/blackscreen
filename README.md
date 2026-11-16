# blackscreen
a browser based monitor screen blackout page.

## 1. Project summary

Black Screen is a small, browser-based utility for turning the current monitor completely black. It gives the user a fast way to enter fullscreen blackout mode, optionally run a countdown that exits blackout mode automatically, and reveal temporary exit controls without leaving the dark screen.

The product should be deliberately minimal: one static page, no accounts, no backend, no external dependencies, and no unnecessary visual effects.

## 2. Goals

- Let the user black out the current monitor with one primary action.
- Make the fullscreen blackout genuinely black, with no persistent controls or light-producing interface.
- Provide an optional countdown that automatically ends the blackout.
- Let the user reveal the remaining time and exit controls with one click or tap.
- Remember timer preferences locally without storing blackout-session history.
- Keep the implementation small, understandable, responsive, and accessible.

## 3. Non-goals

- Controlling other monitors or operating-system display settings.
- Turning off a physical display.
- Saving, reporting, or analyzing previous blackout sessions.
- User accounts, cloud synchronization, analytics, or a backend.
- Operating-system notifications or notification permissions.
- Audio alarms, scheduling, themes, or other productivity features.

## 4. Terminology

- **Control screen:** The normal page containing the title, primary action, and timer settings.
- **Blackout mode:** The pure-black fullscreen state.
- **Exit timer:** The optional countdown that ends blackout mode when it reaches zero. This is the feature previously described as **Keep Time**.
- **Exit prompt:** The temporary overlay shown after the user clicks or taps during blackout mode.

## 5. Primary user flow

1. The user opens the page and sees the control screen on a dark-gray background.
2. The user optionally enables **Set an exit timer** and selects a duration.
3. The user selects **Go Black**.
4. The browser enters fullscreen and the page becomes pure black.
5. If the exit timer is enabled, the countdown starts after fullscreen is successfully entered.
6. The user can click or tap the black screen to reveal the remaining time and exit prompt.
7. The user either dismisses the prompt, exits manually, or allows the countdown to reach zero.
8. The page leaves fullscreen and returns to the control screen.

## 6. Control screen

### Required content

- Product name: **Black Screen**
- Subtitle: **A browser-based monitor blackout tool.**
- Primary button: **Go Black**
- Timer toggle: **Set an exit timer**
- Timer duration controls, shown only when the timer is enabled
- A short instruction explaining that clicking the black screen reveals the timer and exit controls

### Initial state

- The page background is dark gray, not pure black, so blackout mode remains visually distinct.
- The exit timer is off by default for a first-time visitor.
- Timer duration controls are hidden while the exit timer is off.
- A default duration of 10 minutes is selected when the timer is first enabled.

### Timer duration controls

The initial version should offer these presets:

- 10 minutes
- 30 minutes
- 1 hour

It should also support a custom duration in hours and minutes.

Validation rules:

- Minimum duration: 1 minute
- Maximum duration: 24 hours
- The user cannot begin a timed blackout with an empty, zero, negative, or invalid duration.
- Validation feedback appears beside the duration controls and does not rely on color alone.

## 7. Entering blackout mode

Selecting **Go Black** must directly request the browser Fullscreen API because browsers require fullscreen requests to originate from a user action.

On successful fullscreen entry:

- The viewport becomes pure black (`#000000`).
- Normal page controls are hidden.
- No timer or exit control is initially visible.
- The pointer may disappear after a short idle period and return on movement.
- If enabled, the exit countdown starts.

The countdown must begin only after fullscreen entry succeeds. It must not begin while the browser is still awaiting or rejecting the fullscreen request.

## 8. Blackout mode

### Default appearance

- The entire viewport is pure black.
- There is no persistent text, timer, border, glow, or control.
- The page must not scroll.
- Blackout mode continues until it is ended manually, by the optional timer, or by the browser.

### Interaction

Clicking or tapping anywhere in blackout mode reveals the exit prompt. This interaction does not pause, reset, or otherwise alter the countdown.

Repeated clicks while the prompt is open must not create duplicate prompts or timers.

## 9. Exit prompt and visible countdown

When the exit prompt opens:

- The current remaining time appears as a large, subdued countdown behind the prompt.
- The countdown continues updating in real time.
- The countdown uses dark gray rather than bright white to minimize emitted light.
- A compact, high-contrast prompt appears above the countdown.
- The prompt contains **Exit Fullscreen** and **Dismiss** buttons.
- The prompt explains that the `F` key can also exit fullscreen while the prompt is visible.

### Prompt controls

- **Exit Fullscreen:** Immediately leaves fullscreen and returns to the control screen.
- **Dismiss:** Hides both the prompt and visible countdown, returning to a completely black screen.
- Clicking or tapping outside the prompt dismisses it.
- Pressing `F` exits fullscreen only while the prompt is visible.
- Pressing `F` while the prompt is hidden has no effect.
- Browser-reserved fullscreen behavior remains intact; in most browsers, `Escape` will leave fullscreen.

If the exit timer is disabled, the prompt still appears, but no countdown is shown.

If the countdown reaches zero while the prompt is open, blackout mode ends immediately as normal.

## 10. Timer behavior

When enabled:

- The timer starts from the selected duration after successful fullscreen entry.
- It counts down continuously whether the exit prompt is hidden or visible.
- Opening and dismissing the prompt never pauses or resets it.
- At zero, the page requests exit from fullscreen and returns to the control screen.
- Leaving fullscreen early cancels the active countdown.
- Starting a new blackout creates one new countdown from the configured duration.

The timer should calculate remaining time from a fixed end timestamp rather than depending only on interval ticks. This prevents noticeable drift when the browser throttles background work.

No elapsed duration, completed duration, or session history is retained after blackout mode ends.

## 11. Leaving blackout mode

Blackout mode can end when:

- The user selects **Exit Fullscreen**.
- The user presses `F` while the exit prompt is visible.
- The countdown reaches zero.
- The user uses the browser's fullscreen exit control, commonly `Escape`.
- The browser or operating system ends fullscreen unexpectedly.

Whenever fullscreen ends:

- Any active countdown interval is stopped.
- The exit prompt and visible countdown are hidden.
- Temporary blackout state is cleared.
- The control screen is restored.
- The configured timer preferences remain available for the next session.

Cleanup must be safe to run more than once so duplicate fullscreen events do not leave an interval running.

## 12. Fullscreen failure handling

If fullscreen is unsupported, denied, or fails:

- The app remains on the control screen.
- No countdown starts.
- A concise inline message explains that fullscreen could not be started and suggests checking browser permissions.
- The user can try again.

The first version does not need a simulated window-only blackout fallback.

## 13. Local settings

Use browser `localStorage` only for user preferences.

Persist:

- Whether the exit timer is enabled
- The selected preset or custom duration

Do not persist:

- The start or end time of a blackout
- Remaining countdown time
- Elapsed blackout time
- Previous session durations or history
- Whether the exit prompt was open

If stored settings are missing, malformed, or outside the valid duration range, the app should safely restore defaults.

## 14. Visual direction

- Palette: white, gray, and black
- Control-screen background: dark gray
- Blackout background: pure black
- Layout: centered, restrained, and responsive
- Primary action: prominent without decorative animation
- Typography: system font stack; no external font request
- Components: simple controls with clear focus and hover states
- Decorative imagery: none required

The supplied mockup is visual inspiration rather than a layout requirement.

## 15. Accessibility and input support

- All control-screen functions work with keyboard, mouse, and touch.
- Interactive elements use native buttons, inputs, and labels where possible.
- Keyboard focus is visible on the control screen and in the exit prompt.
- When opened, the exit prompt receives focus and is announced appropriately to assistive technology.
- The prompt has an accessible name and modal semantics.
- Button and toggle states are programmatically exposed.
- Touch targets are at least approximately 44 by 44 CSS pixels.
- Text and controls meet WCAG AA contrast expectations.
- Validation errors are associated with the relevant duration fields.
- Motion is minimal and respects reduced-motion preferences.

## 16. Responsive behavior

- The control screen fits comfortably on common desktop and mobile viewports.
- Timer presets may wrap or stack on narrow screens.
- The exit prompt remains centered and fully usable on small displays.
- The large visible countdown scales without overflowing the viewport.
- Mobile fullscreen limitations are handled through the same failure message rather than device-specific instructions.

## 17. Minimal technical approach

The implementation should remain buildless and dependency-free:

- One `index.html` file with embedded CSS and JavaScript
- No framework or component library
- No package manager or build step
- No backend, database, account system, or external API
- No images or external font files
- Fullscreen API for entering and leaving fullscreen
- `fullscreenchange` for authoritative fullscreen-state cleanup
- One countdown interval while a timed blackout is active
- A fixed end timestamp for accurate remaining-time calculation
- `localStorage` only for timer preferences

Additional files should be introduced only if they provide a clear maintenance benefit.

## 18. Application states

The interface has four meaningful states:

1. **Ready:** The control screen is visible and editable.
2. **Entering fullscreen:** A fullscreen request is pending; no timer is running yet.
3. **Blackout active:** The viewport is pure black and the optional timer is running invisibly.
4. **Exit prompt visible:** Blackout remains active while the countdown and exit controls are shown.

Any fullscreen exit returns the application to **Ready**.

## 19. Acceptance criteria

The first version is complete when all of the following are true:

- The page opens on a dark-gray control screen.
- **Go Black** requests fullscreen from the user's button action.
- Successful fullscreen entry produces a pure-black viewport with no visible controls.
- The exit timer defaults to off for a first-time visitor.
- Enabling the timer reveals presets and a valid custom-duration option.
- A timed countdown begins only after fullscreen entry succeeds.
- Clicking or tapping blackout mode reveals the exit prompt.
- When enabled, the remaining countdown appears behind the prompt and continues updating.
- Dismissing the prompt hides both the prompt and countdown without affecting the timer.
- **Exit Fullscreen** exits blackout mode.
- `F` exits only while the exit prompt is visible.
- A countdown reaching zero exits blackout mode automatically.
- Manual or browser-driven fullscreen exit reliably stops the active timer.
- Failed fullscreen entry does not start a timer and produces a useful message.
- Timer preferences survive a reload through local storage.
- No blackout-session times or history are saved.
- Rapid or repeated interactions do not create duplicate intervals or prompts.
- The page remains usable with keyboard, mouse, and touch.
- The implementation has no runtime dependencies or network requirement.

## 20. Deferred ideas

The following ideas are intentionally outside the first release but could be considered later:

- Additional timer presets
- Optional sound or vibration at timer completion
- A window-only fallback when fullscreen is unavailable
- Multiple visual themes for the control screen
- Installable Progressive Web App support

