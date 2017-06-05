#Magento Admin Panel Workshops part #13

###EXPORT / IMPORT PRODUCTS


####EXPORT
Magento 2 Admin Panel has a feature that export products to a file. It's the best way to massive create or edit already created products.

The best way to become familiar with the structure of your database is to export the data and open it in a spreadsheet. Once you become familiar with the process, you’ll find that it is an efficient way to manage large amounts of information.

* TASK:
Export file with all products on your store. 

1. To export products you need to
SYSTEM => Data transfer => Export
2. Choose products
3. Choose a format of exported file
4. Exclude things your not interested. 
Recommended field that should be in file: SKU, Name, ID,Website, Store, Stock, Visibility, Status, Price. 
5. Click continue.

* TASK:
Edit file, change price some of your products. 
Change stock some of your products.

####IMPORT

Data for all product types can be imported into the store. In addition, you can import customer data, customer address data, and product images. Import supports the following operations:

Add/Update
Replace Existing Complex Data
Delete Entities
The size of the import file is determined by the settings in the php.ini file on the server. The system message on the Import page indicates the current size limit.

* TASK:
1. Go to System => Data transfer => Import 
2. Configure import
3. Import edited file.
(Remember to save your file as csv. )
