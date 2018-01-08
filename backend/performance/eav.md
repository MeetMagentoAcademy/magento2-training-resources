# Entity Attribute Value (EAV) explained

Object like products, categories, customers and thier adresses are stored in database differently than other data.
On example of products we have possibility to define a lot of product attribute sets that use different attribbutes for different types of products.
This basically represent "how to handle object inheritance in object relational mapping" problem.

EAV (Entity Attribute Value) implemented in Magento is improvement of concept of storing every attribute in it's separate table.
For optimized database schema one of the solutions to inheritance problem is to store main entity information in one table, and then all optional attributes would be stored in dedicated tables.
Since in PHP and Magento we deal with only few types of basic scalar data types we have only few tables - one for every simple scalar type we will be using.

More over EAV not only solved problem of how to implement ingeritnace in object relational mapping it also allows for easily translating given attribute values for different store scopes.

## Product tables

Based on product tables we can see main product entry in table `catalog_product_entity`.

| Table name | Description |
|------------|-------------|
| `entity_id` | Entity unique internal ID |
| `attribute_set_id` | Indicated available attributes by assigned attribute set |
| `type_id` | Indicated internal product type |
| `sku` | Unqiue human friendly ID | 
| `has_options` | some important information ;) |
| `required_options` | some important information ;) | 
| `created_at` | Entity creation time |
| `updated_at` | Entity update time (should be tracked by application and updated even if this record is not changed) |

Apart of few special cases all attribute value tables have following strucutre.

| Table Name | Description |
|------------|-------------|
| `value_id` | Internal value ID and primary key |
| `attribute_id` | Attribute ID |
| `store_id` | Store IO for this value (`0` for default scope) |
| `entity_id` | Entity internal ID |
| `value` | Atrribute value for given attribute in given scope |

Usually there are a lot of `LEFT JOIN` operations in EAV queries - using `LEFT JOIN` allows us to fetch entities with thier optional attributes.
Here is basic query example.
```sql
SELECT e.id, a1.value AS description
FROM catalog_product_entity e
LEFT JOIN catalog_product_entity_varchar a1
  ON (e.entity_id = a1.entity_id
      AND a1.attribute_id = 123)
```

As mentioned above EAV also provides data structure for translations and scoped values with option to inherit values from main scope.
This is achived by `store_id` column in value tables.
For default scope `store_id` is equal to `0` this allows us to query for values with two different `store_id` - `0` and current store.
If current store value does not exists it means that default value should be used.

```sql
SELECT e.entity_id,
       IF(s1.value_id IS NULL, d1.value, s1.value) AS num
FROM catalog_product_entity e
LEFT JOIN catalog_product_entity_int s1
  ON (e.entity_id = s1.entity_id
      AND s1.attribute_id = 123
      AND s1.store_id = 2)
LEFT JOIN catalog_product_entity_int d1
  ON (e.entity_id = d1.entity_id
      AND d1.attribute_id = 123
      AND d1.store_id = 0)
```

## Profiling database queries

