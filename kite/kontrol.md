# Kontrol

`Kontrol` is a registry to manage a cluster of `Kite`s.

## Interface

```typescript
type KontrolActions = {
  REGISTER: 'register',
  DEREGISTER: 'deregister',
}

interface KontrolOptions {
  name: string;
  prefix: string;
  url: string;
  username: string;
  environment: string;
  version: string;
  region: string;
  logLevel: KiteLoggingLevel;
  
  transportOptions: KiteTransportOptions;
  transportClass(KiteTransportOptions): KiteTransform;
}

interface KontrolCreateKiteOptions {
  kite: KiteDescriptor;
  token: string;
  url: string;
  transportOptions?: KiteTransportOptions;
  autoConnect?: bool;
  autoReconnect?: bool;
}

interface KontrolQuery {
  username: string;
  environment: string;
  name: string;
  version: string;
  region: string;
  hostname: string;
  id: string;
}

interface KontrolKiteDescriptor {
  kite: { name: string, version: string};
  token: string;
  url: string;
}

interface KontrolQueryParams {
  query: KontrolQuery;
  who?: Object<string, any>;
}

interface Kontrol {
  static version: string;
  static Kite(KiteOptions): Kite;
  static actions: KontrolActions;
  
  options: KontrolOptions;
  kite: Kite;
  
  authenticate(options: KontrolOptions): void;
  renewToken(kite: Kite, query: KontrolKiteDescriptor): void;
  
  createKite(options: KontrolCreateKiteOptions): Kite;
  createKites(descriptors: Array<KontrolKiteDescriptor>): Array<Kite>;
  
  fetchKite(params: KontrolQueryParams, callback: (Error, Kite) => void): void;
  fetchKites(params: KontrolQueryParams, callback: (Error, Array<Kite>) => void): void;
  
  connect(): void;
  disconnect(): void;
  register(url: string, callback: Function): void;
}
```

## Usage

`Kontrol` creates an instance for connecting to a remote `Kontrol` instance running on given `url`.

```coffeescript
Kontrol = require 'kite.js/kontrol'

kontrol = new Kontrol
  url: 'ws://my-kontrol-registry.com'
  transportClass: require 'ws'
  username: 'john-doe'
  auth:
    type: 'kiteKey'
    token: '...'
```

This will create an instance to connect to the remote `kontrol`'s `kite` running on `ws://my-kontrol-registry.com`.

We can use this `kontrol` instance to fetch kite's registered to it.

```coffeescript
params =
  query: { name: 'math' }
  
kontrol.fetchKite(params).then (mathKite) ->
  mathKite
    .connect()
    .tell 'square', [4]
    .then (result) -> console.log "Math kite response: #{result}"
# => Math kite response: 16
```
