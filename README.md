# [gulp](https://github.com/gulpjs/gulp)-bundle-assets

| Branch | Status |
| ------ | ------ |
| master | [![Continuous Integration](https://github.com/userfrosting/gulp-uf-bundle-assets/workflows/Continuous%20Integration/badge.svg?branch=master)](https://github.com/userfrosting/gulp-uf-bundle-assets/actions?query=branch:master+workflow:"Continuous+Integration") [![codecov](https://codecov.io/gh/userfrosting/gulp-uf-bundle-assets/branch/master/graph/badge.svg)](https://codecov.io/gh/userfrosting/gulp-uf-bundle-assets/branch/master) |

Orchestrates JS and CSS bundle creation in an efficient and configurable manner.

## Install

```bash
npm i -D  @userfrosting/gulp-bundle-assets
```

## Usage

> **IMPORTANT**<br/>
> This is an ES module package targeting NodeJS `^12.17.0 || >=13.2.0`, refer to the [NodeJS ESM docs](https://nodejs.org/api/esm.html) regarding how to correctly import.
> ESM loaders like `@babel/loader` or `esm` likely won't work as expected.

```js
// gulpfile.mjs
import AssetBundler from "@userfrosting/gulp-bundle-assets";
import Gulp from "gulp";
import cleanCss from "gulp-clean-css";
import concatCss from "gulp-concat-css";
import uglify from "gulp-uglify";
import concatJs from "gulp-concat-js";

export function bundle() {
    const config = {
        bundle: {
            example: {
                scripts: [
                    "foo.js",
                    "bar.js"
                ],
                styles: [
                    "foo.css",
                    "bar.css"
                ]
            }
        }
    };
    const joiner = {
        Scripts(bundleStream, name) {
            return bundleStream
                .pipe(concatJs(name + ".js"))// example.js
                .pipe(uglify());
        },
        Styles(bundleStream, name) {
            return bundleStream
                .pipe(concatCss(name + ".css"))// example.css
                .pipe(cleanCss({
                    compatibility: "ie10"
                }));
        }
    };

    return Gulp.src("src/**")
        .pipe(new AssetBundler(config, joiner))
        .pipe(Gulp.dest("public/assets/"));
}
```

```bash
$ gulp bundle
```

## Integrating bundles into your app

The `Bundler` class exposes a `ResultsMap` property containing a Map where the key is the bundle name and value the full path of the generated file. If any transform stream after `Bundler` that changes path names then the results map will no longer be accurate, so use the built in path transforms if possible.

This approach was decided on as it provides the most efficient means to integrate bundles with any system. No need to touch the file system until its absolutely necessary, and less work to optimise the output (e.g. make a `php` file out of it to reduce IO in production by maximizing use of bytecode caching).

## API

API documentation is regenerated for every release using [API Extractor](https://www.npmjs.com/package/@microsoft/api-extractor) and [API Documenter](https://www.npmjs.com/package/@microsoft/api-documenter).
The results reside in [docs/api](./docs/api/index.md).

## History

This plugin was originally forked from [gulp-bundle-assets](https://github.com/dowjones/gulp-bundle-assets) to fix a CSS import bug.

It has since been entirely reworked to better suit the requirements of the UserFrosting's Sprinkle system and follow the Gulp plugin guidelines (namely not unnecessarily depending on it). Though TypeScript is now the preferred language the output targeted to ES2015 and uses ES Modules (via the `esm` package) to ensure source it can be easily debugged if issues are observed in the wild.

This package was previously published under `gulp-uf-bundle-assets` and as of v3 is published under `@userfrosting/gulp-bundle-assets` to assist in long-term project management.

As of v4 virtual path logic was extracted into a separate package [@userfrosting/vinyl-fs-vpath](https://github.com/userfrosting/vinyl-fs-vpath). This change enabled a significant simplification of core logic along with a significantly faster and more efficient way to support virtual path mappings.

## Release process

Generally speaking, all releases should first traverse through `alpha`, `beta`, and `rc` (release candidate) to catch missed bugs and gather feedback as appropriate. Aside from this however, there are a few steps that **MUST** always be done.

1. Make sure [`CHANGELOG.md`](./CHANGELOG.md) is up to date.
2. Update version via `npm` like `npm version 3.0.0` or `npm version patch`.
3. `npm publish`.
4. Create release on GitHub from tag made by `npm version`.

## License

[MIT](LICENSE)
