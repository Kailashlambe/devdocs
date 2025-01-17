---
group: javascript-developer-guide
title: RequireJS in Commerce
contributor_name: Adarsh Manickam
contributor_link: https://github.com/drpayyne
migrated_to: https://developer.adobe.com/commerce/frontend-core/javascript/requirejs/
layout: migrated
---

This topic describes general concepts of how the [RequireJS library](https://requirejs.org) is used in Magento, with examples. Please refer to official RequireJS documentation for in-depth explanation.

RequireJS is a JavaScript file and module loader. It improves perceived page load times because it allows JavaScript to load in the background. In particular, it enables asynchronous JavaScript loading.

## RequireJS configuration in Magento {#requirejs-config}

All configuration is done in the `requirejs-config.js` file. It has a single root object `config` which contains the configuration options described below. All the configuration settings are optional and are used only when required. The following snippet is a sample `requirejs-config.js` describing the structure of the file. Example [`requirejs-config.js` file]({{ site.mage2bloburl }}/{{ page.guide_version }}/app/code/Magento/Theme/view/base/requirejs-config.js)

```javascript
var config = {
    map: {...},
    paths: {...},
    deps: [...],
    shim: {...},
    config: {
        mixins: {...},
        text: {...}
    }
}
```

### map {#requirejs-config-map}

The `map` configuration maps (connects) any real AMD modules that calls `define()`, to the specified alias. In the snippet below, `*` means all loaded RequireJS modules can use the specified alias. The second mapping applies only in the context of `Vendor_Module/js/amd-module`. So, both types of contexts can be applied: either a global context, or a module specific context.

```javascript
map: {
    '*': {
        alias: 'Vendor_Module/js/complex/path/amd-module'
    },
    'Vendor_Module/js/amd-module': {
        alias-two: 'Vendor_Module/js/complex/path/amd-module-two'
    }
}
```

Now we can use our `Vendor_Module/js/complex/path/module` using `alias` in any RequireJS module or config file without needing to type the entire path. For example, in Magento, `catalogAddToCart` is mapped to `Magento_Catalog/js/catalog-add-to-cart` and can be used anywhere as a RequireJS module name. In the next example, `catalogAddToCart` is mapped to `Magento_Catalog/js/catalog-add-to-cart` only in the context of the `discountCode` module.

```javascript
map: {
    '*': {
        catalogAddToCart: 'Magento_Catalog/js/catalog-add-to-cart'
    },
    'discountCode': {
        catalogAddToCart: 'Magento_Catalog/js/catalog-add-to-cart'
    }
}
```

{:.bs-callout-tip}
You can also use the `map` configuration to override a JS module with a custom JS module. See [Custom JS component]({{ page.baseurl }}/javascript-dev-guide/javascript/custom_js.html#js_replace).

### paths {#requirejs-config-paths}

The `paths` configuration, similar to `map`, is used for aliasing not just any real AMD module that calls `define()`, but also any JS file (even from a URL), HTML templates, etc. Magento uses this to alias URLs and third party libraries.

```javascript
paths: {
    'alias': 'library/file',
    'another-alias': 'https://some-library.com/file'
}
```

{:.bs-callout-info}
When setting a path to an array with multiple script sources, if the first script fails to load, the next is used as a fallback.

```javascript
var config = {
    ...
    paths: {
        'alias': [
            'https://some-library.com/file',
            'vendor_name>_<module_name>/js/file'
        ]
    }
};
```

For external content, resources should be whitelisted; otherwise Magento raises error notices in the browser console. See [Content Security Policies]({{ page.baseurl }}/extension-dev-guide/security/content-security-policies.html).

Consider the example of overwriting an HTML file in the adminhtml.
In this example, the `max-length` value of the text-box in the `adminhtml` is altered. The HTML file is located at `vendor/magento/module_ui/view/base/web/templates/form/element/input.html`.

1. Create a `requirejs-config.js` file under `app/code/<Vendor_Name>/<Module_Name>/view/base/` and add the following code:

    ```javascript
    var config = {
        paths: {
            'ui/template/form/element/input': '<vendor_name>_<module_name>/template/form/element/input'
        }
    };
    ```

1. Create an `input.html` file under `app/code/<Vendor_Name>/<Module_Name>/view/base/web/template/form/element/` and copy the contents of the `input.html` file from the `module_ui` template file.
1. Change the maxlength value to `512`, which was originally set to `256`.
1. Upgrade the Magento application:

   ```bash
   bin/magento setup:upgrade
   ```

1. Generate the dependency injection configuration:

   ```bash
   bin/magento setup:di:compile
   ```

1. Confirm the modification by inspecting the element source code and check the `maxlength` value, which should be `512` as specified in the template.

{:.bs-callout-info}
The path for `Magento_Ui/templates` is set to be `ui/template` in the `requirejs-config.js` module of `module_ui`, hence `ui/template` is used for specifying the path. If no paths are set, `<module_name>/templates` should be used.

### deps {#requirejs-config-deps}

The `deps` configuration is used to add a dependency. It can either be used directly under `var config = {}` or under a [shim configuration](#requirejs-config-shim). Adding modules under an independent `deps` configuration will load the specified modules in all pages.

In this snippet, the custom `Vendor_Module/js/module` will be loaded in all pages.

```javascript
deps: ['Vendor_Module/js/module']
```

### shim {#requirejs-config-shim}

The `shim` configuration is used to build a dependency on a third party library, since we cannot modify it.

When to use the `shim` configuration:

-  To add a new dependency to a third party library
-  To add a new dependency to a third party library which does not use an AMD module
-  To change load order by adding a dependency to a third party library

In this snippet, dependencies are added directly in an array, or it can be specified as an array under the `deps` key. The `exports` key is used to specify under what identifier the module is exported into. This export identifier can be used to access it.

```javascript
shim: {
    '3rd-party-library': ['myJSFile'],
    'another-3rd-party-library': {
        deps: ['myJSFile'],
        exports: 'another3rdPartyLibrary'
    }
}
```

### mixins {#requirejs-config-mixin}

The `mixins` configuration is used to overwrite component methods of an AMD module which returns either a UI component, a jQuery widget, or a JS object. Unlike the above configuration properties, the `mixins` property is under the `config` property, apart from the parent object called **config**.

In this snippet, `Vendor_Module/js/module-mixin` will overwrite the existing component methods in `Vendor_Module/js/module` with the specified component methods. It is a convention to name the mixin by appending a `-mixin` to the original `path/to/js`, although not required.

```javascript
config: {
    mixins: {
        'Vendor_Module/js/module': {
            'Vendor_Module/js/module-mixin': true
        }
    }
}
```

The concept of Javascript mixins itself is explained in depth in [Using Javascript Mixins]({{ page.baseurl }}/javascript-dev-guide/javascript/js_mixins.html).

### text

The `text` configuration is used to set the security request headers using the [`text.js`]({{ site.mage2bloburl }}/{{ page.guide_version }}/lib/web/mage/requirejs/text.js) file.

Without [Cross Origin Resource Sharing (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) it is not possible to add the `X-Requested-With` header to a cross domain XHR request. Set this header to tell the server that the request was initiated from the same domain.

```javascript
config: {
    text: {
        'headers': {
            'X-Requested-With': 'XMLHttpRequest'
        }
    }
}
```
