# native bridge and plugins

dframework allows packaging web applications as native mobile (iOS, Android) and desktop (Tauri) binaries with access to native platform APIs through `App.native`.

## calling native apis

the `App.native` object is globally available across all platforms (web, iOS, Android, Desktop):

```javascript
// device metadata
const platform = await App.native.device.platform(); // 'ios', 'android', 'desktop', 'web'
const model = await App.native.device.model();

// secure device keypair and signatures
const { publicKey } = await App.native.crypto.generate();
const { signature } = await App.native.crypto.sign('payload-base64');

// device session and persistence
await App.native.auth.setSession('session-token-123');
const session = await App.native.auth.getSession();

// generic key value storage
await App.native.storage.set('theme', 'dark');
const theme = await App.native.storage.get('theme');

// local notification
const permission = await App.native.notifications.requestPermission();
if (permission === 'granted') {
  await App.native.notifications.show('sync complete', 'all items updated', 'sync-10');
}

// audio playback with system media session controls
await App.native.audio.load('https://example.com/audio.mp3');
await App.native.audio.setMetadata('song title', 'artist name', 'album', 240);
await App.native.audio.play();
```

## core plugins overview

- `auth`: device session management and key value persistence (`setSession`, `getSession`, `clearSession`, `db_get`, `db_set`)
- `device`: read only platform metadata (`name`, `model`, `platform`)
- `storage`: generic key value store for preferences (`set`, `get`, `remove`)
- `crypto`: hardware backed ECDSA P-256 keypairs, signatures, and SHA-256 digests
- `audio`: single track media playback with lock screen and notification controls
- `notifications`: local push notifications with permission handling and cancellation by id

## zero config plugin architecture

plugins live under `native/plugins/<name>/` and support up to four files:

- `index.js` (required): javascript reference implementation and universal web fallback
- `Plugin.swift` (optional): native iOS implementation
- `Plugin.kt` (optional): native Android implementation
- `Plugin.rs` (optional): native Desktop Tauri implementation

### method matching and fallback

- `index.js` is the source of truth for method names and contracts
- if a native file is omitted or fails on a given platform, the framework automatically executes the javascript reference implementation in the webview
- generate a new plugin scaffold with: `dstrn make:plugin <name>`

## native cli tools

- `dstrn simulate --ios`: boots iOS simulator and attaches dev server
- `dstrn simulate --android`: boots Android emulator with localhost forwarding
- `dstrn simulate --desktop`: opens Tauri desktop testing window
- `dstrn build --[ios|android|desktop]`: produces production binaries
- `dstrn native:doctor`: verifies build tools, SDKs, and plugin wiring