Magento2 comes with database profiler build in we just have to follow [oficial configuration steps](http://devdocs.magento.com/guides/v2.0/config-guide/db-profiler/db-profiler.html) to enable it.

Modify `/app/etc/env.php` to add the following reference to the database profiler class in connection details node:

```
'profiler' => [
   'class' => '\Magento\Framework\DB\Profiler',
   'enabled' => true,
],
```
        
Add following code at the end of the `/pub/index.php` to output gathered information.        
        
```php
/** @var \Magento\Framework\App\ResourceConnection $res */
$res = \Magento\Framework\App\ObjectManager::getInstance()->get('Magento\Framework\App\ResourceConnection');
/** @var Magento\Framework\DB\Profiler $profiler */
$profiler = $res->getConnection('read')->getProfiler();
echo "<table cellpadding='0' cellspacing='0' border='1'>";
echo "<tr>";
echo "<th>Time <br/>[Total Time: ".$profiler->getTotalElapsedSecs()." secs]</th>";
echo "<th>SQL [Total: ".$profiler->getTotalNumQueries()." queries]</th>";
echo "<th>Query Params</th>";
echo "</tr>";
foreach ($profiler->getQueryProfiles() as $query) {
    /** @var Zend_Db_Profiler_Query $query*/
    echo '<tr>';
    echo '<td>', number_format(1000 * $query->getElapsedSecs(), 2), 'ms', '</td>';
    echo '<td>', $query->getQuery(), '</td>';
    echo '<td>', json_encode($query->getQueryParams()), '</td>';
    echo '</tr>';
}
echo "</table>";
```
So lets look on how Magento loaded product data in our example based on product repository.
We can see multiple queries that are used to produce final objects.

```sql
SELECT 
  `e`.*,
  IF(at_status.value_id > 0, at_status.value, at_status_default.value) AS `status`,
  IF(at_visibility.value_id > 0, at_visibility.value, at_visibility_default.value) AS `visibility`,
  `stock_status_index`.`stock_status` AS `is_salable`
FROM 
  `catalog_product_entity` AS `e`
  INNER JOIN `catalog_product_entity_int` AS `at_status_default` ON (`at_status_default`.`entity_id` = `e`.`entity_id`) AND (`at_status_default`.`attribute_id` = '97') AND `at_status_default`.`store_id` = 0
  LEFT JOIN `catalog_product_entity_int` AS `at_status` ON (`at_status`.`entity_id` = `e`.`entity_id`) AND (`at_status`.`attribute_id` = '97') AND (`at_status`.`store_id` = 1)
  INNER JOIN `catalog_product_entity_int` AS `at_visibility_default` ON (`at_visibility_default`.`entity_id` = `e`.`entity_id`) AND (`at_visibility_default`.`attribute_id` = '99') AND `at_visibility_default`.`store_id` = 0
  LEFT JOIN `catalog_product_entity_int` AS `at_visibility` ON (`at_visibility`.`entity_id` = `e`.`entity_id`) AND (`at_visibility`.`attribute_id` = '99') AND (`at_visibility`.`store_id` = 1)
  LEFT JOIN `cataloginventory_stock_status` AS `stock_status_index` ON e.entity_id = stock_status_index.product_id AND stock_status_index.website_id = 0 AND stock_status_index.stock_id = 1
WHERE
  ((`e`.`entity_id` IN('4', '14', '1817', '1561', '6')))
```

We can see that first query executed checks what products can be showed (we are checking if they are visible, in stock and available for purchase)

```sql
SELECT 
  `t_d`.`attribute_id`, `e`.`entity_id`, `t_d`.`value` AS `default_value`,
  `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_varchar` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_varchar` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('73', '84', '86', '87', '88', '89', '96', '100', '104', '106', '109', '110', '111', '114', '116', '118', '126', '127', '129', '130', '132', '134', '135', '136', '137', '138', '139', '140', '148', '149', '150', '151', '152', '153', '154')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))

UNION ALL

SELECT
  `t_d`.`attribute_id`, `e`.`entity_id`,
  `t_d`.`value` AS `default_value`, `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_text` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_text` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('75', '76', '85', '103')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))

UNION ALL

SELECT
  `t_d`.`attribute_id`, `e`.`entity_id`, `t_d`.`value` AS `default_value`,
  `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_decimal` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_decimal` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('77', '78', '81', '82', '92', '98', '117')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))

UNION ALL

SELECT
  `t_d`.`attribute_id`, `e`.`entity_id`, `t_d`.`value` AS `default_value`,
  `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_datetime` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_datetime` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('79', '80', '94', '95', '101', '102')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))

UNION ALL
 
SELECT
  `t_d`.`attribute_id`, `e`.`entity_id`, `t_d`.`value` AS `default_value`,
  `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_int` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_int` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('83', '91', '93', '115', '119', '120', '121', '122', '123', '128', '131', '133', '141', '142', '143', '144', '145', '146', '147')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))
```

Second query is a bit overwhelming but this is basically one simple query repeted few times.
We can see that in order to avoid a lot of `JOIN` operations (also there is hard limit on number of JOIN operations in one query in MySQL implementation) there is one query executed for given attribute value type.
`UNION ALL` statement agregates results of those queries into single response since they all have the same result structure.
You can see that for evey table different set of `attribute_id` is used, since different attributes are stored in those tables.
Those queries also resolve value inhertinace on query level.

```sql
SELECT COUNT(DISTINCT e.entity_id)
FROM
  `catalog_product_entity` AS `e`
  INNER JOIN `catalog_product_entity_int` AS `at_status_default` ON (`at_status_default`.`entity_id` = `e`.`entity_id`) AND (`at_status_default`.`attribute_id` = '97') AND `at_status_default`.`store_id` = 0
  INNER JOIN `catalog_product_entity_int` AS `at_visibility_default` ON (`at_visibility_default`.`entity_id` = `e`.`entity_id`) AND (`at_visibility_default`.`attribute_id` = '99') AND `at_visibility_default`.`store_id` = 0
  INNER JOIN `catalog_product_index_price` AS `price_index` ON price_index.entity_id = e.entity_id AND price_index.website_id = '1' AND price_index.customer_group_id = 0
