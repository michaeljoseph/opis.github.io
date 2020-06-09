---
layout: project
library: orm
version: 1.x
title: Entities 
---

An entity is an object-oriented representation of a table's row, in a form
of a class which extends the `Opis\ORM\Entity` base class.

```php
use Opis\ORM\Entity;

class User extends Entity
{
    // User entity
}
```

The base class provides a single method, named `orm`, which returns a
[data mapper][0] object, that can be used to manipulate the row's records.

```php
use Opis\ORM\Entity;

class User extends Entity
{
    public function name(): string
    {
        return $this->orm()->getColumn('name');
    }
}
```

Since an entity is not meant to be directly instantiable,
the constructor of the base entity class is marked as being `final`, 
in order to prevent of being accidentally overwritten.

```php
use Opis\ORM\Entity;

class User extends Entity
{
    public function __construct()
    {
        // This will throw an exception
    }
}
```

[0]: /1.x/data-mapper.html "Data mappers"
