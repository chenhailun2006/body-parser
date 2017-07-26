# body-parser

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build Status][travis-image]][travis-url]
[![Test Coverage][coveralls-image]][coveralls-url]
[![Gratipay][gratipay-image]][gratipay-url]

Node.js 请求体解析中间件.

在你的处理函数之前使用此中间件对即将到来的请求体进行解析，解析后的数据存放在`req.body`属性中。

[Node.js中的HTTP事务解析](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/).

_此中间件不能处理`multipart`类型的请求体_。对于`multipart`类型的请求体，可以考虑如下的中间件：

  * [busboy](https://www.npmjs.org/package/busboy#readme) and
    [connect-busboy](https://www.npmjs.org/package/connect-busboy#readme)
  * [multiparty](https://www.npmjs.org/package/multiparty#readme) and
    [connect-multiparty](https://www.npmjs.org/package/connect-multiparty#readme)
  * [formidable](https://www.npmjs.org/package/formidable#readme)
  * [multer](https://www.npmjs.org/package/multer#readme)

此中间件提供了如下的解析能力:

  * [JSON请求体解析](#bodyparserjsonoptions)
  * [Raw请求体解析](#bodyparserrawoptions)
  * [Text请求体解析](#bodyparsertextoptions)
  * [URL-encoded表格请求体解析](#bodyparserurlencodedoptions)

对于其他类型的请求体解析，可以考虑以下中间件:

- [body](https://www.npmjs.org/package/body#readme)
- [co-body](https://www.npmjs.org/package/co-body#readme)

## 安装

```sh
$ npm install body-parser
```

## API

<!-- eslint-disable no-unused-vars -->

```js
var bodyParser = require('body-parser')
```

`bodyParser`对象暴露了各种创建中间件的工厂函数。当请求头的`Content-Type`与`type`选项相匹配时，所有的中间件都会使用解析后的请求体或空对象`{}`（没有请求体、`Content-Type`不匹配或发生错误时时）操作`req.body`属性。

关于此模块可能返回的各种错误，请见
[errors section](#errors).

### bodyParser.json(options)

此工厂方法返回的中间件只能解析`json`类型的请求体，并且要求请求头的`Content-Type`与`type`选项向匹配。此解析起可以接收任意的Unicode编码的请求体。

此中间件处理结束后会在`request`对象上添加一个新的`body`对象(比如 `req.body`)，该`body`对象包含了解析后的数据。

#### Options

`json`函数需要一个`options`对象作为参数，该对象可以包含以下的属性： 

##### inflate

当将此属性的值设置为`true`时，被压缩的请求体会被解压；当设置为`false`时, 不允许接收被压缩的请求体。默认值为`true`.

##### limit

此属性用来控制请求体的大小。如果此属性的值为一个数值类型，则以`byte`为单位，如果是一个字符串，则该字符串会被传递给[bytes](https://www.npmjs.com/package/bytes)库用于解析. 默认值为`'100kb'`.

##### reviver

`reviver`属性会直接传递给`JSON.parse` 作为`JSON.parse`的第二个参数。你可以在[JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#Example.3A_Using_the_reviver_parameter)中找到更多相关的内容。

##### strict

当设置为`true`时, `JSON.parse`只会接收数组和对象；当为`false`时， `JSON.parse` 任何形式的数据. 默认为`true`.

##### type

`type`属性用来决定使用何种类型的中间件解析请求体。 此属性的属性值可以是一个函数或者是一个字符串。 `type`属性会被直接传递给[type-is](https://www.npmjs.org/package/type-is#readme)
库，此属性可以是一个扩展名（比如`json`）, 一个媒体类型(比如`application/json`), 或者是一个带有通配符的媒体类型（比如`*/*` 或 `*/json`）。
如果是一个函数，`type`属性将以`fn(req)`的形式被调用，如果它能够返回一个正确的值，请求体就会被解析。默认值为`application/json`.

##### verify

`verify`属性将以`verify(req, res, buf, encoding)`的形式被调用,其中`buf` 是一个原始请求体的`Buffer`，`encoding`是请求体的编码格式。 在解析过程中可以通过抛出一个错误来取消解析。

### bodyParser.raw(options)

Returns middleware that parses all bodies as a `Buffer` and only looks at
requests where the `Content-Type` header matches the `type` option. This
parser supports automatic inflation of `gzip` and `deflate` encodings.

A new `body` object containing the parsed data is populated on the `request`
object after the middleware (i.e. `req.body`). This will be a `Buffer` object
of the body.

#### Options

The `raw` function takes an option `options` object that may contain any of
the following keys:

##### inflate

When set to `true`, then deflated (compressed) bodies will be inflated; when
`false`, deflated bodies are rejected. Defaults to `true`.

##### limit

Controls the maximum request body size. If this is a number, then the value
specifies the number of bytes; if it is a string, the value is passed to the
[bytes](https://www.npmjs.com/package/bytes) library for parsing. Defaults
to `'100kb'`.

##### type

The `type` option is used to determine what media type the middleware will
parse. This option can be a function or a string. If a string, `type` option
is passed directly to the [type-is](https://www.npmjs.org/package/type-is#readme)
library and this can be an extension name (like `bin`), a mime type (like
`application/octet-stream`), or a mime type with a wildcard (like `*/*` or
`application/*`). If a function, the `type` option is called as `fn(req)`
and the request is parsed if it returns a truthy value. Defaults to
`application/octet-stream`.

##### verify

The `verify` option, if supplied, is called as `verify(req, res, buf, encoding)`,
where `buf` is a `Buffer` of the raw request body and `encoding` is the
encoding of the request. The parsing can be aborted by throwing an error.

### bodyParser.text(options)

Returns middleware that parses all bodies as a string and only looks at
requests where the `Content-Type` header matches the `type` option. This
parser supports automatic inflation of `gzip` and `deflate` encodings.

A new `body` string containing the parsed data is populated on the `request`
object after the middleware (i.e. `req.body`). This will be a string of the
body.

#### Options

The `text` function takes an option `options` object that may contain any of
the following keys:

##### defaultCharset

Specify the default character set for the text content if the charset is not
specified in the `Content-Type` header of the request. Defaults to `utf-8`.

##### inflate

When set to `true`, then deflated (compressed) bodies will be inflated; when
`false`, deflated bodies are rejected. Defaults to `true`.

##### limit

Controls the maximum request body size. If this is a number, then the value
specifies the number of bytes; if it is a string, the value is passed to the
[bytes](https://www.npmjs.com/package/bytes) library for parsing. Defaults
to `'100kb'`.

##### type

The `type` option is used to determine what media type the middleware will
parse. This option can be a function or a string. If a string, `type` option
is passed directly to the [type-is](https://www.npmjs.org/package/type-is#readme)
library and this can be an extension name (like `txt`), a mime type (like
`text/plain`), or a mime type with a wildcard (like `*/*` or `text/*`).
If a function, the `type` option is called as `fn(req)` and the request is
parsed if it returns a truthy value. Defaults to `text/plain`.

##### verify

The `verify` option, if supplied, is called as `verify(req, res, buf, encoding)`,
where `buf` is a `Buffer` of the raw request body and `encoding` is the
encoding of the request. The parsing can be aborted by throwing an error.

### bodyParser.urlencoded(options)

Returns middleware that only parses `urlencoded` bodies and only looks at
requests where the `Content-Type` header matches the `type` option. This
parser accepts only UTF-8 encoding of the body and supports automatic
inflation of `gzip` and `deflate` encodings.

A new `body` object containing the parsed data is populated on the `request`
object after the middleware (i.e. `req.body`). This object will contain
key-value pairs, where the value can be a string or array (when `extended` is
`false`), or any type (when `extended` is `true`).

#### Options

The `urlencoded` function takes an option `options` object that may contain
any of the following keys:

##### extended

The `extended` option allows to choose between parsing the URL-encoded data
with the `querystring` library (when `false`) or the `qs` library (when
`true`). The "extended" syntax allows for rich objects and arrays to be
encoded into the URL-encoded format, allowing for a JSON-like experience
with URL-encoded. For more information, please
[see the qs library](https://www.npmjs.org/package/qs#readme).

Defaults to `true`, but using the default has been deprecated. Please
research into the difference between `qs` and `querystring` and choose the
appropriate setting.

##### inflate

When set to `true`, then deflated (compressed) bodies will be inflated; when
`false`, deflated bodies are rejected. Defaults to `true`.

##### limit

Controls the maximum request body size. If this is a number, then the value
specifies the number of bytes; if it is a string, the value is passed to the
[bytes](https://www.npmjs.com/package/bytes) library for parsing. Defaults
to `'100kb'`.

##### parameterLimit

The `parameterLimit` option controls the maximum number of parameters that
are allowed in the URL-encoded data. If a request contains more parameters
than this value, a 413 will be returned to the client. Defaults to `1000`.

##### type

The `type` option is used to determine what media type the middleware will
parse. This option can be a function or a string. If a string, `type` option
is passed directly to the [type-is](https://www.npmjs.org/package/type-is#readme)
library and this can be an extension name (like `urlencoded`), a mime type (like
`application/x-www-form-urlencoded`), or a mime type with a wildcard (like
`*/x-www-form-urlencoded`). If a function, the `type` option is called as
`fn(req)` and the request is parsed if it returns a truthy value. Defaults
to `application/x-www-form-urlencoded`.

##### verify

The `verify` option, if supplied, is called as `verify(req, res, buf, encoding)`,
where `buf` is a `Buffer` of the raw request body and `encoding` is the
encoding of the request. The parsing can be aborted by throwing an error.

## Errors

The middlewares provided by this module create errors depending on the error
condition during parsing. The errors will typically have a `status` property
that contains the suggested HTTP response code and a `body` property containing
the read body, if available.

The following are the common errors emitted, though any error can come through
for various reasons.

### content encoding unsupported

This error will occur when the request had a `Content-Encoding` header that
contained an encoding but the "inflation" option was set to `false`. The
`status` property is set to `415`.

### request aborted

This error will occur when the request is aborted by the client before reading
the body has finished. The `received` property will be set to the number of
bytes received before the request was aborted and the `expected` property is
set to the number of expected bytes. The `status` property is set to `400`.

### request entity too large

This error will occur when the request body's size is larger than the "limit"
option. The `limit` property will be set to the byte limit and the `length`
property will be set to the request body's length. The `status` property is
set to `413`.

### request size did not match content length

This error will occur when the request's length did not match the length from
the `Content-Length` header. This typically occurs when the request is malformed,
typically when the `Content-Length` header was calculated based on characters
instead of bytes. The `status` property is set to `400`.

### stream encoding should not be set

This error will occur when something called the `req.setEncoding` method prior
to this middleware. This module operates directly on bytes only and you cannot
call `req.setEncoding` when using this module. The `status` property is set to
`500`.

### unsupported charset "BOGUS"

This error will occur when the request had a charset parameter in the
`Content-Type` header, but the `iconv-lite` module does not support it OR the
parser does not support it. The charset is contained in the message as well
as in the `charset` property. The `status` property is set to `415`.

### unsupported content encoding "bogus"

This error will occur when the request had a `Content-Encoding` header that
contained an unsupported encoding. The encoding is contained in the message
as well as in the `encoding` property. The `status` property is set to `415`.

## Examples

### Express/Connect top-level generic

This example demonstrates adding a generic JSON and URL-encoded parser as a
top-level middleware, which will parse the bodies of all incoming requests.
This is the simplest setup.

```js
var express = require('express')
var bodyParser = require('body-parser')

var app = express()

// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }))

// parse application/json
app.use(bodyParser.json())

app.use(function (req, res) {
  res.setHeader('Content-Type', 'text/plain')
  res.write('you posted:\n')
  res.end(JSON.stringify(req.body, null, 2))
})
```

### Express route-specific

This example demonstrates adding body parsers specifically to the routes that
need them. In general, this is the most recommended way to use body-parser with
Express.

```js
var express = require('express')
var bodyParser = require('body-parser')

var app = express()

// create application/json parser
var jsonParser = bodyParser.json()

// create application/x-www-form-urlencoded parser
var urlencodedParser = bodyParser.urlencoded({ extended: false })

// POST /login gets urlencoded bodies
app.post('/login', urlencodedParser, function (req, res) {
  if (!req.body) return res.sendStatus(400)
  res.send('welcome, ' + req.body.username)
})

// POST /api/users gets JSON bodies
app.post('/api/users', jsonParser, function (req, res) {
  if (!req.body) return res.sendStatus(400)
  // create user in req.body
})
```

### Change accepted type for parsers

All the parsers accept a `type` option which allows you to change the
`Content-Type` that the middleware will parse.

```js
var express = require('express')
var bodyParser = require('body-parser')

var app = express()

// parse various different custom JSON types as JSON
app.use(bodyParser.json({ type: 'application/*+json' }))

// parse some custom thing into a Buffer
app.use(bodyParser.raw({ type: 'application/vnd.custom-type' }))

// parse an HTML body into a string
app.use(bodyParser.text({ type: 'text/html' }))
```

## License

[MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/body-parser.svg
[npm-url]: https://npmjs.org/package/body-parser
[travis-image]: https://img.shields.io/travis/expressjs/body-parser/master.svg
[travis-url]: https://travis-ci.org/expressjs/body-parser
[coveralls-image]: https://img.shields.io/coveralls/expressjs/body-parser/master.svg
[coveralls-url]: https://coveralls.io/r/expressjs/body-parser?branch=master
[downloads-image]: https://img.shields.io/npm/dm/body-parser.svg
[downloads-url]: https://npmjs.org/package/body-parser
[gratipay-image]: https://img.shields.io/gratipay/dougwilson.svg
[gratipay-url]: https://www.gratipay.com/dougwilson/
