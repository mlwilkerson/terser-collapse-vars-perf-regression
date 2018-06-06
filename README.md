# Description
Reproduce performance regression of `terser --compress collapse_vars=true` on module with 1,000 var defs and complex object values.

# Steps to Reproduce

1. `yarn` or `npm install`

2. `./node_modules/.bin/terser --compress collapse_vars=true --output output.js index.js`

This will produce a tree-shaken output bundle in about 30 seconds, depending on hardware.

Uses a current version of `terser`, the performance of which--in this scenario--is similar to `uglify-es@3.3.9`, which is the package
version that webpack 4.11.0 depends upon.

3. `./node_modules/.bin/terser --compress collapse_vars=false --output output.js index.js`

This will produce a tree-shaken output bundle in <1 second, depending on hardware.

Same version of `terser`.

4. `./node_modules/.bin/uglifyjs --compress collapse_vars=true --output output.js index.js`

This will produce a tree-shaken output bundle in <1 second, depending on hardware.

This uses `uglify-js@2.8.29`, which is the package version that webpack 3.12.0 depends upon.

# Conclusions

Something has changed in the processing of the `collapse_vars` option such that the performance in this scenario has slowed down by
some 60x. I have seen real-world build times up to 80 seconds under `uglify-es@3.3.9` that would have taken 0.5 seconds under
`uglify-js@2.8.29`.

# Associated GitHub Issues:
- [terser #50](https://github.com/fabiosantoscode/terser/issues/50)
- [uglifyjs #3174](https://github.com/mishoo/UglifyJS2/issues/3174)

# Background

This repro is designed to simulate a bundle produced by webpack after importing icon objects from Font Awesome 5 icon packages.
Our icon packages each contain hundreds of var defs with object assignments where each object represents an icon's
SVG definition. Users expect to be able to import a subset of icons from multiple icon packs and have the unused icons eliminated from the
final production bundles. They also expect a fast build time.
