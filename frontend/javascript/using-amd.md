# Calling and initializing JavaScript

**We strongly recommend that you use the described approaches and do not add inline JavaScript.**

# Insert a JS component in a PHTML template

Depending on your task, you might want to use declarative or imperative notation. Both ways are described in the following sections.

## Declarative notation (recommended)

Using the declarative notation to insert a JS component allows preparing all the configuration on the backend side and outputting it to the page source using standard tools. You should use declarative notation if your JavaScript component requires initialization.

In Magento 2 there are two ways of declarative notation:

1. Using the `data-mage-init` attribute
2. Using the `<script type="text/x-magento-init" />` tag

### Declarative notation using the data-mage-init attribute

Use the data-mage-init attribute to insert a JS component in a certain HTML element. The following code sample is an illustration. Here a JS component is inserted in the `<nav/>` element:

```html
<nav data-mage-init='{ "<component_name>": {...} }'></nav>
```

When inserted in a certain element, the script is called only for this particular element. It is not automatically called for other elements of this type on the page.

On DOM ready, the data-mage-init attribute is parsed to extract components’ names and configuration to be applied to the element.

### Declarative notation using the `<script type="text/x-magento-init" />` tag

To call a JS component on a HTML element without direct access to the element or with no relation to a certain element, use the `<script type="text/x-magento-init">` tag and attribute.


The following is a working code sample of a widget call using `<script>`. Here the accordion and navigation widgets are added to the element with the #main-container selector, and the pageCache script is inserted with no binding to any element.

```php
<script type="text/x-magento-init">
  {
      "#main-container": {
          "navigation": <?php echo $block->getNavigationConfig(); ?>,
          "accordion": <?php echo $block->getNavigationAccordionConfig(); ?>
      },
      "*": {
          "pageCache": <?php echo $block->getPageCacheConfig(); ?>
      }
  }
</script>
```

## Imperative notation

Imperative notation allows using raw JavaScript code on the pages and executing particular business logic. The notation using the `<script>` tag, without the `type="text/x-magento-init attribute`, is the imperative notation. The syntax is following:

```html
<script>
  require([
      'jquery',
      'accordion'  // the alias for "mage/accordion"
  ], function ($) {
      $(function () { // to ensure that code evaluates on page load
          $('[data-role=example]')  // we expect that page contains the <tag data-role="example">..</tag> markup
              .accordion({ // now we can use "accordion" as jQuery plugin
                  header:  '[data-role=header]',
                  content: '[data-role=content]',
                  trigger: '[data-role=trigger]',
                  ajaxUrlElement: "a"
              });
      });
  });
</script>
```
