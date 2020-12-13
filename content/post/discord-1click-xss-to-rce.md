---
title: "Discord 1-click XSS to RCE (2020)"
description: "Path traversal in an Electron app leading to NodeJS execution from browser"
date: 2022-10-01
draft: false
---

## Electron Isolation Overview

Discord's desktop app is made with [Electron](https://electronjs.org). The (modern) Electron security model focuses on having the browser sandbox, the renderer process, and the main process. each are isolated from each other as separate processes so require some interface to communicate, like so:
```
    main world (scripts inside the site)
      ↑                            |
      |       contextBridge        |
      |                            ↓
 isolated world (limited node as preload)
      ↑ ipcMain                    |
      |                            |
      |                ipcRenderer ↓
           main process (node)
```

[`ipcMain`](https://www.electronjs.org/docs/latest/api/ipc-main) and [`ipcRenderer`](https://www.electronjs.org/docs/latest/api/ipc-renderer) are event-based ways to communicate between the renderer and main process. The renderer doesn't have access to all capabilities and it's generally recommended to use it largely for such communication for security.

[`contextBridge`](https://www.electronjs.org/docs/latest/api/context-bridge) exposes functions in the preload (isolated world) to the main world in the `window` object.


## Discord Native Modules

Discord have their own native modules system with their updater, allowing separate modules for their purposes. The main modules they have are:
- `discord_desktop_core` - Loaded by the app (`app.asar`) which creates the main window with the client
- `discord_voice` - Native libraries for voice chat / screenshare / etc
- `discord_rpc` - Exposes some Node modules for creating and managing the client's [RPC server](https://discord.com/developers/docs/topics/rpc)

These are exposed to the main world (the Discord webapp loaded) via their own exposed context bridge, `DiscordNative`. Specifically, `DiscordNative.nativeModules.requireModule(name)`:

```js
function requireModule(name) {
  if (!/^discord_[a-z0-9_-]+$/.test(name) && name !== 'erlpack') {
    throw new Error('"' + String(name) + '" is not a whitelisted native module');
  }

  return require(name);
}
```

They use Node's `require`, which looks scary but the input is explicitly checked to begin with `discord_` or being `erlpack`. The paths (`module.paths`) which can be required are carefully controlled earlier on. However, what if we could add our own file in one of the paths which we could then require?


## Discord Native File Saving

Discord have an API in their context bridge for saving files with a UI prompt for where with `DiscordNative.fileManager.saveWithDialog`, this is why we need 1-click, as the user must click save themselves. We can supply our own contents, filename, and default directory. The default directory is appended to the Downloads folder for the running user. However, it isn't sanitized or checked for [path traversal](https://portswigger.net/web-security/file-path-traversal), so we can give it `../../../../../` (etc) to escape to the root dir.

Another context bridge API, `DiscordNative.fileManager.getModulePath()`, gives us the path straight to an allowed path by the native module requirer. So we can combine them for a full path into the directory we want.


## Bringing it all together

Combining these separate functions and knowledge, we can make our exploit:
- Save file with dialog with:
    - Filename: `discord_rce.js`
    - Contents: our Node payload, for example opening calc - `require('child_process').exec('calc.exe')`
    - Base path: `../../../../` (etc) + module path
- Require our self-written file with the native module API
- Node execution (profit!)


## Reporting

- 12th October 2020: Reported to Discord via HackerOne
- 13th October 2020: Fixed by checking path given to native file saving function
- 13th October 2020: Given $100 bounty
