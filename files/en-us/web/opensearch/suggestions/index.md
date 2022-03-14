---
title: Supporting search suggestions in search plugins
slug: Web/OpenSearch/Suggestions
tags:
  - Add-ons
  - Guide
  - OpenSearch
  - Search
  - Search plugins
  - Web
  - Web Standards
---
{{AddonSidebar}}

Firefox supports search suggestions in OpenSearch plugins; as the user types in the search bar, Firefox queries the URL specified by the search plugin to fetch live search suggestions.

Once the list has been retrieved, it's displayed in a popup box that appears under the search bar, which lets the user select a suggested search term. If the user continues to type, a new set of suggestions is requested from the search engine, and the displayed list is refreshed.

## Implementing suggestion support in the search plugin

To support search suggestions, a search plugin needs to define an extra `<Url>` element with its `type` attribute set to `"application/x-suggestions+json"`. (This means that a suggestion-supporting engine plugin will have two `<Url>` elements, the other one being the main `text/html` search URL.)

For example, the Yahoo search plugin has this `<Url>` entry:

```xml
<Url type="application/x-suggestions+json" template="http://ff.search.yahoo.com/gossip?output=fxjson&command={searchTerms}"/>
```

If the user types "fir" into the search bar, then pauses, Firefox inserts "fir" in place of `{searchTerms}` and queries that URL:

```xml
<Url type="application/x-suggestions+json" template="http://ff.search.yahoo.com/gossip?output=fxjson&command=fir"/>
```

The results are used to construct the suggestion list box.

See [Creating OpenSearch plugins for Firefox](#dead_link) to learn more about how to implement a search plugin.

## Implementing search suggestion support on the server

The majority of the work in handling search suggestions is actually implemented on the server side. If you're a web site designer, and want to support search suggestions, you need to implement support for returning the suggestions in [JavaScript Object Notation](https://json.org/) (JSON) given a search term.

When the browser wants to fetch possible matches for a search term, it then sends an HTTP GET request to the URL specified by the `<Url>` element.

Your server should then decide upon the suggestions to offer using whatever means it sees fit, and return a JSON array of results:

- query string
  - : The first element in the array is the original query string. This allows Firefox to verify that the suggestions match the current search term.

- completion list
  - : An array of suggested search terms. The array should be enclosed in square brackets. For example: `["term 1", "term 2", "term 3", "term 4"]`

- descriptions
  - : This optional element is an array of descriptions for each of the suggestions in the _completion list_. These can be any additional information the search engine might want to return to be displayed by the browser, such as the number of results available for that search.

> Descriptions are not supported in Firefox, and are ignored if any are specified.

- query URLs
  - : This optional element is an array of alternate URLs for each of the suggestions in thecompletion list . For example, if you want to offer a map link instead of just a search result page for a given suggestion, you can return an URL to a map in this array.

    If you don't specify a query URL, the default query is used based on the search described by the `<Url>` element in the search plugin's XML description.

> Query URLs are not supported in Firefox, and are ignored.
> This enhancement request - the handling of a selected suggestion, namely calling of a full specified URL as [proposed in the opensearch standard](http://web.archive.org/web/20190630001749/http://www.opensearch.org/Specifications/OpenSearch/Extensions/Suggestions/1.0#Query_URLs) - is tracked in [bug 386591](https://bugzilla.mozilla.org/show_bug.cgi?id=386591).

For example, if the search term is "fir", and you don't need to return descriptions or alternate query URLs, you might return the following JSON:

```json
["fir", ["firefox", "first choice", "mozilla firefox"]]
```

Note that in this example, only the query string and completion array are specified, leaving out the optional elements.

Your completion list can include as many suggestions as you like, although it should be kept manageable, given that the display will be updating live while the user is typing their search string. In addition, the method you use to select suggestions is entirely up to you.

> **Note:** Firefox requires that suggest requests complete within 500ms for suggestions to be displayed. If the request for the suggest URL does not return data before 500ms have elapsed, no search suggestions will be shown.
