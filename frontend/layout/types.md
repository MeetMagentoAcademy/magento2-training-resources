# Layout file types

## Page layout
A page layout file defines the page wireframe, for example, one-column layout. Technically page layout is an .xml file defining the structure inside the `<body>` section of the HTML page markup. Page layouts feature only containers. All page layouts used for page rendering should be declared in the page layout declaration file.

### Allowed layout instructions:
- `<container>`
- `<referenceContainer>`
- `<move>`
- `<update>`

### Sample page layout:

`<Magento_Theme_module_dir>/view/frontend/page_layout/2columns-left.xml`

```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <update handle="1column"/>
    <referenceContainer name="columns">
        <container name="div.sidebar.main" htmlTag="div" htmlClass="sidebar sidebar-main" after="main">
            <container name="sidebar.main" as="sidebar_main" label="Sidebar Main"/>
        </container>
        <container name="div.sidebar.additional" htmlTag="div" htmlClass="sidebar sidebar-additional" after="div.sidebar.main">
            <container name="sidebar.additional" as="sidebar_additional" label="Sidebar Additional"/>
        </container>
    </referenceContainer>
</layout>
```

To be able to use a layout for actual page rendering, you need to declare it in `layouts.xml`.

## Page configuration

Page configuration is also an `.xml` file. It defines the detailed structure (page header, footer, etc.), contents and page meta information, including the page layout used. Page configuration features both main elements, blocks of particular classes and containers.


