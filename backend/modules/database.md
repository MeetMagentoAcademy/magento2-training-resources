# Accessing database

In Magento 2 database persistence layer is direct port of Magento 1 solution and we expect it's major overhaul in next releases.

Object relation mapping default implementation is quite simple - object structure is inferred based on result of `DESCRIBE TABLE` operation.
Because of that no mapping definition exists in Magento.
This also means that all database schema altering operations must be performed explicitly in module setup scripts.

## Migration scripts

Every time Magento CLI command `setup:upgrade` is run system compares module versions in `setup_module` with those defined in `etc/module.xml` files for every module.
Based on that list of modules that require database migration is complied.

In database table `setup_module` we can see that two versions are tracked: `schema_version` and `data_version`.
Reason behind it is quite simple - there are two types of update scripts that can be defined: schema updates that alter structure of database schema and data updates that introduce or change data in database without changing database structure.
All data scripts are run after all schema scripts to ensure that all required database structure changes were performed first.

> Data migration scripts are often used to provide default data for module to work for.
> But there are very useful when structure of module database structure changes and there is need to add new information to existing records to keep them valid.
> Data migration scripts could be ran even without schema changes for example when migratin from Magento 2.1 to Magento 2.2 there were security decision made to replace PHP serialization with JSON serialization - here data migration script can be used to recalculate data existing in database to new serialization format.

Magento will look for following files:
* `Setup/InstallSchema.php` must implement `Magento\Framework\Setup\InstallSchemaInterface` will be executed _once_ to perform initial database schema setup for module.
* `Setup/UpgradeSchema.php` must implement `Magento\Framework\Setup\UpgradeSchemaInterface` will be executed every time change in module version is found.
* `Setup/InstallData.php` must implement `Magento\Framework\Setup\InstallDataInterface` will be executed _once_.
* `Setup/UpgradeData.php` must implement `Magento\Framework\Setup\UpgradeDataInterface` will be executed every time change in module version is found.

Note that almost all `UpgradeSchema` classes You will find use version number comparision in order to find what upgrades should be called (You can expect multiple changes in database scheme during module lifetime, so expect changed to `UpgradeSchema` class) while this solution is simple to use I would urge You to use different approach.
`SchemaSetupInterface` passed as migration script argument is capable of checking existence of tables, table columns, indexes or even describing table structure, please consider using those method to calculate actions required to achieve final scheme state.

### Example install script

Here is example install script that will create new database table `mmacademy_micro_review` with four columns:
* `review_id` as integer primary key with auto increment
* `product_id` as integer field
* `message` as text field no longer than 255 characters
* `created_at` as timestamp field

```php
<?php

namespace MMAcademy\MicroReview\Setup;

use Magento\Framework\DB\Ddl\Table;
use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class InstallSchema implements InstallSchemaInterface
{
    /**
     * Installs DB schema for a module
     *
     * @param SchemaSetupInterface   $setup
     * @param ModuleContextInterface $context
     * @return void
     * @throws \Zend_Db_Exception
     * @SuppressWarnings(PHPMD.UnusedFormalParameter)
     */
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();

        $tableName = $setup->getTable('mmacademy_micro_review');

        $table = $setup->getConnection()->newTable($tableName);
        $table
            ->addColumn(
                'review_id',
                Table::TYPE_INTEGER,
                null,
                ['identity' => true, 'nullable' => false, 'primary' => true],
                'Review ID'
            )
            ->addColumn('product_id', Table::TYPE_INTEGER, null, [], 'Product ID')
            ->addColumn('message', Table::TYPE_TEXT, 255, [], 'Review message')
            ->addColumn('created_at', Table::TYPE_DATETIME, null, [], 'Created At');

        $setup->getConnection()->createTable($table);

        $setup->endSetup();
    }
}
```

## Database access objects

In order to access data stored in database tables we need three model classes defined for every simple database entity.
* Model class - allows operation on data in active record like fashion and provides place for basic business logic
* Resource model class - is responsible for performing database operation on behalf of model class
* Collection class - acts as interface for retrieving large number of object of given type with ability to apply database level filtering.

Note that those three classes will be required for all logical datatbase entities but not for all database tables as there are complex strucutres in Magento that persist one logical entity in mutiple database tables.
This is also reason why such elaborate structure is needed - for simple plain oen table entity mapping default implementation is provided but for complex structures that require loading data from multiple tables there is still clear separation of concerns.
Model object will be used in the same way regardless of how complex it's persistence code is, and this code will be always separated from business logic in resource model.

### Model clasess example

Minimal model class has only to indicate what resource model knows how to persist it by executing `_init` method.

```php
<?php

namespace MMAcademy\MicroReview\Model;

use Magento\Framework\Model\AbstractModel;

class Review extends AbstractModel
{
    protected function _construct()
    {
        $this->_init(ResourceModel\Review::class);
    }
}
```

Minimal resource model class has only to indicate what database table it operates on and column that acts as primary index.

```php
<?php

namespace MMAcademy\MicroReview\Model\ResourceModel;

use Magento\Framework\Model\ResourceModel\Db\AbstractDb;

class Review extends AbstractDb
{
    /**
     * Resource initialization
     *
     * @return void
     */
    protected function _construct()
    {
        $this->_init('mmacademy_micro_review', 'review_id');
    }
}
```

Collection class has only to indicate model and resource models it is connected to.

```php
<?php

namespace MMAcademy\MicroReview\Model\ResourceModel\Review;

use Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection;
use MMAcademy\MicroReview\Model\Review;

class Collection extends AbstractCollection
{
    protected function _construct()
    {
        $this->_init(Review::class, \MMAcademy\MicroReview\Model\ResourceModel\Review::class);
    }
}
```


## Next Steps

* [APIs](service_contracts.md)
