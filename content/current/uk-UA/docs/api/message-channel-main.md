# MessageChannelMain

`MessageChannelMain` is the main-process-side equivalent of the DOM [`MessageChannel`][] object. Its singular function is to create a pair of connected [`MessagePortMain`](message-port-main.md) objects.

See the [Channel Messaging API][] documentation for more information on using channel messaging.

## Class: MessageChannelMain

Приклад:
```js
const { port1, port2 } = new MessageChannelMain()
w.webContents.postMessage('port', null, [port2])
port1.postMessage({ some: 'message' })
```

### Властивості Екземпляра

#### `channel.port1`

A [`MessagePortMain`](message-port-main.md) property.

#### `channel.port2`

A [`MessagePortMain`](message-port-main.md) property.

[`MessageChannel`]: https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel
[Channel Messaging API]: https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API