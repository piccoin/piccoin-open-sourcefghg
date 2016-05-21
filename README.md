# piccoin-open-sourcefghgMigrate hubot's translate to it's own package
 master  v0.2.0 v0.1.0
0 parents commit f46095092d42080345f5388eb1013db9c8700f15 @technicalpickles technicalpickles committed on 27 Aug 2014
Unified Split
Showing  11 changed files  with 299 additions and 0 deletions.
12  .editorconfig
@@ -0,0 +1,12 @@
+# http://editorconfig.org
+root = true
+
+[*]
+indent_style = space
+indent_size = 2
+charset = utf-8
+trim_trailing_whitespace = true
+insert_final_newline = true
+
+[*.md]
+trim_trailing_whitespace = false
1  .gitignore
@@ -0,0 +1 @@
+node_modules 
6  .travis.yml
@@ -0,0 +1,6 @@
+language: node_js
+node_js:
+  - "0.11"
+  - "0.10"
+notifications:
+  email: false 
40  Gruntfile.js
@@ -0,0 +1,40 @@
+'use strict';
+
+module.exports = function(grunt) {
+
+  grunt.loadNpmTasks('grunt-mocha-test');
+  grunt.loadNpmTasks('grunt-release');
+
+  grunt.initConfig({
+    mochaTest: {
+      test: {
+        options: {
+          reporter: 'spec',
+          require: 'coffee-script'
+        },
+        src: ['test/**/*.coffee']
+      }
+    },
+    release: {
+      options: {
+        tagName: 'v<%= version %>',
+        commitMessage: 'Prepared to release <%= version %>.'
+      }
+    },
+    watch: {
+      files: ['Gruntfile.js', 'test/**/*.coffee'],
+      tasks: ['test']
+    }
+  });
+
+  grunt.event.on('watch', function(action, filepath, target) {
+    grunt.log.writeln(target + ': ' + filepath + ' has ' + action);
+  });
+
+  // load all grunt tasks
+  require('matchdep').filterDev('grunt-*').forEach(grunt.loadNpmTasks);
+
+  grunt.registerTask('test', ['mochaTest']);
+  grunt.registerTask('test:watch', ['watch']);
+  grunt.registerTask('default', ['test']);
+};
28  README.md
@@ -0,0 +1,28 @@
+# hubot-google-translate
+
+Allows Hubot to know many languages using Google Translate
+
+See [`src/google-translate.coffee`](src/google-translate.coffee) for full documentation.
+
+## Installation
+
+In hubot project repo, run:
+
+`npm install hubot-google-translate --save`
+
+Then add **hubot-google-translate** to your `external-scripts.json`:
+
+```json
+[
+  "hubot-google-translate"
+]
+```
+
+## Sample Interaction
+
+```
+user> hubot translate me bienvenu
+hubot> " bienvenu" is Turkish for " Bienvenu "
+user> hubot translate me from french into english bienvenu
+hubot> The French " bienvenu" translates as " Welcome " in English
+```
12  index.coffee
@@ -0,0 +1,12 @@
+fs = require 'fs'
+path = require 'path'
+
+module.exports = (robot, scripts) ->
+  scriptsPath = path.resolve(__dirname, 'src')
+  fs.exists scriptsPath, (exists) ->
+    if exists
+      for script in fs.readdirSync(scriptsPath)
+        if scripts? and '*' not in scripts
+          robot.loadFile(scriptsPath, script) if script in scripts
+        else
+          robot.loadFile(scriptsPath, script) 
43  package.json
@@ -0,0 +1,43 @@
+{
+  "name": "hubot-google-translate",
+  "description": "Allows Hubot to know many languages using Google Translate",
+  "version": "0.0.0",
+  "author": "Josh Nichols <technicalpickles@github.com>",
+  "license": "MIT",
+
+  "keywords": "hubot, hubot-scripts, translate, translation, google translate",
+
+  "repository": {
+    "type": "git",
+    "url": "git://github.com/hubot-scripts/hubot-google-translate.git"
+  },
+
+  "bugs": {
+    "url": "https://github.com/hubot-scripts/hubot-google-translate/issues"
+  },
+
+  "dependencies": {
+  },
+
+  "peerDependencies": {
+    "hubot": "2.x"
+  },
+
+  "devDependencies": {
+    "hubot": "2.x",
+    "mocha": "*",
+    "chai": "*",
+    "sinon-chai": "*",
+    "sinon": "*",
+    "grunt-mocha-test": "~0.7.0",
+    "grunt-release": "~0.6.0",
+    "matchdep": "~0.1.2",
+    "grunt-contrib-watch": "~0.5.3"
+  },
+
+  "main": "index.coffee",
+
+  "scripts": {
+    "test": "grunt test"
+  }
+}
18  script/bootstrap
@@ -0,0 +1,18 @@
+#!/bin/bash
+
+# Make sure everything is development forever
+export NODE_ENV=development
+
+# Load environment specific environment variables
+if [ -f .env ]; then 
+  source .env
+fi
+
+if [ -f .env.${NODE_ENV} ]; then
+  source .env.${NODE_ENV}
+fi
+
+npm install
+
+# Make sure coffee and mocha are on the path
+export PATH="node_modules/.bin:$PATH" 
6  script/test
@@ -0,0 +1,6 @@
+#!/bin/bash
+
+# bootstrap environment
+source script/bootstrap
+
+mocha --compilers coffee:coffee-script   
114  src/google-translate.coffee
@@ -0,0 +1,114 @@
+# Description:
+#   Allows Hubot to know many languages.
+#
+# Commands:
+#   hubot translate me <phrase> - Searches for a translation for the <phrase> and then prints that bad boy out.
+#   hubot translate me from <source> into <target> <phrase> - Translates <phrase> from <source> into <target>. Both <source> and <target> are optional
+
+languages =
+  "af": "Afrikaans",
+  "sq": "Albanian",
+  "ar": "Arabic",
+  "az": "Azerbaijani",
+  "eu": "Basque",
+  "bn": "Bengali",
+  "be": "Belarusian",
+  "bg": "Bulgarian",
+  "ca": "Catalan",
+  "zh-CN": "Simplified Chinese",
+  "zh-TW": "Traditional Chinese",
+  "hr": "Croatian",
+  "cs": "Czech",
+  "da": "Danish",
+  "nl": "Dutch",
+  "en": "English",
+  "eo": "Esperanto",
+  "et": "Estonian",
+  "tl": "Filipino",
+  "fi": "Finnish",
+  "fr": "French",
+  "gl": "Galician",
+  "ka": "Georgian",
+  "de": "German",
+  "el": "Greek",
+  "gu": "Gujarati",
+  "ht": "Haitian Creole",
+  "iw": "Hebrew",
+  "hi": "Hindi",
+  "hu": "Hungarian",
+  "is": "Icelandic",
+  "id": "Indonesian",
+  "ga": "Irish",
+  "it": "Italian",
+  "ja": "Japanese",
+  "kn": "Kannada",
+  "ko": "Korean",
+  "la": "Latin",
+  "lv": "Latvian",
+  "lt": "Lithuanian",
+  "mk": "Macedonian",
+  "ms": "Malay",
+  "mt": "Maltese",
+  "no": "Norwegian",
+  "fa": "Persian",
+  "pl": "Polish",
+  "pt": "Portuguese",
+  "ro": "Romanian",
+  "ru": "Russian",
+  "sr": "Serbian",
+  "sk": "Slovak",
+  "sl": "Slovenian",
+  "es": "Spanish",
+  "sw": "Swahili",
+  "sv": "Swedish",
+  "ta": "Tamil",
+  "te": "Telugu",
+  "th": "Thai",
+  "tr": "Turkish",
+  "uk": "Ukrainian",
+  "ur": "Urdu",
+  "vi": "Vietnamese",
+  "cy": "Welsh",
+  "yi": "Yiddish"
+
+getCode = (language,languages) ->
+  for code, lang of languages
+      return code if lang.toLowerCase() is language.toLowerCase()
+
+module.exports = (robot) ->
+  language_choices = (language for _, language of languages).sort().join('|')
+  pattern = new RegExp('translate(?: me)?' +
+                       "(?: from (#{language_choices}))?" +
+                       "(?: (?:in)?to (#{language_choices}))?" +
+                       '(.*)', 'i')
+  robot.respond pattern, (msg) ->
+    term   = "\"#{msg.match[3]}\""
+    origin = if msg.match[1] isnt undefined then getCode(msg.match[1], languages) else 'auto'
+    target = if msg.match[2] isnt undefined then getCode(msg.match[2], languages) else 'en'
+    
+    msg.http("https://translate.google.com/translate_a/t")
+      .query({
+        client: 't'
+        hl: 'en'
+        multires: 1
+        sc: 1
+        sl: origin
+        ssel: 0
+        tl: target
+        tsel: 0
+        uptl: "en"
+        text: term
+      })
+      .header('User-Agent', 'Mozilla/5.0')
+      .get() (err, res, body) ->
+        data   = body
+        if data.length > 4 and data[0] == '['
+          parsed = eval(data)
+          language =languages[parsed[2]]
+          parsed = parsed[0] and parsed[0][0] and parsed[0][0][0]
+          if parsed
+            if msg.match[2] is undefined
+              msg.send "#{term} is #{language} for #{parsed}"
+            else
+              msg.send "The #{language} #{term} translates as #{parsed} in #{languages[target]}"
+
19  test/google-translate-test.coffee
@@ -0,0 +1,19 @@
+chai = require 'chai'
+sinon = require 'sinon'
+chai.use require 'sinon-chai'
+
+expect = chai.expect
+
+describe 'google-translate', ->
+  beforeEach ->
+    @robot =
+      respond: sinon.spy()
+      hear: sinon.spy()
+
+    require('../src/google-translate')(@robot)
+
+  it 'registers a respond listener', ->
+    expect(@robot.respond).to.have.been.calledWith(/hello/)
+
+  it 'registers a hear listener', ->
+    expect(@robot.hear).to.have.been.calledWith(/orly/)
