# Auto inject version using vue.js cli environment

# Requirement
* vue.js
* vue.js cli 3.0+

# webpack-auto-inject-version
For version control, we usually want a version number for identifying the development and production instance.
We can use [webpack-auto-inject-version](https://github.com/radswiat/webpack-auto-inject-version) to inject version number into desired place in the project code.

This package can do the following for us:
* Inject version number specified in `package.json` into your code with specified format
* Auto increment the version number when running build scripts by passing parameters
  * For example for version 1.2.3, 1 means major version, 2 means minor version while 3 means patch version, [more info](https://en.wikipedia.org/wiki/Software_versioning)

First we install the package:
```
npm install webpack-auto-inject-version --save-dev
```

Modify your `vue.config.js`, insert the webpack plugin config:
```
var WebpackAutoInject = require('webpack-auto-inject-version');

module.exports = {
  configureWebpack: {
    //// Something
    plugins: [
      new WebpackAutoInject({
        // specify the name of the tag in the outputed files eg
        // bundle.js: [SHORT]  Version: 0.13.36 ...
        SHORT: 'CUSTOM',
        SILENT: false,
        PACKAGE_JSON_PATH: './package.json',
        PACKAGE_JSON_INDENT: 2,
        components: {
          AutoIncreaseVersion: true,
          InjectAsComment: true,
          InjectByTag: true
        },
        componentsOptions: {
          AutoIncreaseVersion: {
            runInWatchMode: true // it will increase version with every single build!
          },
          InjectAsComment: {
            tag: 'Version: {version} - {date}',
            dateFormat: 'UTC:h:MM:ss', // change timezone: `UTC:h:MM:ss` or `GMT:h:MM:ss`
            multiLineCommentType: false, // use `/** */` instead of `//` as comment block
          },
          InjectByTag: {
            fileRegex: /\.+/,
            // regexp to find [AIV] tag inside html, if you tag contains unallowed characters you can adjust the regex
            // but also you can change [AIV] tag to anything you want
            AIVTagRegexp: /(\[AIV])(([a-zA-Z{} ,:;!()_@\-"'\\\/])+)(\[\/AIV])/g,
            dateFormat: 'UTC:h:MM:ss'
          }
        },
        LOGS_TEXT: {
          AIS_START: 'WebpackAutoInject started'
        }
      })
    ]
  }
}
```

Insert version number in your code, for example:
```
<small>
    [AIV]{version}[/AIV]
</small>
<small>
    [AIV]{date}[/AIV]
</small>
```

Modify `package.json` in order to make use of auto increment of version number when build:
```
"scripts": {
  "build": "vue-cli-service build --mode production",
  "patch": "vue-cli-service build --env.patch",
  "minor": "vue-cli-service build --env.minor",
  "major": "vue-cli-service build --env.major",
},
```

For example, if the version specified in `package.json` is `1.0.0`:
* When running `npm run build`, the version specified in `package.json` will remain `1.0.0` and the version number in your built instance will show `1.0.0`.
* When running `npm run patch`, the version specified in `package.json` will changed to `1.0.1` and the version number in your built instance will show `1.0.1`.
* When running `npm run minor`, the version specified in `package.json` will changed to `1.1.0` and the version number in your built instance will show `1.1.0`.
* When running `npm run major`, the version specified in `package.json` will changed to `1.0.0` and the version number in your built instance will show `2.0.0`.
