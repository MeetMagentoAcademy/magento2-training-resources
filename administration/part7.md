#Magento Admin Panel Workshops part #7

###1. Customers - basic info

In default magento 2 customers who open an account with your store enjoy a range of benefits, including:
* Faster checkout. Registered customers move through checkout faster because much of the information is already in their accounts.
* Self service. Registered customers can update their information, check the status of orders, and even reorder from their account dashboard.

* User of your store has an access to his account, where user can gets more info about his orders, add shipping address, payment address etc. 

* All views u can see here:http://docs.magento.com/m2/ce/user_guide/customers/account-dashboard.html
Or by creating your own account as user.

###2. Conf. Customer Accounts
####1. Online Session Length
Online session length is one of the most important thing for customers - even if they don't feel it. But in 2017 many of customers use their shopping cart as a wishlist, even if shop has such a feature itself. So set up time for how long user will be logged after log in can be a key issue. 
   * TASK: Set up online session length

Help: http://docs.magento.com/m2/ce/user_guide/customers/customer-online-options.html
If client is using shopping cart as a wishlist, it's good to enable **Persistent Cart**
Enabled **Persistent Cart** means that the the contents of log in customer cart are saved for the next time they sign in to their accounts. 
   * TASK: Check if your shop has enabled **Persistent Cart**, if not enabled it.
   * TASK: Make a test, and check of persistent cart works on your store.

####2. Customer Account Scope
In case of having more than one website, we can decide if client that will create his account on website 1 can use it in all website or not.
  * TASK: Set up Customer Account Scope
Help: http://docs.magento.com/m2/ce/user_guide/customers/account-scope.html

####3. New account options
Basically new account options is the part of customer configuration, where we can choose to what group client should be assignee after creating account, what information u want to get from customers, set up if any created account should be confirmed by mail. etc.

More info: http://docs.magento.com/m2/ce/user_guide/customers/account-options-new.html

####4. Address option configuration
This part is for configure if we want to get info about date of birth our customers, gender etc.
  * TASK:
Configure address option.
Steps: http://docs.magento.com/m2/ce/user_guide/customers/name-address-options.html

####5. Password option configuration
  * TASK: 
Configure password option. 
Steps: http://docs.magento.com/m2/ce/user_guide/customers/password-options.html
