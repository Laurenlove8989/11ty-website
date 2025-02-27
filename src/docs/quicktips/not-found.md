---
tipindex: "006"
tiptitle: "Adding a 404 Not Found Page to your Static Site"
date: 2018-09-20
---
A `404: Not Found` page is a pretty standard thing on the web. It’s particularly useful when someone shares a broken or mistyped link. You don’t need dynamic middleware to do this, it’s pretty easy to add to a statically generated site. There are a few options, depending on your web host:

But first let’s make our `404.html` template:

{% raw %}
```html
---
title: Oops! Not Found
permalink: 404.html
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
  </head>
  <body>
    <h1>{{ title }}</h1>
    <p>This is where you should tell the user how to find their content. Maybe on the <a href="/">home page?</a></p>
  </body>
</html>
```
{% endraw %}

Eleventy will output this template to `404.html`.

If you’re using [GitHub Pages](https://help.github.com/en/github/working-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site) or [Netlify](https://docs.netlify.com/routing/redirects/redirect-options/#custom-404-page-handling), there is no step two! A `404.html` file in your output directory is all you need.

Netlify even has lovely [multi-language 404 page support too using `Redirects`](https://docs.netlify.com/routing/redirects/redirect-options/#custom-404-page-handling).

## .htaccess

For other hosts, if you use `.htaccess` for configuration you can use `ErrorDocument`. Make sure your root directory matches here!

```
ErrorDocument 404 /404.html
```

## With `--serve`

{% callout "info", "md" -%}
If you’re using the new [Eleventy Dev Server](/docs/watch-serve/#eleventy-dev-server) {% addedin "2.0.0" %}, the `404.html` convention is now provided to you for free! You can skip this section!

If you’d like to backport this functionality to Browsersync, keep reading. (Note also that you can [use Browsersync with Eleventy 2.0!](/docs/dev-server/#swap-back-to-browsersync))
{% endcallout %}

If you're using Browsersync with Eleventy (with `eleventy --serve`), you can configure it to do the 404 routing by passing a callback in your config. Read more on [the BrowserSyncConfig option](/docs/config/#override-browsersync-server-options), the [Browsersync callbacks option](https://browsersync.io/docs/options#option-callbacks), and [how to provide a 404 using a Browsersync callback](https://github.com/browsersync/browser-sync/issues/1398).

{% codetitle ".eleventy.js" %}

```js
const fs = require("fs");
const NOT_FOUND_PATH = "_site/404.html";

module.exports = function(eleventyConfig) {
  eleventyConfig.setBrowserSyncConfig({
    callbacks: {
      ready: function(err, bs) {

        bs.addMiddleware("*", (req, res) => {
          if (!fs.existsSync(NOT_FOUND_PATH)) {
            throw new Error(`Expected a \`${NOT_FOUND_PATH}\` file but could not find one. Did you create a 404.html template?`);
          }

          const content_404 = fs.readFileSync(NOT_FOUND_PATH);
          // Add 404 http status code in request header.
          res.writeHead(404, { "Content-Type": "text/html; charset=UTF-8" });
          // Provides the 404 content without redirect.
          res.write(content_404);
          res.end();
        });
      }
    }
  });
};
```

_Thank you [Cassey Lottman](https://github.com/clottman)._
