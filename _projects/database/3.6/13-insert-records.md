---
layout: project
permalink: database/3.6/insert-records
redirect_from: /database/insert.html
title: Insert records
description: Insert new records into a table
---
# Insert records

Adding new records into a table is done using the `insert` method. 
The method accepts a single argument that represents a `key => value` mapped array
where the `key` is the name of the column and `value` is the actual value that 
will be inserted into the column.

```php
$result = $db->insert(array(
                'name' => 'John Doe',
                'email' => 'john.doe@example.com'
            ))
            ->into('users');
```
```sql
INSERT INTO `users` (`name`, `email`) VALUES ("John Doe", "john.doe@example.com")
```

This method returns `true` on success and `false` otherwise.
 