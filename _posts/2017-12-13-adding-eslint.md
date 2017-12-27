---
layout: post
title: Adding Eslint
date: 2017-12-13 12:00:00 -0700
icon: code
tags: nodejs
---

# Adding Eslint

With this lesson our environment will be ready to start any nodejs application that we want, It includes:

- Template engine
- Style compiler
- Logger
- Grammar checkr (current lesson)

[Eslint](https://eslint.org/) is an utility that help us to detect problems in our code before running it and get a better quality of our code. 

Javascript is an interpreted language and in some cases is not compiled, so the errors came at runtime and sometimes is difficult to find them. But with the linter, we find errors like unused variables or wrong names in variables, in current nodejs and web development, is a must to include it in our projects.

Here is an example of eslint running

```bash
$ npm run eslint

> nodejscourse@1.0.0 eslint /Users/rgonzalezibarra/Documents/nodejscourse
> eslint .


/Users/rgonzalezibarra/Documents/nodejscourse/index.js
  7:54  error  'vaca' is defined but never used  no-unused-vars

✖ 1 problem (1 error, 0 warnings)
```

It's highly configurable and easy to use, and we can easily integrate it with our VSCode editor as well as other editors like Atom.

## Adding eslint

The first thing that we need to do is to install eslint dependency:

```bash
$ npm install --save eslint
```

## Creating eslint configuration

At the top of our project, we need to include a file called **.eslintrc**. With the next content:

```json
{
	"extends": [
		"eslint:recommended"
	],
	"plugins": [],
	"globals": {
	},
	"parserOptions": {
		"ecmaVersion": 6,
		"sourceType": "module",
		"ecmaFeatures": {
			"jsx": true
		}
	},
	"env": {
		"es6": true,
		"node": true
	},
	"rules": {
		"indent": ["error", 2, { "SwitchCase": 1 }],
		"linebreak-style": ["error", "unix"],
		"quotes": ["error", "single"],
		"semi": ["error", "always"],
		"eqeqeq": ["error", "always"],
		"no-console": "error",
		"eol-last": "error",
		"no-debugger": "error",
		"block-scoped-var": "error",
		"no-div-regex": "error",
		"no-eval": "error",
		"array-bracket-spacing": "error",
		"brace-style": "error",
		"comma-spacing": ["error", {"before": false, "after": true}],
		"block-spacing": "error",
		"space-before-blocks": ["error", {"functions": "always", "keywords": "always"}],
		"max-len": ["error", {"code": 120, "ignoreUrls": true}],
		"key-spacing": ["error", {"beforeColon": false, "afterColon": true}],
		"no-var": "error",
		"no-unused-vars": "error",
		"no-underscore-dangle": ["error", { "allow": ["__REDUX_DEVTOOLS_EXTENSION__"] }],
		"no-trailing-spaces": "error",
		"no-alert": "error",
		"no-lone-blocks": "error",
		"jsx-quotes": "error",
		"no-multiple-empty-lines": ["error", {"max": 1, "maxEOF": 0}]
	}
}
```

this are the rules that I usually use, feel free to modify them according to your necessities.

All the rules are defined here https://eslint.org/docs/rules/ .

## Adding task in package.json

To run eslint we need to add a npm task in the package.json file:

```json
  "scripts": {
    "test": "test",
    "dev-server": "nodemon index.js",
    "eslint": "eslint ."
  },
```

Now we can execute:

```bash
$ npm run eslint
```

## Plugin for VSCode

This is the plugin that I use for VScode [Eslint-plugin](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint). It automatically detects our eslint configuration and show the errors directly over the code, so we don't need to run eslint task all the time.

## Ignoring files

We can create a **.eslintignore** file to prevent to lint some files that we know are fine, but we trust in its content or is a library. 

This files works in the same way as the **.gitignore** file:  

- Full path:  src/folder/file.js
- Folder:     directory/
- From root:  /directory/
