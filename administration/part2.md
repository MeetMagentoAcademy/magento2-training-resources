# Magento Admin Panel Workshops part #2

This part is about creating your own products in magento


##I. Basic elements/ views of magento store

Before you start to create your catalog it is good categorize store elements by views.

1. Open your store in browser and try to find a location with:
  * homepage view
  * category view
  * product view
  * sign in/ log in view - create your user test account 
  * my account view
  * cart view
  * checkout view
  * page view

##II. Creating catalog

####1. Site structure
![alt text](/administration/media/structure.png "test")
  * TASK:  
  By looking on your store, and graphic above choose if sentence if true of false:
      1. One product can be assignee to two different categories
      2. Products can be assignee only to one catalog.
      3. Wishlist is a list of product that client can see below category view.

####2. Websites, stores, store views.

  Let's say you want to start your own business by selling rubber duckies, and rubber chickens online. 
  You already have bought domain for your shops. But it's getting harder, because u can sell your rubber duckies only to Poland and France, and rubber chickens to Poland and Japan. So what now? You have two domains, so basically do you need to create two admin panels, as it's two completely different stores? 

  Magento has an easy way to manage 2 stores on the same admin panel, also, there's an easy way to prepare different views for different language of your store.
  
  * TASK:  
  Create two websites with one store and 3 different store views.

####3. First steps of creating catalog - creating simple products. 

  As u could see on a graphic above, products is the deepest thing in simple shop structure. 
  So when we want to create our own catalog it's good to start with creating simple product.

  More info: http://docs.magento.com/m2/ce/user_guide/catalog/product-create-simple.html

  * TASK:
    1. Find a nice image of rubber ducky
    1. Go to your admin panel
    2. Go to Products => Catalog => ...
    3. Try to create 2 simple product with this info:
      * name: rubber ducky
      * description: it's cool ducky
      * color: yellow
      * add an product image
    4. Choose product type
    5. Choose attribute set
          
          You can already see that this information i gave u above it's not enough to create product.
          To create simple product you need filled up this information:
          Product name, sku, price.

         _What is SKU: In the field of inventory management, a **stock keeping unit**._
  * TASK:
      1. Find a nice image of rubber chicken.
      2. Create next 2 simple product.
      2. Information about product: I'm counting on your creativity.

####4. Creating configurable products.

  Your a rubber ducky seller, and u want to extended your sell by selling same model of the rubber ducky but in a different colors. This is where can use configurable products for giving your clinets a chance to choose the color of your products in product view.

  A configurable product looks like a single product with drop-down lists of options for each variation. Each option is actually a separate simple product with a unique SKU, which makes it possible to track inventory for each product variation. You could achieve a similar effect by using a simple product with custom options, but without the ability to track inventory for each variation.
  More info: http://docs.magento.com/m2/ce/user_guide/catalog/product-create-configurable.html

  * TASK: 
    1. Create configurable product
      * Go to catalog
      * Click on "Add product"
      * Choose a type of product
      * Choose attribute set
      * Add all needed information: name, sku, price
      * Find a tab "Configurations" 
      * Choose attribute from the list( in this case we choose "color")
      * Choose values of attribute
      * Add images for each color
      * Add price for each color
      * Add stock value for each color
      * Save
    2. Add product to some category.
    3. Save product

####5. Creating grouped product

  Let's say u want to give your customer a chance to buy rubber ducky, and rubber chicken by visit only one product. That's when we can create grouped product.

  A grouped product is made up of simple standalone products that are presented as a group. 
  More info: http://docs.magento.com/m2/ce/user_guide/catalog/product-create-grouped.html

  * TASK:
    1. Create grouped product:
      * Go to products => catalog
      * Click on "Add product"
      * Choose a type of product
      * Choose attribute set
      * Add all needed information: name, sku, price etc.
      * Find tab called "Grouped product"
      * Add yours products to group
      * Save your grouped product

####6. Creating virtual and bundle products 
    http://docs.magento.com/m2/ce/user_guide/catalog/product-types.html



 
