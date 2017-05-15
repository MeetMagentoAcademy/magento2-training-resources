# Magento Admin Panel Workshops part #5

This part of workshops will teach you how to create Catalog Price Rules.

More infor: http://docs.magento.com/m2/ce/user_guide/marketing/price-rules-catalog.html

##1. Catalog price rules
Making a huge sale in your store can be complicated. But Magento has some nice tools to make it easier to, for example: disount all products in category with 30% of price.
And thats what we gonna do now. 
##2. How to move your idea for sale to Catalog Price Rules.
Before you start to create any caltalog/ cart price rule, it's good to find answers for questions below:
  * What kinds of products should be discount? (localization of product - choose category)
  * Should this sale be enabled for guest/ log in users or both?
  * How long this sale should be active.

##3. Creating new catalog price rule.

  * TASK:
    1. Go to marketing => Catalog Price Rules
    2. Add new rule
    3. Fill up "Rule information" (start date, end date etc)
    4. Conditions = "IF THIS IS TRUE" or "IF THIS IS FALSE"
    This part of creating catalog price rule ask you, what are the condition to add discount to products in category, and to what products. In our case we want Magento to check if product is in category calles "SALE" so we:
        * Set condition "If category is ..." 
    5. Actions = "THEN DO THIS"
      This part is about what should happend if condition are ok. 
      So, if product is assigne to category "SALE" then we want to discount it.
      * Set action to "Apply as % of orginal"
      * Discount ammount 30%
      * Discard subsequent rules: YES
    6. Save the rule

    Now, each product asignnee to catgeory "SALE" will get a 30% discount

###4. Types of action:
  * **Apply as percentage of original** - Discounts item by subtracting a percentage from the 
  * **Apply as fixed amount** - Discounts item by subtracting a fixed amount from original price.
  * **Adjust final price to this percentage** - Discounts item by defining the final price based on percentage. For example: Enter 10 in Discount Amount for an updated price that is 10% of the original price.

##5. Creating adavnced catalog price rule:
  * TASK:
    1. Create rule that will discount only yellow duckies in sale category. Discount= -10$/zl
        (don't forget to deactivate last calatog rule, to see correct discount in your store site, or use priority of rules) Disount should be active only for users.





