# Magento2 Modules

All system features are organized in modules. 
Modules are located in separate directories having all required files (code, templates, configuration, assets) within that directory.

## First look at directory tree

Module directories can be found in two places:
* `app/code` directory - those are local store modules that were not installed using composer (If You downloaded Magento2 from github also here You will find all of core modules) 
* `vendor` directory - in this directory all modules installed via composer are to be found (If You installed Magento2 via composer meta package all core module will be here)

When trying to look for core code You should also look in `lib/internal/Magento/Framework` directory.
This is where framework code is located. 
It is responsible for most basic building blocks of Magento2 instance.

## Using Magento CLI to list modules

Magento2 has provided command line tool (CLI) one of it's feeatures is ability yo list all installed moduules.
Enter Magento2 installation directory and run following command:
```
$ php bin/magento module:status
```

If Your store is properly installed You should see list of modules - for empty store all of them will have `Magento` in their name.
```
List of enabled modules:
Magento_Store
Magento_AdvancedPricingImportExport
Magento_Directory
Magento_Theme
Magento_Backend
Magento_Backup
Magento_Eav
Magento_Customer
Magento_BundleImportExport
Magento_CacheInvalidate
Magento_AdminNotification
Magento_Indexer
Magento_CatalogImportExport
Magento_Cms
Magento_Rule
Magento_Catalog
Magento_Search
Magento_CatalogUrlRewrite
Magento_Widget
Magento_Quote
Magento_SalesSequence
Magento_Payment
Magento_CmsUrlRewrite
Magento_Config
Magento_ConfigurableImportExport
Magento_Msrp
Magento_CatalogInventory
Magento_Contact
Magento_Cookie
Magento_Cron
Magento_CurrencySymbol
Magento_Bundle
Magento_CustomerImportExport
Magento_Deploy
Magento_Developer
Magento_Dhl
Magento_Authorization
Magento_Downloadable
Magento_ImportExport
Magento_Sales
Magento_Email
Magento_User
Magento_Fedex
Magento_GiftMessage
Magento_Checkout
Magento_GoogleAnalytics
Magento_Ui
Magento_GroupedImportExport
Magento_GroupedProduct
Magento_DownloadableImportExport
Magento_CatalogRule
Magento_Security
Magento_LayeredNavigation
Magento_Marketplace
Magento_MediaStorage
Magento_ConfigurableProduct
Magento_Multishipping
Magento_NewRelicReporting
Magento_Newsletter
Magento_OfflinePayments
Magento_SalesRule
Magento_PageCache
Magento_Captcha
Magento_Vault
Magento_Persistent
Magento_ProductAlert
Magento_ProductVideo
Magento_CheckoutAgreements
Magento_Reports
Magento_RequireJs
Magento_Review
Magento_Robots
Magento_Rss
Magento_CatalogRuleConfigurable
Magento_Authorizenet
Magento_SalesInventory
Magento_OfflineShipping
Magento_ConfigurableProductSales
Magento_UrlRewrite
Magento_CatalogSearch
Magento_Integration
Magento_SendFriend
Magento_Shipping
Magento_Sitemap
Magento_Paypal
Magento_Swagger
Magento_Swatches
Magento_SwatchesLayeredNavigation
Magento_Tax
Magento_TaxImportExport
Magento_GoogleAdwords
Magento_Translation
Magento_GoogleOptimizer
Magento_Ups
Magento_SampleData
Magento_EncryptionKey
Magento_Usps
Magento_Variable
Magento_Braintree
Magento_Version
Magento_Webapi
Magento_WebapiSecurity
Magento_Weee
Magento_CatalogWidget
Magento_Wishlist

List of disabled modules:
None
```

As You can see there is a naming convention here.
In module name You can see it's vendor and it's actual name.
For example `Magento_Sitemap` module was created by Magento and provides sitemap generation functionality.

## Next steps

* [Create Magneto2 module](modules/create.md)
* [Tips on working with composer](modules/composer.md)