WHERE ((`e`.`entity_id` IN('4', '14', '1817', '1561', '6')))
```

Another query if executed in order to fetch product price from price index.
Since there is quire extensive logic to price calculation big portion of it is precalulated and stored in designated table called price index.

```sql
SELECT `url_rewrite`.*, `relation`.* FROM `url_rewrite` LEFT JOIN `catalog_url_rewrite_product_category` AS `relation` ON url_rewrite.url_rewrite_id = relation.url_rewrite_id WHERE (url_rewrite.entity_id IN ('4')) AND (url_rewrite.entity_type = 'product') AND (url_rewrite.store_id IN ('1')) AND (relation.category_id IS NULL)
SELECT `url_rewrite`.*, `relation`.* FROM `url_rewrite` LEFT JOIN `catalog_url_rewrite_product_category` AS `relation` ON url_rewrite.url_rewrite_id = relation.url_rewrite_id WHERE (url_rewrite.entity_id IN ('14')) AND (url_rewrite.entity_type = 'product') AND (url_rewrite.store_id IN ('1')) AND (relation.category_id IS NULL)
SELECT `url_rewrite`.*, `relation`.* FROM `url_rewrite` LEFT JOIN `catalog_url_rewrite_product_category` AS `relation` ON url_rewrite.url_rewrite_id = relation.url_rewrite_id WHERE (url_rewrite.entity_id IN ('1817')) AND (url_rewrite.entity_type = 'product') AND (url_rewrite.store_id IN ('1')) AND (relation.category_id IS NULL)
SELECT `url_rewrite`.*, `relation`.* FROM `url_rewrite` LEFT JOIN `catalog_url_rewrite_product_category` AS `relation` ON url_rewrite.url_rewrite_id = relation.url_rewrite_id WHERE (url_rewrite.entity_id IN ('1561')) AND (url_rewrite.entity_type = 'product') AND (url_rewrite.store_id IN ('1')) AND (relation.category_id IS NULL)
SELECT `url_rewrite`.*, `relation`.* FROM `url_rewrite` LEFT JOIN `catalog_url_rewrite_product_category` AS `relation` ON url_rewrite.url_rewrite_id = relation.url_rewrite_id WHERE (url_rewrite.entity_id IN ('6')) AND (url_rewrite.entity_type = 'product') AND (url_rewrite.store_id IN ('1')) AND (relation.category_id IS NULL)
```

Last group of queries executed is fetching product URL rewrites for every requested product.
Since `url_rewrite` table is usually quite big You want to avoid performing `JOIN` operation on it.
Also note that those quereies were not executed by product repository, but were triggered during block rendering when `getProductUrl` method were called on product object.

## Flat Index

While EAV is considered normalized database schema it is not always the best for performance.
As we could see query complexity for product data fetching is quite high.
In order to optimize performance Magento introduced flat indexes for product and category data.
Those indexes consists of one additaional data table per store view containing selected subset of attributes of products used while rendering product lists.
Data stored within those tables is already resolved for given scope.
Note that are redundant data - increasing size of Your database in order to improve operation performance.

```sql
SELECT
  1 AS `status`,
  `e`.`entity_id`,
  `e`.`attribute_set_id`,
  `e`.`type_id`,
  `e`.`created_at`,
  `e`.`updated_at`,
  `e`.`sku`,
  `e`.`name`,
  `e`.`sku`,
  `e`.`description`,
  `e`.`short_description`,
  `e`.`price`,
  `e`.`special_price`,
  `e`.`special_from_date`,
  `e`.`special_to_date`,
  `e`.`cost`,
  `e`.`weight`,
  `e`.`image`,
  `e`.`small_image`,
  `e`.`thumbnail`,
  `e`.`color`,
  `e`.`color_value`,
  `e`.`news_from_date`,
  `e`.`news_to_date`,
  `e`.`visibility`,
  `e`.`required_options`,
  `e`.`has_options`,
  `e`.`image_label`,
  `e`.`small_image_label`,
  `e`.`thumbnail_label`,
  `e`.`created_at`,
  `e`.`updated_at`,
  `e`.`msrp`,
  `e`.`msrp_display_actual_price_type`,
  `e`.`price_type`,
  `e`.`sku_type`,
  `e`.`weight_type`,
  `e`.`price_view`,
  `e`.`url_key`,
  `e`.`url_path`, 
  `e`.`links_purchased_separately`,
  `e`.`links_title`,
  `e`.`links_exist`,
  `e`.`gift_message_available`,
  `e`.`tax_class_id`,
  `e`.`activity`,
  `e`.`style_bags`,
  `e`.`material`,
  `e`.`strap_bags`,
  `e`.`features_bags`,
  `e`.`gender`,
  `e`.`category_gear`,
  `e`.`size`,
  `e`.`size_value`,
  `e`.`eco_collection`,
  `e`.`performance_fabric`,
  `e`.`erin_recommends`,
  `e`.`new`,
  `e`.`sale`,
  `e`.`format`,
  `e`.`format_value`,
  `e`.`style_bottom`,
  `e`.`style_general`,
  `e`.`sleeve`,
  `e`.`collar`,
  `e`.`pattern`,
  `e`.`climate`,
  `e`.`swatch_image`,
  IF(at_status.value_id > 0, at_status.value, at_status_default.value) AS `status`,
  IF(at_visibility.value_id > 0, at_visibility.value, at_visibility_default.value) AS `visibility`,
  `stock_status_index`.`stock_status` AS `is_salable`
