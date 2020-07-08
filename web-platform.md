# Tumblr's New Web Platform

Hello! In 2020, we're rolling out a new version of the Tumblr web app. It's fresh and fancy, built on React and other fun things. This documentation is a guide to how to develop on top of it - browser extensions and that sort of thing. All of the following documentation assumes you are using JavaScript in the console or in the context of a browser extension.

This is fairly short at the moment, but it's a work in progress.

## How do I tell which version of the site I'm on?

A good question! This rollout is not going to happen all at once - different pages on the site will be converted at different times. There's a simple way to tell the difference, though. Just check to see if `window.tumblr` is defined. Note the lowercase - it's the real distinction here, as the other version of the site does define a `window.Tumblr`.

## What can I do with `window.tumblr`?

### `getCssMap`

One major difference in our new version of the site is the way our stylesheets and class names are compiled. No longer are the class names engineers write in the source code the ones that end up in the actual version of the site. This makes DOM manipulation a tricky proposal, from an extension's point of view. This function is intended to bridge that gap.

In TypeScript terms, the function is typed like this:

```ts
() => Promise<{ [Key: string]: string[] }>
```

So it's an async function, returning a promise that resolves to an object. That object has string keys - semantic class names - and values that are arrays of strings. Those strings in the array are the actual CSS class names as they exist on the page.

Let's have an example. Say you wanted to make every post currently on the page turn upside down. That could look like this:

```js
window.tumblr.getCssMap().then(cssMap => {
  // Always make sure the class name you're looking for actually exists on the object!
  if (!cssMap.post) {
    return;
  }

  const classNames = cssMap.post.map(className => `.${className}`);
  const selector = classNames.join(', ');
  const posts = document.querySelectorAll(selector);

  posts.forEach(post => {
    post.style.transform = 'rotate(180deg)';
  });
});
```

`cssMap.post` in there could look something like this:

```js
['_2OqKn', '_2-mU3', '_1yQrs', '_1Wz4U']
```

An array of all class names that are, in our source code, labeled as `'post'`. A lot of the values are going to be arrays with a length of 1, but in some cases, like this, we have more than one style of post. That snippet flips 'em all.

### `languageData`
If you want to avoid dealing with class names entirely, one way of querying for elements consistently might be to look for other element properties, like `aria-label`, that don't get hashed. Only problem there is that they are localized to each user's display language.

This object provides a map of English root strings to the equivalents in the current language. For a user looking at Tumblr in Spanish, it would look something like this:
```js
{
  code: 'es_ES',
  translations: {
    // Truncated - it's a large object
    Edit: 'Editar',
    'Edit Appearance': 'Modificar apariencia',
    'Eh?': '¿Eh?',
  },
}
```

If you wanted to query for all the edit buttons on a page, that could look like this:
```js
document.querySelectorAll(`button[aria-label="${window.tumblr.languageData.translations['Edit'] || 'Edit'}"`)
```

When a user has their language set to English, the `translations` object will be empty - hence the `|| 'Edit'` in the example above.

### `apiFetch`
This function is a helper, used to make requests to the Tumblr API in the exact same way all our in-house web code does.

Here's a simple example of how it would work.

```ts
window.tumblr.apiFetch('/v2/tagged', { method: 'GET', queryParams: { tag: 'furby' }}).then(response => {
  console.log(response);
})
```

The first parameter is the path for the API route, without the host. You can see documentation of our API routes [right here](https://github.com/tumblr/docs/blob/master/api.md).

Internally, this is using the native `fetch` function, and so the second parameter object accepts all keys that [the native one](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch) does, with two major differences:

1. `queryParams` - As in the example above, `apiFetch` accepts a `queryParams` object. This will encode the params and append them to the request URL, so you don't need to do that yourself.
2. `body` - When making `POST` requests, `apiFetch` accepts an object as the `body` param, and stringifies it for you, unlike native `fetch`.

## Help

If something's not working as expected, or you've got feature requests, please create an issue in this here repo!
