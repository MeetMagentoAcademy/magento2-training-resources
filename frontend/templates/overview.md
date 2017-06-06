# Templates basic concepts

## How templates are initiated

Templates are usually initiated in layout files. Each layout block has an associated template. The template is specified in the template attribute of the layout instruction. For example, from `<Magento_Catalog_module_dir>/view/frontend/layout/catalog_category_view.xml`:

```xml
<block class="Magento\Catalog\Block\Category\View"
       name="category.image"
       template="Magento_Catalog::category/image.phtml"
/>
```

This means that the category.image block is rendered by the `image.phtml` template, which is located in the category subdirectory of the `Magento_Catalog` module templates directory.

The templates directory of `Magento_Catalog` is `<Magento_Catalog_module_dir>/view/frontend/templates`.

## Conventional templates location

Templates are stored in the following locations:

- Module templates: `<module_dir>/view/frontend/templates/<path_to_templates>`
- Theme templates: `<theme_dir>/<Namespace>_<Module>/templates/<path_to_templates>`

Here <path_to_templates> might have several levels of directory nesting, or might be empty. Examples:
- `<Magento_Catalog_module_dir>/view/frontend/templates/product/widget/new/content/new_grid.phtml`
- `<Magento_Checkout_module_dir>/view/frontend/templates/cart.phtml`

## Templates overriding

For template files with the same name, the following is true: theme templates override module templates, and those of a child theme override parent theme templates.

This mechanism is the basis of the template customization concept in Magento application: to change the output defined by a certain default template, you need to override one in your custom theme.

## Root template

In Magento there’s a special template which serves as root template for all storefront pages in the application: `<Magento_Theme_module_dir>/view/base/templates/root.phtml`.

Unlike other templates, root.phtml contains the doctype specification and contributes to `<head>` and `<body>` sections of all pages rendered by Magento application. But similar to other templates, root.phtml can be overridden in a theme.

## HTML templates
Not every template is going to be defined through XML layouts. We have "UI Components" which combines power of XML, PHP and JS togheter, to create reusable components and they are (mostly )using HTML templates with Knockout.js syntax inside to render data.

It's a complex thing and we are not going to cover it in this course, so just be aware of existence of Knockout flavored HTML templates.

# "HTML" email templates
HTML templates are also used to build emails, but it's not just HTML code inside.
`app/code/Magento/Sales/view/frontend/email/order_new.html`

```html
<!--
/**
 * Copyright © 2013-2017 Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
-->
<!--@subject {{trans "Your %store_name order confirmation" store_name=$store.getFrontendName()}} @-->
<!--@vars {
"var formattedBillingAddress|raw":"Billing Address",
"var order.getEmailCustomerNote()":"Email Order Note",
"var order.increment_id":"Order Id",
"layout handle=\"sales_email_order_items\" order=$order area=\"frontend\"":"Order Items Grid",
"var payment_html|raw":"Payment Details",
"var formattedShippingAddress|raw":"Shipping Address",
"var order.getShippingDescription()":"Shipping Description",
"var shipping_msg":"Shipping message"
} @-->

{{template config_path="design/email/header_template"}}

<table>
    <tr class="email-intro">
        <td>
            <p class="greeting">{{trans "%customer_name," customer_name=$order.getCustomerName()}}</p>
            <p>
                {{trans "Thank you for your order from %store_name." store_name=$store.getFrontendName()}}
                {{trans "Once your package ships we will send you a tracking number."}}
                {{trans 'You can check the status of your order by <a href="%account_url">logging into your account</a>.' account_url=$this.getUrl($store,'customer/account/',[_nosid:1]) |raw}}
            </p>
            <p>
                {{trans 'If you have questions about your order, you can email us at <a href="mailto:%store_email">%store_email</a>' store_email=$store_email |raw}}{{depend store_phone}} {{trans 'or call us at <a href="tel:%store_phone">%store_phone</a>' store_phone=$store_phone |raw}}{{/depend}}.
                {{depend store_hours}}
                    {{trans 'Our hours are <span class="no-link">%store_hours</span>.' store_hours=$store_hours |raw}}
                {{/depend}}
            </p>
        </td>
    </tr>
    <tr class="email-summary">
        <td>
            <h1>{{trans 'Your Order <span class="no-link">#%increment_id</span>' increment_id=$order.increment_id |raw}}</h1>
            <p>{{trans 'Placed on <span class="no-link">%created_at</span>' created_at=$order.getCreatedAtFormatted(2) |raw}}</p>
        </td>
    </tr>
    <tr class="email-information">
        <td>
            {{depend order.getEmailCustomerNote()}}
            <table class="message-info">
                <tr>
                    <td>
                        {{var order.getEmailCustomerNote()|escape|nl2br}}
                    </td>
                </tr>
            </table>
            {{/depend}}
            <table class="order-details">
                <tr>
                    <td class="address-details">
                        <h3>{{trans "Billing Info"}}</h3>
                        <p>{{var formattedBillingAddress|raw}}</p>
                    </td>
                    {{depend order.getIsNotVirtual()}}
                    <td class="address-details">
                        <h3>{{trans "Shipping Info"}}</h3>
                        <p>{{var formattedShippingAddress|raw}}</p>
                    </td>
                    {{/depend}}
                </tr>
                <tr>
                    <td class="method-info">
                        <h3>{{trans "Payment Method"}}</h3>
                        {{var payment_html|raw}}
                    </td>
                    {{depend order.getIsNotVirtual()}}
                    <td class="method-info">
                        <h3>{{trans "Shipping Method"}}</h3>
                        <p>{{var order.getShippingDescription()}}</p>
                        {{if shipping_msg}}
                        <p>{{var shipping_msg}}</p>
                        {{/if}}
                    </td>
                    {{/depend}}
                </tr>
            </table>
            {{layout handle="sales_email_order_items" order=$order area="frontend"}}
        </td>
    </tr>
</table>

{{template config_path="design/email/footer_template"}}

```
