# protocol (NetworkService) (Draft)

This document describes the new protocol APIs based on the [NetworkService](https://www.chromium.org/servicification).

We don't currently have an estimate of when we will enable the `NetworkService` by default in Electron, but as Chromium is already removing non-`NetworkService` code, we will probably switch before Electron 10.

The content of this document should be moved to `protocol.md` after we have enabled the `NetworkService` by default in Electron.

> カスタムプロトコルを登録し、既存のプロトコルリクエストを傍受します。

プロセス: [Main](../glossary.md#main-process)

`file://` プロトコルと同じ効果を持つプロトコルの実装の例:

```javascript
const { app, protocol } = require('electron')
const path = require('path')

app.on('ready', () => {
  protocol.registerFileProtocol('atom', (request, callback) => {
    const url = request.url.substr(7)
    callback({ path: path.normalize(`${__dirname}/${url}`) })
  })
})
```

**注釈:** 指定されていないすべてのメソッドは、`app` モジュールの `ready` イベントが発生した後にのみ使用できます。

## `protocol` をカスタムの `partition` や `session` で使用する

A protocol is registered to a specific Electron [`session`](./session.md) object. If you don't specify a session, then your `protocol` will be applied to the default session that Electron uses. However, if you define a `partition` or `session` on your `browserWindow`'s `webPreferences`, then that window will use a different session and your custom protocol will not work if you just use `electron.protocol.XXX`.

To have your custom protocol work in combination with a custom session, you need to register it to that session explicitly.

```javascript
const { session, app, protocol } = require('electron')
const path = require('path')

app.on('ready', () => {
  const partition = 'persist:example'
  const ses = session.fromPartition(partition)

  ses.protocol.registerFileProtocol('atom', (request, callback) => {
    const url = request.url.substr(7)
    callback({ path: path.normalize(`${__dirname}/${url}`) })
  })

  mainWindow = new BrowserWindow({ webPreferences: { partition } })
})
```

## メソッド

`protocol` モジュールには以下のメソッドがあります。

### `protocol.registerSchemesAsPrivileged(customSchemes)`

* `customSchemes` [CustomScheme[]](structures/custom-scheme.md)

**注意:** このメソッドは、`app` モジュールの `ready` イベントが発行される前にのみ使用でき、一度だけ呼び出すことができます。

Registers the `scheme` as standard, secure, bypasses content security policy for resources, allows registering ServiceWorker and supports fetch API. Specify a privilege with the value of `true` to enable the capability.

An example of registering a privileged scheme, that bypasses Content Security Policy:

```javascript
const { protocol } = require('electron')
protocol.registerSchemesAsPrivileged([
  { scheme: 'foo', privileges: { bypassCSP: true } }
])
```

標準スキームは、RFC 3986 で [Generic URI Syntax](https://tools.ietf.org/html/rfc3986#section-3) と呼ぶものに準拠しています。 例えば `http` と `https` は標準スキームですが、`file` はそうではありません。

Registering a scheme as standard allows relative and absolute resources to be resolved correctly when served. そうでないと、スキームは `file` プロトコルのように動作しますが、相対 URL を解決することはできません。

たとえば、標準スキームとして登録せずにカスタムプロトコルで以下のページをロードすると、非標準スキームが相対URLを認識できないため、イメージはロードされません。

```html
<body>
  <img src='test.png'>
</body>
```

スキームを標準で登録すると、[FileSystem API](https://developer.mozilla.org/en-US/docs/Web/API/LocalFileSystem) を介してファイルにアクセスできます。 そうしない場合、レンダラーはスキームのセキュリティエラーをスローします。

By default web storage apis (localStorage, sessionStorage, webSQL, indexedDB, cookies) are disabled for non standard schemes. So in general if you want to register a custom protocol to replace the `http` protocol, you have to register it as a standard scheme.

### `protocol.registerFileProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (String | [ProtocolResponse](structures/protocol-response.md))

Registers a protocol of `scheme` that will send a file as the response. The `handler` will be called with `request` and `callback` where `request` is an incoming request for the `scheme`.

`request` を処理するには、`callback` を、ファイルのパスまたは `path` プロパティを持つオブジェクトのいずれかを使用して、例えば、`callback(filePath)` や `callback({ path: filePath })` で呼び出す必要があります。 The `filePath` must be an absolute path.

By default the `scheme` is treated like `http:`, which is parsed differently from protocols that follow the "generic URI syntax" like `file:`.

### `protocol.registerBufferProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (Buffer | [ProtocolResponse](structures/protocol-response.md))

`Buffer` をレスポンスとして送信する `scheme` のプロトコルを登録します。

The usage is the same with `registerFileProtocol`, except that the `callback` should be called with either a `Buffer` object or an object that has the `data` property.

サンプル:

```javascript
protocol.registerBufferProtocol('atom', (request, callback) => {
  callback({ mimeType: 'text/html', data: Buffer.from('<h5>Response</h5>') })
})
```

### `protocol.registerStringProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (String | [ProtocolResponse](structures/protocol-response.md))

`String` をレスポンスとして送信する `scheme` のプロトコルを登録します。

The usage is the same with `registerFileProtocol`, except that the `callback` should be called with either a `String` or an object that has the `data` property.

### `protocol.registerHttpProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` ProtocolResponse

HTTP リクエストをレスポンスとして送信する `scheme` のプロトコルを登録します。

The usage is the same with `registerFileProtocol`, except that the `callback` should be called with an object that has the `url` property.

### `protocol.registerStreamProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (ReadableStream | [ProtocolResponse](structures/protocol-response.md))

Registers a protocol of `scheme` that will send a stream as a response.

The usage is the same with `registerFileProtocol`, except that the `callback` should be called with either a [`ReadableStream`](https://nodejs.org/api/stream.html#stream_class_stream_readable) object or an object that has the `data` property.

サンプル:

```javascript
const { protocol } = require('electron')
const { PassThrough } = require('stream')

function createStream (text) {
  const rv = new PassThrough() // PassThrough is also a Readable stream
  rv.push(text)
  rv.push(null)
  return rv
}

protocol.registerStreamProtocol('atom', (request, callback) => {
  callback({
    statusCode: 200,
    headers: {
      'content-type': 'text/html'
    },
    data: createStream('<h5>Response</h5>')
  })
})
```

It is possible to pass any object that implements the readable stream API (emits `data`/`end`/`error` events). For example, here's how a file could be returned:

```javascript
protocol.registerStreamProtocol('atom', (request, callback) => {
  callback(fs.createReadStream('index.html'))
})
```

### `protocol.unregisterProtocol(scheme)`

* `scheme` String

`scheme` のカスタムプロトコルを登録解除します。

### `protocol.isProtocolRegistered(scheme)`

* `scheme` String

Returns `Boolean` - Whether `scheme` is already registered.

### `protocol.interceptFileProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (String | [ProtocolResponse](structures/protocol-response.md))

`scheme` プロトコルを傍受し、ファイルをレスポンスとして送信するプロトコルの新しいハンドラとして `handler` を使用します。

### `protocol.interceptStringProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (String | [ProtocolResponse](structures/protocol-response.md))

`scheme` プロトコルを傍受し、`String` をレスポンスとして送信するプロトコルの新しいハンドラとして `handler` を使用します。

### `protocol.interceptBufferProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (Buffer | [ProtocolResponse](structures/protocol-response.md))

`scheme` プロトコルを傍受し、`Buffer` をレスポンスとして送信するプロトコルの新しいハンドラとして `handler` を使用します。

### `protocol.interceptHttpProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` ProtocolResponse

`scheme` プロトコルを傍受し、新しい HTTP リクエストをレスポンスとして送信するプロトコルの新しいハンドラとして `handler` を使用します。

### `protocol.interceptStreamProtocol(scheme, handler)`

* `scheme` String
* `handler` Function
  * `request` ProtocolRequest
  * `callback` Function
    * `response` (ReadableStream | [ProtocolResponse](structures/protocol-response.md))

`protocol.registerStreamProtocol` と同じですが、既存のプロトコルハンドラを置き換える点が異なります。

### `protocol.uninterceptProtocol(scheme)`

* `scheme` String

`scheme` のためにインストールされた傍受するハンドラを削除し、元のハンドラを復元します。

### `protocol.isProtocolIntercepted(scheme)`

* `scheme` String

Returns `Boolean` - Whether `scheme` is already intercepted.