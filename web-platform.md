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

It's also important to know that, because Tumblr's API works with JSON objects, `apiFetch` returns the JSON response to you in the `then()` block of the Promise. Because it does some response processing to get that JSON, in certain cases it will reject the Promise or even `throw`. Here is a matrix that describes `apiFetch()`'s behavior:

| `response.headers.get('content-type')` | `response.ok` | What `apiFetch()` does |
| ----- | ----- | ----- |
| `application/javascript` or `application/json` | `true` | Resolves the promise with the JSON of the response |
| `application/javascript` or `application/json` | `false` | Rejects the promise with an error object |
| any non-json content-type | `true` | Resolves the promise with `{}` |
| any non-json content-type | `false` | `throw`s an error object |

### `navigate`

Another key difference from the old website is that the modern web app handles the loading of different pages on the client, rather than forcing a new document to be fetched from the server when any link is clicked. This is great for the default experience, but anchors added by third-party tools won't trigger this by default.

This function bridges that gap, allowing you to perform client-side navigations. It takes one parameter: the path of the destination page.

```js
window.tumblr.navigate('/explore/trending');    // Loads the trending topics page
window.tumblr.navigate('/staff');               // Loads the Staff blog as a modal
```

For a more complete example, we can use this to construct a fancy anchor that loads the #photography hub on the client when clicked:

```js
const handleClick = event => {
  if (event.altKey || event.ctrlKey || event.metaKey || event.shiftKey) {
    // Allow default behaviour of modifier key + click
    return;
  }

  event.preventDefault();
  window.tumblr.navigate(event.currentTarget.href);
};

const anchor = document.createElement('a');
anchor.href = '/tagged/photography';
anchor.textContent = '#photography';
anchor.addEventListener('click', handleClick);
```

Please note that this function has no return value and will not throw if given an invalid argument. It also cannot facilitate navigation to pages that exist under `www.tumblr.com` but not within the web app. For example, a plain anchor with `href="/help"` will go to Tumblr's Help Center, while `navigate('/help')` will not.

### `on()` and `off()`
These are used to add and remove callbacks for various events that the website sends. The first parameter is the event name, and the second parameter is a function to call when the event happens. This function will return `true` or `false` to indicate whether the addition or removal was successful.

_Right now, `navigation` is the only supported event name._ Here is an example of how to monitor client-side navigation changes:

```ts
const navigationHandler = () => console.log(`The page just changed to ${window.location.href}.`);
window.tumblr.on('navigation', navigationHandler);

// Later, when you no longer want to listen to navigation events
window.tumblr.off('navigation', navigationHandler);
```

## Help

If something's not working as expected, or you've got feature requests, please create an issue in this here repo!
