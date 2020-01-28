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

## Help

If something's not working as expected, or you've got feature requests, please create an issue in this here repo!
