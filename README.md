# Implementing (partly) auto-generated license notices for a frontend JavaScript bundle

This repository includes example files showing how one could deliver license and other notices for bundled frontend JavaScript code in a project with dependencies under [permissive](https://tldrlegal.com/licenses/tags/Permissive) open source licenses. The objective is to generate the required notices automatically, as far as possible, from the project code base. Where adequate notices cannot be generated automatically, this approach leaves room for a separate notice file which can be manually updated as needed. Please note that additional accommodations would probably be necessary for any dependencies under [copyleft](https://tldrlegal.com/licenses/tags/Copyleft) (also called _reciprocal_) licenses.

## Structure

The contents of `dist/` are meant to represent the published code, i.e., the JavaScript run in the browser of the end-user and other similarly accessible files. For purposes of this example, the files in `dist/` are the following:

* `bundle.js`: Represents the bundled version of the JavaScript source code in the project. Includes a comment referring the viewer to the contents of `licenses.txt`
* `licenses.txt`: Includes the auto-generated notices for the bundled code. Includes an explanation of the file contents and a reference to the file `additional-license-info.txt`. Should be accessible at the same URL directory as `bundle.js` (unless otherwise instructed in `bundle.js`)
* `additional-license-info.txt`: Includes manually generated additional legal information, if any, regarding the bundled code, e.g., copyright or license notices missing from the auto-generated notices in `licenses.txt`. Should be accessible at the same URL directory as the above files.

## Implementation

There are several ways to technically implement the above and similar end results, for example:

1. Using a bundler plugin: If you're using [Webpack](https://webpack.js.org/) as your bundler, [license-webpack-plugin](https://www.npmjs.com/package/license-webpack-plugin) might be the best choice. For users of [Browserify](http://browserify.org/), there is [browserify-licenses](https://www.npmjs.com/package/browserify-licenses). At least for now, it is left as an exercise for the reader to figure out how to configure the relevant plugin for desired results.

2. If you're not using any of the above bundlers, or if they don't work for you for one reason or another, you may have no other option than to cook up an _ad hoc_ notice generator. Currently, I'm not aware of a general tool for this purpose. Further down, there are some example specs for this.

Aside from not having to write any code of your own, there are several upsides to using a bundler plugin:

* Being closer to the bundling process, the plugin should have a better idea of _what actually goes into the bundle_. There is usually no point in generating a notice for code that never gets bundled to the end result. This would be the case e.g. for many (but not necessarily all) development dependencies in your project.

* The above-mentioned plugins also generate various warnings that may be useful for legal review purposes. If you have someone doing any kind of closer review of third-party licensing, they might be interested in knowing, for example, if a dependency is missing a `LICENSE` file and/or `license` metadata in `package.json`, or if there are indications of third-party licenses that you want to avoid altogether.

## Example specs (if no bundler plugin available)

This kind of tool should preferably be able to at least:

* Go through the project dependencies and their source code, including transitive dependencies
* For each dependency, pick up the contents of any package root level file called `LICENSE`, `LICENSE.md` or `LICENSE.txt` (case insensitive), and get the values for the following `package.json` fields: `name`, `version`, `homepage`, `repository`, `author`, `license`
* Print out a warning for the dependency if there is no `LICENSE` file or if the `license` field in `package.json` is missing or empty
* Generate a notice file in the designated location
  * The file should include separate slots for each dependency with the following information for each (where available): name and version, homepage, repository (url), download location, author, license, and contents of the license file (see `dist/licenses.txt` in this repository for one possible approach).
  * You should decide (or let the user of the tool decide) what to do with dependencies that have missing license information. Usually, it might be best to leave out and handle manually at least any dependencies that have neither valid `license` information in `package.json` nor a recognizable `LICENSE` file.
  * To allow for variations, consider enabling some kind of templating for the notice file.

### Other considerations

While a valid [SPDX license expression](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60) string in the `license` field is the correct way to specify the package license(s) in `package.json`, you still sometimes see license information given out in the following manner:

```json
"licenses": [
  {
    "type": "AFLv2.1",
    "url": "https://github.com/dojo/dojo/blob/master/LICENSE"
  },
  {
    "type": "BSD 3-Clause",
    "url": "https://github.com/dojo/dojo/blob/master/LICENSE"
  }
]
```

There are problems with this approach. First of all, the above license expressions are not valid SPDX and thus not optimal for automated analysis. Also, with a mere list of licenses, it is not clear whether the licensing status is `AFL-2.1 OR BSD-3-Clause` or `AFL-2.1 AND BSD-3-Clause`. From the point of view of the notice tool, it is of course possible to parse and build a notice even with this information, so that's always an option. Another approach would be to ignore such defective dependencies in the auto-generated notice file and create the notices for them manually.

Also, while outside the scope of this example, checking third-party licensing against SPDX-based rules is possibly something you may want to consider using in addition to mere license notice generation. [Licensee.js](https://www.npmjs.com/package/licensee) is a nice tool for that. It comes with dependency traversal built in, so it could also be a good starting point for your ad hoc notice generator.