FROM
  `catalog_product_flat_1` AS `e`
  INNER JOIN `catalog_product_entity_int` AS `at_status_default` ON (`at_status_default`.`entity_id` = `e`.`entity_id`) AND (`at_status_default`.`attribute_id` = '97') AND `at_status_default`.`store_id` = 0
  LEFT JOIN `catalog_product_entity_int` AS `at_status` ON (`at_status`.`entity_id` = `e`.`entity_id`) AND (`at_status`.`attribute_id` = '97') AND (`at_status`.`store_id` = 1)
  INNER JOIN `catalog_product_entity_int` AS `at_visibility_default` ON (`at_visibility_default`.`entity_id` = `e`.`entity_id`) AND (`at_visibility_default`.`attribute_id` = '99') AND `at_visibility_default`.`store_id` = 0
  LEFT JOIN `catalog_product_entity_int` AS `at_visibility` ON (`at_visibility`.`entity_id` = `e`.`entity_id`) AND (`at_visibility`.`attribute_id` = '99') AND (`at_visibility`.`store_id` = 1)
  LEFT JOIN `cataloginventory_stock_status` AS `stock_status_index` ON e.entity_id = stock_status_index.product_id AND stock_status_index.website_id = 0 AND stock_status_index.stock_id = 1
WHERE ((`e`.`entity_id` IN('4', '14', '1817', '1561', '6')))
```

Although all those attributes are stored in flat table we can still see two attributes are joined directly from EAV storage - `status` and `visibility`.
Those are quite special attributes that controll if product can be displayed on site.
In this special case we always want most current data - we do not want to wait for index maintenance process to update index, as enablind and disabling products is one of the most crucial tasks for store operators.

## Using Magento collections directly

When compared amount of data returned by product repository and amount of data needed by our solution we can see that a lot of it is fetched needlessly.
In our current example we only need product name and it's URL.
Attribute limitation cannot be applied to product repository, but since we know that it is using product collection objects underneath we can use those object directly/

> Note: this solution is braking "use only service contracts" rule.
> Such solutions should not be used with modules You indent to distribute for other stores.
> Since this will be implementation independent implementation it may brake after some Magento update or be incompatible with other third party modules.

```php
        $collection = $this->productCollectionFactory->create();
        $collection->addAttributeToFilter('entity_id', ['in' => $productIds]);
        $collection->addAttributeToSelect('name');
```

After we acquire instance of `Magento\Catalog\Model\ResourceModel\Product\Collection` object we apply at it basic filter restricting product IDs and require only one attribute `name`.
As we discussed earlier product URLs will be fetched when `getProductUrl` method will be called.

```sql
SELECT
  `e`.*,
  `stock_status_index`.`stock_status` AS `is_salable`
FROM
  `catalog_product_entity` AS `e`
  LEFT JOIN `cataloginventory_stock_status` AS `stock_status_index` ON e.entity_id = stock_status_index.product_id AND stock_status_index.website_id = 0 AND stock_status_index.stock_id = 1
WHERE (`e`.`entity_id` IN('4', '14', '1817', '1561', '6'))
```

As we can see Magento will still perform stock status check.

```sql
SELECT
  `t_d`.`attribute_id`, `e`.`entity_id`, `t_d`.`value` AS `default_value`,
  `t_s`.`value` AS `store_value`,
  IF(t_s.value_id IS NULL, t_d.value, t_s.value) AS `value`
FROM
  `catalog_product_entity_varchar` AS `t_d`
  INNER JOIN `catalog_product_entity` AS `e` ON e.entity_id = t_d.entity_id
  LEFT JOIN `catalog_product_entity_varchar` AS `t_s` ON t_s.attribute_id = t_d.attribute_id AND t_s.entity_id = t_d.entity_id AND t_s.store_id = 1
WHERE (e.entity_id IN (4, 6, 14, 1561, 1817)) AND (t_d.attribute_id IN ('73')) AND (t_d.store_id = IFNULL(t_s.store_id, 0))
```

Resulting query is still similar to ones previously seen, but it is much, much simpler as it fetched only selected data we need.
In comparison to first solution we implemented this one provides almost 85% improvement in block rendering time.

 
## Next Steps

* [Build more reusable code with Dependency Injection](/backend/di.md)
