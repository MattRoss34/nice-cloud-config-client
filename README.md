# Spring Cloud Config Client for NodeJS

[![Build Status](https://travis-ci.com/mattross34/nice-cloud-config-client.svg?branch=master)](https://travis-ci.org/mattross34/nice-cloud-config-client)

Requires: NodeJS 8+

Feature requests are welcome.

## Install

    npm i nice-cloud-config-client

## Usage

```js
const client = require("nice-cloud-config-client");
client.load({
    application: "invoices"
}).then((config) => {
    // Look for a key
    const value1 = config.get("this.is.a.key");

    // Using a prefix, this is equivalent to .get("this.is.another.key");
    const value2 = config.get("this.is", "another.key");
});

```

### `load` function

[Load implementation](./index.js)

#### Parameters

* options - `Object`, mandatory:
  * endpoint `string`, optional, default=`http://localhost:8888`: Config server URL.
  * rejectUnauthorized - `boolean`, optional, default = `true`: if `false` accepts self-signed certificates
  * application - `string`, **deprecated, use name**: Load configuration for this application.
  * name - `string`, mandatory: Load the configuration with this name.
  * profiles `string|string[]`, optional, default=`"default"`: Load profiles.
  * label - `string`, optional: Load environment.
  * auth - `Object`, optional: Basic Authentication for access config server (e.g.: `{ user: "username", pass: "password"}`).
    _endpoint_ accepts also basic auth (e.g. `http://user:pass@localhost:8888`).
    * user - `string`, mandatory
    * pass - `string`, mandatory
  * agent - `http.Agent|https.Agent`, optional: Agent for the request. (e.g. use a proxy) (Since version 1.2.0)
  * context - `object`, optional: Context for substitution (see context section below) (Since version 1.4.0)
* callback - `function(error: Error, config: Config)`, optional: node style callback. If missing, the method will return a promise.

```js
client.load(options)
.then((config) => { ... })
.catch((error) => { ... })

// or

client.load(options, (error, config) => { ... })

// or

async function foo () {
    const config = await client.load(options)
    //...
}

```

### Context references (Since version 1.4.0)

Strings could contain context references, if a contexts is provided those references will be replaced by the actual values in the `Config` object reference.

YAML example:

```yaml
key01: Hello ${NAME:World}!!!
```

This is not related to spEl and those references are not an expression language. 

Reference structure: `${CONTEXT_KEY:DEFAULT_VALUE?}`:

* `CONTEXT_KEY`: Context key, if the value for the key is missing it will be use the default value or return the original string
* `DEFAULT_VALUE`: Optional default value

Environment variables as context:

```js
Client.load({
  endpoint: 'http://server:8888',
  name: 'application',
  context: process.env })
.then(config => {
  // config loaded
}).catch(console.error)
```

### `Config` object

[Config class implementation](./lib/config.js)

#### Properties

* `raw`: Spring raw response data.
* `properties`: computed properties as per Spring specification:
  > Property keys in more specifically named files override those in application.properties or application.yml.

#### Methods

* `get(...parts)`: Retrieve a value at a given path or undefined. Multiple parameters can be used to calculate the key.
  * parts - `string`, variable, mandatory:
* `forEach(callback, includeOverridden)`: Iterates over every key/value in the config.
  * callback - `function(key: string, value: string)`, mandatory: iteration callback.
  * includeOverridden - `boolean`, optional, default=`false`: if true, include overridden keys.
* `toString(spaces): string`: Returns a string representation of `raw` property.
  * spaces - `number`, optional: spaces to use in format.
* `toObject(): object`: Returns the whole configuration as an object. (Since version 1.3.0)

```js
config.get("this.is.a.key");
config.get("this.is", "a.key");
config.get("this", "is", "a", "key");

config.forEach((key, value) => console.log(key + ":" + value));
```

## Behind a proxy

Example (adapted from https-proxy-agent site):

```js
const HttpsProxyAgent = require('https-proxy-agent')
const client = require('nice-cloud-config-client')

const proxy = process.env.http_proxy || 'http://168.63.76.32:3128'
console.log('using proxy server %j', proxy)
const agent = new HttpsProxyAgent(proxy)

const options = {
  application: 'demo',
  profiles: ['test', 'timeout'],
  agent
}
client.load(options).then((cfg) => {
  console.log(cfg.get('test.users', 'multi.uid'))
  console.log(cfg.toString(2))
}).catch((error) => console.error(error))
```

## References

* [Spring Cloud Config](http://cloud.spring.io/spring-cloud-config/)
