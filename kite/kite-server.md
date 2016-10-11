# Kite Server

Creates a kite, and creates a server using its `serverClass` to accepts connections to this kite.

## Usage

Let's assume you run a websocket server on host `ws://kite-hub.mysite.com`

```coffeescript
# server.coffee on host kite-hub.mysite.com
KiteServer = require 'kite.js/server'

server = new KiteServer
  name: 'math'
  auth: no
  prefix: '/math'
  api:
    square: (number, callback) -> callback null, number * number
    sum: (numbers..., calback) ->
      callback null, (numbers.reduce((sum, number) -> sum + number), 0)
      
server.listen 2342
```

And you have an application which will make use of this `kite`:

```coffeescript
# app.coffee on host mysite.com which will eventually be served as http://mysite.com/bundle.js
Kite = require 'kite.js/promises'

# it's gonna immediately connect to the kite instance.
kite = new Kite 'ws://kite-hub.mysite.com/math'

kite
  .tell 'square', [4]
  .then (result) -> console.log "result: #{result}"
# => result: 16
```
