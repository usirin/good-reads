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
