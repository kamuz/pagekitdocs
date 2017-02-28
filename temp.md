### Adding theme options

Pagekit uses [Vue.js](https://vuejs.org/) to build its administration interface. Here is a [Video tutorial](https://youtu.be/3gPGyhTroSA?list=PL2rU5GxE-MQ7aYIcxTmDh4-BTHRM-9lto) on Vue.js in Pagekit.

A frequently requested feature is for the navbar to remain fixed at the top of the browser window when scrolling down the site. In the following steps, we are going to add this as an option to our theme.

#### Step 1

First, we need to create the folder `app/components` and in it the file `site-theme.vue`. Settings stored in this file affect the entire website and can be found under *Theme* in the *Settings* tab of the Site Tree. They cannot be applied to a specific page.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-site-tree.png" alt="Site Tree">
    <figcaption class="uk-thumbnail-caption">You can add any kind of Settings screen to the admin area.</figcaption>
</figure>

In the freshly created file `site-theme.vue` we add the option which will be displayed in the Pagekit administration.

```
<template>

    <div class="uk-margin uk-flex uk-flex-space-between uk-flex-wrap" data-uk-margin>
        <div data-uk-margin>
            <h2 class="uk-margin-remove">{{ 'Theme' | trans }}</h2>
        </div>
        <div data-uk-margin>
            <button class="uk-button uk-button-primary" type="submit">{{ 'Save' | trans }}</button>
        </div>
    </div>

    <div class="uk-form uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-navbar-mode" class="uk-form-label">{{ 'Navbar Mode' | trans }}</label>
            <p class="uk-form-controls-condensed">
                <label><input type="checkbox" v-model="config.navbar_sticky"> {{ 'Sticky Navigation' | trans }}</label>
            </p>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the Site Tree. To do so, we can create a *Theme* tab in the interface by adding the following script below the option in the `site-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            icon: 'pk-icon-large-brush',
            priority: 15
        },

        data: function () {
            return _.extend({config: {}}, window.$theme);
        },

        events: {

            save: function() {

                this.$http.post('admin/system/settings/config', {name: this.name, config: this.config}).catch(function (res) {
                    this.$notify(res.data, 'danger');
                });

            }

        }

    };

    window.Site.components['site-theme'] = module.exports;

</script>
```

#### Step 3

To load the script and add the option to the Site Tree, you also need to add the following to the `index.php` file.

```
'events' => [

    'view.system/site/admin/settings' => function ($event, $view) use ($app) {
        $view->script('site-theme', 'theme:app/bundle/site-theme.js', 'site-settings');
        $view->data('$theme', $this);
    }

]
```

#### Step 4

Add the default setting for the navbar mode in the `index.php` file.

```
'config' => [
    'navbar_sticky' => false
],
```

#### Step 5

Vue components need to be compiled into JavaScript using Webpack. To do so, create the file `webpack.config.js` in your theme folder:

```
module.exports = [

    {
        entry: {
            "site-theme": "./app/components/site-theme.vue"
        },
        output: {
            filename: "./app/bundle/[name].js"
        },
        module: {
            loaders: [
                { test: /\.vue$/, loader: "vue" }
            ]
        }
    }

];
```

#### Step 6

After that, run the command `webpack` inside the theme folder and `site-theme.vue` will be compiled into `/bundle/site-theme.js` with the template markup converted to an inline string.

If you haven't worked with it before, you will quickly need to [install webpack globally](http://webpack.github.io/docs/installation.html) and then also run the following in your project directory to have a local webpack version and a Vue compiler available.

```
npm install webpack vue-loader vue-html-loader babel-core babel-loader babel-preset-es2015 babel-plugin-transform-runtime --save-dev
```

Whenever you now apply changes to the Vue component, you need to run this task again. Alternatively, you can run `webpack --watch` or `webpack -w` which will stay active and automatically recompile whenever you change the Vue component. You can quit this command with the shortcut *Ctrl + C* For more information on Vue and Webpack, take a closer look at [this doc](https://pagekit.com/docs/developer/vuejs-and-webpack).

#### Step 7

Lastly, we want to load the necessary JavaScript dependencies in the head of our `views/template.php` file. In our case we are using the [Sticky component](http://getuikit.com/docs/sticky.html) from UIkit. Since it is not included in the framework core, it needs to be loaded explicitely. YOu can do that by adding a dependency to the sticky component when loading the theme's JavaScript.

```
<?php $view->script('theme', 'theme:js/theme.js', ['uikit-sticky']) ?>
```

Now all you need to do is render the option into the actual navbar in `template.php`.

```
<nav class="uk-navbar uk-position-z-index" <?= $params['navbar_sticky'] ? ' data-uk-sticky' : '' ?>>
```

In the admin area, go to *Site &gt; Settings &gt; Theme*  and enable the _Sticky Navigation_ option to see it take effect on your website.

## Widgets

Widget positions allow users to publish widgets in several locations of your theme markup. They appear in the *Widgets* area of the Pagekit admin panel and can be selected by the user when setting up a widget.

#### Step 1

To render a new widget position, you first need to register it with the `index.php` file. For example, if we want a create a new *Top* position, we will define it through the `positions` property.

```
'positions' => [

    'sidebar' => 'Sidebar',
    'top' => 'Top'

],
```

#### Step 2

Now that we have made the new position known to Pagekit, we need also to create a position renderer. We could skip this step and use a default renderer, but then all widgets would simply render below each pther. So to lay out the widgets in a grid, create the file `views/position-grid.php`:

```
<?php foreach ($widgets as $widget) : ?>
<div class="uk-width-medium-1-<?= count($widgets) ?>">

    <div class="uk-panel">

        <h3 class="uk-panel-title"><?= $widget->title ?></h3>

        <?= $widget->get('result') ?>

    </div>

