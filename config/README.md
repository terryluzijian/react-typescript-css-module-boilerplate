# Problems faced during upgrading from Webpack 3 to 4

_IMPORTANT_: Re-install dependencies by deleting `node_modules` and `package-lock.json` if there is still any error even if the proper procedures are followed.

## Problem 1: HtmlWebpackPlugin error

```
Plugin could not be registered at 'html-webpack-plugin-before-html-processing'. Hook was not found.
BREAKING CHANGE:
There need to exist a hook at 'this.hooks'. To create a compatiblity layer for this hook, hook into 'this._pluginCompat'.
```

This problem could be solved by changing the order of plugins in webpack config:

```js
[
    ...,
    new InterpolateHtmlPlugin(env.raw),
    new HtmlWebpackPlugin({
        inject: true,
        template: paths.appHtml,
    }),
    ...
]
```

to

```js
[
    ...,
    new HtmlWebpackPlugin({
            inject: true,
            template: paths.appHtml,
        }),
    new InterpolateHtmlPlugin(env.raw),
    ...
]
```

## Problem 2: 'Fail to compile: webpack is not a function' error

Change the line in start.js file under config folder from:

```js
const compiler = createCompiler(webpack, config, appName, urls, useYarn);
```

to:

```js
const compiler = createCompiler({ webpack, config, appName, urls, useYarn });
```

I upgraded react-dev-utils to the latest version as well.

## Problem 3: 'this.htmlWebpackPlugin.getHooks is not a function' error

Install the latest version of HTML webpack plugin by running:

`npm i html-webpack-plugin@next`

and change the config file from:

```js
[
    ...,
    new HtmlWebpackPlugin({
            inject: true,
            template: paths.appHtml,
        }),
    new InterpolateHtmlPlugin(env.raw),
    ...
]
```

to

```js
[
    ...,
    new HtmlWebpackPlugin({
            inject: true,
            template: paths.appHtml,
        }),
    new InterpolateHtmlPlugin(HtmlWebpackPlugin, env.raw),
    ...
]
```

## Problem 4: 'Chunk.entrypoints: Use Chunks.groupsIterable and filter by instanceof Entrypoint instead' error

Install the latest extract-text-webpack-plugin by running:

`npm install extract-text-webpack-plugin@next`

## Problem 5: 'Error: Cannot find module 'css-loader/locals'' error

Downgrade css-loader to version ^1.0.1 by running:

`npm install css-loader@^1.0.1`

## Problem 6: 'The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.' warning

Add the following lines in config files:

```js
module.exports = {
    mode: 'development' // or 'production',
    ...
};
```

## _(For Production)_ Problem 7: 'Path variable [contenthash:8] not implemented in this context: static/css/[name].[contenthash:8].css' error

Change the following line in webpack production config:

```js
const cssFilename = 'static/css/[name].[contenthash:8].css';
```

to

```js
const cssFilename = 'static/css/[name].[hash].css';
```

## _(For Production)_ Problem 7: UglifyJs error

```
static/js/main.0349eac5.js from UglifyJs
`safari10` is not a supported option
```

Simply remove the following lines in webpack production config:

```js
[
    ...,
    // Minify the code.
    new UglifyJsPlugin({
        uglifyOptions: {
            parse: {
                // we want uglify-js to parse ecma 8 code. However we want it to output
                // ecma 5 compliant code, to avoid issues with older browsers, this is
                // whey we put `ecma: 5` to the compress and output section
                // https://github.com/facebook/create-react-app/pull/4234
                ecma: 8,
            },
            compress: {
                ecma: 5,
                warnings: false,
                // Disabled because of an issue with Uglify breaking seemingly valid code:
                // https://github.com/facebook/create-react-app/issues/2376
                // Pending further investigation:
                // https://github.com/mishoo/UglifyJS2/issues/2011
                comparisons: false,
                // Don't inline functions with arguments, to avoid name collisions:
                // https://github.com/mishoo/UglifyJS2/issues/2842
                inline: 1,
            },
            mangle: {
                safari10: true,
            },
            output: {
                ecma: 5,
                comments: false,
                // Turned on because emoji and regex is not minified properly using default
                // https://github.com/facebook/create-react-app/issues/2488
                ascii_only: true,
            },
        },
        // Use multi-process parallel running to improve the build speed
        // Default number of concurrent runs: os.cpus().length - 1
        parallel: true,
        // Enable file caching
        cache: true,
        sourceMap: shouldUseSourceMap,
    }), // Note: this won't work without ExtractTextPlugin.extract(..) in `loaders`.
    ...
]
```

Explanation found in Stack Overflow:

> Setting `mode` to `production` in Webpack v4 should do enough optimizations, so there's no need to specifically require the Uglify plugin. Try remove `uglifyjs-webpack-plugin` and there's also no need for passing the `-p` flag for the `build` script.
