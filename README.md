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
import hamber from 'rollup-plugin-hamber';
import resolve from '@rollup/plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'public/bundle.js',
    format: 'iife'
  },
  plugins: [
    hamber({
      // By default, all ".hamber" files are compiled
      extensions: ['.my-custom-extension'],

      // You can restrict which files are compiled
      // using `include` and `exclude`
      include: 'src/components/**/*.hamber',

      // By default, the client-side compiler is used. You
      // can also use the server-side rendering compiler
      generate: 'ssr',

      // ensure that extra attributes are added to head
      // elements for hydration (used with generate: 'ssr')
      hydratable: true,

      // Optionally, preprocess components with hamber.preprocess:
      // https://hamberjs.web.app/docs#hamber_preprocess
      preprocess: {
        style: ({ content }) => {
          return transformStyles(content);
        }
      },

      // Emit CSS as "files" for other plugins to process. default is true
      emitCss: false,

      // You can optionally set 'customElement' to 'true' to compile
      // your components to custom elements (aka web elements)
      customElement: false,

      // Warnings are normally passed straight to Rollup. You can
      // optionally handle them here, for example to squelch
      // warnings with a particular code
      onwarn: (warning, handler) => {
        // e.g. don't warn on <marquee> elements, cos they're cool
        if (warning.code === 'a11y-distracting-elements') return;

        // let Rollup handle all other warnings normally
        handler(warning);
      }
    }),
    // see NOTICE below
    resolve({ browser: true }),
    // ...
  ]
}
```

> **NOTICE:** You will need additional Rollup plugins. <br>Alone, this plugin translates Hamber components into CSS and JavaScript files. <br>You will need to include [`@rollup/plugin-node-resolve`](https://www.npmjs.com/package/@rollup/plugin-node-resolve) – and probably [`@rollup/plugin-commonjs`](https://www.npmjs.com/package/@rollup/plugin-commonjs) – in your Rollup config.


## Preprocessing and dependencies

If you are using the `preprocess` feature, then your callback responses may — in addition to the `code` and `map` values described in the Hamber compile docs — also optionally include a `dependencies` array. This should be the paths of additional files that the preprocessor result in some way depends upon. In Rollup 0.61+ in watch mode, any changes to these additional files will also trigger re-builds.


## `pkg.hamber`

If you're importing a component from your node_modules folder, and that component's package.json has a `"hamber"` property...

```js
{
  "name": "some-component",

  // this means 'some-component' resolves to 'some-component/src/SomeComponent.hamber'
  "hamber": "src/MyComponent.hamber"
}
```

...then this plugin will ensure that your app imports the *uncompiled* component source code. That will result in a smaller, faster app (because code is deduplicated, and shared functions get optimized quicker), and makes it less likely that you'll run into bugs caused by your app using a different version of Hamber to the component.

Conversely, if you're *publishing* a component to npm, you should ship the uncompiled source (together with the compiled distributable, for people who aren't using Hamber elsewhere in their app) and include the `"hamber"` property in your package.json.

If you are publishing a package containing multiple components, you can create an `index.js` file that re-exports all the components, like this:

```js
export { default as Component1 } from './Component1.hamber';
export { default as Component2 } from './Component2.hamber';
```

and so on. Then, in `package.json`, set the `hamber` property to point to this `index.js` file.


## Extracting CSS

By default (when `emitCss: true`) the CSS styles will be emitted into a virtual file, allowing another Rollup plugin – for example, [`rollup-plugin-css-only`](https://www.npmjs.com/package/rollup-plugin-css-only), [`rollup-plugin-postcss`](https://www.npmjs.com/package/rollup-plugin-postcss), etc. – to take responsibility for the new stylesheet. In fact, emitting CSS files _requires_ that you use a Rollup plugin to handle the CSS. Otherwise, your build(s) will fail! This is because this plugin will add an `import` statement to import the emitted CSS file. It's not valid JS to import a CSS file into a JS file, but it allows the CSS to be linked to its respective JS file and is a common pattern that other Rollup CSS plugins know how to handle.

If you set `emitCss: false` and your Hamber components contain `<style>` tags, the compiler will add JavaScript that injects those styles into the page when the component is rendered. That's not the default, because it adds weight to your JavaScript, prevents styles from being fetched in parallel with your code, and can even cause CSP violations.

## License

MIT
