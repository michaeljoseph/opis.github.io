---
layout: project
library: json-schema
version: 1.x
title: Creating Opis JSON Schema filters
description: the opis json schema validation using custom filters
keywords: opis, json, schema, validation, filter
---

A filter is an object of a class implementing `Opis\JsonSchema\IFilter` interface.
The `validate` method receives two arguments, and must return a `boolean` (`true` if valid, `false` otherwise)
- `mixed $value`: the current value to validate
- `array $args`: an associative array of variables

```php
<?php

use Opis\JsonSchema\IFilter;

class ModuloFilter implements IFilter
{
    public function validate($value, array $args): bool {
        $divisor = $args['divisor'] ?? 1;
        $reminder = $args['reminder'] ?? 0;
        return $value % $divisor == $reminder;
    }
}
```

## Using filters

Before using the [`$filters` keyword](filters.html) in your schemas, make sure
to register them in a `Opis\JsonSchema\IFilterContainer` object, and pass
that object to [`Opis\JsonSchema\IValidator::setFilters()`](php-validator.html#setfilters).

When your register a filter you must specify:
- json data type (boolean, number, integer, string, null, array, object)
- name: the name you will use in your schemas
- the filter object that implements `Opis\JsonSchema\IFilter`

```php
<?php

use Opis\JsonSchema\{
    Validator,
    FilterContainer
};

// Create a new FilterContainer
$filters = new FilterContainer();

// Register our modulo filter
$filters->add("number", "modulo", new ModuloFilter());

// Create a IValidator
$validator = new Validator();

// Set filters to be used by validator
$validator->setFilters($filters);

// Validation ...

```

Here is an example of schema that uses our modulo filter

```json
{
  "type": "number",
  "$filters": {
    "$func": "modulo",
    "$vars": {
      "divider": 4,
      "reminder": 3
    }
  }
}
```

This schema validates `7` _(7 % 4 == 3)_ but does not validate `17` _(17 % 4 == 1 != 3)_.

### Creating a "password match" filter

```php
<?php

use Opis\JsonSchema\IFilter;

class MatchFilter implements IFilter
{
    public function validate($value, array $args): bool {
        if (!$args || !array_key_exists('value', $args)) {
            return false;
        }
        return $value === $args['value'];
    }
}
```

You can use the above filter (considering that is named `match`) in a schema like this

```json
{
  "type": "object",
  "properties": {
    "password": {
      "type": "string",
      "minLength": 8
    },
    "repeatPassword": {
      "type": "string",
      "$filters": {
        "$func": "match",
        "$vars": {
          "value": {"$ref": "1/password"}
        }
      }
    }
  },
  "required": ["password", "repeatPassword"]
}
```