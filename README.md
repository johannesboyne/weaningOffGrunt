# Weaning Off Grunt

Many of you like [Grunt](http://gruntjs.com/), and that is OK! Personally I really like how [npm](http://npmjs.org) works thus most of the time I just don't have to use Grunt.

Substack put it quite right in his blog post [task automation with npm run](http://www.substack.net/task_automation_with_npm_run).

I want to show you how I am dealing with `task automation` and setup processes:

```
$ pkginit
$ autopkginit autoConf.json
$ npm install
```

So, what does each of the commands do?

### pkginit

`pkginit` is a great tool by [Substack](http://github.com/substack), it creates a new package json for you. It is highly customizable by configuration files. E.g. here is mine:

```json
{
  "name" : basename.replace(/^node-/, ''),
  "version" : "0.0.0",
  "description" : prompt('description, what are you working on?', ''),
  "main" : prompt('entry point', 'index.js'),
  "bin" : function (cb) {
    var path = require('path');
    var fs = require('fs');
    var exists = fs.exists || path.exists;
    exists('bin/cmd.js', function (ex) {
      var bin = {};
      if (ex) bin[basename.replace(/^node-/, '')] = 'bin/cmd.js';
      cb(null, bin);
    });
  },
  "directories" : {
    "example" : "example",
    "test" : "test"
    },
  "dependencies" : {},
  "devDependencies" : {
    "tap" : "~0.2.5"
  },
  "scripts" : {
    "test" : "tap test/*.js"
  },
  "repository" : {
    "type" : "git",
    "url" : "git://github.com/johannesboyne/" + basename + ".git"
  },
  "homepage" : "https://github.com/johannesboyne/" + basename,
  "keywords" : prompt(function (s) { return s.split(/\s+/) }),
  "author" : {
    "name" : "Johannes Boyne",
    "email" : "johannes@boyne.de"
  },
  "license" : "MIT",
  "engine" : { "node" : ">=0.6" }
}
```

It looks more complex as it really is, the handy function `pkginit add default yourDefault.json` enables you to reuse config-json files.

### autopkginit autoConf.json

[autopkginit](https://github.com/johannesboyne/autopkginit) is a tool, written by myself to automate / standardize the creation of `package.json` files with task automation enabled. Like in Substack's blog post [task automation with npm run](http://www.substack.net/task_automation_with_npm_run). You have to pass a config-json to `autopkginit` as well, and it can look like:

```json
{
  "do you want to use browserify? (Y/n) ": {
    "dependencies": {
      "browserify": "~2.35.2",
      "uglifyjs": "~2.3.6"
    },
    "scripts": {
      "build-js": "browserify browser/main.js | uglifyjs -mc > static/bundle.js"
    },
    "default": true
  },
  "do you want to use a watcher for the JS files? (Y/n) ": {
    "devDependencies": {
      "watchify": "~0.1.0"
    },
    "scripts": {
      "watch-js": "watchify browser/main.js -o static/bundle.js -dv"
    },
    "default": true
  }
}
```

I really like this configuration...may you like it too. `autopkginit` uses [answer-question-mapper](https://github.com/johannesboyne/answer-question-mapper) and [json-combinator](https://github.com/johannesboyne/json-combinator) to check what do you want to use from this json and to merge it with the package.json, created by `pkginit`. For example, if you answer like below:

```
do you want to use browserify? (Y/n) Y
do you want to use a watcher for the JS files? (Y/n) n
```

It combines the pre-initialized `package.json` by `pkginit` and parts from the `autoConf.json` to:

```json
{
  "name": "<NAME>",
  "version": "<VERSION>",
  "description": "<DESCRIPTION>",
  "main": "<START>",
  "bin": {},
  "directories": {
    "example": "example",
    "test": "test"
  },
  "dependencies": {
    "browserify": "~2.35.2",
    "uglifyjs": "~2.3.6"
  },
  "devDependencies": {
    "tap": "~0.2.5"
  },
  "scripts": {
    "test": "tap test/*.js",
    "build-js": "browserify browser/main.js | uglifyjs -mc > static/bundle.js"
  },
  "repository": {
    "type": "git",
    "url": "git://github.com/johannesboyne/<NAME>.git"
  },
  "homepage": "https://github.com/johannesboyne/<NAME>",
  "keywords": [<KEYWORDS>],
  "author": {
    "name": "Johannes Boyne",
    "email": "johannes@boyne.de"
  },
  "license": "MIT",
  "engine": {
    "node": ">=0.6"
  }
}
```

Tell me whether you like it or not, or if you have any suggestions!
