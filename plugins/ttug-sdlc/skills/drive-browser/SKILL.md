---
name: drive-browser
description: Drives a real browser page through chrome-devtools-mcp to check a change — your own tab, the cheapest read that answers the question (a page script, then a text snapshot, then a screenshot), held keys for games, and nothing left behind. Use for a build-to-merge "check for real" step that needs a page exercised, when the owner says "open it in Chrome", "check it in the browser" or "drive the page", and before recording a video of a page.
---

# Drive a browser page

For looking at a running page yourself. A case you will want to run again belongs in the repo's own test runner, not here.

## Before the first call

- **The tools may not be there.** A session whose chrome-devtools server failed to start shows no error; the tools are just missing. Look for them before planning around them, and say so if they are absent.
- **Connect-only setups need a browser first.** When the server attaches to a running Chrome over a debug port, start that Chrome with the machine's launcher — the repo or machine settings name it. Never attach to, or close, the owner's everyday browser.
- **One controller per browser.** The MCP server and a Playwright run connected over CDP to the same Chrome hang each other, with no error. Run one, then the other.

## Your own tab

Open a tab with `new_page` (it needs a `url`; `background: true` keeps it from taking the front). It returns a list of every tab in the browser: yours is the line marked `[selected]`. Keep that id and pass it as `pageId` on every call. Other agents may be working in the same browser: do not navigate, select or close a tab you did not open. Ids change whenever the server reconnects or the browser restarts, and the tool says so. After a reconnect find your tab again by its URL in `list_pages`; after a restart it is gone, so open a new one.

## Read the cheapest thing that answers the question

| Order | Tool | Good for | Keep it small |
| :-- | :-- | :-- | :-- |
| 1 | `evaluate_script` | A value: app state, one element's text, a count, a computed style | Return the fields you need, never the DOM. `filePath` sends a large result to a file |
| 2 | `take_snapshot` | What is on the page, and the element ids that `click` and `fill` need | Leave `verbose` off. It goes stale on navigation and when the page re-renders the element: take a new one |
| 3 | `take_screenshot` | How it looks: layout, colour, overlap | `uid` for one element instead of the page; `format: "jpeg"`; `filePath` to save it and read it only if you need to. The tool fixes the extension to match the format (`shot.jpg` is written as `shot.jpeg`): use the path its reply names |
| 4 | A video | How it changes over time | See `ttug-sdlc:video-evidence` |

A script answer is a few hundred characters; a screenshot is an image to read. Most "did it work?" questions are a value.

**On a high-density display, shrink the tab before a screenshot.** A screenshot comes back at the display's pixel ratio: a 932-pixel-wide page on a 2x display is a 1864-pixel image. `emulate` with `viewport: "1280x800x1"` on your own tab gives a 1280x800 image, about 70% fewer pixels, and leaves the window itself alone. `evaluate_script` returning `devicePixelRatio` tells you whether it is needed.

- **Console:** `list_console_messages` with `types: ["error", "warn"]`, not the whole log. Add `"issue"` for CSP and CORS problems and `"assert"` for failed asserts.
- **Network:** `list_network_requests` with `resourceTypes` (say `["xhr", "fetch"]`) and a `pageSize`.
- **An empty console is not always a clean one.** The console list restarts at every navigation, and a client-side route change counts: an error logged before the route changed is gone from the default list. `includePreservedMessages: true` reaches back three navigations. The network list survives a route change; `includePreservedRequests` does the same for it.

## Act

- **Click and fill by `uid`** from the latest snapshot. `includeSnapshot: true` on `click` returns the new one in the same call.
- **Wait for text, not for time:** `wait_for` with `text: ["…"]` (a list) that proves the step finished. It gives up after 5 s unless you pass a `timeout`, and its reply already holds a fresh snapshot. It cannot see text drawn on a canvas: for a game, poll the readout with `evaluate_script`.
- **Hold keys, never tap, for anything polled per frame** (games). `press_key` goes down and up at once, and a press that falls between two frames is never seen. Hold through `evaluate_script`, sampling the app's readout in the same call:

  ```js
  async () => {
    const read = () => window.__app.state();   // the app's own readout
    read();                                    // a wrong name fails here, before any key goes down
    const key = (t, k, c, kc) => (document.activeElement ?? document.body).dispatchEvent(new KeyboardEvent(t, { key: k, code: c, keyCode: kc, which: kc, bubbles: true }));
    const frame = () => new Promise((r) => { requestAnimationFrame(r); setTimeout(r, 100); });
    const hold = async (label, keys, ms) => {
      const s = [read()], end = performance.now() + ms;
      keys.forEach((k) => key("keydown", ...k));
      try { while (performance.now() < end) { await frame(); s.push(read()); } }
      finally { keys.forEach((k) => key("keyup", ...k)); }
      return { label, samples: s.length, perSecond: Math.round((s.length - 1) / (ms / 1000)), dx: Math.round(s.at(-1).x - s[0].x) };
    };
    return [await hold("run right", [["ArrowRight", "ArrowRight", 39]], 1000)];
  }
  ```

  - The events go to the focused element and bubble, so listeners on the element, `document` and `window` all get them; dispatched on `window` they never reach a `document` listener.
  - `finally` sends the key-up even when the readout throws, so a failed call never leaves a key held.
  - `perSecond` near the display's refresh rate means the page was painting. Near 10 means it was not — the timeout was driving the loop — and a `dx` of 0 then says nothing about the app.
  - Replace the whole `window.__app.state()` call with the repo's readout, and `x` with the field it reports.
- **Synthetic input has limits.** It moves a character; it cannot grant what needs a real user gesture (pointer lock, sound). Hand those flows to the owner.
- **Sign-ins are the owner's.** They type the credentials. A profile that holds a sign-in is no longer throwaway: say so before deleting it.

## Finish

- Close the tabs you opened.
- Stop the browser only if you started it, and only that one, by its profile. Killing by image name takes the owner's browser too.
- Delete a throwaway profile you created. A fresh Chrome profile is over 100 MB.
- Report what you read and how: a value from a script, a snapshot, or a screenshot you looked at.

## Repo and machine settings

Keys from build-to-merge's `settings.md`: `Browser:` (the launcher and its port), `Readout:` (the app's debug readout), `Ports:` (the dev server). A readout such as `window.__game.debug()` replaces the whole `window.__app.state()` call in the script above.
