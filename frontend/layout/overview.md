# Layout overview

Magento application implements the Model-view-controller (MVC) architecture pattern, meaning, the Magento software is architected into layers, including the view layer.

The major part of the view layer of Magento application is layout. Functionally, layout is a page structure, represented by hierarchy of elements (element tree), which can be of two types: blocks and containers. Technically, layout is defined in the `.xml` files, which contain element declarations and element manipulation instructions.

## There are three kinds of layout handles:

- *page type layout handles* – Synonyms of the page type identifiers. Correspond to “full action names” of controller actions, for example, `catalog_product_view`.
- *page layout handles* – Identifiers of specific pages. Correspond to controller actions with parameters that identify specific pages, for example, `catalog_product_view_type_simple_id_128`.
- *arbitrary handles* - Do not correspond to any page type, but other handles use them by including.

## Basic layout elements

The basic components of Magento page design are blocks and containers.

A container exists for the sole purpose of assigning content structure to a page. A container has no additional content except the content of included elements. Examples of containers include the header, left column, main column, and footer.

The following figure shows an example:

![](http://devdocs.magento.com/common/images/layouts_containers_defn.jpg)

A block represents each feature on a page and employs templates to generate the HTML to insert into its parent structural block. Examples of blocks include a category list, a mini cart, product tags, and product listing.

The following figure shows an example:

![](http://devdocs.magento.com/common/images/layouts_block_defn.jpg)

## Basic layouts

The basic view of all Magento storefront pages is defined in two page configuration layout files located in the Magento_Theme module:

`<Magento_Theme_module_dir>/view/frontend/layout/default.xml`: defines the page layout.
`<Magento_Theme_module_dir>/view/frontend/layout/default_head_blocks.xml`: defines the scripts, images, and meta data included in pages’ `<head>` section.
These basic page configuration layouts are extended in other Magento modules and in Magento themes.

You can also extend or override these files in your custom theme.
