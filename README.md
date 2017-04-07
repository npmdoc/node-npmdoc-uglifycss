# api documentation for  [uglifycss (v0.0.25)](https://github.com/fmarcia/UglifyCSS)  [![npm package](https://img.shields.io/npm/v/npmdoc-uglifycss.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-uglifycss) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-uglifycss.svg)](https://travis-ci.org/npmdoc/node-npmdoc-uglifycss)
#### Port of YUI CSS Compressor to NodeJS

[![NPM](https://nodei.co/npm/uglifycss.png?downloads=true)](https://www.npmjs.com/package/uglifycss)

[![apidoc](https://npmdoc.github.io/node-npmdoc-uglifycss/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-uglifycss_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-uglifycss/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-uglifycss/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-uglifycss/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Franck Marcia",
        "email": "franck.marcia@gmail.com",
        "url": "https://github.com/fmarcia"
    },
    "bin": {
        "uglifycss": "./uglifycss"
    },
    "bugs": {
        "url": "https://github.com/fmarcia/UglifyCSS/issues"
    },
    "dependencies": {},
    "description": "Port of YUI CSS Compressor to NodeJS",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "bea72bf4979eacef13a302cf47b2d1af3f344197",
        "tarball": "https://registry.npmjs.org/uglifycss/-/uglifycss-0.0.25.tgz"
    },
    "gitHead": "f069004529e57023a464db170110bcfa12712b48",
    "homepage": "https://github.com/fmarcia/UglifyCSS",
    "keywords": [
        "css",
        "stylesheet",
        "uglify",
        "minify"
    ],
    "license": "MIT",
    "main": "./index.js",
    "maintainers": [
        {
            "name": "fmarcia",
            "email": "franck.marcia@gmail.com"
        }
    ],
    "name": "uglifycss",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/fmarcia/UglifyCSS.git"
    },
    "scripts": {},
    "version": "0.0.25"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module uglifycss](#apidoc.module.uglifycss)
1.  [function <span class="apidocSignatureSpan">uglifycss.</span>processFiles (filenames, options)](#apidoc.element.uglifycss.processFiles)
1.  [function <span class="apidocSignatureSpan">uglifycss.</span>processString (content, options)](#apidoc.element.uglifycss.processString)
1.  object <span class="apidocSignatureSpan">uglifycss.</span>defaultOptions



# <a name="apidoc.module.uglifycss"></a>[module uglifycss](#apidoc.module.uglifycss)

#### <a name="apidoc.element.uglifycss.processFiles"></a>[function <span class="apidocSignatureSpan">uglifycss.</span>processFiles (filenames, options)](#apidoc.element.uglifycss.processFiles)
- description and source-code
```javascript
function processFiles(filenames, options) {

    var nFiles = filenames.length,
        uglies = [],
        index,
        filename,
        content;

    options = options || defaultOptions;

    if (options.convertUrls) {
        options.target = path.resolve(process.cwd(), options.convertUrls).split(PATH_SEP);
    }

    // process files
    for (index = 0; index < nFiles; index += 1) {
        filename = filenames[index];
        try {
            content = fs.readFileSync(filename, 'utf8');
            if (content.length) {
                if (options.convertUrls) {
                    options.source = path.resolve(process.cwd(), filename).split(PATH_SEP);
                    options.source.pop();
                }
                uglies.push(processString(content, options));
            }
        } catch (e) {
            if (options.debug) {
                console.error('uglifycss: unable to process "' + filename + '"\n' + e.stack);
            } else {
                console.error('uglifycss: unable to process "' + filename + '"\n\t' + e);
            }
            process.exit(1);
        }
    }

    // return concat'd results
    return uglies.join('');
}
```
- example usage
```shell
...
Both functions return uglified css.

#### Example

'''js
var uglifycss = require('uglifycss');

var uglified = uglifycss.processFiles(
    [ 'file1', 'file2' ],
    { maxLineLen: 500, expandVars: true }
);

console.log(uglified);
'''
...
```

#### <a name="apidoc.element.uglifycss.processString"></a>[function <span class="apidocSignatureSpan">uglifycss.</span>processString (content, options)](#apidoc.element.uglifycss.processString)
- description and source-code
```javascript
function processString(content, options) {

    var startIndex,
        comments = [],
        preservedTokens = [],
        token,
        len = content.length,
        pattern,
        quote,
        rgbcolors,
        hexcolor,
        placeholder,
        val,
        i,
        c,
        line = [],
        lines = [],
        vars = {};

    options = options || defaultOptions;
    content = extractDataUrls(content, preservedTokens);
    content = convertRelativeUrls(content, options, preservedTokens);
    content = collectComments(content, comments);

    // preserve strings so their content doesn't get accidentally minified
    pattern = /("([^\\"]|\\.|\\)*")|('([^\\']|\\.|\\)*')/g;
    content = content.replace(pattern, function (token) {
        quote = token.substring(0, 1);
        token = token.slice(1, -1);
        // maybe the string contains a comment-like substring or more? put'em back then
        if (token.indexOf("___PRESERVE_CANDIDATE_COMMENT_") >= 0) {
            for (i = 0, len = comments.length; i < len; i += 1) {
                token = token.replace("___PRESERVE_CANDIDATE_COMMENT_" + i + "___", comments[i]);
            }
        }
        // minify alpha opacity in filter strings
        token = token.replace(/progid:DXImageTransform.Microsoft.Alpha\(Opacity=/gi, "alpha(opacity=");
        preservedTokens.push(token);
        return quote + ___PRESERVED_TOKEN_ + (preservedTokens.length - 1) + "___" + quote;
    });

    // strings are safe, now wrestle the comments
    for (i = 0, len = comments.length; i < len; i += 1) {

        token = comments[i];
        placeholder = "___PRESERVE_CANDIDATE_COMMENT_" + i + "___";

        // ! in the first position of the comment means preserve
        // so push to the preserved tokens keeping the !
        if (token.charAt(0) === "!") {
            if (options.cuteComments) {
                preservedTokens.push(token.substring(1));
            } else if (options.uglyComments) {
                preservedTokens.push(token.substring(1).replace(/[\r\n]/g, ''));
            } else {
                preservedTokens.push(token);
            }
            content = content.replace(placeholder, ___PRESERVED_TOKEN_ + (preservedTokens.length - 1) + "___");
            continue;
        }

        // \ in the last position looks like hack for Mac/IE5
        // shorten that to<span class="apidocCodeCommentSpan"> /*\*/ and the next one to /**/
</span>        if (token.charAt(token.length - 1) === "\\") {
            preservedTokens.push("\\");
            content = content.replace(placeholder, ___PRESERVED_TOKEN_ + (preservedTokens.length - 1) + "___");
            i = i + 1; // attn: advancing the loop
            preservedTokens.push("");
            content = content.replace(
                "___PRESERVE_CANDIDATE_COMMENT_" + i + "___",
                ___PRESERVED_TOKEN_ + (preservedTokens.length - 1) + "___"
            );
            continue;
        }

        // keep empty comments after child selectors (IE7 hack)
        // e.g. html >/**/ body
        if (token.length === 0) {
            startIndex = content.indexOf(placeholder);
            if (startIndex > 2) {
                if (content.charAt(startIndex - 3) === '>') {
                    preservedTokens.push("");
                    content = content.replace(placeholder, ___PRESERVED_TOKEN_ + (preservedTokens.length - 1) + "___");
                }
            }
        }

        // in all other cases kill the comment
        content = content.replace("/*" + placeholder + "*/", "");
    }

    if (options.expandVars) {
        // parse simple @variables blocks and remove them
        pattern = /@variables\s*\{\s*([^\}]+)\s*\}/g;
        content = content.replace(pattern, function (ignore, f1) {
            pattern = /\s*([a-z0-9\-]+)\s*:\s*([^;\}]+)\s*/gi;
            f1.replace(pattern, function (ignore, f1, f2) {
                if (f1 && f2) {
                vars[f1] = f2;
                }
                return '';
            });
            return '';
        });

        // replace var(x) with the value of x
        pattern = /va ...
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
