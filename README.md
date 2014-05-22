# angular-localization

*AngularJS Localization done right.*

[![Build Status](https://travis-ci.org/doshprompt/angular-localization.svg?branch=master)](https://travis-ci.org/doshprompt/angular-localization)
[![Dependency Status](https://david-dm.org/doshprompt/angular-localization.svg?theme=shields.io)](https://david-dm.org/doshpromot/angular-localization)
[![devDependency Status](https://david-dm.org/doshprompt/angular-localization/dev-status.svg?theme=shields.io)](https://david-dm.org/angular-localization#info=devDependencies)
[![Donate to help support angular-localization development](http://img.shields.io/gittip/doshprompt.svg)](https://www.gittip.com/doshprompt/)
[![Analytics](https://ga-beacon.appspot.com/UA-51468215-1/angular-localization/README.md)](https://github.com/igrigorik/ga-beacon)

___

## Table of Contents

- [Overview](#overview)
    - [Build Dependencies](#build-dependencies)
    - [Dear Developer](#dear-developer)
- [Getting Started](#getting-started)
    - [Quick Start](#quick-start)
    - [Wiring It Up](#wiring-it-up)
    - [Module Setup](#module-setup)
    - [Localization File Formats](#localization-file-formats)
- [Usage Examples](#usage-examples)
    - [i18n directive](#i18n-directive)
        - [Localize Using the i18n attribute](#localize-using-the-i18n-attribute)
        - [Localize with Dynamic User Data](#localize-with-dynamic-user-data)
    - [i18nAttr directive](#i18nAttr-directive)
    - [locale service](#locale-service)
    - [i18n filter](#i18n-filter)
    - [Sample App](#sample-app)
- [License](#license)

## Overview

A localization module for [AngularJS](http://angularjs.org/) complete with core service and accompanying filter, directives etc.

It is based on a number of angularjs localization modules already available out there on the web,
and borrows heavily from the following list including but not limited to:

- http://dailyjs.com/2014/04/08/angular-localize/
- https://github.com/blueimp/angular-localize
- http://alicoding.com/how-to-localized-angularjs-app/

It was inspired by [Jim Lavin's AngularJS Resource Localization Service](https://github.com/lavinjj/angularjs-localizationservice)
who made an excellent first tutorial for his original version at [Coding Smackdown TV](http://codingsmackdown.tv/blog/2012/12/14/localizing-your-angularjs-app/)
which was later updated to include performance improvements [seen here](http://codingsmackdown.tv/blog/2013/04/23/localizing-your-angularjs-apps-update/).

##### Main Points of Difference

1. Simplified format of the translation file.
2. Support for parameterized messages.

##### Major Improvements

1. Parameters in the directive may be bound to `$scope` variables from the nearest parent controller.
2. HTML element tag attributes can also be natively localized.
3. Ability to configure a whole slew of things for more customizability to play nicely with your application.

### Build Dependencies

- node.js >= 0.10.x
- npm
- grunt CLI `npm install -g grunt-cli`
- bower `npm install -g bower`

### Dear Developer

##### tl;dr

```shell
$ npm install -d
$ npm test
```

## Getting Started

### Quick Start

The easiest way to install the `ngLocalize` module is via [Bower](http://bower.io/):

```shell
bower install angular-localization --save
```

You can then include `angular-localization` after including its dependencies, [angular](https://github.com/angular/bower-angular) and [angular-cookies](https://github.com/angular/bower-angular-cookies):

```html
<script src="bower_components/angular/angular.js"></script>
<script src="bower_components/angular-cookies/angular-cookies.js"></script>
<script src="bower_components/angular-localization/angular-localization.js"></script>
```

### Wiring It Up

1. Include the required libraries
2. Ensure that you inject `ngLocalize` into your app by adding it to the dependency list.

```js
angular.module('myApp', ['ngLocalize']);
```

### Module Setup

All overridable configuration options are part of `localeConf` within the `ngLocalize.Config` module that comes bundled along with this plugin and works alongside `ngLocalize`

###### basePath @ `languages`
> the folder off the root of your wb app where the resource files are located (it can also be used as a relative path)

###### defaultLocale @ `en-US`
> the locale that your app is initialized with for a new user

###### sharedDictionary @ `common`
> commonly occuring words, phrases, strings etc. are stored in this file(it is used to check whether the service itself is ready or not as it is loaded during bootstrap)

###### fileExtension @ `.lang.json`
> the extension for all resource files spanning across all languages

###### persistSelection @ `true`
> whether to save the selected language to cookies for retrieval on later uses of the app

###### cookieName @ `COOKIE_LOCALE_LANG`
> works in conjuntion with `persistSelection` and provides the cookie name for storage

###### observableAttrs @ `\^data-(?!ng-|i18n)\`
> a regular expression which is used to match which custom data attibutes may be watched by the `i18n` directive for 2-way bindings to replace in a tokenized string

###### delimiter @ `::`
> the delimiter to be used when passing params along with the string token to the service, filter etc.

```js
angular.module('myApp', [
    'ngLocalize',
    'ngLocalize.Config'
]).value('localeConf', {
    basePath: 'languages',
    defaultLocale: 'en-US',
    sharedDictionary: 'common',
    fileExtension: '.lang.json',
    persistSelection: true,
    cookieName: 'COOKIE_LOCALE_LANG',
    observableAttrs: new RegExp('^data-(?!ng-|i18n)'),
    delimiter: '::'
});
```

__NOTE:__ There is one caveat; if you want to override at least one particular option,
then you must explicitly set any other options explicitly to their default values (shown above).

##### Availability of Events Fired

Publicly exposed events may be found as part of `localeEvents` situated within the `ngLocalize.Events` module that comes pre-loaded along with this plugin and is leveraged internally by the service.

- `localeEvents.resourceUpdates`: when a new language file is pulled into memory
- `localeEvents.localeChanges`: when the user chooses a different locale

```js
angular.module('myApp', [
    'ngLocalize',
    'ngLocalize.Events'
]).controller('myAppControl', ['$scope', 'localeEvents',
    function ($scope, localeEvents) {
        $scope.$on(localeEvents.resourceUpdates, function () {
            // do something
        });
        $scope.$on(localeEvents.localeChanges, function (loc) {
            console.log('new locale chosen: ' + loc);
        });
    }
]);
```

__NOTE:__ All of these are sent by `$rootScope` so that they may be accessible to all other `$scopes` within your app.
Otherwise you could alternatively always choose to inject `$rootScope` and consume (ingest) the aforementioned events.

##### Declaration of New Languages

This plugin relies on the developer(s) to let it know of all languages currently part of the codebase.
These languages must be defined as per the [Language Culture Name specification] (http://msdn.microsoft.com/en-us/library/ee825488(v=cs.20).aspx).
It is broken up into two parts, both of which are located in the `ngLocalize.InstalledLanguages` modules separated out as `localeSupported` and `localeFallbacks`.
The first is responsible for the list of languages currently supported as the name suggests while the latter takes care of fallbacks when a particular language is present but not for the region from which the app is being opened (e.g. en-GB).
As a last resort, if none of these are valid and/or available, it will fallback to the defaultLocale configured as part of the options during initialization.

```js
angular.module('myApp', [
    'ngLocalize',
    'ngLocalize.InstalledLanguages'
])
.value('localeSupported', [
    'en-US',
    'fr-FR',
])
.value('localeFallbacks', {
    'en': 'en-US',
    'fr': 'fr-FR'
});
```

__NOTE:__ Since the plugin does not rely on auto-discovery mechanisms of any kind, and none of the sort is planned for the future,
it is possible to start creating language resource files before they must be fully integrated.
Simply leaving them out of the declaration will suffice meaning it will cause them to be ignored when deciding on which language to show up on the page(s).

### Localization File Formats

Each localization file is pretty simple. It consists of a flat JSON object:

```json
{
    "helloWorld": "Hello World",
    "yes": "Yes",
    "no": "No"
}
```

The key is used to look up the localized string, the value will be returned from the lookup.

##### Parameterized Substitutions of String Text in Tokens

As mentioned earlier, this plugin is able to handle substitutions of tokens based on arguments passed to it along with the token, provided that the token contains some indication of where it is to be placed within that said token. There are a few ways of doing this:

```json
{
    "helloWorld": "Hello %name"
}
```

```json
{
    "helloWorld": "Hello {1}",
}
```

```json
{
    "helloWorld": "Hello {firstName} {lastName}"
}
```

```json
{
    "helloWorld": "Hello %1 %2"
}
```

Please take note of the fact that multiple ordered params may also be given to it.

## Usage Examples

### i18n directive

#### Localize Using the i18n attribute

The localization key can be defined as the value of the `i18n` attribute:

```html
<any i18n="common.helloWorld"></any>
```

If the attribute value is not a valid token, then it will itself be used as the element's content.

#### Localize with Dynamic User Data

It is also possible to provide dynamic user data to the translation function.

The `i18n` directive observes all non-directive `data-*` attributes and passes them as a normalized map of key/value pairs to the translation function:

```html
<p data-i18n="common.helloWorld" data-name="{{ user.name }}"></p>
```

Whenever `user.name` is updated, it's indicator in the token `helloWorld` is subsitituted for the new value when the translation function gets called with an object, e.g. `{ name: 'Bob' }` as an argument and the element content is then updated accordingly.

### i18nAttr directive

TODO

### locale Service

The `locale` service is an equivalent to the `i18n` directive and can be used to generate localized results in situations where the directive cannot be used:

```js
angular.module('myApp', ['ngLocalize'])
    .controller('exampleCtrl', ['$scope', 'locale',
        function ($scope, locale) {
            locale.ready('common').then(function () {
                $scope.sampleText = locale.getString('common.helloWorld', {
                    firstName: 'John',
                    lastName: 'Smith'
                });
            });
        }
    ]);
```

As you can see, The `locale` service expects the localization key as the first argument and an optional {Object|Array|String} with user data as the second argument.

### i18n filter

The `i18n` filter provides the same functionality as the service.  
It can be useful in templates where the localization strings are dynamic, e.g. for error messages:

```html
<any>{{ errorMessage | i18n }}</any>
```

It is also possible to pass an object or an array with localization arguments or even a single string to the `i18n` filter:

```html
<p>{{ errorMessage | i18n:data }}</p>
```

The filter also serves a special purpose use-case targeted at maximum compatibilty with other third party plugin modules, directives, components etc. where it is not possible to provide the parameters as a separate argument.
In such a situation, the token is modified to have the substitution text params appended to it like so:

```js
angular.module('myApp', ['ngLocalize'])
    .controller('exampleCtrl', ['$scope', '$filter', 'locale',
        function ($scope, $filter, locale) {
            locale.ready('common').then(function () {
                $scope.sampleText = $filter('i18n')('common.helloWorld::["John", "Smith"]');
            });
        }
    ]);
```

Notice how we use the `delimiter` from `ngLocalize.Config`'s `localeConf` which can be changed to suit different needs and requirements as well as you see fit in your own app.
The part after the token must be in a valid JSON format as either an array, object or simple string (if it is a single param that needs to be passed in).

### Sample App

TODO

## License

The MIT License (MIT)

Copyright (c) 2014 Rahul Doshi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.