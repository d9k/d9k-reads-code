# Express: body parser

- :speech_balloon: [node.js - How do I consume the JSON POST data in an Express application | SO](https://stackoverflow.com/questions/10005939/how-do-i-consume-the-json-post-data-in-an-express-application)
	- Body parser was split out into its own npm package after v4, requires a separate install `npm install body-parser`

## How body-parser for express works? v.

- `function bodyParser (options) {`
	- `return function bodyParser (req, res, next) {`
		- `_json(req, res, function (err) {`

```js
object.defineProperty(exports, 'json', {
  configurable: true,
  enumerable: true,
  get: createParserGetter('json')
})
```

- `function createParserGetter (name) {`
	- …`loadParser(name)`
		- `case 'json':`
			- `parser = require('./lib/types/json')`
				- `return JSON.parse(body, reviver)`

## Checking `Content-Type`

- :beginner: [README](https://github.com/expressjs/body-parser?tab=readme-ov-file#bodyparserjsonoptions): _req.body property with the parsed body when the Content-Type request header matches the type option._

- `var shouldParse = typeof type !== 'function' ? typeChecker(type)`