# Get entities

## Summary

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [By ids](#by-ids)
  - [Get many entities by ids](#get-many-entities-by-ids)
  - [By id and revision](#by-id-and-revision)
- [By Sitelinks](#by-sitelinks)
- [Examples](#examples)
  - [A little Promises workflow demo](#a-little-promises-workflow-demo)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## By ids
*associated Wikibase API doc: [wbgetentities](https://www.wikidata.org/w/api.php?action=help&modules=wbgetentities)*

on the same pattern

```js
const ids = 'Q571' // could also be several ids as an array: [ 'Q1', 'Q5', 'Q571' ]
const languages = [ 'en', 'fr', 'de' ] // returns all languages if not specified
const props = [ 'info', 'claims' ] // returns all data if not specified
const format = 'xml' // defaults to json
const redirections = false // defaults to true
const url = wbk.getEntities(ids, languages, props, format, redirections)
```

props being wikidata entities' properties: info, sitelinks, labels, descriptions, claims.

ids, languages, props can get either one single value as a string or several values in a array


And Again, this can also be passed as an object:
```js
const url = wbk.getEntities({
  ids: [ 'Q1', 'Q5', 'Q571' ],
  languages: [ 'en', 'fr', 'de' ], // returns all languages if not specified
  props: [ 'info', 'claims' ], // returns all data if not specified
  format: 'xml', // defaults to json
  redirections: false // defaults to true
})
```

### Get many entities by ids
Above 50 ids, `wbk.getEntities` will warn you that your request won't be fully fullfiled by Wikidata API due to its limitations policy.
You can use `wbk.getManyEntities` instead to generate several request urls to work around this limitation:

The arguments API is the same as getEntities:
```js
const urls = wbk.getManyEntities([ 'Q1', 'Q2', 'Q3', ..., 'Q123' ])
// or
const urls = wbk.getManyEntities([ 'Q1', 'Q2', 'Q3', ..., 'Q123' ], [ 'en', 'fr', 'de' ], [ 'info', 'claims' ], 'json', false)
// or
const urls = wbk.getManyEntities({
  ids: [ 'Q1', 'Q2', 'Q3', ..., 'Q123' ],
  languages: [ 'en', 'fr', 'de' ],
  props: [ 'info', 'claims' ],
  format: 'json',
  redirections: false // defaults to true
})
```
but it returns an array of urls instead.

:warning: This limitation policy was probably there for a reason, right? This should be the exception, make sure to set an interval between your requests (500ms, 1s?), and if you really need a lot of entities, consider using [dumps](https://www.wikidata.org/wiki/Wikidata:Database_download#JSON_dumps_.28recommended.29): there are [great tools](https://github.com/maxlath/wikidata-filter) to work with those too! ;)


### By id and revision
At some point in your love story with Wikidata, you might end up needing to access data from an entity at a different revision than the current one, so there's a function for that:
```js
// multiple args interface
const url = wbk.getEntityRevision('Q3548931', 775908525)
// OR object interface
const url = wbk.getEntityRevision({ id: 'Q3548931', revision: 775908525 })
```

The revision id can be obtained using [`wbk.getRevisions`](https://github.com/maxlath/wikidata-sdk/blob/main/docs/get_revisions.md#get-revisions): look for the `revid`.

The returned data can then be [simplified](https://github.com/maxlath/wikidata-sdk/blob/main/docs/simplify_entities_data.md#simplify-entities-data) as for normal entity data.

## By Sitelinks
*associated Wikibase API doc: [wbgetentities](https://www.wikidata.org/w/api.php?action=help&modules=wbgetentities)*

Typical usecase: you're working with a list of Wikipedia articles in a given language and would like to move to Wikidata for all the awesomeness it provides: you can fetch the entities on Wikidata using the Wikipedia articles titles:
```js
const url = wbk.getEntitiesFromSitelinks('Hamburg')
//=> 'https://www.wikidata.org/w/api.php?action=wbgetentities&titles=Hamburg&sites=enwiki&format=json'

const url = wbk.getEntitiesFromSitelinks([ 'Hamburg', 'London', 'Lisbon' ])
// => 'https://www.wikidata.org/w/api.php?action=wbgetentities&titles=Hamburg%7CLyon%7CBerlin&sites=enwiki&format=json'
```

By default, it looks in the English Wikipedia, but we can change that:
```js
const url = wbk.getEntitiesFromSitelinks([ 'Hambourg', 'Londres', 'Lisbonne' ], 'frwiki')
// => 'https://www.wikidata.org/w/api.php?action=wbgetentities&titles=Hamburg%7CLyon%7CBerlin&sites=enwiki&format=json'
```
You can customize different things by passing more arguments
```js
const titles = 'Hamburg'
const sites = 'enwikivoyage'
const languages = [ 'en', 'fr', 'de' ] // those are the languages in which we would like the entities data
const props = [ 'info', 'claims' ] // default: info, sitelinks, aliases, labels, descriptions, claims, datatype
const format = 'json' // default: json
const redirections = false // default: true
const url = wbk.getEntitiesFromSitelinks(titles, sites, languages, props, format, redirections)
```
or by using the object interface:
```js
const url = wbk.getEntitiesFromSitelinks({
  titles: 'Hamburg',
  sites: 'enwikivoyage',
  languages: [ 'en', 'fr', 'de' ],
  props: [ 'info', 'claims' ],
  format: 'json', // defaults to json
  redirections: false // defaults to true
})
```

## Examples

### A little [Promises](https://www.promisejs.org) workflow demo
That's how I love to work :)

To keep things simple, `fetch` is the browser standard [`fetch` function](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch) or its NodeJS equivalent:
```js
const fetch = require('node-fetch')
```

```js
const url = wbk.getEntities({
  ids: [ 'Q647268', 'Q771376', 'Q860998', 'Q965704' ],
  language: user.language
})

fetch(url)
.then(response => response.json())
// Turns the response in an array of simplified entities
.then(wbk.parse.wb.entities)
.then(entities => // do your thing with those entities data)
```
