---
title: JavaScript 自学笔记
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-04-01 16:12:54
tags: 
	- javascript
	- code
	- note
categories: javascript
description: JavaScript自学笔记整理
cover: https://raw.githubusercontent.com/8128/PicGo/master/20200401161455.png
top_img:
keywords:
---

# joi

**joi** lets you describe your data using a simple, intuitive, and readable language. Like the rest of the [hapi ecosystem](https://hapi.dev/) it fits in, **joi** allows you to describe your data for both input and output validation, as part of a hapi HTTP server or standalone.

总的来说就是个数据验证的工具，可以给properties设置一些限制和数据格式

```js
const Joi = require('@hapi/joi');

const schema = Joi.object({
    username: Joi.string()
        .alphanum()
        .min(3)
        .max(30)
        .required(),

    password: Joi.string()
        .pattern(new RegExp('^[a-zA-Z0-9]{3,30}$')),

    repeat_password: Joi.ref('password'),

    access_token: [
        Joi.string(),
        Joi.number()
    ],

    birth_year: Joi.number()
        .integer()
        .min(1900)
        .max(2013),

    email: Joi.string()
        .email({ minDomainSegments: 2, tlds: { allow: ['com', 'net'] } })
})
    .with('username', 'birth_year')
    .xor('password', 'access_token')
    .with('password', 'repeat_password');


schema.validate({ username: 'abc', birth_year: 1994 });
// -> { value: { username: 'abc', birth_year: 1994 } }

schema.validate({});
// -> { value: {}, error: '"username" is required' }

// Also -

try {
    const value = await schema.validateAsync({ username: 'abc', birth_year: 1994 });
}
catch (err) { }
```

The above schema defines the following constraints:

- `username`

  - a required string
  - must contain only alphanumeric characters
  - at least 3 characters long but no more than 30
  - must be accompanied by `birth_year`

- `password`

  - an optional string
  - must satisfy the custom regex pattern
  - cannot appear together with `access_token`
  - must be accompanied by `repeat_password` and equal to it

- `access_token`

  - an optional, unconstrained string or number

- `birth_year`

  - an integer between 1900 and 2013

- `email`

  - a valid email address string
  - must have two domain parts e.g. `example.com`
  - TLD must be `.com` or `.net`



### General Usage

Usage is a two steps process:

First, a schema is constructed using the provided types and constraints:

```js
const schema = Joi.object({
    a: Joi.string()
});
```

Note that **joi** schema objects are immutable which means every additional rule added (e.g. `.min(5)`) will return a new schema object.

Second, the value is validated against the defined schema:

```js
const { error, value } = schema.validate({ a: 'a string' });
```

If the input is valid, then the `error` will be `undefined`. If the input is invalid, `error` is assigned a [`ValidationError`](https://github.com/hapijs/joi/blob/master/API.md#validationerror) object providing more information.

The schema can be a plain JavaScript object where every key is assigned a **joi** type, or it can be a **joi** type directly:

```js
const schema = Joi.string().min(10);
```

If the schema is a **joi** type, the `schema.validate(value)` can be called directly on the type. When passing a non-type schema object, the module converts it internally to an object() type equivalent to:

```js
const schema = Joi.object().keys({
    a: Joi.string()
});
```

When validating a schema:

- Values (or keys in case of objects) are optional by default.

  ```js
  Joi.string().validate(undefined); // validates fine
  ```

  To disallow this behavior, you can either set the schema as `required()`, or set `presence` to `"required"` when passing `options`:

  ```js
  Joi.string().required().validate(undefined);
  // or
  Joi.string().validate(undefined, /* options */ { presence: "required" });
  ```

- Strings are utf-8 encoded by default.

- Rules are defined in an additive fashion and evaluated in order, first the inclusive rules, then the exclusive rules.