<table>
  <tbody>
    <tr>
      <th>Element</th>
      <th colspan="1">Attributes</th>
      <th colspan="1">Parent of</th>
      <th>Description</th>
    </tr>
    <tr>
      <td>
        <code><span>&lt;page&gt;&lt;/page&gt;</span></code>
      </td>
      <td colspan="1">
        <ul>
          <li>
            <code>layout = {layout}</code>
          </li>
          <li>
            <code>xsi:noNamespaceSchemaLocation ="{path_to_schema}"</code>
          </li>
        </ul>
      </td>
      <td colspan="1">
        <ul>
          <li><code>&lt;html&gt;</code></li>
          <li><code>&lt;head&gt;</code></li>
          <li><code>&lt;body&gt;</code></li>
          <li><code>&lt;update&gt;</code></li>
        </ul>
      </td>
      <td>Mandatory root element.</td>
    </tr>
    <tr>
      <td colspan="1"><code>&lt;html&gt;&lt;/html&gt;</code></td>
      <td colspan="1">
        <p>none</p>
      </td>
      <td colspan="1">
        <ul>
          <li><code>&lt;attribute&gt;</code></li>
        </ul>
      </td>
      <td colspan="1">
        <span>Mandatory element.</span>
      </td>
    </tr>
    <tr>
      <td colspan="1"><code>&lt;head&gt;&lt;/head&gt;</code></td>
      <td colspan="1">none</td>
      <td colspan="1">
        <ul>
          <li><code>&lt;title&gt;</code></li>
          <li><code>&lt;meta&gt;</code></li>
          <li><code>&lt;link&gt;</code></li>
          <li><code>&lt;css&gt;</code></li>
          <li><code>&lt;script&gt;</code></li>
        </ul>
      </td>
      <td colspan="1"></td>
    </tr>
    <tr>
      <td colspan="1"><code>&lt;body&gt;&lt;/body&gt;</code></td>
      <td colspan="1">none</td>
      <td colspan="1">
        <ul>
          <li><code>&lt;block&gt;</code></li>
          <li><code>&lt;container&gt;</code></li>
          <li><code>&lt;move&gt;</code></li>
       <li><code>&lt;attribute&gt;</code></li>
          <li><code>&lt;referenceBlock&gt;</code></li>
          <li><code>&lt;referenceContainer&gt;</code></li>
          <li><code>&lt;action&gt;</code></li>
        </ul>
      </td>
      <td colspan="1"></td>
    </tr>
    <tr>
      <td colspan="1"><code>&lt;attribute&gt;</code></td>
      <td colspan="1">
        <ul>
          <li><code>name = {arbitrary_name}</code>
          </li>
          <li><code>value = {arbitrary_value}</code>
          </li>
        </ul>
      </td>
      <td colspan="1"></td>
      <td colspan="1">
        <p>Specified for <code>&lt;html&gt;</code>, rendered like following:</p>
        <p><code>&lt;html name="value'&gt;</code></p>
      </td>
    </tr>
    <tr>
      <td colspan="1">
        <p><code>&lt;title&gt;</code></p>
      </td>
      <td colspan="1">none</td>
      <td colspan="1">none</td>
      <td colspan="1">Page title</td>
    </tr>
    <tr>
      <td colspan="1">
        <p><code>&lt;meta&gt;</code></p>
      </td>
      <td colspan="1">
        <ul>
          <li>
            <span><code>content</code></span>
          </li>
          <li>
            <span><code>charset</code></span>
          </li>
          <li>
            <span><code>http-equiv</code></span>
          </li>
          <li>
            <span><code>name</code></span>
          </li>
          <li>
            <span><code>scheme</code></span>
          </li>
        </ul>
      </td>
      <td colspan="1">
        <span>none</span>
      </td>
      <td colspan="1"></td>
    </tr>
    <tr>
      <td colspan="1">
        <p><code>&lt;link&gt;</code></p>
      </td>
      <td colspan="1">
        <ul>
          <li>
            <span><code>defer</code></span>
          </li>
          <li>
            <span><code>ie_condition</code></span>
          </li>
          <li>
            <span><code>charset</code></span>
          </li>
          <li>
            <span><code>hreflang</code></span>
          </li>
          <li>
            <span><code>media</code></span>
          </li>
          <li>
            <span><code>rel</code></span>
          </li>
          <li>
            <span><code>rev</code></span>
          </li>
          <li>
            <span><code>sizes</code></span>
          </li>
          <li>
            <span><code>src</code></span>
          </li>
          <li>
            <span><code>src_type</code></span>
          </li>
          <li>
            <span><code>target</code></span>
          </li>
          <li>
            <span><code>type</code></span>
          </li>
        </ul>
      </td>
      <td colspan="1">
        <span>none</span>
      </td>
      <td colspan="1"> </td>
    </tr>
    <tr>
      <td colspan="1">
        <span><code>&lt;css&gt;</code></span>
      </td>
      <td colspan="1">
        <ul>
          <li>
            <span><code>defer</code></span>
          </li>
          <li>
            <span><code>ie_condition</code></span>
          </li>
          <li>
            <span><code>charset</code></span>
          </li>
          <li>
            <span><code>hreflang</code></span>
          </li>
          <li>
            <span><code>media</code></span>
          </li>
          <li>
            <span><code>rel</code></span>
          </li>
          <li>
            <span><code>rev</code></span>
          </li>
          <li>
            <span><code>sizes</code></span>
          </li>
          <li>
            <span><code>src</code></span>
          </li>
          <li>
            <span><code>src_type</code></span>
          </li>
          <li>
            <span><code>target</code></span>
          </li>
          <li>
            <span><code>type</code></span>
          </li>
        </ul>
      </td>
      <td colspan="1">
        <span>none</span>
      </td>
      <td colspan="1"></td>
    </tr>
    <tr>
      <td colspan="1">
        <p><code>&lt;script&gt;</code></p>
      </td>
      <td colspan="1">
        <ul>
          <li>
            <span><code>defer</code></span>
          </li>
          <li>
            <span><code>ie_condition</code></span>
          </li>
          <li>
            <span><code>async</code></span>
          </li>
          <li>
            <span><code>charset</code></span>
          </li>
          <li>
            <span><code>src</code></span>
          </li>
          <li>
            <span><code>src_type</code></span>
          </li>
          <li>
            <span><code>type</code></span>
          </li>
        </ul>
      </td>
      <td colspan="1">
        <span>none</span>
      </td>
      <td colspan="1"></td>
    </tr>
  </tbody>
</table>

<style>
  .markdown-section table td, .markdown-section table th {
    padding: 4px !important;
  }
</style>


## Generic layouts

They are `.xml` files which define the contents and detailed structure inside the `<body>` section of the HTML page markup. These files are used for pages returned by AJAX requests, emails, HTML snippets and so on.