</div>
<?php endforeach ?>
```

#### Step 3

To render the new position in the theme's markup, we need to add it to the `views/template.php` file, after the closing `</nav>` tag:

```
<?php if ($view->position()->exists('top')) : ?>
<div id="top" class="tm-top">
    <div class="uk-container uk-container-center">

        <section class="uk-grid uk-grid-match" data-uk-grid-margin>
            <?= $view->position('top', 'position-grid.php') ?>
        </section>

    </div>
</div>
<?php endif ?>
```

Now can select *Top* for any widget that you want to render in the newly created position.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-select-position.png" alt="Widget position">
    <figcaption class="uk-thumbnail-caption">Select the new position when creating a widget.</figcaption>
</figure>

### Adding position options
The example before added configuration options which applied for the whole site. We can also extend the Site Tree with configuration that only applies for a certain page. Let us add the option to apply a different background color to our new Top position.

#### Step 1

First, we need to create the file `node-theme.vue` inside the folder `app/components`. Here we add the option which will be displayed in the Pagekit administration. Settings that are stored in this file can be applied to each page separately by entering the appropriate item in the site tree and clicking the *Theme* tab.

```
<template>

    <div class="uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-top-style" class="uk-form-label">Top {{ 'Position' | trans }}</label>
            <div class="uk-form-controls">
                <select id="form-top-style" class="uk-form-width-large" v-model="node.theme.top_style">
                    <option value="uk-block-default">{{ 'Default' | trans }}</option>
                    <option value="uk-block-muted">{{ 'Muted' | trans }}</option>
                </select>
            </div>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the Site Tree. To do so, we can create a *Theme* tab in the interface by adding the following to the `node-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            priority: 90
        },

        props: ['node']

    };

    window.Site.components['node-theme'] = module.exports;

</script>
```

#### Step 3

In the chapter about theme options, we inserted the event listener to the `index.php` file in order to load the script and add the option to the Site Tree. Now we need to do the same thing for the site setting. The complete `events` section should then look like this.

```
'events' => [

    'view.system/site/admin/settings' => function ($event, $view) use ($app) {
        $view->script('site-theme', 'theme:app/bundle/site-theme.js', 'site-settings');
        $view->data('$theme', $this);
    },

    'view.system/site/admin/edit' => function ($event, $view) {
        $view->script('node-theme', 'theme:app/bundle/node-theme.js', 'site-edit');
    }

]
```

#### Step 4

The default setting for the widget position also needs to be added in the `index.php`.

```
'node' => [

    'top_style' => 'uk-block-muted'

],
```

#### Step 5

In the chapter about theme options we created the file `webpack.config.js`. Our `node-theme.vue` file also needs to be registered to be compiled into JavaScript.

```
entry: {
    "node-theme": "./app/components/node-theme.vue",
    "site-theme": "./app/components/site-theme.vue"
},
```

#### Step 6

Now you can run the command *webpack* inside the theme folder and `node-theme.vue` will be compiled into `app/bundle/node-theme.js`.

#### Step 7

Lastly, to actually render the chosen setting into the widget position, we need to add the `.uk-block` class and the style parameter to the position itself in the `template.php` file.

```
<div id="top" class="tm-top uk-block <?= $params['top_style'] ?>">
```

#### Step 8

