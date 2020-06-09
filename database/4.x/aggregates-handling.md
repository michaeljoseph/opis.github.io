---
layout: project
library: database
version: 4.x
title: Working with aggregates
description: Learn how you can use the 'GROUP BY' and 'HAVING' clauses
keywords: group by, having, aggregates
---

## Grouping

Grouping the result-set by a column is done by using the `groupBy` method.

{% capture php%}
```php
$result = $db->from('customers')
             ->leftJoin('orders', function($join){
               $join->on('customers.id', 'orders.cid');
             })
             ->groupBy('customers.name')
             ->select(function($include){
               $include->count('orders.id', 'total_orders')
                       ->column('customers.name', 'name');
             })
             ->all();
```
{% endcapture %}
{% capture sql %}
```sql
SELECT COUNT(`orders`.`id`) AS `total_orders`,
`customers`.`name` AS `name` FROM `customers`
LEFT JOIN `orders` ON `customers`.`id` = `orders`.`cid`
GROUP BY `customers`.`name`
```
{% endcapture %}
{% include tabs.html 1="PHP" 2="SQL" _1=php _2=sql %}

## Having clause

You can filter the result of aggregate functions by using the `having` method 
in conjunction with an aggregate method and a condition method. 
The available aggregating methods are `count`, `sum`, `min`, `max` and `avg`. 
The condition methods are `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `notIn`, `between` and `notBetween`.
Each condition method was described in the [filters](filters) section.

{% capture php %}
```php
$result = $db->from('customers')
             ->leftJoin('orders', function($join){
               $join->on('customers.id', 'orders.cid');
             })
             ->groupBy('customers.name')
             ->having('orders.id', function($column){
               $column->count()->gt(10);
             })
             ->select(function($include){
               $include->count('orders.id', 'total_orders')
                       ->column('customers.name', 'name');
             })
             ->all();
```
{% endcapture %}
{% capture sql %}
```sql
SELECT COUNT(`orders`.`id`) AS `total_orders`,
`customers`.`name` AS `name` FROM `customers`
LEFT JOIN `orders` ON `customers`.`id` = `orders`.`cid`
GROUP BY `customers`.`name`
HAVING COUNT(`orders`.`id`) > 10
```
{% endcapture %}
{% include tabs.html 1="PHP" 2="SQL" _1=php _2=sql %}

You can use multiple having filters by using the `andHaving` and `orHaving` methods.

{% capture php %}
```php
$result = $db->from('customers')
             ->leftJoin('orders', function($join){
               $join->on('customers.id', 'orders.cid');
             })
             ->groupBy('customers.name')
             ->having('orders.id', function($column){
               $column->count()->gt(10);
             })
             ->andHaving('orders.value', function($column){
                $column->sum()->gte(1000);
             })
             ->select(function($include){
               $include->count('orders.id', 'total_orders')
                       ->column('customers.name', 'name');
             })
             ->all();
```
{% endcapture %}
{% capture sql %}
```sql
SELECT COUNT(`orders`.`id`) AS `total_orders`,
`customers`.`name` AS `name` FROM `customers`
LEFT JOIN `orders` ON `customers`.`id` = `orders`.`cid`
GROUP BY `customers`.`name`
HAVING COUNT(`orders`.`id`) > 10 AND SUM(`orders`.`value`) >= 1000
```
{% endcapture %}
{% include tabs.html 1="PHP" 2="SQL" _1=php _2=sql %}

{% capture php %}
```php
$result = $db->from('customers')
             ->leftJoin('orders', function($join){
               $join->on('customers.id', 'orders.cid');
             })
             ->groupBy('customers.name')
             ->having('orders.id', function($column){
               $column->count()->gt(10);
             })
             ->orHaving('orders.value', function($column){
                $column->sum()->gte(1000);
             })
             ->select(function($include){
               $include->count('orders.id', 'total_orders')
                       ->column('customers.name', 'name');
             })
             ->all();
```
{% endcapture %}
{% capture sql %}
```sql
SELECT COUNT(`orders`.`id`) AS `total_orders`,
`customers`.`name` AS `name` FROM `customers`
LEFT JOIN `orders` ON `customers`.`id` = `orders`.`cid`
GROUP BY `customers`.`name`
HAVING COUNT(`orders`.`id`) > 10 OR SUM(`orders`.`value`) >= 1000
```
{% endcapture %}
{% include tabs.html 1="PHP" 2="SQL" _1=php _2=sql %}

Grouping conditions is done by passing a closure as the first argument to the 
`having`, `andHaving` or `orHaving` methods.

{% capture php %}
```php
$result = $db->from('customers')
             ->leftJoin('orders', function($join){
               $join->on('customers.id', 'orders.cid');
             })
             ->groupBy('customers.name')
             ->having('orders.id', function($column){
               $column->count()->gt(10);
             })
             ->andHaving(function($group){
               $group->having('orders.value', function($column){
                  $column->sum()->gte(1000);
               })
               ->orHaving('orders.value', function($column){
                  $column->min()->gte(500);
               });
             })
             ->select(function($include){
               $include->count('orders.id', 'total_orders')
                       ->column('customers.name', 'name');
             })
             ->all();
```
{% endcapture %}
{% capture sql %}
```sql
SELECT COUNT(`orders`.`id`) AS `total_orders`,
`customers`.`name` AS `name` FROM `customers`
LEFT JOIN `orders` ON `customers`.`id` = `orders`.`cid`
GROUP BY `customers`.`name`
HAVING COUNT(`orders`.`id`) > 10 AND
    (SUM(`orders`.`value`) >= 1000 OR MIN(`orders`.`value`) >= 500)
```
{% endcapture %}
{% include tabs.html 1="PHP" 2="SQL" _1=php _2=sql %}