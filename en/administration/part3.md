# Magento Admin Panel Workshops part #3
This section will teach you how to create new attributes, customise your products, optimize your navigation in catalog and create attribute sets.

##1. Attributes
Attributes are the building blocks of your product catalog, and describe specific characteristics of a product. Attributes determine the type of input control that is used for product options, provide additional information for product pages, and are used as search parameters and criteria for layered navigation.
* TASK:
    Let's say u want add a new attribute that will give customer information about sex of the rubber ducky you're selling. (male/female/unisex) 

    1. Go to Stores - Attributes - Product attributes
    2. Click "Add new attribute"
    3. Using instructions from: http://docs.magento.com/m2/ce/user_guide/catalog/product-attributes-add.html create attribute:
      * should be added to all kind of product types
      * Catalog Input - multiselect
      * Attribute code "sex"
      * Scope - Global

##2. Attributes set
The attributes are organized into groups that determine where they appear in the product record. Your store comes with an initial attribute set called “default” which includes a set of commonly-used attributes. If you would like to add only a small number of attributes, you can add them to the default attribute set. However, if you sell products that require specific types of information, such as cameras, it might be better to create a dedicated attribute set that includes the specific attributes that are needed to describe the product.

* TASK: 
    After creating new attribute it should be assignee to attribute set. 
    1. Go to Stores => Attributes => Attribute set
    2. Choose default set
    3. Find your "sex" attribute on the list and add it to "product detail folder" after price attribute.
    4. Save attribute set.

* TASK:
    As u can see in default set there's already many attributes there.
    Let's create new set, with new group called "Basic info"
    1. Go to Stores => Attributes => Attribute set
    2. Add new attribute set called "Rubber set"
    3. Choose based on "default" set.
    4. Add group "Basic info"
    5. Add NAME, DESCRIPTION, SKU, PRICE, PRICE TYPE, TAX CLASS, STATUS, VISIBILITY, STORE VIEW
    6. Save new attribute set