In the site tree, you now see a _Theme_ tab when editing a page. Here you can configure the new option. This configuration only applies for this specific page.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-node.png" alt="Node options added by the theme">
    <figcaption class="uk-thumbnail-caption">Node options added by the theme.</figcaption>
</figure>

### Adding widget options
You can also add specific options to widgets themselves. In this case, we would like to provide a panel style option that can be selected for each widget.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-widget-options.png" alt="Widget options">
    <figcaption class="uk-thumbnail-caption">A theme can add any kind of options to the Widget editor.</figcaption>
</figure>

#### Step 1

First, we need to create a `app/components/widget-theme.vue` file inside the folder `app/components`. This renders the select box from which we will be able to choose the widget's style.

```
<template>

    <div class="uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-theme-panel" class="uk-form-label">{{ 'Panel Style' | trans }}</label>
            <div class="uk-form-controls">
                <select id="form-theme-panel" class="uk-form-width-large" v-model="widget.theme.panel">
                    <option value="">{{ 'None' | trans }}</option>
                    <option value="uk-panel-box">{{ 'Box' | trans }}</option>
                    <option value="uk-panel-box uk-panel-box-primary">{{ 'Box Primary' | trans }}</option>
                    <option value="uk-panel-box uk-panel-box-secondary">{{ 'Box Secondary' | trans }}</option>
                    <option value="uk-panel-header">{{ 'Header' | trans }}</option>
                </select>
            </div>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the widget administration. To do so, we can create a *Theme* tab in the interface by adding the following to the `widget-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            priority: 90
        },

        props: ['widget', 'config']

    };

    window.Widgets.components['theme'] = module.exports;

</script>
```

#### Step 3

To make the option available in the widget administration, we can create a *Theme* tab to the interface by adding the following to the `events` section in the file `index.php`.

```
'view.system/widget/edit' => function ($event, $view) {
    $view->script('widget-theme', 'theme:app/bundle/widget-theme.js', 'widget-edit');
}
```

#### Step 4

The default setting for the widget also needs to be added in the `index.php`.

```
'widget' => [

    'panel' => ''

],
```

#### Step 5

Add the `widget-theme` entry to the `webpack.config.json` file, so it will be compiled to JavaScript (don't forget to run `webpack` in the theme folder afterwards).

```
entry: {
    "node-theme": "./app/components/node-theme.vue",
    "site-theme": "./app/components/site-theme.vue",
    "widget-theme": "./app/components/widget-theme.vue"
},
```

#### Step 6

Now all we need to do is render the chosen setting into the widget in the `position-grid.php` file.

```
<div class="uk-panel <?= $widget->theme['panel'] ?>">
```

#### Step 7

When editing a widget, you will now see a _Theme_ tab on top of the editor, where you can access the _Panel Style_ configuration that we have just added.

## Overwrite system views

A theme can also customize the way how Pagekit renders static pages and blog posts. You simply overwrite system view files to apply your own layout. To do so, create corresponding folders inside your theme to mimic the original structure of the Pagekit system and put the template files there:

File                         | Original view file                       | Description
---------------------------- | ---------------------------------------- | ------------------------
`views/system/site/page.php` | `/app/system/site/views/page.php`        | Default static page view
`views/blog/post.php`        | `/packages/pagekit/blog/views/post.php`  | Blog post single view
`views/blog/posts.php`       | `/packages/pagekit/blog/views/posts.php` | Blog posts list view

To understand which variables are available in these views, look at the markup in the original view file.

## Wrapping up

In this tutorial, you have learned the basic knowledge and tools to create themes for Pagekit. Let us summarize which topics we have covered.

**Note** The [completed theme](https://github.com/pagekit/example-theme) can be found on Github.

- You are now familiar with the **file structure** of a Pagekit theme. The main entry point for configuration and custom code is the theme's `index.php`, the main template file is located at `views/template.php` inside the theme.
- We have presented an approach for a modern **front-end workflow** with tools such as LESS, Gulp and Bower. You can customize this setup to your own preferences.
- To **add JavaScript** to your theme, you can add your own code and include the script files in your template. You can also add third party libraries like UIkit and jQuery that help you to add interaction to your website.
- **Widgets and menus** are managed from the Pagekit admin area and are rendered in special positions of your theme. You have learned how to create these positions and how to change the default widget rendering.
- To make sure your theme is customizable from the admin area, you can add your own **settings screens**. These can be added to the Site Tree, to the Site settings and to the Widget editor.
- To **overwrite system views**, your theme can include custom view files to alter the way static pages and blog posts are rendered.

With these skills you are now in a position to create Pagekit themes, both for client projects and also for the Pagekit marketplace.
