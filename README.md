# Preact Helmet
## A document head manager for Preact

> This project is a port of [react-helmet](https://github.com/nfl/react-helmet) 
to [Preact](https://preactjs.com), the 3kB lightweight React alternative. 

[![npm Version](https://img.shields.io/npm/v/preact-helmet.svg?style=flat-square)](https://www.npmjs.org/package/preact-helmet)
[![Build Status](https://img.shields.io/travis/Download/preact-helmet/master.svg?style=flat-square)](https://travis-ci.org/Download/preact-helmet)
[![Dependency Status](https://img.shields.io/david/Download/preact-helmet.svg?style=flat-square)](https://david-dm.org/Download/preact-helmet)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md#pull-requests)

This Preact component will manage all of your changes to the document head with support 
for document title, meta, link, style, script, noscript, and base tags.

Inspired by:
* [react-document-title](https://github.com/gaearon/react-document-title)
* [react-side-effect](https://github.com/gaearon/react-side-effect)
* [preact-side-effect](https://github.com/ooade/preact-side-effect) (port)
* [react-helmet](https://github.com/nfl/react-helmet) (obviously)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Examples](#examples)
- [Features](#features)
- [Installation](#installation)
- [Server Usage](#server-usage)
  - [As string output](#as-string-output)
  - [As Preact components](#as-preact-components)
- [Use Cases](#use-cases)
- [Contributing to this project](#contributing-to-this-project)
- [License](#license)
- [More Examples](#more-examples)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Examples
```javascript
import {h} from "preact"; /** @jsx h */
import Helmet from "preact-helmet";

export default function Application () {
    return (
        <div className="application">
            <Helmet title="My Title" />
            ...
        </div>
    );
};
```

```javascript
import {h} from "preact"; /** @jsx h */
import Helmet from "preact-helmet";

export default function Application () {
    return (
        <div className="application">
            <Helmet
                htmlAttributes={{lang: "en", amp: undefined}} // amp takes no value
                title="My Title"
                titleTemplate="MySite.com - %s"
                defaultTitle="My Default Title"
                titleAttributes={{itemprop: "name", lang: "en"}}
                base={{target: "_blank", href: "http://mysite.com/"}}
                meta={[
                    {name: "description", content: "Helmet application"},
                    {property: "og:type", content: "article"}
                ]}
                link={[
                    {rel: "canonical", href: "http://mysite.com/example"},
                    {rel: "apple-touch-icon", href: "http://mysite.com/img/apple-touch-icon-57x57.png"},
                    {rel: "apple-touch-icon", sizes: "72x72", href: "http://mysite.com/img/apple-touch-icon-72x72.png"}
                ]}
                script={[
                    {src: "http://include.com/pathtojs.js", type: "text/javascript"},
                    {type: "application/ld+json", innerHTML: `{ "@context": "http://schema.org" }`}
                ]}
                noscript={[
                    {innerHTML: `<link rel="stylesheet" type="text/css" href="foo.css" />`}
                ]}
                style={[
                  {type: "text/css", cssText: "body {background-color: blue;} p {font-size: 12px;}"}
                ]}
                onChangeClientState={(newState) => console.log(newState)}
            />
            ...
        </div>
    );
};
```

## Features
- Supports `title`, `base`, `meta`, `link`, `script`, `noscript`, and `style` tags.
- Attributes for `html` and `title` tags.
- Supports isomorphic/universal environment.
- Nested components override duplicate head changes.
- Duplicate head changes preserved when specified in same component (support for tags like "apple-touch-icon").
- Callback for tracking DOM changes.

## Installation
```
npm install --save preact-helmet
```

## Server Usage
To use on the server, call `rewind()` after using `render` from `preact-render-to-string` 
to get the head data for use in your prerender.

Because this component keeps track of mounted instances, **you have to make sure to call `rewind` on server**, or you'll get a memory leak.

```javascript
import { render } from 'preact-render-to-string'
import Helmet from 'preact-helmet'

const markup = renderToString(<MyApp />);
const head = Helmet.rewind();

// populate some document template using `markup` and `head`
const html = `
    <!doctype html>
    <html>
        <head>
            ${head.title.toString()}
            ${head.meta.toString()}
            ${head.link.toString()}
        </head>
        <body>
            <div id="content">
                ${markup}
            </div>
        </body>
    </html>
`;

```

`head` contains the following properties:
- `htmlAttributes`
- `title`
- `base`
- `meta`
- `link`
- `script`
- `noscript`
- `style`

Each property contains `toComponent()` and `toString()` methods. Use whichever is appropriate 
for your environment. For htmlAttributes, use the JSX spread operator on the object returned 
by `toComponent()`. E.g:

### As string output
```javascript
const html = `
    <!doctype html>
    <html ${head.htmlAttributes.toString()}>
        <head>
            ${head.title.toString()}
            ${head.meta.toString()}
            ${head.link.toString()}
        </head>
        <body>
            <div id="content">
                ${markup}
            </div>
        </body>
    </html>
`;
```

### As Preact components
If you are doing server side rendering with Preact, it may be easier to render the 
document template with Preact as well:

```javascript
function HTML({head}) {
    const attrs = head.htmlAttributes.toComponent();

    return (
        <html {...attrs}>
            <head>
                {head.title.toComponent()}
                {head.meta.toComponent()}
                {head.link.toComponent()}
            </head>
            <body>
                <div id="content">
                    <MyApp />
                </div>
            </body>
        </html>
    );
}
```

## Use Cases
1. Nested or latter components will override duplicate changes.
  ```javascript
  <Helmet
      title="My Title"
      meta={[
          {"name": "description", "content": "Helmet application"}
      ]}
  />
  <Helmet
      title="Nested Title"
      meta={[
          {"name": "description", "content": "Nested component"}
      ]}
  />
  ```
  Yields:
  ```
  <head>
      <title>Nested Title</title>
      <meta name="description" content="Nested component">
  </head>
  ```

2. Use a titleTemplate to format title text in your page title
  ```javascript
  <Helmet
      title="My Title"
      titleTemplate="%s | MyAwesomeWebsite.com"
  />
  <Helmet
      title="Nested Title"
  />
  ```
  Yields:
  ```
  <head>
      <title>Nested Title | MyAwesomeWebsite.com</title>
  </head>
  ```

3. Duplicate `meta` and/or `link` tags in the same component are preserved
  ```javascript
  <Helmet
      link={[
          {"rel": "apple-touch-icon", "href": "http://mysite.com/img/apple-touch-icon-57x57.png"},
          {"rel": "apple-touch-icon", "sizes": "72x72", "href": "http://mysite.com/img/apple-touch-icon-72x72.png"}
      ]}
  />
  ```
  Yields:
  ```
  <head>
      <link rel="apple-touch-icon" href="http://mysite.com/img/apple-touch-icon-57x57.png">
      <link rel="apple-touch-icon" sizes="72x72" href="http://mysite.com/img/apple-touch-icon-72x72.png">
  </head>
  ```

4. Duplicate tags can still be overwritten
  ```javascript
  <Helmet
      link={[
          {"rel": "apple-touch-icon", "href": "http://mysite.com/img/apple-touch-icon-57x57.png"},
          {"rel": "apple-touch-icon", "sizes": "72x72", "href": "http://mysite.com/img/apple-touch-icon-72x72.png"}
      ]}
  />
  <Helmet
      link={[
          {"rel": "apple-touch-icon", "href": "http://mysite.com/img/apple-touch-icon-180x180.png"}
      ]}
  />
  ```
  Yields:
  ```
  <head>
      <link rel="apple-touch-icon" href="http://mysite.com/img/apple-touch-icon-180x180.png">
  </head>
  ```

5. Only one base tag is allowed
  ```javascript
  <Helmet
      base={{"href": "http://mysite.com/"}}
  />
  <Helmet
      base={{"href": "http://mysite.com/blog"}}
  />
  ```
  Yields:
  ```
  <head>
      <base href="http://mysite.com/blog">
  </head>
  ```

6. defaultTitle will be used as a fallback when the template does not want to be used in the current Helmet
  ```javascript
  <Helmet
      defaultTitle="My Site"
      titleTemplate="My Site - %s"
  />
  ```
  Yields:
  ```
  <head>
      <title>My Site</title>
  </head>
  ```

  But a child route with a title will use the titleTemplate, giving users a way to declare a titleTemplate for their app, but not have it apply to the root.

  ```javascript
  <Helmet
      defaultTitle="My Site"
      titleTemplate="My Site - %s"
  />

  <Helmet
      title="Nested Title"
  />
  ```
  Yields:
  ```
  <head>
      <title>My Site - Nested Title</title>
  </head>
  ```

  And other child route components without a Helmet will inherit the defaultTitle.

7. Usage with `<script>` tags:
  ```javascript
  <Helmet
      script={[{
          "type": "application/ld+json",
          "innerHTML": `{
              "@context": "http://schema.org",
              "@type": "NewsArticle"
          }`
      }]}
  />
  ```
  Yields:
  ```
  <head>
      <script type="application/ld+json">
        {
            "@context": "http://schema.org",
            "@type": "NewsArticle"
        }
      </script>
  </head>
  ```

8. Usage with `<style>` tags:
  ```javascript
  <Helmet
      style={[{
          "cssText": `
              body {
                  background-color: green;
              }
          `
      }]}
  />
  ```
  Yields:
  ```
  <head>
      <style>
          body {
              background-color: green;
          }
      </style>
  </head>
  ```

## Contributing to this project
Please take a moment to review the [guidelines for contributing](CONTRIBUTING.md).

* [Pull requests](CONTRIBUTING.md#pull-requests)
* [Development Process](CONTRIBUTING.md#development)

## License

MIT

## More Examples
[react-helmet-example](https://github.com/mattdennewitz/react-helmet-example) (for React, but still useful)
