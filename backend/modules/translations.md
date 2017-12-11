# Translations

Magento framework allows us to provide translations for modules we prepare.
Translation system is based on `.csv` files containing original text and translated text in every line.

Using `Magento\Framework\Phrase` we can access translated string.
More over there is handy shortcut defined in `app/functions.php` file - the `__()` function:

```php
/**
 * Create value-object \Magento\Framework\Phrase
 *
 * @return \Magento\Framework\Phrase
 */
function __()
{
    $argc = func_get_args();

    $text = array_shift($argc);
    if (!empty($argc) && is_array($argc[0])) {
        $argc = $argc[0];
    }

    return new \Magento\Framework\Phrase($text, $argc);
}
```

Function `__()` takes variable number of parameters but at least one is required - text we want to fetch translation for.
Following parameters can be used to substitute placeholders in original text.

Placeholders are constructed with `%` sign followed by number denoting function call argument number.
This allows translators to swap placeholder positions in translated text to mach language grammar (in Magento 1 it was not supported).

All translation files should be placed in `i18n` directory with file name corresponding to language code (for example `i18n/pl_PL.csv`)

## Example

So lets take a look at our example template after translations have been applied:

```php
<?php /** @var \MMAcademy\Hello\Block\Hello $block */ ?>
<h1><?= __('Hello there') ?></h1>
<?php if($block->getName()): ?>
    <p><?= __('How are you %1?', $block->escapeHtml($block->getName())); ?></p>
<?php else: ?>
    <p><?= __('Hello, how are You?') ?></p>
<?php endif ?>
```

Here every translatable sentence was passed as first parameter to `__()` function.
In line 4 we can find example of placeholder substitution - where user supplied value is passed as second parameter providing data for first placeholder.

It is common to provide translation file for original language code where original text in 100% matches translated text - this provides repository of all translatable texts.
So here we would see file `i18n/en_US.csv`:

```csv
Hello there,Hello there
How are you %1?,How are you %1?
"Hello, how are You?","Hello, how are You?"
```

Note that it is possible to pass HTML markup in translation files.
This can be used as possible attack vector against store installations.
Attacker can use translation files to inject `<script>` tag and execute malicious Java Script code.
In order to avoid that it is best to always wrap all translated strings in `escapeHtml` methods calls.

```php
<?php /* @var \MMAcademy\Hello\Block\Hello $block */ ?>
<h1><?= $block->escapeHtml(__('Hello there')) ?></h1>
<?php if($block->getName()): ?>
    <p><?= $block->escapeHtml(__('How are you %1?', $block->getName())); ?></p>
<?php else: ?>
    <p><?= $block->escapeHtml(__('Hello, how are You?')) ?></p>
<?php endif ?>
```
Note that in line 4 we no longer escape user supplied value directly, but it still will be escaped by call wrapping `__()` function.

## Next steps

[Store configuration](store_configuration.md)
