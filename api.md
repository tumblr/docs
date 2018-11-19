# Tumblr API

Welcome to the Tumblr API! There isn't anything we enjoy more than seeing talented designers and engineers using Tumblr to invent whole new forms of creative expression. We've put a tremendous amount of care into making this API functional and flexible enough for any projects you throw at it. Join us in our [discussion group](https://groups.google.com/group/tumblr-api/) to talk about how to use it, what could be better, and all the amazing things you're going to build with it. Follow our [Engineering Blog](https://engineering.tumblr.com/) for important news and announcements. Please use the API [responsibly](https://www.tumblr.com/docs/api_agreement), and [send us your feedback](mailto:api@tumblr.com). Enjoy!

Tumblr also supports an API that delivers content according to the [oEmbed standard](http://oembed.com/). Our oEmbed API endpoint is `https://www.tumblr.com/oembed`, which supports post URLs in the format `https://*.tumblr.com/post/*`.

If you're looking for documentation for the old API, you can find it [here](https://www.tumblr.com/docs/api/v1).

## Table of Contents

- [What You Need](#what-you-need)
- [Console](#console)
- [API Overview](#api-overview)
    - [URI Structure](#uri-structure)
    - [Blog Identifiers](#blog-identifiers)
    - [Response Format](#response-format)
    - [About the API Documentation](#about-the-api-documentation)
- [Official Tumblr API Client Libraries](#official-tumblr-api-client-libraries)
- [Authentication](#authentication)
    - [OAuth](#oauth)
- [Common Response Elements](#common-response-elements)
    - [Links](#links)
    - [Tag Objects](#tag-objects)
    - [Neue Post Format objects](#neue-post-format-objects)
- [Blog Methods](#blog-methods)
    - [`/info` - Retrieve Blog Info](#info---retrieve-blog-info)
    - [`/avatar` — Retrieve a Blog Avatar](#avatar--retrieve-a-blog-avatar)
    - [`/likes` — Retrieve Blog's Likes](#likes--retrieve-blogs-likes)
    - [`/following` — Retrieve Blog's following](#following--retrieve-blogs-following)
    - [`/followers` — Retrieve a Blog's Followers](#followers--retrieve-a-blogs-followers)
    - [`/posts` – Retrieve Published Posts](#posts--retrieve-published-posts)
    - [`/posts/queue` — Retrieve Queued Posts](#postsqueue--retrieve-queued-posts)
    - [`/posts/draft` — Retrieve Draft Posts](#postsdraft--retrieve-draft-posts)
    - [`/posts/submission` — Retrieve Submission Posts](#postssubmission--retrieve-submission-posts)
    - [`/post` — Create a New Blog Post (Legacy)](#post--create-a-new-blog-post-legacy)
    - [`/post/edit` – Edit a Blog Post (Legacy)](#postedit--edit-a-blog-post-legacy)
    - [`/post/reblog` – Reblog a Post (Legacy)](#postreblog--reblog-a-post-legacy)
    - [`/posts` - Create/Reblog a Post (Neue Post Format)](#posts---createreblog-a-post-neue-post-format)
    - [`/posts/{post-id}` - Fetching a Post (Neue Post Format)](#postspost-id---fetching-a-post-neue-post-format)
    - [`/posts/{post-id}` - Editing a Post (Neue Post Format)](#postspost-id---editing-a-post-neue-post-format)
    - [`/post/delete` – Delete a Post](#postdelete--delete-a-post)
- [User Methods](#user-methods)
    - [`/user/info` – Get a User's Information](#userinfo--get-a-users-information)
    - [`/user/dashboard` – Retrieve a User's Dashboard](#userdashboard--retrieve-a-users-dashboard)
    - [`/user/likes` — Retrieve a User's Likes](#userlikes--retrieve-a-users-likes)
    - [`/user/following` – Retrieve the Blogs a User Is Following](#userfollowing--retrieve-the-blogs-a-user-is-following)
    - [`/user/follow` – Follow a blog](#userfollow--follow-a-blog)
    - [`/user/unfollow` – Unfollow a blog](#userunfollow--unfollow-a-blog)
    - [`/user/like` – Like a Post](#userlike--like-a-post)
    - [`/user/unlike` – Unlike a Post](#userunlike--unlike-a-post)
- [Tagged Method](#tagged-method)
    - [`/tagged` – Get Posts with Tag](#tagged--get-posts-with-tag)

## What You Need

To get an OAuth key you must [register an application](https://www.tumblr.com/oauth/apps).

You'll need this to get your API key, even if you don't ever need to use a fully signed OAuth request.

For more details, see [Authentication](#authentication) below.

## Console

Check out our [developer API console](https://api.tumblr.com/console) to test calls and see live examples of using our official clients.

## API Overview

Before you dive in, review these essentials.

### URI Structure

All Tumblr API requests start with `https://api.tumblr.com`.

The next segment of the URI path depends on the type of request you want to make. For example, getting blog data or to make a post on a blog uses the `/v2/blog/{blog-identifier}/...` endpoint, so the full URL would be `https://api.tumblr.com/v2/blog/{blog-identifier}/...`

See the different Methods sections below for complete details on the available routes.

### Blog Identifiers

Each blog has a unique hostname which can be used as its **identifier**. The hostname can be **standard** or **custom**.

- **Standard hostname**: the blog short name + `.tumblr.com`. Example: `greentype.tumblr.com`
- **Custom hostname**: these can be anything at all, as determined by a [DNS CNAME entry](https://tumblr.zendesk.com/hc/en-us/articles/231256548-Custom-domains). Example: `www.davidslog.com`

You'll need the blog hostname to use as its identifier anytime you work with a blog. When you see the placeholder `{blog-identifier}` in these API docs, substitute the blog's standard or custom hostname.

#### Blog Unique Identifiers

Each blog also has a unique identifier that you can retrieve from any API response that includes a blog, in the `uuid` field (example: `t:DvRFDGL05g8KB0gwiBJv1A`). The `{blog-identifier}` placeholder can also be replaced by this unique identifier.

The benefits of using a unique identifier instead of a hostname are that the unique identifier will not change if the blog name or custom domain changes. It can be used as a stable, persistent identifier for a blog.

However, note that under exceptional circumstances, a unique identifier can change. Your own blog's unique identifier will only be changed with your knowing.

### Response Format

The API returns JSON-encoded objects (`Content-Type: application/json`). Responses vary according to the method/endpoint used, but every response envelope includes these common parts:

| Field | Notes |
| ----- | ----- |
| **meta** | The meta object matches the HTTP response message |
| **response** | Endpoint-specific results |

The `meta` object should always contain:

| Field | Notes |
| ----- | ----- |
| **status** | the 3-digit HTTP Status-Code (e.g., `200`) |
| **msg** | the HTTP Reason-Phrase (e.g., `OK`) |

#### Example

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": { ... }
}
```

All requests made with HTTP GET are [JSONP](http://en.wikipedia.org/wiki/JSONP)-enabled. To use JSONP, append `jsonp=` or `callback=` and the name of your callback function to the request. JSONP requests will always return an HTTP status code of 200 but will reflect the real status code in the meta field of the JSON response.

### About the API Documentation

These docs include some working examples. Please click them to your heart's content, but don't use the embedded API key for your own nefarious purposes. [Register your application](https://www.tumblr.com/oauth/apps) to get your own API key (your OAuth Consumer Key).

#### URI Conventions

| Notation | Meaning | Example |
| -------- | ------- | ------- |
| **Curly brackets { }** | Required item | `api.tumblr.com/v2/blog/{blog-identifier}/posts` (the blog identifier is required) |
| **Square brackets [ ]** | Optional item | `api.tumblr.com/v2/blog/{blog-identifier}/posts[/type]` (specifying the type is optional) |

## Official Tumblr API Client Libraries

Here is a list of official API client which will help you get up and running with your favorite language in no time.

- [Javascript](http://github.com/tumblr/tumblr.js)
- [Ruby](http://github.com/tumblr/tumblr_client)
- [PHP](http://github.com/tumblr/tumblr.php)
- [Java](http://github.com/tumblr/jumblr)
- [Python](http://github.com/tumblr/pytumblr)
- [Objective-C](http://github.com/tumblr/TMTumblrSDK)
- [Go](http://github.com/tumblr/tumblrclient.go)

## Authentication

The API uses three different levels of authentication, depending on the method.

- **None:** No authentication. Anybody can query the method.
- **API key:** Requires an API key. Use your OAuth Consumer Key as your `api_key`.
  - Example: `api_key=PyezS3Q4Smivb24d9SzZGYSuh--IaMfAkE`
  - Get an OAuth key: [register an application](https://www.tumblr.com/oauth/apps)
- **OAuth:** Requires a signed request that meets the [OAuth 1.0a Protocol](http://oauth.net/).

Each method description below includes a note about the authentication level.

### OAuth

The API supports the [OAuth 1.0a Protocol](http://oauth.net/), accepting parameters via the `Authorization` header, with the `HMAC-SHA1` signature method only. There's probably already an [OAuth client library](http://oauth.net/code/) for your platform.

#### Endpoints

- **Request-token URL:** `https://www.tumblr.com/oauth/request_token`
- **Authorize URL:** `https://www.tumblr.com/oauth/authorize`
- **Access-token URL:** `https://www.tumblr.com/oauth/access_token`

## Common Response Elements

Beyond the [overall response format described above](#response-format), there are some elements that are common throughout the API responses.

### Links

Links describe an in-app or navigation behavior. A response may contain one or more of these contained under the `_links` response object key.

An object in the `_links` hash will always contain `type` and `href` keys.

A `_links` object may be of type `navigation` or `action`.

- `navigation`: A reference to some external URI to which the user can go.
- `action`: A reference to some internal client state change in the application. The most common example would be to open a different view.

A `navigation` link will only contain a `href` to which the user should be directed. An `action` type link will contain details on what request method as well as what query or body parameters should be used to take the action.

**Example:**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "_links": {
         "previous": {
            "type": "action",
            "method": "GET",
            "href": "https://api.tumblr.com/v2/endpoint",
            "query_params": {
               "page": 1,
            }
         },
         "next": {
            "type": "action",
            "method": "GET",
            "href": "https://api.tumblr.com/v2/endpoint",
            "query_params": {
               "page": 3,
            }
         },
         "terms_of_service": {
            "type": "navigation",
            "href": "https://www.tumblr.com/policy/terms-of-service",
         },
         ...
      }
   }
}
```

### Tag Objects

A tag object represents a tag, and could have these fields:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **tag** | String | The tag name |
| **thumb_url** | String | Returns an image (75x75) associated with the tag; may be `null` |
| **is_tracked** | Boolean | Indicates whether the requesting user is tracking this tag |
| **featured** | Boolean | Indicates whether the tag is a featured tag |

### Neue Post Format objects

Some routes will return Posts that have `type: blocks` and/or `is_blocks_post_format: true`, which means their content will be in the Neue Post Format. [See the NPF specification docs for more info!](npf-spec.md)

## Blog Methods

### `/info` - Retrieve Blog Info

This method returns general information about the blog, such as the title, number of posts, and other high-level data.

**Examples**

- `https://api.tumblr.com/v2/blog/scipsy.tumblr.com/info`
- `https://api.tumblr.com/v2/blog/good.tumblr.com/info`

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/info?api_key={key}` | GET | [API Key](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier See the [Blog Identifiers](#blog-identifiers) for more details. | N/A | Yes |
| **api_key** | String | Your OAuth Consumer Key See [Authentication](#authentication) for more details. | N/A | Yes |

#### Response

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **title** | String | The display title of the blog | Compare name |
| **posts** | Number | The total number of posts to this blog | |
| **name** | String | The short blog name that appears before tumblr.com in a standard blog hostname | Compare title |
| **updated** | Number | The time of the most recent post, in seconds since the epoch |  |
| **description** | String | You guessed it! The blog's description |  |
| **ask** | Boolean | Indicates whether the blog allows questions |  |
| **ask_anon** | Boolean | Indicates whether the blog allows anonymous questions; returned only if ask is true |  |
| **likes** | Number | Number of likes for this user | Returned only if this is the user's primary blog and sharing of likes is enabled |  |
| **is_blocked_from_primary** | Boolean | Indicates whether this blog has been blocked by the calling user's primary blog; returned only if there is an authenticated user making this call |  |

**Example**

[https://api.tumblr.com/v2/blog/david.tumblr.com/info?api_key={key}](https://api.tumblr.com/v2/blog/david.tumblr.com/info?api_key=fuiKNFp9vQFvjLNvx4sUwti4Yb5yGutBN4Xh10LXZhhRKjWlV4)

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": {
         "title": "David's Log",
         "posts": 3456,
         "name": "david",
         "url": "http:\/\/david.tumblr.com\/",
         "updated": 1308953007,
         "description": "<p><strong>Mr. Karp<\/strong> is tall and skinny, with
            unflinching blue eyes a mop of brown hair.\r\n
            He speaks incredibly fast and in complete paragraphs.</p>",
         "ask": true,
         "ask_anon": false,
         "likes": 12345
      }
   }
}
```

### `/avatar` — Retrieve a Blog Avatar

You can get a blog's avatar in 9 different sizes. The default size is 64x64.

**Examples**

- `https://api.tumblr.com/v2/blog/david.tumblr.com/avatar/512`
- `https://api.tumblr.com/v2/blog/david.tumblr.com/avatar`

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/avatar[/size]` | GET | None |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier See the [Blog Identifiers](#blog-identifiers) for more details. | N/A | Yes |
| **size** | Number | The size of the avatar (square, one value for both length and width). Must be one of the values: 16, 24, 30, 40, 48, 64, 96, 128, 512 | 64 | No |

#### Response

Requests that are _not_ signed using OAuth1 will receive the requested avatar in PNG format, while requests that are signed will receive a response of the following form:

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **avatar_url** | String | The URL of the avatar image. This is also returned in the [Location HTTP header field](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) (see note). | An [HTTP Location header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) field is returned — the response points to the avatar image. That means you can embed this method in an `img` tag in HTML. |

### `/likes` — Retrieve Blog's Likes

This method can be used to retrieve the publicly exposed likes from a blog.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/likes?api_key={key}` | GET | [API Key](https://www.tumblr.com/docs/en/api/v2#auth) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Liked post number to start at | 0 (first post) | No |
| **before** | Integer | Retrieve posts liked before the specified timestamp | None | No |
| **after** | Integer | Retrieve posts liked after the specified timestamp | None | No |

**Notes**

- You can only provide either before, after, or offset. If you provide more than one of these options together you will get an error.
- You can still use limit with any of those three options to limit your result set.
- When using the offset parameter the maximum limit on the offset is 1000. If you would like to get more results than that use either before or after.

#### Response

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **liked_posts** | Array | An array of post objects (posts liked by the user) |
| **liked_count** | Number | Total number of liked posts |

### `/following` — Retrieve Blog's following

This method can be used to retrieve the publicly exposed list of blogs that a blog follows, in order from most recently-followed to first.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/following` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Followed blog index to start at | 0 (most-recently followed) | No |

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **blogs** | Array | An array of short blog infos that this blog follows, in order from most recently-followed to first. |
| **total_blogs** | Number | Total number of followed blogs |

### `/followers` — Retrieve a Blog's Followers

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/followers` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier See the Overview for more details. | N/A | Yes |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Result to start at | 0 (first follower) | No |

#### Response

These fields are wrapped in a `users` object:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **total_users** | Number | The number of users currently following the blog |
| **users** | Array | Each item is a follower, containing these fields: |
| **name** | String | The user's name on tumblr |
| **following** | Boolean | Whether the caller is following the user |
| **url** | String | The URL of the user's primary blog |
| **updated** | Number | The time of the user's most recent post, in seconds since the epoch |

**Example**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "total_users": 2684,
      "users":  [
            {
               "name": "david",
               "following": true,
               "url": "http:\/\/www.davidslog.com",
               "updated": 1308781073
            },
            {
               "name": "ben",
               "following": true,
               "url": "http:\/\/bengold.tv",
               "updated": 1308841333
            },
            ...
         ]
   }
}
```

### `/posts` – Retrieve Published Posts

**Examples**

- `https://api.tumblr.com/v2/blog/peacecorps.tumblr.com/posts/text?notes_info=true`
- `https://api.tumblr.com/v2/blog/pitchersandpoets.tumblr.com/posts/photo?tag=new+york+yankees`
- `https://api.tumblr.com/v2/blog/staff.tumblr.com/posts?before=1496289599`

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts[/type]?api_key={key}&[optional-params=]` | GET | [API Key](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier See the Overview for more details. | N/A | Yes |
| **api_key** | String | Your OAuth Consumer Key See Authentication for more details. | N/A | Yes |
| **type** | String | The type of post to return. Specify one of the following: text, quote, link, answer, video, audio, photo, chat | None – return all types | No |
| **id** | Number | A specific post ID. Returns the single post specified or (if not found) a 404 error. | None | No |
| **tag** | String | Limits the response to posts with the specified tag | None | No |
| **limit** | Number | The number of posts to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Post number to start at | 0 (first post) | No |
| **reblog_info** | Boolean | Indicates whether to return reblog information (specify true or false). Returns the various reblogged_ fields. | False | No |
| **notes_info** | Boolean | Indicates whether to return notes information (specify true or false). Returns note count and note metadata. | False | No |
| **filter** | String | Specifies the post format to return, other than HTML: text – Plain text, no HTML; raw – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |
| **before** | Number | Returns posts published earlier than a specified Unix timestamp, in seconds. | False | No |

#### Response

Each response includes a `blog` object that is the equivalent of an `/info` [response](#blog-info). Posts are returned as an array attached to the `posts` field.

**Fields available for all Post types:**

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **blog_name** | String | The short name used to uniquely identify a blog | |
| **id** | Number | The post's unique ID | |
| **post_url** | String | The location of the post | |
| **type** | String | The type of post | See the `type` request parameter |
| **timestamp** | Number | The time of the post, in seconds since the epoch | |
| **date** | String | The GMT date and time of the post, as a string | |
| **format** | String | The post format: `html` or `markdown` | |
| **reblog_key** | String | The key used to reblog this post | See the `/post/reblog` method |
| **tags** | Array of strings | Tags applied to the post | |
| **bookmarklet** | Boolean | Indicates whether the post was created via the Tumblr bookmarklet | Exists only if true |
| **mobile** | Boolean | Indicates whether the post was created via mobile/email publishing | Exists only if true |
| **source_url** | String | The URL for the source of the content (for quotes, reblogs, etc.) | Exists only if there's a content source |
| **source_title** | String | The title of the source site | Exists only if there's a content source |
| **liked** | Boolean | Indicates if a user has already liked a post or not | Exists only if the request is fully authenticated with OAuth. |
| **state** | String | Indicates the current state of the post | States are published, queued, draft and private |
| **total_posts** | Number | The total number of post available for this request, useful for paginating through results | |

#### Legacy Text Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **title** | String | The optional title of the post |
| **body** | String | The full post body |

**Example**

`https://api.tumblr.com/v2/blog/citriccomics.tumblr.com/posts/text?api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "citriccomics",
            "id": 3507845453,
            "post_url": "http:\/\/citriccomics.tumblr.com\/post\/3507845453",
            "type": "text",
            "date": "2011-02-25 20:27:00 GMT",
            "timestamp": 1298665620,
            "state": "published",
            "format": "html",
            "reblog_key": "b0baQtsl",
            "tags": [
               "tumblrize",
               "milky dog",
               "mini comic"
            ],
            "note_count": 14,
            "title": "Milky Dog",
            "body": "<p><img src=\"http:\/\/media.tumblr.com\
               /tumblr_lh6x8d7LBB1qa6gy3.jpg\"\/><a href=\"http:\/\
               /citriccomics.com\/blog\/?p=487\" target=\"_blank\">TO READ
               THE REST CLICK HERE<\/a><br\/>\n\nMilky Dog was inspired by
               something <a href=\"http:\/\/gunadie.com\/naomi\"
               target=\"_blank\">Naomi Gee<\/a> wrote on twitter, I really
               liked the hash tag <a href=\"http:\/\/twitter.com\/
               search?q=%23MILKYDOG\" target=\"_blank\">#milkydog<\/a>
               and quickly came up with a little comic about it. You can
               (and should) follow Naomi on twitter <a href=\"http:\/\
               /twitter.com\/ngun\" target=\"_blank\">@ngun<\/a> I'm on
               twitter as well <a href=\"http:\/\/twitter.com\
               /weflewairplanes\"target=\"_blank\">@weflewairplanes<\/a>
               <\/p>\n\nAlso, if youâ€™re a Reddit user (or even if
               you're not) I submitted this there, if you could up vote
               it I'd be super grateful just <a href=\"http:\/\
               /tinyurl.com\/5wj3tqz\" target=\"_blank\">CLICK HERE<\/a>"
         },
         ...
      ],
      "total_posts": 3
   }
}
```

#### Legacy Photo Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **caption** | String | The user-supplied caption |
| **width** | Number | The width of the photo or photoset |
| **height** | Number | The height of the photo or photoset |
| **photos** | Array | Photo objects with properties: |
| **caption** | String | user supplied caption for the individual photo (Photosets only) |
| **alt_sizes** | Array | alternate photo sizes, each with: |
| **width** | Number | width of the photo, in pixels |
| **height** | Number | height of the photo, in pixels |
| **url** | String | Location of the photo file (either a JPG, GIF, or PNG) |

> Multi-photo Photo posts, called **Photosets**, will send return multiple photo objects in the `photos` array.

**Example**

`https://api.tumblr.com/v2/blog/derekg.org/posts/photo?id=7431599279&api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "derekg",
            "id": 7431599279,
            "post_url": "http:\/\/derekg.org\/post\/7431599279",
            "type": "photo",
            "date": "2011-07-09 22:09:47 GMT",
            "timestamp": 1310249387,
            "format": "html",
            "reblog_key": "749amggU",
            "tags": [],
            "note_count": 18,
            "caption": "<p>my arm is getting tired.<\/p>",
            "photos": [
               {
                  "caption": "",
                  "alt_sizes": [
                     {
                        "width": 1280,
                        "height": 722,
                        "url": "http:\/\/derekg.org\/photo\/1280\/7431599279\/1\/
                           tumblr_lo36wbWqqq1qanqww"
                     },
                     {
                        "width": 500,
                        "height": 282,
                        "url": "http:\/\/30.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_500.jpg"
                     },
                     {
                        "width": 400,
                        "height": 225,
                        "url": "http:\/\/29.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_400.jpg"
                     },
                     {
                        "width": 250,
                        "height": 141,
                        "url": "http:\/\/26.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_250.jpg"
                     },
                     {
                        "width": 100,
                        "height": 56,
                        "url": "http:\/\/24.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_100.jpg"
                     },
                     {
                        "width": 75,
                        "height": 75,
                        "url": "http:\/\/30.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_75sq.jpg"
                     }
                  ]
               }
            ]
         }
      ],
      "total_posts": 1
   }
}
```

#### Legacy Quote Posts

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **text** | String | The text of the quote (can be modified by the user when posting) | |
| **source** | String | Full HTML for the source of the quote Example: `<a href="...">Steve Jobs</a>` | See also the table of common response fields |

**Example**

`https://api.tumblr.com/v2/blog/museumsandstuff.tumblr.com/posts/quote?api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "museumsandstuff",
            "id": 4742980381,
            "post_url": "http:\/\/museumsandstuff.tumblr.com\/post\/4742980381",
            "type": "quote",
            "date": "2011-04-19 08:52:34 GMT",
            "timestamp": 1303203154,
            "format": "html",
            "reblog_key": "KLA85e6c",
            "tags": [],
            "note_count": 23,
            "source_url": "http:\/\/museumtwo.blogspot.com\/2011\/04\/
               guest-post-convivial-museum-photo-essay.html",
            "source_title": "museumtwo.blogspot.com",
            "text": "Why do visitors still report discomfort, confusion,
               elitism, exclusion?",
            "source": "<a href=\"http:\/\/museumtwo.blogspot.com\/2011\/04\/
               guest-post-convivial-museum-photo-essay.html\"
               target=\"_blank\">Museum 2.0: Guest Post: The Convivial
               Museum Photo Essay<\/a> (via <a href=\"http:\/\/
               www.joshrobinson.org\/\"target=\"_blank\">joshrobinsonblog
               <\/a>)"
         },
         ...
      ],
      "total_posts": 9
   }
}
```

#### Legacy Link Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **title** | String | The title of the page the link points to |
| **description** | String | A user-supplied description |
| **url** | String | The link! |
| **author** | String | The author of the article the link points to |
| **excerpt** | String | An excerpt from the article the link points to |
| **publisher** | String | The publisher of the article the link points to |
| **photos** | Array | Photo objects with properties: |
| **caption** | String | The thumbnail caption |
| **original_size** | Object | The photo at its original size |
| **width** | Number | Width of the image, in pixels |
| **height** | Number | Height of the image, in pixels |
| **url** | String | Location of the image file (either a JPG, GIF, or PNG) |
| **alt_sizes** | Array | Alternate photo sizes, each with the same properties as above. |

**Example**

`https://api.tumblr.com/v2/blog/travellingcameraclub.com/posts/link?api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "travellingcameraclub",
            "id": 688472164,
            "post_url": "http:\/\/travellingcameraclub.com\/post\/688472164",
            "type": "link",
            "date": "2010-06-11 23:17:08 GMT",
            "timestamp": 1276298228,
            "format": "html",
            "reblog_key": "RWhJJWia",
            "tags": [],
            "note_count": 9,
            "title": "Esther Aarts Illustration | News and Blog: Faq: How do
               you make those marker doodles?",
            "url": "http:\/\/blog.estadiezijn.nl\/post\/
               459075181\/faq-how-do-you-make-those-marker-doodles",
            "author": "Ester Aarts",
            "excerpt": "How I make marker doodles",
            "publisher": "blog.estadiezijn.nl",
            "photos": [
               {
                  "caption": "",
                  "original_size":
                     {
                        "width": 500,
                        "height": 500,
                        "url": "http:\/\/40.media.tumblr.com
                        \/1393850e5c331da2e3c9fb478a30310d
                        \/tumblr_inline_nm3lwntw8k1rplry2_500.jpg"
                     },
                  "alt_sizes": []
               }
            ],
            "description": "<blockquote>\n<p>On my Ipad, of course!<\/p>
                <p>Nothing better than the latest technology to get the job done.
                Look at all my apps!<\/p> <p><img height=\"555\" width=\"500\"
                src=\"http:\/\/farm5.static.flickr.com\/4006\/
                4445161463_31da0327c2_o.jpg\" alt=\"my iPad\"\/><\/p>
                <p>My favourite markers are an Edding 400, a Sharpie and a
                Copic Ciao. The white gouache is from Dr Martins and does a
                decent job covering up whatever needs to be covered up, and
                flows.<\/p><\/blockquote>"
         }
         ...
      ],
      "total_posts": 7
   }
}
```

#### Legacy Chat Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **title** | String | The optional title of the post |
| **body** | String | The full chat body |
| **dialogue** | Array | Array of objects with the following properties: |
| **name** | String | Name of the speaker |
| **label** | String | Label of the speaker |
| **phrase** | String | Text. |

**Example**

`https://api.tumblr.com/v2/blog/www.davidslog.com/posts/chat?api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "david",
            "id": 5845345725,
            "post_url": "http:\/\/www.davidslog.com\/5845345725\/if-youre-okay-with-it-destroying-tumblr",
            "type": "chat",
            "date": "2011-05-25 22:32:00 GMT",
            "timestamp": 1306362720,
            "format": "html",
            "reblog_key": "wlxAsElf",
            "tags": [],
            "note_count": 114,
            "title": null,
            "body": "David: http://staff.tumblr.com/post/5845153911
                [draft] Topherchris: are you freaking serious\r\n
                David: If you're okay with it - I'd love to :)\r\n
                Topherchris: i'm okay with it, if you're okay with
                it destroying tumblr",
            "dialogue": [
                {
                   "label": "David:",
                   "name": "David",
                   "phrase": "http://staff.tumblr.com/post/5845153911 [draft]\r",
                },
                {
                   "label": "Topherchris:",
                   "name": "Topherchris",
                   "phrase": "are you freaking serious\r"
                },
                {
                   "label": "David:",
                   "name": "David",
                   "phrase": "If you're okay with it - I'd love to :)"
                },
                {
                   "label": "Topherchris:",
                   "name": "Topherchris",
                   "phrase": "i'm okay with it, if you're okay with it
                      destroying tumblr",
                }
            ]
         }
         ...
      ],
      "total_posts": 7
   }
}
```

#### Legacy Audio Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **caption** | String | The user-supplied caption |
| **player** | String | HTML for embedding the audio player |
| **plays** | Number | Number of times the audio post has been played |
| **album_art** | String | Location of the audio file's ID3 album art image |
| **artist** | String | The audio file's ID3 artist value |
| **album** | String | The audio file's ID3 album value |
| **track_name** | String | The audio file's ID3 title value |
| **track_number** | Number | The audio file's ID3 track value |
| **year** | Number | The audio file's ID3 year value |

**Example**

`https://api.tumblr.com/v2/blog/derekg.org/posts?id=5578378101&api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "derekg",
            "id": 5578378101,
            "post_url": "http:\/\/derekg.org\/post\/5578378101/otis-redding",
            "type": "audio",
            "date": "2011-05-17 16:21:05 GMT",
            "timestamp": 1305649265,
            "format": "html",
            "reblog_key": "Wa65rR5l",
            "tags": [],
            "note_count": 3,
            "source_url": "http:\/\/soundcloud.com\/mariam-cabal\/otis-redding",
            "source_title": "SoundCloud \/ Mariam Caballero",
            "id3_title": "Otis Redding - Cigarettes And Coffee",
            "caption": "<p>Otis Redding never fails me.\u00a0<\/p>",
            "embed": "<iframe class=\"tumblr_audio_player tumblr_audio_player_5578378101\" src=\"http:\/\/derekg.org\/post\/5578378101/otis-redding/audio_player_iframe" frameborder=\"0\" allowtransparency=\"true\" scrolling=\"no\" width=\"540\" height=\"85\"></iframe>",
            "plays": 1576
         }
      ],
      "total_posts": 1
   }
}
```

#### Legacy Video Posts

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **caption** | String | The user-supplied caption | |
| **player** | Array of embed objects | Object fields within the array: | Values vary by video source |
| **width** | Number | Width of video player, in pixels | |
| **embed_code** | String | HTML for embedding the video player | |

**Example**

`https://api.tumblr.com/v2/blog/john.io/posts/video?api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "john",
            "id": 6522991678,
            "post_url": "http:\/\/john.io\/post\/6522991678",
            "type": "video",
            "date": "2011-06-14 15:51:21 GMT",
            "timestamp": 1308066681,
            "format": "html",
            "reblog_key": "sWRdVJrI",
            "tags": [],
            "note_count": 17,
            "source_url": "http:\/\/www.WatchMojo.com",
            "source_title": "WatchMojo.com",
            "caption": "<p><a href=\"http:\/\/foreverneilyoung.tumblr.com\/
               post\/6522738445\" target=\"_blank\">foreverneilyoung<\/a>:
               <\/p>\n<blockquote>\n<p><a href=\"http:\/\/watchmojo.tumblr.com\/
               post\/6521201320\" target=\"_blank\">watchmojo<\/a>:<\/p>\n
               <blockquote>\n<p>Neil Young\u2019s live album \u201cA Treasure\
               u201d is available today. To celebrate, we take a look at the
               life and career of the Canadian singer-songwriter.
               <\/p>\n<\/blockquote>\n<p>Neil 101 for you new fans out
               there.<\/p>\n<\/blockquote>\n<p><strong>If you don't
               know\/appreciate Neil Young's impressive body of work,
               this will help<\/strong><\/p>",
            "player": [
               {
                  "width": 250,
                  "embed_code": "<object width=\"248\" height=\"169\"><param
                     name=\"movie\" value=\"http:\/\/www.youtube.com\/
                     v\/4Q1aI7xPo0Y&rel=0&egm=0&
                     showinfo=0&fs=1\"><\/param><param name=\"wmode\"
                     value=\"transparent\"><\/param><param name=\"
                     allowFullScreen\" value=\"true\"><\/param><embed
                     src=\"http:\/\/www.youtube.com\/v\/
                     4Q1aI7xPo0Y&rel=0&egm=0&showinfo=
                     0&fs=1\" type=\"application\/x-shockwave-flash\"
                     width=\"248\" height=\"169\" allowFullScreen=\"true\"
                     wmode=\"transparent\"><\/embed><\/object>"
               },
               {
                  "width": 400,
                  "embed_code": "<object width=\"400\" height=\"251\">
                     <param name=\"movie\" value=\"http:\/\/www.youtube.com\/
                     v\/4Q1aI7xPo0Y&rel=0&egm=0&showinfo=0&fs=1\"><\/
                     param><param name=\"wmode\" value=\"transparent
                     \"><\/param><param name=\"allowFullScreen\" value=
                     \"true\"><\/param><embed src=\"http:\/\/www.youtube.com
                     \/v\/4Q1aI7xPo0Y&rel=0&egm=0&
                     showinfo=0&fs=1\" type=\"application\/
                     x-shockwave-flash\" width=\"400\" height=\"251\"
                     allowFullScreen=\"true\" wmode=\"transparent\">
                     <\/embed><\/object>"
               },
               {
                  "width": 500,
                  "embed_code": "<object width=\"500\" height=\"305\"><param
                     name=\"movie\" value=\"http:\/\/www.youtube.com\/
                     v\/4Q1aI7xPo0Y&rel=0&egm=0&
                     showinfo=0&fs=1\"><\/param><param name=\"wmode\"
                     value=\"transparent\"><\/param><param name=\"
                     allowFullScreen\" value=\"true\"><\/param><embed
                     src=\"http:\/\/www.youtube.com\/v\/4Q1aI7xPo0Y&
                     rel=0&egm=0&showinfo=0&fs=1\"
                     type=\"application\/x-shockwave-flash\" width=\"500\"
                     height=\"305\" allowFullScreen=\"true\"
                     wmode=\"transparent\"><\/embed><\/object>"
               }
            ]
         },
         ...
      ],
      "total_posts": 48
   }
}
```

#### Legacy Answer Posts

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **asking_name** | String | The blog that sent this ask, or answered it if it was privately answered |
| **asking_url** | String | The blog URL that sent this ask, or answered it if it was privately answered |
| **question** | String | The question being asked |
| **answer** | String | The answer given |

**Example**

`https://api.tumblr.com/v2/blog/david.tumblr.com/posts?id=7504154594&api_key={key}`

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blog": { ... },
      "posts": [
         {
            "blog_name": "david",
            "id": 7504154594,
            "post_url": "http://www.davidslog.com/7504154594",
            "type": "answer",
            "date": "2011-07-11 20:24:14 GMT",
            "timestamp": 1310415854,
            "format": "html",
            "reblog_key": "HNvqLd5G",
            "tags": [],
            "asking_name": "aperfectfacade",
            "asking_url": "http://aperfectfacade.tumblr.com/",
            "question": "I thought Tumblr started in 2007, yet you
               have posts from 2006?",
            "answer": "<p>Good catch! Tumblr <strong>launched</strong> in
               February 2007. We were testing it for a few months before
               then.</p>\n<p><strong>Tumblr Trivia:</strong> Before Tumblr,
               my blog (davidslog.com) was a manually edited, single page,
               HTML tumblelog.</p>"
         }
      ],
      "total_posts": 1
   }
}
```

#### Neue Post Format (NPF) Posts

Posts that have `type: blocks` and/or `is_blocks_post_format: true` will have three fields for the post's content:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **content** | Array | The content of the post. |
| **layout** | Array | The layout of the post content. |
| **trail** | Array | The reblog trail items, if any. |

The specification for what objects you can find in these fields is [documented here](npf-spec.md).

### `/posts/queue` — Retrieve Queued Posts

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/queue` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **offset** | String | Post number to start at | 0 (first post) | No |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |

### `/posts/draft` — Retrieve Draft Posts

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/draft` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **before_id** | Number | Return posts that have appeared before this ID; Use this parameter to page through the results: first get a set of posts, and then get posts since the last ID of the previous set. | 0 | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |

### `/posts/submission` — Retrieve Submission Posts

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/submission` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **offset** | String | Post number to start at | 0 (first post) | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **id** | Number | The ID of the submitted post |
| **post_url** | String | The location of the post |
| **slug** | String | Short text summary to the end of the post URL |
| **type** | String | The type of post. One of the following: text, photo, quote, link, video |
| **date** | String | The GMT date and time of the post |
| **timestamp** | Number | The time of the post, in seconds since the epoch |
| **state** | String | Indicates the current state of the post (submission) |
| **format** | String | Format type of post. |
| **reblog_key** | String | The reblog key for the post |
| **tags** | Array | Tags applied to the post |
| **short_url** | String | Short url for the post |
| **post_author** | String | Author of post, only available when submission is not anonymous |
| **is_submission** | Boolean | Indicates post is a submission (true) |
| **anonymous_name** | String | Name on an anonymous submission |
| **anonymous_email** | String | Email on an anonymous submission |

### `/post` — Create a New Blog Post (Legacy)

These legacy posting flows are still available, but we encourage you to use the [Neue Post Format creation route](#posts---createreblog-a-post-neue-post-format).

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post` | POST | [OAuth](#authentication) |

#### Request Parameters

These parameters are used for `/post`, `/post/edit`, and `/post/reblog` methods.

**Fields available for all Post types:**

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **type** | String | The type of post to create. Specify one of the following:  text, photo, quote, link, chat, audio, video | text | Yes |
| **state** | String | The state of the post. Specify one of the following:  published, draft, queue, private | published | No |
| **tags** | String | Comma-separated tags for this post | None | No |
| **tweet** | String | Manages the autotweet (if enabled) for this post: set to off for no tweet, or enter text to override the default tweet | None | No |
| **date** | String | The GMT date and time of the post, as a string | The date and time of the POST request | No |
| **format** | String | Sets the format type of post. Supported formats are: html & markdown | html | No |
| **slug** | String | Add a short text summary to the end of the post URL | Dynamically generated if enabled on the blog | No |
| **native_inline_images** | Boolean | Convert any external image URLs to Tumblr image URLs | false | No |

#### Text Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **title** | String | The optional title of the post, HTML entities must be escaped | None | No |
| **body** | String | The full post body, HTML allowed | None | Yes |

#### Photo Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **caption** | String | The user-supplied caption, HTML allowed | None | No |
| **link** | String | The "click-through URL" for the photo | None | No |
| **source** | String | The photo source URL | None | Yes, either source or `data` or `data64` |
| **data** | Array (URL-encoded binary contents) | One or more image files (submit multiple times to create a slide show), limit 10MB | None | Yes, either `source` or `data` or `data64` |
| **data64** | String | The contents of an image file encoded using base64, limit 10MB | None | Yes, either `source` or `data` or `data64` |

#### Quote Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **quote** | String | The full text of the quote, HTML entities must be escaped | None | Yes |
| **source** | String | Cited source, HTML allowed | None | No |

#### Link Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **title** | String | The title of the page the link points to, HTML entities should be escaped | None | No |
| **url** | String | The link | None | Yes |
| **description** | Strin | A user-supplied description, HTML allowed | None | No |
| **thumbnail** | String | The url of an image to use as a thumbnail for the post | None | No |
| **excerpt** | String | An excerpt from the page the link points to, HTML entities should be escaped | None | No |
| **author** | String | The name of the author from the page the link points to, HTML entities should be escaped | None | No |

#### Chat Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **title** | String | The title of the chat | None | No |
| **conversation** | String | The text of the conversation/chat, with dialogue labels (no HTML) | None | Yes |

#### Audio Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **caption** | String | The user-supplied caption | None | No |
| **external_url** | String | The URL of the site that hosts the audio file (not tumblr) | None | Yes, either external_url or data |
| **data** | String (URL-encoded binary contents) | An audio file, limit 10MB | None | Yes, either external_url or data |

#### Video Posts

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **caption** | String | The user-supplied caption | None | No |
| **embed** | String | HTML embed code for the video or a URI to the video. If you provide an unsupported service's URI you may receive a 400 response. | None | Yes, either a URI, embed, or data. |
| **data** | String (URL-encoded binary contents) | A video file, limit 100MB | None | Yes, either embed or data |

#### Response

Returns `201: Created` or an error code.

### `/post/edit` – Edit a Blog Post (Legacy)

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post/edit` | POST | [OAuth](#authentication) |

#### Request Parameters

These parameters are in addition to the common parameters listed under `/post`.

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to edit | None | Yes |

#### Response

Returns `200: OK` (successfully edited) or an error code.

### `/post/reblog` – Reblog a Post (Legacy)

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post/reblog` | POST | [OAuth](#authentication) |

#### Request Parameters

These parameters are in addition to the common parameters listed under [`/post`](#post--create-a-new-blog-post-legacy).

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the reblogged post on tumblelog | None | Yes |
| **reblog_key** | Number | The reblog key for the reblogged post – get the reblog key with a /posts request | None | Yes |
| **comment** | String | A comment added to the reblogged post | None | No |
| **native_inline_images** | Boolean | Convert external image URLs in the comment to Tumblr image URLs | false | No |

#### Response

Returns `201: Created` or an error code.

### `/posts` - Create/Reblog a Post (Neue Post Format)

This route allows you to create posts (and reblogs) using the Neue Post Format. The specification for NPF is [here](npf-spec.md).

#### Note about Post States

Posts can be in the following "states" as indicated in requests to the post creation/editing endpoints:

- `"published"` means the post should be publicly published immediately.
- `"queue"` means the post should be added to the end of the blog's post queue.
- `"draft"` means the post should be saved as a draft.
- `"private"` means the post should be privately published immediately.

If omitted, the state parameter on a new post defaults to `"published"`.

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts` | POST | [OAuth](#authentication) |

### Request Parameters

**For creating a new post or reblog,** the following parameters are expected as a JSON object in the request body:

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **content** | Array | An array of [NPF content blocks](npf-spec.md) to be used to make the post. | N/A | Yes |
| **layout** | Array | An array of [NPF layout objects](npf-spec.md) to be used to lay out the post content. | `[]` | No |
| **state** | String | The initial [state of the new post](#note-about-post-states), such as "published" or "queued". | `"published"` | No |
| **publish_on** | String | The exact date and time (ISO 8601 format) to publish the post, if desired. This parameter will be ignored unless the `state` parameter is `"queue"`. | Now | No |
| **tags** | String | A comma-separated list of tags to associate with the post. | None | No |
| **source_url** | String | A source attribution for the post content. | None | No |
| **send_to_twitter** | Boolean | Whether or not to share this via any connected Twitter account on post publish. Defaults to the blog's global setting. | `false` | No |
| **send_to_facebook** | Boolean | Whether or not to share this via any connected Facebook account on post publish. Defaults to the blog's global setting. | `false` | No |

**If the post being created is a reblog, all of the above parameters are expected, along with:**

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **parent_tumblelog_uuid** | String | The unique public identifier of the Tumblelog that's being reblogged from. | N/A | Yes |
| **parent_post_id** | Integer | The unique public post ID being reblogged. | N/A | Yes |
| **reblog_key** | String | The unique per-post hash validating that this is a genuine reblog action. | N/A | Yes |
| **hide_trail** | Boolean | Whether or not to hide the reblog trail with this new post. Defaults to false. | `false` | No |

#### User Uploaded Media

In order to support user uploaded media (photos, videos, etc.), the creation and edit routes also support a multi-part form request where the first part contains the JSON body and subsequent parts contain media data.

To specify which media data pertains to which block, we use a unique identifier in the JSON body and this identifier is used as the key in the form data.

```JSON
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/jpeg",
                    "identifier": "some-identifier",
                    "width": 250,
                    "height": 200
                }
            ]
        }
    ]
}
```

```
--TumblrBoundary
Content-Disposition: form-data; name="json"
Content-Type: application/json

{JSON encoded parameters}
--TumblrBoundary
Content-Disposition: form-data; name="some-identifier"; filename="filename.jpg"
Content-Type: image/jpeg

{image data}
--TumblrBoundary--
```

Uploading a video for a `video` content block follows the same `identifier` pattern as above:

```JSON
{
    "content": [
        {
            "type": "video",
            "media": {
                "type": "video/mp4",
                "identifier": "some-identifier",
                "width": 250,
                "height": 200
            }
        }
    ]
}
```

Normally we would require a `provider` field inside the video block, but on post creation this is assumed to be `tumblr` if none is provided.

### Response

This returns a `201 Created` on successful post creation, or an error code. The response JSON object will contain the `id` of the created post, which is intentionally a string instead of an integer for 32bit device compatibility.

**Example response:**

```JSON
{
    "meta": {
        "msg": "Created",
        "status": 201
    },
    "response": {
        "id": "1234567891234567"
    }
}
```

#### Errors and Error Subcodes

The error codes here are the HTTP statuses that'll be returned in error states, the error subcodes are the specific subcodes also returned in some situations.

- `400 Bad Request` when the request has not provided the required data.
    - `400.8001` when an NPF JSON parameter is invalid or a bad format.
    - `400.8002` when the reblog parent post and/or blog is invalid or cannot be found.
    - `400.8005` when the uploaded media is an invalid format we cannot accept.
- `401 Unauthorized` when the requester is an unauthorized client.
- `403 Forbidden` when the requester is not allowed to reblog the parent post they specified.
    - `403.8004` when the requesting user cannot upload more media today.
    - `403.8008` when trying to upload a video in reblog content, which is not allowed.
    - `403.8010` when there is a video upload still transcoding, so you can't upload another video yet.
    - `403.8011` when the requesting user cannot upload more videos today.
- `404 Not Found` when the request is not HTTPS.
- `404 Not Found` when the Tumblelog identifier in the path does not resolve to an existing blog.
- `404 Not Found` when the given post ID for editing/fetching doesn't exist.
- `500 Internal Server Error` when something has gone very wrong.
    - `500.8006` when there was an unknown upload error (not video related).
    - `500.8009` when there was an unknown upload error (video related).
- `503 Service Unavailable` when all posting, or posting via the API, is disabled.

For example:

```JSON
{
    "meta": {
        "msg": "Bad Request",
        "status": 400
    },
    "errors": [
        {
            "title": "'content' must be an array'",
            "code": 8001
        }
    ]
}
```

### `/posts/{post-id}` - Fetching a Post (Neue Post Format)

This does NOT fetch _any_ post and return it in the Neue Post Format. This route fetches only single posts created in the Neue Post Format, with the intention of editing it. The specification for NPF is [here](npf-spec.md).

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}` | GET | [OAuth](#authentication) |

### Response

This returns a `200 OK` on successful post fetching, or an error code.

The response JSON object will contain:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **id** | String | The post ID, intentionally a string instead of an integer, for 32bit device compatibility. |
| **tumblelog_uuid** | String | The posting blog's unique identifier. |
| **parent_post_id** | String | The parent post ID, if the post being fetched is a reblog. |
| **parent_tumblelog_uuid** | String | The parent posting blog's UUID, if the post being fetched is a reblog. |
| **reblog_key** | String | The reblog key needed as an additional authentication token. |
| **trail** | Array | Array of trail items, if the post being fetched is a reblog. |
| **content** | Array | Array of the content blocks of the post itself. |
| **layout** | Array | Array of the post's layout objects. |

**Example response:**

An example response of an original post being fetched:

```JSON
{
    "meta": {
        "msg": "OK",
        "status": 200
    },
    "response": {
        "id": "1234567891234567",
        "tumblelog_uuid": "t:1234abcdef",
        "content": [
            {
                "type": "text",
                "text": "ello i'm oli"
            }
        ],
        "layout": []
    }
}
```

An example response of a reblogged post being fetched:

```JSON
{
    "meta": {
        "msg": "OK",
        "status": 200
    },
    "response": {
        "id": "1234567891234567",
        "tumblelog_uuid": "t:1234abcdef",
        "parent_post_id": "987654321",
        "parent_tumblelog_uuid": "t:ello9876",
        "reblog_key": "elloelloelloello",
        "trail": [],
        "content": [
            {
                "type": "text",
                "text": "ello i'm oli"
            }
        ],
        "layout": []
    }
}
```

#### Errors and Error Subcodes

See the notes from the NPF Post creation route for info about this.

### `/posts/{post-id}` - Editing a Post (Neue Post Format)

This route allows you to edit posts using the Neue Post Format. Note that you can only edit posts in NPF if they were originally created in NPF, or are legacy text posts. The specification for NPF is [here](npf-spec.md).

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}` | PUT | [OAuth](#authentication) |

### Request Parameters

For editing a post, all of the request parameters from the NPF Post Creation route are expected (depending on if it's an original post or reblog), along with the Post's ID in the query path.

### Response

This returns a `200 OK` on successful post editing, or an error code. The response JSON object will contain the `post_id` of the edited post, which is intentionally a string instead of an integer for 32bit device compatibility.

**Example response:**

```JSON
{
    "meta": {
        "msg": "OK",
        "status": 200
    },
    "response": {
        "id": "1234567891234567"
    }
}
```

#### Errors and Error Subcodes

See the notes from the NPF Post creation route for info about this.

### `/post/delete` – Delete a Post

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post/delete` | POST | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to delete | N/A | Yes |

#### Response

Returns `200: OK` (successfully deleted) or an error code.

## User Methods

### `/user/info` – Get a User's Information

Use this method to retrieve the user's account information that matches the OAuth credentials submitted with the request.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/info` | GET | [OAuth](#authentication) |

#### Response

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| **following** | Number | The number of blogs the user is following |
| **default_post_format** | String | The default posting format - `html`, `markdown`, or `raw` |
| **name** | String | The user's tumblr short name |
| **likes** | Number | The total count of the user's likes |
| **blogs** | Array | Each item is a blog the user has permissions to post to, containing these fields: |
| **blogs.name** | String | the short name of the blog |
| **blogs.url** | String | the URL of the blog |
| **blogs.title** | String | the title of the blog |
| **blogs.primary** | Boolean | indicates if this is the user's primary blog |
| **blogs.followers** | Number | total count of followers for this blog |
| **blogs.tweet** | String | indicate if posts are tweeted auto, `Y`, `N` |
| **blogs.facebook** | String | indicate if posts are sent to facebook `Y`, `N` |
| **blogs.type** | String | indicates whether a blog is `public` or `private` |

**Example**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
     "user": {
       "following": 263,
       "default_post_format": "html",
       "name": "derekg",
       "likes": 606,
       "blogs": [
          {
           "name": "derekg",
           "title": "Derek Gottfrid",
           "url": "http://derekg.org/",
           "tweet": "auto",
           "primary": true,
           "followers": 33004929,
          },
          {
           "name": "ihatehipstrz",
           "title": "I Hate Hipstrz",
           ...
          }
        ]
     }
  }
}
```

### `/user/dashboard` – Retrieve a User's Dashboard

Use this method to retrieve the dashboard that matches the OAuth credentials submitted with the request.

> ⚠️ Note: Please don't re-implement the Dashboard, and don't recreate complete Tumblr functions or clients on a platform where Tumblr already has an official client. See our API policies [here](https://www.tumblr.com/docs/api_agreement).

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/dashboard` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Post number to start at | 0 (first post) | No |
| **type** | String | The type of post to return. Specify one of the following:  text, photo, quote, link, chat, audio, video, answer | None – return all types | No |
| **since_id** | Number | Return posts that have appeared after this ID; Use this parameter to page through the results: first get a set of posts, and then get posts since the last ID of the previous set. | 0 | No |
| **reblog_info** | Boolean | Indicates whether to return reblog information (specify true or false). Returns the various `reblogged_` fields. | False | No |
| **notes_info** | Boolean | Indicates whether to return notes information (specify true or false). Returns note count and note metadata. | False | No |

#### Response

Dashboard responses include the fields returned in `/posts` [responses](#post--create-a-new-blog-post-legacy) (with all the various type-specific fields), but without the blog object. Instead, a blog_name field identifies the blog for each post returned.

**Example**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "posts": [
         {
            "blog_name": "laughingsquid",
            "id": 6828225215,
            "post_url": "http:\/\/links.laughingsquid.com\/post\/6828225215",
            ...
         }
      ]
   }
}
```

### `/user/likes` — Retrieve a User's Likes

Use this method to retrieve the liked posts that match the OAuth credentials submitted with the request.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/likes` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Liked post number to start at | 0 (first post) | No |
| **before** | Integer | Retrieve posts liked before the specified timestamp | None | No |
| **after** | Integer | Retrieve posts liked after the specified timestamp | None | No |

> - You can only provide either before, after, or offset. If you provide more than one of these options together you will get an error.
> - You can still use limit with any of those three options to limit your result set.
> - When using the offset parameter the maximum limit on the offset is 1000. If you would like to get more results than that use either before or after.

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **liked_posts** | Array | An array of post objects (posts liked by the user) |
| **liked_count** | Number | Total number of liked posts |

### `/user/following` – Retrieve the Blogs a User Is Following

Use this method to retrieve the blogs followed by the user whose OAuth credentials are submitted with the request.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/following` | GET | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Result number to start at | 0 (first followee) | No |

#### Response

These fields are wrapped in a `blog` object:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **total_blogs** | Number | The number of blogs the user is following |
| **blogs** | Array | Each item is a blog that's being followed, containing these fields: |
| **name** | String | the user name attached to the blog that's being followed |
| **url** | String | the URL of the blog that's being followed |
| **updated** | Number | the time of the most recent post, in seconds since the epoch |
| **title** | String | the title of the blog |
| **description** | String | the description of the blog |

**Example**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
       "total_blogs": 259,
       "blogs": [
          {
             "name": "david",
             "url": "http:\/\/www.davidslog.com",
             "updated": 1308781073,
             "title": "David’s Log",
             "description": "“Mr. Karp is tall and skinny, with unflinching blue eyes and a mop of brown hair. He speaks incredibly fast and in complete paragraphs.” – NY Observer"
          },
          {
             "name": "matthew",
             "url": "http:\/\/matthew.tumblr.com",
             "updated": 1306436424,
             "title": "Matt Hackett",
             "description": " I obsess over the engineering and dissemination of technology for creative people.

\nI'm happily torn between work on the visible (marketing) and the invisible (architecture, management) components of media-product-making machinery.\n

\nMost recently, I saw as Tumblr's team grew from 10 to 110 and monthly audience from 25 million to 150 million from my vantage point as VP Engineering and later Head of Brand Strategy.

\nMore about Matt Hackett"
          },
          ...
       ]
   }
}
```

### `/user/follow` – Follow a blog

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/follow` | POST | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| url | String | The URL of the blog to follow | None | Yes |

#### Response

Returns `200: OK` (blog successfully followed) or a 404 (blog was not found)

### `/user/unfollow` – Unfollow a blog

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/unfollow` | POST | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **url** | String | The URL of the blog to unfollow | None | Yes |

#### Response

Returns `200: OK` (blog successfully unfollowed) or a `404` (if the blog was not found)

### `/user/like` – Like a Post

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/like` | POST | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to like | None | Yes |
| **reblog_key** | String | The reblog key for the post id | None | Yes |

#### Response

Returns `200: OK` (post successfully liked ) or a `404` (post ID or `reblog_key` was not found)

### `/user/unlike` – Unlike a Post

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/unlike` | POST | [OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to unlike | None | Yes |
| **reblog_key** | String | The reblog key for the post id | None | Yes |

#### Response

Returns `200: OK` (post successfully unliked ) or a `404` (post ID or `reblog_key` was not found)

## Tagged Method

### `/tagged` – Get Posts with Tag

**Examples**

- `https://api.tumblr.com/v2/tagged?tag=gif`
- `https://api.tumblr.com/v2/tagged?tag=lol`

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/tagged` | GET | [API Key or OAuth](#authentication) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **tag** | String | The tag on the posts you'd like to retrieve | None | Yes |
| **before** | Integer | The timestamp of when you'd like to see posts before. If the Tag is a "featured" tag, use the "featured_timestamp" on the post object for pagination. | Current timestamp | No |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML; | None (HTML) | No |

