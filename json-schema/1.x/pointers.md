---
layout: project
library: json-schema
version: 1.x
title: Json Pointers
description: using absolute and relative json pointers in opis json schema
keywords: opis, json, schema, validation, pointer, relative, absolute, $ref
---

A JSON pointer is a way of traversing a json document by specifying the path
to the desired value.
**Opis JSON Schema** supports both [absolute](#absolute-pointers) and [relative](#relative-pointers) JSON schema pointers.

## Absolute pointers

Absolute json pointers are used to search for values starting at the root of the document.
That's why absolute json pointers always start with `/` (slash). The property names (keys) used to descend
into children are also separated by `/` (slash). 

If a property name contains
`~`, then inside the pointer it must be escaped using `~0`, or if contains `/` then
it must be escaped using `~1`. 

Here are some examples about how absolute json pointers work:

```json
{
  "name": "some product",
  "price": 10.5,
  "features": [
    "easy to use",
    {
      "name": "environment friendly",
      "url": "http://example.com"
    }
  ],
  "info": {
    "onStock": true
  },
  "a/b": "a"
}
```

| Pointer | Value |
|---------|-------|
| `/` | _the document itself (root)_ |
| `/name` | `"some product"`{:.language-json} |
| `/price` | `10.5`{:.language-json} |
| `/features/0` | `"easy to use"`{:.language-json} |
| `/features/1/url` | `"http://example.com"`{:.language-json} |
| `/info` | `{"onStock": true}`{:.language-json} |
| `/info/onStock` | `true`{:.language-json} |
| `/a~1b` | `"a"`{:.language-json} |
| `/inexistent/path` | *error*{:.text-danger.text-normal} |
{:.table}

You can find more details about the structure of absolute JSON pointers [here](https://tools.ietf.org/html/rfc6901){:target="_blank"}.

### Using absolute pointers

You can use absolute json pointers to fetch data/subschemas defined in the document.
You must specify the pointer in the fragment component of the URI, this means
that you should place your pointer after `#`.

Consider the following json schema document

```json
{
  "$id": "http://example.com/schema.json#",
  
  "definitions": {
    "name": {
      "type": "string",
      "minLength": 1
    },
    "personal": {
      "email": {
        "type": "string",
        "format": "email"
      },
      "birthday": {
        "type": "string",
        "format": "date"
      }
    }
  }
}
```

Here is a table containing the absolute URI and pointer to fetch
the desired schemas.

| Absolute URI and pointer | Fetched schema |
|---------|-------|
| `http://example.com/schema.json#` | _the document itself_ |
| `http://example.com/schema.json#/` | _the document itself_ |
| `http://example.com/schema.json#/definitions/name` | `{"type": "string", "minLength": 1}`{:.language-json} |
| `http://example.com/schema.json#/definitions/personal/email` | `{"type": "string", "format": "email"}`{:.language-json} |
| `http://example.com/schema.json#/definitions/personal/birthday` | `{"type": "string", "format": "date"}`{:.language-json} |
| `http://example.com/schema.json#/inexistent/path` | *error*{:.text-danger.text-normal} |
{:.table}

Now lets see a complex example that uses the [`$ref` keyword](ref-keyword.html#ref).

```json
{
  "$id": "http://example.com/path/to/user.json",
  
  "type": "object",
  "properties": {
    "email": {
      "$ref": "#/definitions/personal/email"
    },
    "birthday": {
      "$ref": "#/definitions/personal/birthday"
    },
    "settings": {
      "$ref": "user-settings.json#/definitions/settings"
    },
    "info": {
      "$ref": "../info.json#"
    },
    "root": {
      "$ref": "/other/path/to/schema.json#/definitions/root"
    },
    "external": {
      "$ref": "http://external.example.com/some-schema.json#/definitions/name"
    }
  },
  
  "definitions": {
    "personal": {
      "email": {
        "type": "string",
        "format": "email"
      },
      "birthday": {
        "type": "string",
        "format": "date"
      }
    }
  }
}
```

First thing we have to notice is that this document contains the [`$id` keyword](string.html#id-keyword).
This means that all values of the `$ref` keyword will be resolved using the 
`$id` as base.

The following table contains the resolved (absolute) URIs for `$ref`s inside
the `properties` keyword.

| Property name | Resolved $ref |
|---------|-------|
| email | `http://example.com/path/to/user.json#/definitions/personal/email` |
| birthday | `http://example.com/path/to/user.json#/definitions/personal/birthday` |
| settings | `http://example.com/path/to/user-settings.json#/definitions/settings` |
| info | `http://example.com/path/info.json#` |
| root | `http://example.com/other/path/to/schema.json#/definitions/root` |
| external | `http://external.example.com/some-schema.json#/definitions/name` |
{:.table.table-striped}

These are the steps in order to perform validation for __email__ property

1. Get the value of `$ref` => `#/definitions/personal/email`
2. Get the absolute URI, using `$id` as base => `http://example.com/path/to/user.json#/definitions/personal/email`
3. Load the schema document having the `$id` equal to `http://example.com/path/to/user.json` (in this case it is the same document)
4. Apply the json pointer `/definitions/personal/email` to get the subschema => `{"type": "string", "format": "email"}`
5. Use the subschema for validation (in our case we validate the value of _email_ property)

## Relative pointers

Relative json pointers are used to search for values starting at the _current_
location. We can go upwards by specifying the numbers of levels to ascend,
and then we can go downwards by using a pointer composed by multiple property names (keys) separated
by `/` (slash). The level and the pointer are also separated by `/` (slash).
Additionally, we can append `#` which will return the property name (key)
where our value was found (this is very useful for arrays). 
The level is always
required and must be a non-negative integer. Level `0` points to the _current_ location,
level `1` points to the parent of _current_ location, and so on. 
You cannot use a level that will go past document root.

If a property name contains
`~`, then inside the pointer it must be escaped using `~0`, or if contains `/` then
it must be escaped using `~1`. 

Here are some examples about how relative json pointers work:

```json
{
  "name": "some product",
  "price": 10.5,
  "features": [
    "easy to use",
    {
      "name": "environment friendly",
      "url": "http://example.com"
    }
  ],
  "info": {
    "onStock": true
  },
  "a/b": "a"
}
```

Considering that our current location is `10.5` (absolute json pointer `/price`)
we have the following table

| Pointer | Value |
|---------|-------|
| `0` | `10.5`{:.language-json} |
| `0#` | `"price"`{:.language-json} |
| `1` | _the document itself (root)_ |
| `1#` | *error*{:.text-danger.text-normal} |
| `1/name` | `"some product"`{:.language-json} |
| `1/info` | `{"onStock": true}`{:.language-json} |
| `1/info/onStock` | `true`{:.language-json} |
| `1/a~1b` | `"a"`{:.language-json} |
| `1/inexstent/path` | *error*{:.text-danger.text-normal} |
| `2` | *error*{:.text-danger.text-normal} |
{:.table}

Considering that our current location is `"http://example.com"` (absolute json pointer `/features/1/url`)
we have the following table

| Pointer | Value |
|---------|-------|
| `0` | `"http://example.com"`{:.language-json} |
| `0#` | `"url"`{:.language-json} |
| `1#` | `1`{:.language-json} (array index) |
| `1/name` | `"environment friendly"`{:.language-json} |
| `2#` | `"features"`{:.language-json} |
| `2/0` | `"easy to use"`{:.language-json} |
| `2/0#` | `0`{:.language-json} (array index) |
| `3` | _the document itself (root)_ |
| `3/price` | `10.5`{:.language-json} |
| `3/info/onStock` | `true`{:.language-json} |
| `3/inexstent/path` | *error*{:.text-danger.text-normal} |
| `3#` | *error*{:.text-danger.text-normal} |
| `4` | *error*{:.text-danger.text-normal} |
{:.table}

You can find more details about the structure of relative json pointers [here](https://tools.ietf.org/html/draft-luff-relative-json-pointer-00){:target="_blank"}.

### Using relative pointers

You can use absolute JSON pointers to fetch data/subschemas defined in the document.
You cannot use relative pointers in the way you use [absolute pointers](#absolute-pointers),
so you cannot use them in an URI.

Here is an example using the [`$ref` keyword](ref-keyword.html).

{% capture schema %}
```json
{
  "type": "object",
  "properties": {
    "first_email": {
      "type": "string",
      "format": "email"
    },
    "second_email": {
      "$ref": "1/first_email"
    }
  }
}
```
{% endcapture %}
{% capture examples %}
|Input|Status|
|-----|------|
| `{"first_email": "john@example.com", "second_email: "opis@example.com"}`{:.language-json} | *valid*{:.text-success.text-normal} |
| `{"second_email: "opis@example.com"}`{:.language-json} | *valid*{:.text-success.text-normal} |
| `{"first_email": "john@example.com", "second_email: "invalid-email"}`{:.language-json} | *invalid*{:.text-danger.text-normal} |
| `{"second_email: "invalid-email"}`{:.language-json} | *invalid*{:.text-danger.text-normal} |
{:.table}
{% endcapture %}
{% include tabs.html 1="Schema" 2="Examples" _1=schema _2=examples %}

These are the steps taken in order to perform validation of `second_email` property:

1. The _current_ location is `{"$ref": "1/first_email"}`{:.language-json} (the value of `second_email` property)
2. Ascending `1` level we end up at the value of `properties` property
3. Descending into `first_email` and we get `{"type": "string", "format": "email"}`{:.language-json}
4. Use the subschema to validate the value of `second_email` property