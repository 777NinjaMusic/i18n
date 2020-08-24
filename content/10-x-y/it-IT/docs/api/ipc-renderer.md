# ipcRenderer

> Comunicazione asincrona dal un processo di rendering al processo principale.

Processo: [Renderer](../glossary.md#renderer-process)

The `ipcRenderer` module is an  [EventEmitter][event-emitter]. Fornisce alcuni memtodi che permettono l'invio di messaggi sincroni e assincroni dal processo di rendiring(pagina web) al processo principale. È inoltre possibile ricevere risposte dal processo principale.

Vedere [ipcMain](ipc-main.md) per esempi di codice.

## Metodi

Il modulo di `ipcRenderer` ha il seguente metodo per attendere gli eventi e inviare messaggi:

### `ipcRenderer.on(channel, listener)`

* `channel` String
* `listener` Function
  * `event` IpcRendererEvent
  * `...args` any[]

Ascola sul `channel`, quando arriva un nuovo messaggio il `listener` verrà chiamato con `listener(event, args...)`.

### `ipcRenderer.once(channel, listener)`

* `channel` String
* `listener` Function
  * `event` IpcRendererEvent
  * `...args` any[]

Adds a one time `listener` function for the event. This `listener` is invoked only the next time a message is sent to `channel`, after which it is removed.

### `ipcRenderer.removeListener(channel, listener)`

* `channel` String
* `listener` Function
  * `...args` any[]

Il `listener` specificato viene rimosso dalla lista dei listener del `channel` indicato.

### `ipcRenderer.removeAllListeners(channel)`

* `channel` String

Rimuove tutti i listeners, oppure quelli del `channel` specificato.

### `ipcRenderer.send(channel, ...args)`

* `channel` String
* `...args` any[]

Send an asynchronous message to the main process via `channel`, along with arguments. Arguments will be serialized with the [Structured Clone Algorithm][SCA], just like [`window.postMessage`][], so prototype chains will not be included. Sending Functions, Promises, Symbols, WeakMaps, or WeakSets will throw an exception.

> **NOTE**: Sending non-standard JavaScript types such as DOM objects or special Electron objects is deprecated, and will begin throwing an exception starting with Electron 9.

The main process handles it by listening for `channel` with the [`ipcMain`](ipc-main.md) module.

If you need to transfer a [`MessagePort`][] to the main process, use [`ipcRenderer.postMessage`](#ipcrendererpostmessagechannel-message-transfer).

If you want to receive a single response from the main process, like the result of a method call, consider using [`ipcRenderer.invoke`](#ipcrendererinvokechannel-args).

### `ipcRenderer.invoke(channel, ...args)`

* `channel` String
* `...args` any[]

Returns `Promise<any>` - Resolves with the response from the main process.

Send a message to the main process via `channel` and expect a result asynchronously. Arguments will be serialized with the [Structured Clone Algorithm][SCA], just like [`window.postMessage`][], so prototype chains will not be included. Sending Functions, Promises, Symbols, WeakMaps, or WeakSets will throw an exception.

> **NOTE**: Sending non-standard JavaScript types such as DOM objects or special Electron objects is deprecated, and will begin throwing an exception starting with Electron 9.

The main process should listen for `channel` with [`ipcMain.handle()`](ipc-main.md#ipcmainhandlechannel-listener).

Ad esempio:
```javascript
// Renderer process
ipcRenderer.invoke('some-name', someArgument).then((result) => {
  // ...
})

// Main process
ipcMain.handle('some-name', async (event, someArgument) => {
  const result = await doSomeWork(someArgument)
  return result
})
```

If you need to transfer a [`MessagePort`][] to the main process, use [`ipcRenderer.postMessage`](#ipcrendererpostmessagechannel-message-transfer).

If you do not need a respons to the message, consider using [`ipcRenderer.send`](#ipcrenderersendchannel-args).

### `ipcRenderer.sendSync(channel, ...args)`

* `channel` String
* `...args` any[]

Restituisce `any` - il valore inviato indietro dal gestore [`ipcMain`](ipc-main.md).

Send a message to the main process via `channel` and expect a result synchronously. Arguments will be serialized with the [Structured Clone Algorithm][SCA], just like [`window.postMessage`][], so prototype chains will not be included. Sending Functions, Promises, Symbols, WeakMaps, or WeakSets will throw an exception.

> **NOTE**: Sending non-standard JavaScript types such as DOM objects or special Electron objects is deprecated, and will begin throwing an exception starting with Electron 9.

Il processo principale lo gestisce ascoltando il `channel` con il modulo [`ipcMain`](ipc-main.md), e risponde impostando un valore `event.returnValue`.

> :warning: **WARNING**: Sending a synchronous message will block the whole renderer process until the reply is received, so use this method only as a last resort. It's much better to use the asynchronous version, [`invoke()`](ipc-renderer.md#ipcrendererinvokechannel-args).

### `ipcRenderer.postMessage(channel, message, [transfer])`

* `channel` String
* `message` any
* `transfer` MessagePort[] (optional)

Send a message to the main process, optionally transferring ownership of zero or more [`MessagePort`][] objects.

The transferred `MessagePort` objects will be available in the main process as [`MessagePortMain`](message-port-main.md) objects by accessing the `ports` property of the emitted event.

Ad esempio:
```js
// Renderer process
const { port1, port2 } = new MessageChannel()
ipcRenderer.postMessage('port', { message: 'hello' }, [port1])

// Main process
ipcMain.on('port', (e, msg) => {
  const [port] = e.ports
  // ...
})
```

For more information on using `MessagePort` and `MessageChannel`, see the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel).

### `ipcRenderer.sendTo(webContentsId, channel, ...args)`

* `webContentsId` Number
* `channel` String
* `...args` any[]

Sends a message to a window with `webContentsId` via `channel`.

### `ipcRenderer.sendToHost(channel, ...args)`

* `channel` String
* `...args` any[]

Come `ipcRenderer.send` ma l'evento verrà inviato al `<webview>`elemento nella pagina host anziché il processo principale.

## L'oggetto Event

The documentation for the `event` object passed to the `callback` can be found in the [`ipc-renderer-event`](structures/ipc-renderer-event.md) structure docs.

[event-emitter]: https://nodejs.org/api/events.html#events_class_eventemitter
[SCA]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm
[`window.postMessage`]: https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage
[`MessagePort`]: https://developer.mozilla.org/en-US/docs/Web/API/MessagePort