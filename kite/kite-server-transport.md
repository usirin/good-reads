## Kite Server Transport

A class that is used by a `kite` as `transformClass` option.

### Interface

```typescript
declare type TransportEvent = {
  OPEN: 'open',
  CLOSE: 'close',
  MESSAGE: 'message',
  ERROR: 'error',
  INFO: 'info'
}

declare type TransportEventHandler = {
  [TransportEvent.OPEN]: () => void,
  [TransportEvent.CLOSE]: (event: Object) => void,
  [TransportEvent.MESSAGE]: ({ data: Object }) => void,
  [TransportEvent.ERROR]: (error: Object) => void,
  [TransportEvent.INFO]: (info: Object) => void
}

declare interface KiteServerTransport {
  addEventListener(eventName: TransportEvent, handler: Function);
  
  close();
  send(message: string);
}
```
