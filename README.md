# rollup-plugin-hamber

Compile Hamber components.


## Installation

```bash
npm install --save-dev hamber rollup-plugin-hamber
```

Note that we need to install Hamber as well as the plugin, as it's a 'peer dependency'.


## Usage

```js
// rollup.config.js
import * as fs from 'fs';
import hamber from 'rollup-plugin-hamber';

export default {
  input: 'src/main.js',
  output: {
    file: 'public/bundle.js',
    format: 'iife'
  },
  plugins: [
    hamber({
      // By default, all .html and .hamber files are compiled
      extensions: [ '.my-custom-extension' ],

      // You can restrict which files are compiled
      // using `include` and `exclude`
      include: 'src/components/**/*.html',

      // By default, the client-side compiler is used. You
      // can also use the server-side rendering compiler
      generate: 'ssr',

      // Optionally, preprocess components with hamber.preprocess:
      // https://github.com/hamberjs/hamber#preprocessor-options
      preprocess: {
        style: ({ content }) => {
          return transformStyles(content);
        }
      },

      // Emit CSS as "files" for other plugins to process
      emitCss: true,

      // Extract CSS into a separate file (recommended).
      // See note below
      css: function (css) {
        console.log(css.code); // the concatenated CSS
        console.log(css.map); // a sourcemap

        // creates `main.css` and `main.css.map` — pass `false`
        // as the second argument if you don't want the sourcemap
        css.write('public/main.css');
      }
    })
  ]
}
```


## Preprocessing and dependencies

If you are using the `preprocess` feature, then your callback responses may — in addition to the `code` and `map` values described in the Hamber compile docs — also optionally include a `dependencies` array. This should be the paths of additional files that the preprocessor result in some way depends upon. In Rollup 0.61+ in watch mode, any changes to these additional files will also trigger re-builds.


## `pkg.hamber`

If you're importing a component from your node_modules folder, and that component's package.json has a `"hamber"` property...

```js
{
  "name": "some-component",

  // this means 'some-component' resolves to 'some-component/src/SomeComponent.html'
  "hamber": "src/MyComponent.html"
}
```

...then this plugin will ensure that your app imports the *uncompiled* component source code. That will result in a smaller, faster app (because code is deduplicated, and shared functions get optimized quicker), and makes it less likely that you'll run into bugs caused by your app using a different version of Hamber to the component.

Conversely, if you're *publishing* a component to npm, you should ship the uncompiled source (together with the compiled distributable, for people who aren't using Hamber elsewhere in their app) and include the `"hamber"` property in your package.json.


## Extracting CSS

If your Hamber components contain `<style>` tags, by default the compiler will add JavaScript that injects those styles into the page when the component is rendered. That's not ideal, because it adds weight to your JavaScript, prevents styles from being fetched in parallel with your code, and can even cause CSP violations.

A better option is to extract the CSS into a separate file. Using the `css` option as shown above would cause a `public/main.css` file to be generated each time the bundle is built (or rebuilt, if you're using rollup-watch), with the normal scoping rules applied.

If you have other plugins processing your CSS (e.g. rollup-plugin-scss), and want your styles passed through to them to be bundled together, you can use `emitCss: true`.

Alternatively, if you're handling styles in some other way and just want to prevent the CSS being added to your JavaScript bundle, use `css: false`.


## License

MIT
