# Kite

## Interface

```typescript
declare enum KiteReadyState {
  NOTREADY = 0,
  READY = 1,
  CLOSED = 3,
  CONNECTING = 5,
}

declare class KiteError implements Error {
  name: 'KiteError';
  message: string;
  
  static codeIs(code: string): (err: Error) => (bool);
  static codeIsnt(code: string): (err: Error) => (bool);
}

declare type KiteTimerHandle = 'heartbeatHandle' | 'expiryHandle' | 'backoffHandle'

declare enum KiteLoggingLevel {
  CRITICAL = 0,
  ERROR,
  WARNING,
  NOTICE,
  INFO,
  DEBUG,
}

interface KiteApiDescription {
  [propName: string]: (..., callback) => void;
}

declare interface KiteOptions {
  name: string;
  url: string;
  
  username: string;
  environment: string;
  version: string;
  region: string;
  hostname: string;
  
  autoConnect?: bool;
  autoReconnect?: bool;
  prefix?: bool;
  logLevel: KiteLoggingLevel;
  api: KiteApiDescription;
  auth: { type: string, key: KiteAuthToken };
  
  transportClass(url: string, _reserved?: any, options?: KiteTransportOptions): KiteTransport;
  transportOptions: KiteTransportOptions;
}

declare interface Kite {
  id: string;
  options: KiteOptions;
  proto: DnodeProtocol;
  readyState: KiteReadyState;
  ws: KiteTransport;
  
  getToken(): KiteAuthToken;
  setToken(token: KiteAuthToken): void;
  
  connect(): void;
  disconnect(reconnect?: bool): void;
  
  onOpen(): void;
  onClose(): void;
  onMessage(message: KiteTransportMessage): void;
  onError(err: KiteTransportError): void;
  
  getKiteInfo(params?: KiteInfoParams): KiteInfo;
  wrapMessage(method: string, params: Array<any>, callback: Function): DnodeScrubberMessage;
  tell(method: string, params: Array<any>, callback: Function): void;
  
  expireTokenOnExpiry(): void;
  expireToken(callback: Function): void;
  
  ready(callback: Function): void;
  
  ping(callback: Function): void;
  
  static disconnect(kites: Array<Kite>): void;
  static random(kites: Array<Kite>): Kite;
}
```

## Usage

```coffeescript
Kite = require 'kite.js'

kite = new Kite 'ws://math-service.com'
# which is equal to
kite = new Kite
  url: 'ws://math-service.com'
  autoConnect: yes
  autoReconnect: yes
  transportClass: require 'ws' # WebSocket constructor
  logLevel: KiteLoggingLevel.INFO
  backoff: 
    initialDelayMs: 700
    multiplyFactor: 1.4
    maxDelayMs: 1000 * 15 # 15 seconds
    maxReconnectAttempts: 50
```

`new Kite(options: KiteOptions)` creates a `kite` instance, which expects to connect to a
[KiteServer](kite-server.md) instance runs on the url `ws://math-service.com`.

In more detail:
- It will enable the logging with given logging level (`KiteLoggingLevel.INFO`).
- It will set `readyState` to `KiteReadyState.NOTREADY`.
- It's gonna initialize backoff handlers since `autoReconnect` is passed as `yes`. This will cause backoff timers to be set for automatically trying to `reconnect` with using a backoff strategy once a connection is closed with the `transport` instance and it fires `close` event.
- It's start to create a connection immediately since `autoConnect` passed as `yes`. This will cause series of events:
  - It will set `readyState` is gonna be `KiteReadyState.CONNECTING`
    ```coffeescript
    # kite.coffee
    class Kite extends EventEmitter
      ...

      connect: ->
        ...
        @readyState = KiteReadyState.CONNECTING
        { transportClass: Transport, url } = @options
        @ws = new Transport url
    ```
  - It will create a connection with its `transportClass` which can be accesses via `kite.ws`:
    ```coffeescript
    # kite.coffee
    connect: ->
      ...
      { transportClass: Transport, url } = @options
      @ws = new Transport url
    ```
  - It will bind handlers to `transport` instance events:
  
    ```coffeescript
    # kite.coffee
    connect: ->
      ...
      @ws.addEventListener 'open', @bound 'onOpen' # Runs when transport is ready for calls.
      @ws.addEventListener 'close', @bound 'onClose' # Runs when connection with transport is closed.
      @ws.addEventListener 'message', @bound 'onMessage' # Runs when a message is received from server.
      @ws.addEventListener 'error', @bound 'onError' # Runs when an error happens in the transport and/or connection with it.
      @ws.addEventListener 'info', @bound 'onInfo' # Runs when an info request is sent.
    ```
- Once the connection with `transport` instance is established and `open` event is fired from it, `kite.onOpen` method will be executed.
- `backoff` timers will be resetted since a connection is established and no need to try to `reconnect`.

Let's recap:

```coffeescript
Kite = require 'kite.js'

# create kite on math-service.com and immediately start connecting using a websocket.
kite = new Kite 'ws://math-service.com'
# => [kite] INFO Trying to connect to ws://math-service.com'
# => [kite] NOTICE Connected to Kite: ws://math-service.com'
```

Your `kite` instance is ready to talk with the server:
```coffeescript
kite.ping, (response) -> console.log "response from server: #{response}"
# => response from server: pong
```
