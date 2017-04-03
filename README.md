# api documentation for  [css (v2.2.1)](https://github.com/reworkcss/css#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-css.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-css) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-css.svg)](https://travis-ci.org/npmdoc/node-npmdoc-css)
#### CSS parser / stringifier

[![NPM](https://nodei.co/npm/css.png?downloads=true)](https://www.npmjs.com/package/css)

[![apidoc](https://npmdoc.github.io/node-npmdoc-css/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-css_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-css/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-css/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-css/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "TJ Holowaychuk",
        "email": "tj@vision-media.ca"
    },
    "bugs": {
        "url": "https://github.com/reworkcss/css/issues"
    },
    "dependencies": {
        "inherits": "^2.0.1",
        "source-map": "^0.1.38",
        "source-map-resolve": "^0.3.0",
        "urix": "^0.1.0"
    },
    "description": "CSS parser / stringifier",
    "devDependencies": {
        "bytes": "^1.0.0",
        "matcha": "^0.5.0",
        "mocha": "^1.21.3",
        "should": "^4.0.4"
    },
    "directories": {},
    "dist": {
        "shasum": "73a4c81de85db664d4ee674f7d47085e3b2d55dc",
        "tarball": "https://registry.npmjs.org/css/-/css-2.2.1.tgz"
    },
    "files": [
        "index.js",
        "lib"
    ],
    "gitHead": "e38b6f1cc03aa36ff161a3da96b5c7510bd41ca7",
    "homepage": "https://github.com/reworkcss/css#readme",
    "keywords": [
        "css",
        "parser",
        "stringifier",
        "stylesheet"
    ],
    "license": "MIT",
    "main": "index",
    "maintainers": [
        {
            "name": "tjholowaychuk",
            "email": "tj@vision-media.ca"
        },
        {
            "name": "jonathanong",
            "email": "jonathanrichardong@gmail.com"
        },
        {
            "name": "jongleberry",
            "email": "jonathanrichardong@gmail.com"
        },
        {
            "name": "conradz",
            "email": "me@conradz.com"
        },
        {
            "name": "necolas",
            "email": "nicolasgallagher@gmail.com"
        },
        {
            "name": "anthonyshort",
            "email": "antshort@gmail.com"
        },
        {
            "name": "ianstormtaylor",
            "email": "ian@ianstormtaylor.com"
        },
        {
            "name": "moox",
            "email": "m@moox.io"
        },
        {
            "name": "clintwood",
            "email": "clint@anotherway.co.za"
        },
        {
            "name": "lydell",
            "email": "simon.lydell@gmail.com"
        },
        {
            "name": "slexaxton",
            "email": "alexsexton@gmail.com"
        }
    ],
    "name": "css",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/reworkcss/css.git"
    },
    "scripts": {
        "benchmark": "matcha",
        "test": "mocha --require should --reporter spec --bail test/*.js"
    },
    "version": "2.2.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module css](#apidoc.module.css)
1.  [function <span class="apidocSignatureSpan">css.</span>parse (css, options)](#apidoc.element.css.parse)
1.  [function <span class="apidocSignatureSpan">css.</span>stringify (node, options)](#apidoc.element.css.stringify)



# <a name="apidoc.module.css"></a>[module css](#apidoc.module.css)

#### <a name="apidoc.element.css.parse"></a>[function <span class="apidocSignatureSpan">css.</span>parse (css, options)](#apidoc.element.css.parse)
- description and source-code
```javascript
parse = function (css, options){
  options = options || {};

<span class="apidocCodeCommentSpan">  /**
   * Positional.
   */
</span>
  var lineno = 1;
  var column = 1;

  /**
   * Update lineno and column based on 'str'.
   */

  function updatePosition(str) {
    var lines = str.match(/\n/g);
    if (lines) lineno += lines.length;
    var i = str.lastIndexOf('\n');
    column = ~i ? str.length - i : column + str.length;
  }

  /**
   * Mark position and patch 'node.position'.
   */

  function position() {
    var start = { line: lineno, column: column };
    return function(node){
      node.position = new Position(start);
      whitespace();
      return node;
    };
  }

  /**
   * Store position information for a node
   */

  function Position(start) {
    this.start = start;
    this.end = { line: lineno, column: column };
    this.source = options.source;
  }

  /**
   * Non-enumerable source string
   */

  Position.prototype.content = css;

  /**
   * Error 'msg'.
   */

  var errorsList = [];

  function error(msg) {
    var err = new Error(options.source + ':' + lineno + ':' + column + ': ' + msg);
    err.reason = msg;
    err.filename = options.source;
    err.line = lineno;
    err.column = column;
    err.source = css;

    if (options.silent) {
      errorsList.push(err);
    } else {
      throw err;
    }
  }

  /**
   * Parse stylesheet.
   */

  function stylesheet() {
    var rulesList = rules();

    return {
      type: 'stylesheet',
      stylesheet: {
        rules: rulesList,
        parsingErrors: errorsList
      }
    };
  }

  /**
   * Opening brace.
   */

  function open() {
    return match(/^{\s*/);
  }

  /**
   * Closing brace.
   */

  function close() {
    return match(/^}/);
  }

  /**
   * Parse ruleset.
   */

  function rules() {
    var node;
    var rules = [];
    whitespace();
    comments(rules);
    while (css.length && css.charAt(0) != '}' && (node = atrule() || rule())) {
      if (node !== false) {
        rules.push(node);
        comments(rules);
      }
    }
    return rules;
  }

  /**
   * Match 're' and return captures.
   */

  function match(re) {
    var m = re.exec(css);
    if (!m) return;
    var str = m[0];
    updatePosition(str);
    css = css.slice(str.length);
    return m;
  }

  /**
   * Parse whitespace.
   */

  function whitespace() {
    match(/^\s*/);
  }

  /**
   * Parse comments;
   */

  function comments(rules) {
    var c;
    rules = rules || [];
    while (c = comment()) {
      if (c !== false) {
        rules.push(c);
      }
    }
    return rules;
  }

  /**
   * Parse comment.
   */

  function comment() {
    var pos = position();
    if ('/' != css.charAt(0) || '*' != css.charAt(1)) return;

    var i = 2;
    while ("" != css.charAt(i) && ('*' != css.charAt(i) || '/' != css.charAt(i + 1))) ++i;
    i += 2;

    if ("" === css.charAt(i-1)) {
      return error('End of comment missing');
    }

    var str = css.slice(2, i - 2);
    column += 2;
    updatePosition(str);
    css = css.slice(i);
    column += 2;

    return pos({
      type: 'comment',
      comment: str
    });
  }

  /**
   * Parse selector.
   */

  function selector() {
    var m = match(/^([^{]+)/);
    if (!m) return;
    /* @fix Remove all comments from selectors
     * http://ostermiller.org/findcomment.html */
    return trim(m[0])
      .replace(/\/\*([^*]|[\r\n]|(\*+([^*/]|[\r\n])))*\*\/+/g, '')
      .replace(/"(?:\\"|[^"])*"|'(?:\\'|[^'])*'/g, function(m) {
        return m.replace(/,/g, '\u200C');
      })
      .split(/\s*(?![^(]*\)),\s*/)
      .map(function(s) {
        return s.replace(/\u200C/g, ',');
      });
  }

  /**
   * Parse declaration.
   */

  function declaration() {
    var pos = position();

    // prop
    var prop = match(/^(\*?[-#\/\*\\\w]+(\[[0-9a-z_-]+\])?)\s*/);
    if (!prop) return;
    prop = trim(prop[0]);

    // :
    if (!match(/^:\s*/)) return error("property missing ':'");

    // val
    var val = match(/^((?:'(?:\\'|.)*?'|"(?:\\"|.)*?"|\([^\)]*?\)|[^};])+)/);

    var ret = pos({
      type: 'declaration',
      property: prop.replace(commentre, ''),
      value: v ...
```
- example usage
```shell
...

    $ npm install css

## Usage

'''js
var css = require('css');
var obj = css.parse('body { font-size: 12px; }', options);
css.stringify(obj, options);
'''

## API

### css.parse(code, [options])
...
```

#### <a name="apidoc.element.css.stringify"></a>[function <span class="apidocSignatureSpan">css.</span>stringify (node, options)](#apidoc.element.css.stringify)
- description and source-code
```javascript
stringify = function (node, options){
  options = options || {};

  var compiler = options.compress
    ? new Compressed(options)
    : new Identity(options);

  // source maps
  if (options.sourcemap) {
    var sourcemaps = require('./source-map-support');
    sourcemaps(compiler);

    var code = compiler.compile(node);
    compiler.applySourceMaps();

    var map = options.sourcemap === 'generator'
      ? compiler.map
      : compiler.map.toJSON();

    return { code: code, map: map };
  }

  var code = compiler.compile(node);
  return code;
}
```
- example usage
```shell
...
    $ npm install css

## Usage

'''js
var css = require('css');
var obj = css.parse('body { font-size: 12px; }', options);
css.stringify(obj, options);
'''

## API

### css.parse(code, [options])

Accepts a CSS string and returns an AST 'object'.
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
