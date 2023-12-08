# Tumblr API

Welcome to the Tumblr API! There isn't anything we enjoy more than seeing talented designers and engineers using Tumblr to invent whole new forms of creative expression. We've put a tremendous amount of care into making this API functional and flexible enough for any projects you throw at it. Drop a line in the [GitHub Issues](https://github.com/tumblr/docs/issues) if you have ideas for what could be better, or want to discuss all the amazing things you're going to build with it. Follow our [Engineering Blog](https://engineering.tumblr.com/) for important news and announcements. Please use the API responsibly and read the [Application Developer and API License Agreement](https://www.tumblr.com/docs/api_agreement). If you run into problems with the Tumblr API, please use [our official support form](https://www.tumblr.com/support) to request help. Enjoy!

Tumblr also supports an API that delivers content according to the [oEmbed standard](http://oembed.com/). Our oEmbed API endpoint is `https://www.tumblr.com/oembed`, which supports post URLs in the format `https://*.tumblr.com/post/*`.

If you're looking for documentation for the old v1 API, you can find it [here](https://www.tumblr.com/docs/api/v1).

## Table of Contents

- [What You Need](#what-you-need)
- [Console](#console)
- [API Overview](#api-overview)
    - [URI Structure](#uri-structure)
    - [Request User Agent](#request-user-agent)
    - [Request Content Types](#request-content-types)
    - [Post Identifiers](#post-identifiers)
    - [Blog Identifiers](#blog-identifiers)
    - [Response Format](#response-format)
    - [Rate Limits](#rate-limits)
    - [Reporting Issues](#reporting-issues)
    - [About the API Documentation](#about-the-api-documentation)
- [Official Tumblr API Client Libraries](#official-tumblr-api-client-libraries)
- [Authentication](#authentication)
- [OAuth1 Authorization](#oauth1-authorization)
    - [Temporary Credentials Endpoint](#temporary-credentials-endpoint)
    - [Resource Owner Authorization Endpoint](#resource-owner-authorization-endpoint)
    - [Access Token Endpoint](#access-token-endpoint)
- [OAuth2 Authorization](#oauth2-authorization)
    - [`/oauth2/authorize` - Authorization Request](#oauth2authorize---authorization-request)
    - [`/v2/oauth2/token` - Authorization Code Grant Request](#v2oauth2token---authorization-code-grant-request)
    - [`/v2/oauth2/token` - Refresh Token Grant Request](#v2oauth2token---refresh-token-grant-request)
    - [`/v2/oauth2/exchange` - OAuth1 to OAuth2 Token Exchange](#v2oauth2exchange---oauth1-to-oauth2-token-exchange)
- [Common Response Elements](#common-response-elements)
    - [Links](#links)
    - [Tag Objects](#tag-objects)
    - [Neue Post Format objects](#neue-post-format-objects)
- [Blog Methods](#blog-methods)
    - [`/info` - Retrieve Blog Info](#info---retrieve-blog-info)
    - [`/avatar` — Retrieve a Blog Avatar](#avatar--retrieve-a-blog-avatar)
    - [`/blocks` – Retrieve Blog's Blocks](#blocks--retrieve-blogs-blocks)
    - [`/blocks` – Block a Blog](#blocks--block-a-blog)
    - [`/blocks/bulk` – Block a list of Blogs](#blocksbulk--block-a-list-of-blogs)
    - [`/blocks` – Remove a Block](#blocks--remove-a-block)
    - [`/likes` — Retrieve Blog's Likes](#likes--retrieve-blogs-likes)
    - [`/following` — Retrieve Blog's following](#following--retrieve-blogs-following)
    - [`/followers` — Retrieve a Blog's Followers](#followers--retrieve-a-blogs-followers)
    - [`/followed_by` — Check If Followed By Blog](#followed_by--check-if-followed-by-blog)
    - [`/posts` – Retrieve Published Posts](#posts--retrieve-published-posts)
    - [`/posts/queue` — Retrieve Queued Posts](#postsqueue--retrieve-queued-posts)
    - [`/posts/queue/reorder` — Reorder Queued Posts](#postsqueuereorder--reorder-queued-posts)
    - [`/posts/queue/shuffle` — Shuffle Queued Posts](#postsqueueshuffle---shuffle-queued-posts)
    - [`/posts/draft` — Retrieve Draft Posts](#postsdraft--retrieve-draft-posts)
    - [`/posts/submission` — Retrieve Submission Posts](#postssubmission--retrieve-submission-posts)
    - [`/notifications` — Retrieve Blog's Activity Feed](#notifications--retrieve-blogs-activity-feed)
    - [`/post` — Create a New Blog Post (Legacy)](#post--create-a-new-blog-post-legacy)
    - [`/post/edit` – Edit a Blog Post (Legacy)](#postedit--edit-a-blog-post-legacy)
    - [`/post/reblog` – Reblog a Post (Legacy)](#postreblog--reblog-a-post-legacy)
    - [`/posts` - Create/Reblog a Post (Neue Post Format)](#posts---createreblog-a-post-neue-post-format)
    - [`/posts/{post-id}` - Fetching a Post (Neue Post Format)](#postspost-id---fetching-a-post-neue-post-format)
    - [`/posts/{post-id}` - Editing a Post (Neue Post Format)](#postspost-id---editing-a-post-neue-post-format)
    - [`/post/delete` – Delete a Post](#postdelete--delete-a-post)
    - [`/posts/{post-id}/mute` - Muting a Post's Notifications](#postspost-idmute---muting-a-posts-notifications)
    - [`/notes` - Get notes for a specific Post](#notes---get-notes-for-a-specific-post)
- [User Methods](#user-methods)
    - [`/user/info` – Get a User's Information](#userinfo--get-a-users-information)
    - [`/user/limits` – Get a User's Limits](#userlimits--get-a-users-limits)
    - [`/user/dashboard` – Retrieve a User's Dashboard](#userdashboard--retrieve-a-users-dashboard)
    - [`/user/likes` — Retrieve a User's Likes](#userlikes--retrieve-a-users-likes)
    - [`/user/following` – Retrieve the Blogs a User Is Following](#userfollowing--retrieve-the-blogs-a-user-is-following)
    - [`/user/follow` – Follow a blog](#userfollow--follow-a-blog)
    - [`/user/unfollow` – Unfollow a blog](#userunfollow--unfollow-a-blog)
    - [`/user/like` – Like a Post](#userlike--like-a-post)
    - [`/user/unlike` – Unlike a Post](#userunlike--unlike-a-post)
    - [`/user/filtered_tags` - Tag Filtering](#userfiltered_tags---tag-filtering)
    - [`/user/filtered_content` - Content Filtering](#userfiltered_content---content-filtering)
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

### Request User Agent

The `User-Agent` header is required and every application is expected to use a consistent value across requests. Your application may be suspended if it uses a variety of values for the `User-Agent` header.

### Request Content Types

Unless otherwise noted, Tumblr API endpoints that accept body parameters expect the body parameters to be encoded using one of the following content types:
- `application/json`
- `application/x-www-form-urlencoded`
- `multipart/form-data`

Note that the `Content-Type` header must be set accordingly, and the absence of a `Content-Type` header may result in unexpected behavior.

### Post Identifiers

Please note that Tumblr's Post IDs are 64-bit integers. Some languages, like Javascript, don't handle 64-bit numbers very well. With that in mind, there is also an `id_string` field available so you can handle these safely.

### Blog Identifiers

Each blog has a few different unique identifiers that can be used to reference that specific blog in the API payload. They are interchangeable and can be used in any place that `{blog-identifier}` appears in these API docs. The three types of blog identifiers are:

- **Blog name**: The name of the blog that appears in the URL. Examples:
  - `staff` for `https://staff.tumblr.com/`
  - `changes` for `https://www.tumblr.com/changes` or `https://www.tumblr.com/@changes`
- [**Hostname**](#blog-unique-hostnames): The hostname that's used to access the blog, excluding `www.tumblr.com`. Examples:
  - `staff.tumblr.com` for `https://staff.tumblr.com/`
  - `changes.tumblr.com` for `https://www.tumblr.com/changes` (*not* `www.tumblr.com`)
  - `www.davidslog.com` for `https://www.davidslog.com/`
- [**Universally unique identifier**](#blog-unique-identifiers): The UUID of the blog [retrieved from the API](#info---retrieve-blog-info). Examples:
  - `t:0aY0xL2Fi1OFJg4YxpmegQ` for `staff`, `staff.tumblr.com`, and even `t:0aY0xL2Fi1OFJg4YxpmegQ`
  - `t:WX5xItRbCiJH_GJDsIdhAQ` for `changes`, `changes.tumblr.com`, and even `t:WX5xItRbCiJH_GJDsIdhAQ`

#### Blog Unique Hostnames

Each blog has a unique hostname which can be used as its **identifier**. The hostname can be **standard** or **custom**.

- **Standard hostname**: the blog short name + `.tumblr.com`. Example: `greentype.tumblr.com`
- **Custom hostname**: these can be anything at all, as determined by a [DNS CNAME entry](https://help.tumblr.com/hc/en-us/articles/231256548-Custom-domains). Example: `www.davidslog.com`

#### Blog Unique Identifiers

Each blog also has a unique identifier that you can retrieve from any API response that includes a blog, in the `uuid` field (example: `t:0aY0xL2Fi1OFJg4YxpmegQ`). The `{blog-identifier}` placeholder can also be replaced by this unique identifier.

The benefits of using a unique identifier instead of a blog name or hostname are that the unique identifier will not change if the blog name or custom domain changes. It can be used as a stable, persistent identifier for a blog.

However, note that under exceptional circumstances, a unique identifier can change. Your own blog's unique identifier will only be changed with your knowing.

### Response Format

Unless otherwise noted, the API returns JSON-encoded objects (`Content-Type: application/json`) and every response body includes these standard properties:

| Field | Notes |
| ----- | ----- |
| **meta** | The meta object matches the HTTP response message |
| **response** | Endpoint-specific results |

The `meta` object should always contain:

| Field | Notes |
| ----- | ----- |
| **status** | The 3-digit HTTP Status-Code (e.g., `200`) |
| **msg** | The HTTP Reason-Phrase (e.g., `OK`) |

Even 4xx/5xx responses such as `400 Bad Request` or `500 Internal Server Error` should include this information, and the `response` may include more information about why you received the 4xx response.

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

All requests made with HTTP GET are [JSONP](https://en.wikipedia.org/wiki/JSONP) enabled. To use JSONP, append `jsonp=` or `callback=` and the name of your callback function to the request. JSONP requests will always return an HTTP status code of 200 but will reflect the real status code in the meta field of the JSON response.

### Rate Limits

Usage of the Tumblr API is rate limited in a few ways, but we respond with a `429 Limit Exceeded` response whenever a consumer hits one of these limits. There is a global rate limit for all usage of the API per consumer, as well as a few per-feature rate limits, such as how many posts you can make per day. The error message or headers you receive with the `429 Limit Exceeded` response should indicate what limit you've hit.

These rate limits include:

- 300 API calls per minute, per IP address.
- 18,000 API calls per hour, per IP address.
- 432,000 API calls per day, per IP address.
- 1,000 API calls per hour, per consumer key.
- 5,000 API calls per day, per consumer key.
- 250 new published posts (including reblogs) per day, per user.
- 250 images uploaded per day, per user.
- 200 follows per day, per user.
- 1,000 likes per day, per user.
- 10 new blogs per day, per user.
- 20 videos uploaded per day, per user.
- 60 minutes of total video uploaded per day, per user.

Note that these rate limits are based on the Tumblr servers' internal clocks and that the daily limits are per calendar day.

There are also overall limits to a few specific actions, including:

- A blog can only follow 5,000 other blogs at a time.
- A blog can only have 1,000 queued posts at a time.
- You can only filter up to 1,000 tags at a time.

### Reporting Issues

If you run into problems with the Tumblr API, please use [our official support form](https://www.tumblr.com/support) to request help.

### About the API Documentation

These docs include some working examples. Please click them to your heart's content, but don't use the embedded API key for your own nefarious purposes. [Register your application](https://www.tumblr.com/oauth/apps) to get your own API key (your OAuth Consumer Key).

#### URI Conventions

| Notation | Meaning | Example |
| -------- | ------- | ------- |
| **Curly brackets { }** | Required item | `api.tumblr.com/v2/blog/{blog-identifier}/posts` (the [blog identifier](#blog-identifiers) is required) |
| **Square brackets [ ]** | Optional item | `api.tumblr.com/v2/blog/{blog-identifier}/posts[/type]` (specifying the type is optional) |

## Official Tumblr API Client Libraries

Here is a list of official API client which will help you get up and running with your favorite language in no time.

- [Javascript](https://github.com/tumblr/tumblr.js)
- [Ruby](https://github.com/tumblr/tumblr_client)
- [PHP](https://github.com/tumblr/tumblr.php)
- [Java](https://github.com/tumblr/jumblr)
- [Python](https://github.com/tumblr/pytumblr)
- [Objective-C](https://github.com/tumblr/TMTumblrSDK)
- [Go](https://github.com/tumblr/tumblrclient.go)

## Authentication

The API uses three different levels of authentication, depending on the method.

- **None:** No authentication. Anybody can query the method.
- **API key:** Requires an API key. Use your OAuth Consumer Key as your `api_key`.
  - Example: `api_key=PyezS3Q4Smivb24d9SzZGYSuh--IaMfAkE`
  - Get an OAuth key: [register an application](https://www.tumblr.com/oauth/apps)
- **OAuth:** Requires a signed request that meets the [OAuth 1.0a Protocol](https://oauth.net/1/).

Each method description below includes a note about the authentication level.

## OAuth1 Authorization

The API supports the [OAuth 1.0a Protocol](https://oauth.net/1/), accepting parameters via the `Authorization` header, with the `HMAC-SHA1` signature method only.

### Temporary Credentials Endpoint

This route is used to create a temporary oauth1 token to be used for oauth1 authorization. See [https://tools.ietf.org/html/rfc5849#section-2.1](https://tools.ietf.org/html/rfc5849#section-2.1) for further information.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `https://www.tumblr.com/oauth/request_token` | POST | OAuth (client credentials) |

#### Request Parameters

None.

#### Response

Returns `200 OK` or an error code. The response body parameters are url encoded (`Content-Type: application/x-www-form-urlencoded`), and the standard response properties are omitted.

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **oauth_token** | String | The access token to use during the authorization process |
| **oauth_token_secret** | String | The access token secret to use during the authorization process |
| **oauth_callback_confirmed** | String | Indicates whether the protocol is OAuth 1.0a |

### Resource Owner Authorization Endpoint

After retrieving temporary credentials, redirect the user to this endpoint so they can authorize your app. See [https://tools.ietf.org/html/rfc5849#section-2.2](https://tools.ietf.org/html/rfc5849#section-2.2) for further information.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `https://www.tumblr.com/oauth/authorize` | GET | None |

#### Request Parameters

Include these parameters in the query string when you redirect the user.

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **oauth_token** | String | The temporary access token from the temporary credentials request  | N/A | Yes |
| **source** | String | An optional identifier that indicates where the process started | N/A | No |

#### Response

Upon successful authorization, the user is redirected to your callback url with the following parameters in the query string:

- **oauth_token** - The access token retrieved from the temporary credentials request
- **oauth_verifier** - The token required to retrieve an access token
- **source** - The optional identifier that indicates where the process started

### Access Token Endpoint

The endpoint that is used to exchange an `oauth_verifier` token for an access token and secret for the user. See [https://tools.ietf.org/html/rfc5849#section-2.3](https://tools.ietf.org/html/rfc5849#section-2.3) for further information.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `https://www.tumblr.com/oauth/access_token` | GET | OAuth (temporary credentials) |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **oauth_verifier** | String | The token from the authorize redirect query string | N/A | Yes |

#### Response

Returns `200 OK` or an error code. The response body parameters are url encoded (`Content-Type: application/x-www-form-urlencoded`), and the standard response properties are omitted.

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **oauth_token** | String | The user's access token |
| **oauth_token_secret** | String | The user's access token secret |

## OAuth2 Authorization

The API supports the [OAuth 2.0 Protocol](https://oauth.net/2/), accepting a bearer token via the `Authorization` header like so:

```
Authorization: Bearer {access_token}
```

API clients have access to [Authorization Code](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3.1), [Client Credentials](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3.4), and [Refresh Token](https://datatracker.ietf.org/doc/html/rfc6749#section-1.5) grants. Available scopes include `basic`, `write`, and `offline_access`. Note that you must specify a valid OAuth2 redirect URL for OAuth2 to be available to your application. You can find this field by editing [one of your applications](https://www.tumblr.com/oauth/apps).

### The OAuth2 Authorization flow

#### Step One: Redirect a user to Tumblr

Redirect the user to [Tumblr's OAuth2 authorization endpoint](#oauth2authorize---authorization-request). This will prompt the user to allow your application to access the Tumblr API on their behalf.

#### Step Two: Handle the callback request

Once a user decides to allow or disallow your application access to their account, Tumblr will redirect them to the OAuth2 redirect URL that you set when creating your application. If there was an error processing the request, the URL will contain an `error` query parameter. If the request was successful, the URL will contain `code` and `state` query parameters. You must verify the `state` value has the same value that you included when you redirected the user to Tumblr's OAuth2 authorization endpoint, and the `code` value can be used to retrieve an access token for the user.

#### Step Three: Retrieve an access token

Use your applications credentials and the `code` from the previous step to issue a request to [Tumblr's OAuth2 access token endpoint](#v2oauth2token---authorization-code-grant-request). For example, using the `curl` command line tool:

```
curl -F grant_type=authorization_code -F code={code} -F client_id={Your OAuth consumer key} -F client_secret={Your OAuth secret key} https://api.tumblr.com/v2/oauth2/token
```

If you provided a redirect URI in the initial authorization request, you must provide the same redirect URI in the token request (as `redirect_uri`).

A successful request will yield a response that contains the following object:

```json
{
    "access_token": "{access_token}",
    "expires_in": 2520,
    "id_token": false,
    "refresh_token": "{refresh_token}",
    "scope": "write offline_access",
    "token_type": "bearer"
}
```

_Special tip: [jq](https://stedolan.github.io/jq/) is a handy tool for formatting JSON on the command line, you can pipe the output of `curl` through `jq` to easily format the API response, like so:_

```
curl -F ... https://api.tumblr.com/v2/oauth2/token | jq
```

You'll need to store this information somewhere so you can issue requests on behalf of the user, and so you can refresh the access token when it has expired.

#### Step Four: Make your API requests

Using the `access_token`, you can now talk directly to the Tumblr API. For example, this `curl` request will retrieve the [user's account information](#userinfo--get-a-users-information):

```
curl -H 'Authorization: Bearer {access_token}' https://api.tumblr.com/v2/user/info
```

#### Step Five: Refresh an access token

Your application can use the `expires_in` property in the access token response to determine if the user's access token has expired. If it has, you can use [the refresh token to retrieve a new access token and refresh token](#v2oauth2token---refresh-token-grant-request). For example, using `curl` again:

```
curl -F grant_type=refresh_token -F refresh_token={refresh_token} -F client_id={Your OAuth consumer key} -F client_secret={Your OAuth consumer secret} https://api.tumblr.com/v2/oauth2/token
```

### `/oauth2/authorize` - Authorization Request

This request is used to request permission from a user to gain access to their account. Once a user grants your application access, they will be returned to your OAuth2 redirect Url as defined on your [OAuth Application](https://www.tumblr.com/oauth/apps).

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `www.tumblr.com/oauth2/authorize` | GET | N/A |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **client_id** | String | Your OAuth consumer key | n/a | Yes |
| **response_type** | String | Always `code` | n/a | Yes |
| **scope** | String | A space delimited list of scopes | n/a | Yes |
| **state** | String | A unique value used to maintain state | n/a | Yes |
| **redirect_uri** | String | A redirect URI | n/a | Maybe, see note below |

A redirect URI must be supplied when multiple redirect URIs are registered.

### `/v2/oauth2/token` - Authorization Code Grant Request

This request allows you to exchange an authorization code for an access token.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/oauth2/token` | POST | N/A |

#### Request Parameters

| Parameter | Type | Description                                                      | Default | Required? |
| --------- | ---- |------------------------------------------------------------------| ------- |-----------|
| **grant_type** | String | Always `authorization_code`                                      | n/a | Yes       |
| **code** | String | The code recieved at your OAuth2 redirect Url                    | n/a | Yes       |
| **client_id** | String | Your OAuth consumer key                                          | n/a | Yes       |
| **client_secret** | String | Your OAuth consumer secret                                       | n/a | Yes       |
| **redirect_uri** | String | A redirect URI (if it was included in the authorization request) | n/a | No        |

#### Response

- `200` Operation was successful
- `401` Unauthorized

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| access_token | String | The OAuth2 access token |
| expires_in | Int | The access token TTL in seconds  |
| token_type | String | The type of the access token |
| scope | String | The OAuth2 access token scopes |
| refresh_token | String | An OAuth2 refresh token (if `offline_access` scope was requested) |

### `/v2/oauth2/token` - Refresh Token Grant Request

This request is used to exchange an OAuth2 refresh token for a new OAuth2 access token and refresh token.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/oauth2/token` | POST | OAuth2 |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **grant_type** | String | Always a string `refresh_token`	| n/a | Yes |
| **client_id** | String | Your OAuth consumer key | n/a | Yes |
| **client_secret** | String | Your OAuth consumer secret | n/a | Yes |
| **refresh_token** | String | An OAuth2 refresh token | n/a | Yes |

**Example:**

```JSON
{
    "grant_type": "refresh_token",
    "client_id": "...",
    "refresh_token": "{refresh_token goes here}"
}
```

#### Response

- `200` Operation was successful
- `401` Unauthorized, refresh token has expired

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| access_token | String | The OAuth2 access token |
| expires_in | Int | The access token TTL in seconds  |
| token_type | String | The type of the access token |
| scope | String | The OAuth2 access token scopes |
| refresh_token | String | An OAuth2 refresh token |

### `/v2/oauth2/exchange` - OAuth1 to OAuth2 Token Exchange

This route is used to exchange an OAuth1 access token for an OAuth2 access token and refresh token. Clients must expect the OAuth1 token to be invalidated if the exchange is successful.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/oauth2/exchange` | POST | OAuth1 |

#### Request Parameters

None.

#### Response

- `200` Operation was successful
- `400` Incorrect authorization type (requests must use OAuth1)
- `401` Authentication failed

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| access_token | String | The OAuth2 access token |
| expires_in | Int | The access token TTL in seconds  |
| token_type | String | The type of the access token |
| scope | String | The OAuth2 access token scopes (always `basic write`) |
| refresh_token | String | An OAuth2 refresh token |

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

Some routes will return Posts that have `type: blocks` and/or `is_blocks_post_format: true`, which means their content is available in the Neue Post Format. [See the NPF specification docs for more info!](npf-spec.md)

Also, when making any request to our API that returns Posts, you may supply a `npf=true` query parameter to specify that you'd like all of the Posts' content returned in the Neue Post Format rather than the legacy format.

## Blog Methods

### `/info` - Retrieve Blog Info

This method returns general information about the blog, such as the title, number of posts, and other high-level data.

**Examples**

- `https://api.tumblr.com/v2/blog/scipsy.tumblr.com/info`
- `https://api.tumblr.com/v2/blog/good.tumblr.com/info`

#### Method

| URI                                                           | HTTP Method | Authentication             |
|---------------------------------------------------------------|-------------|----------------------------|
| `api.tumblr.com/v2/blog/{blog-identifier}/info?api_key={key}` | GET         | [API Key](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter           | Type   | Description                                                                         | Default | Required? |
|---------------------|--------|-------------------------------------------------------------------------------------|---------|-----------|
| **api_key**         | String | Your OAuth Consumer Key. See [Authentication](#authentication) for more details.    | N/A     | Yes       |
| **fields**          | Object | See next section: "Partial Responses".                                              | N/A     | No        |

#### Partial Responses

Note that `blog` objects via the Tumblr API support "partial responses", which allows the request to specify which fields are returned any time a `blog` object is present by using the `fields` query parameter.

For example, if you would like to always only see the `name` and `updated` fields in `blog` objects:

```
https://api.tumblr.com/v2/blog/david/info?fields[blogs]=name,updated
```

That will return only the `name` and `updated` fields in the `blog` object. You can use this in _any_ API request that includes blog objects, not just this blog info endpoint. Some fields are also "optional", and are preceded in the list by a `?` (encoded to `%3F`).

Those optional fields are _only available upon request_, such as:

| Response Field                  | Type    | Description                                                                                      |
|---------------------------------|---------|--------------------------------------------------------------------------------------------------|
| **is_following_you**            | Boolean | Whether or not the blog is following your primary blog                                           |
| **duration_blog_following_you** | Integer | How long (in seconds) that blog has been following your primary blog (`-1` if they aren't)       |
| **duration_following_blog**     | Integer | How long (in seconds) you've been following that blog (`-1` if you aren't)                       |
| **timezone**                    | String  | The blog's configured timezone, such as "US/Eastern". Only viewable by blog member.              |
| **timezone_offset**             | String  | The blog's configured timezone as a GMT offset such as "GMT+0800". Only viewable by blog member. |

To access these, specify them in the `fields[blogs]` query parameter list, like so:

```
https://api.tumblr.com/v2/blog/david/info?fields[blogs]=name,updated,%3Fis_following_you,%3Fduration_following_blog
```

Which will return a response like:

```json
{
    "meta": {
        "status": 200,
        "msg": "OK"
    },
    "response": {
        "blog": {
            "name": "david",
            "updated": 1591035760,
            "is_following_you": false,
            "duration_following_blog": 123456
        }
    }
}
```

Omitting the `fields` query parameter will return the default set of fields below.

#### Response

*Note: Field ordering below may differ from actual response.* Also, any fields present that aren't included here are not officially supported for third-party use.

| Response Field              | Type    | Description                                                                                                                                       |
|-----------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| **title**                   | String  | The display title of the blog                                                                                                                     |
| **posts**                   | Number  | The total number of posts to this blog                                                                                                            |
| **name**                    | String  | The short blog name that appears before tumblr.com in a standard blog hostname                                                                    |
| **updated**                 | Number  | The time of the most recent post, in seconds since the epoch                                                                                      |
| **description**             | String  | You guessed it! The blog's description                                                                                                            |
| **ask**                     | Boolean | Indicates whether the blog allows questions                                                                                                       |
| **ask_anon**                | Boolean | Indicates whether the blog allows anonymous questions; returned only if ask is true                                                               |
| **followed**                | Boolean | Whether you're following the blog, returned only if this request has an authenticated user                                                        |
| **likes**                   | Number  | Number of likes for this user, returned only if this is the user's primary blog and sharing of likes is enabled                                   |
| **is_blocked_from_primary** | Boolean | Indicates whether this blog has been blocked by the calling user's primary blog; returned only if there is an authenticated user making this call |
| **avatar**                  | Array   | An array of avatar objects, each a different size, which should each have a width, height, and URL.                                               |
| **url**                     | String  | The blog's canonical URL                                                                                                                          |
| **theme**                   | Object  | The blog's general theme options, which may not be useful if the blog uses a custom theme. See next table.                                        |

Specific fields inside of the `theme` object and what they mean:

| Field | Type | Description                                                                                                                                                                                                                        |
| ----- | ---- |------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `avatar_shape` | String | "circle" or "square", this is the shape of the mask over the user's avatar.                                                                                                                                                        |
| `background_color` | String | The intended hex color used for the blog's background color.                                                                                                                                                                       |
| `body_font` | String | The font that the blog has selected as their "body" font.                                                                                                                                                                          |
| `header_bounds` | Mixed | If the blog's header should be cropped, this is a comma-separated list of top/right/bottom/left coordinates to use.                                                                                                                |
| `header_image` | String | The URL of the blog's original, full header image. Note that this may be a default Tumblr header image.                                                                                                                            |
| `header_image_npf`     | Object  | [NPF image block format](https://www.tumblr.com/docs/npf#content-block-type-image) of the blog's header image. It will include one or more image sizes to choose from, in order from largest to smallest.                                                                                  |
| `header_image_focused` | String | If the blog cropped/repositioned their header image, this will be that version, which should be preferred over the original.                                                                                                       |
| `header_image_poster` | String | The URL of a single-frame "poster" version of the blog's header image, if it's an animated image. Note that this may be an empty string if no poster could be made or is not needed.                                               |
| `header_image_scaled` | String | If the blog _only_ scaled their header image, this will be that scaled version. Note that this may be a default Tumblr header image in the case that they scaled _and_ repositioned it, in which case, use the `_focused` version. |
| `header_stretch` | Boolean | Whether or not the blog's header is meant to be stretched to aspect-fill any given space where it's used.                                                                                                                          |
| `link_color` | String | The intended hex color of any links in the blog's description.                                                                                                                                                                     |
| `show_avatar` | Boolean | Whether or not the blog's avatar should be displayed, even if it's given in the API payload.                                                                                                                                       |
| `show_description` | Boolean | Whether or not the blog's description should be displayed, even if it's given in the API payload.                                                                                                                                  |
| `show_header_image` | Boolean | Whether or not the blog's header image should be displayed, even if it's given in the API payload.                                                                                                                                 |
| `show_title` | Boolean | Whether or not the blog's title should be displayed, even if it's given in the API payload.                                                                                                                                        |
| `title_color` | String | The intended hex color of the blog's title.                                                                                                                                                                                        |
| `title_font` | String | The intended font to use when displaying the blog's title.                                                                                                                                                                         |
| `title_font_weight` | String | The intended font weight to use when displaying the blog's title.                                                                                                                                                                  |

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
         "url": "https:\/\/david.tumblr.com\/",
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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |
| **size** | Number | The size of the avatar (square, one value for both length and width). Must be one of the values: 16, 24, 30, 40, 48, 64, 96, 128, 512 | 64 | No |

#### Response

Requests that are _not_ signed using OAuth1 will receive the requested avatar in PNG format, while requests that are signed will receive a response of the following form:

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **avatar_url** | String | The URL of the avatar image. This is also returned in the [Location HTTP header field](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) (see note). | An [HTTP Location header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) field is returned — the response points to the avatar image. That means you can embed this method in an `img` tag in HTML. |

### `/blocks` – Retrieve Blog's Blocks

Get the blogs that the requested blog is currently blocking. The requesting user must be an admin of the blog to retrieve this list.

Note that this endpoint is rate limited to 60 requests per minute.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/blocks` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **offset** | Number | Block number to start at | 0 | No |
| **limit** | Number | The number of blocks to retrieve, 1-20, inclusive | 20 | No |

#### Response

Returns `200 OK` or an error code.

| Response Field | Type | Description | Notes |
| -------------- | ---- | ----------- | ----- |
| **blocked_tumblelogs** | Array | Blog objects that are blocked. | |
| **_links** | Object | A standard `_links` object with pagination objects. | |

**Example response:**

```JSON
{
   "meta": {
      "status": 200,
      "msg": "OK"
   },
   "response": {
      "blocked_tumblelogs": [
         {
            "name": "joe",
            "url": "http:\/\/www.spammyjoe.com",
            "updated": 1308797076,
            "title": "Spammy Joe",
            "description": "Posting things you don't really care for"
         },
         {
            "name": "harry",
            "url": "harryblog19.tumblr.com",
            "updated": 1808841333,
            "title": "The 19th Harry Blog",
            "description": ""
         },
         ...
      ],
      "_links": {
         "next": {
            "href": "/v2/blog/andysblog.tumblr.com/blocks?offset=20",
            "method": "GET",
            "query_params": {
               "offset": 20
            }
         }
      }
   }
}
```

### `/blocks` – Block a Blog

Note that this endpoint is rate limited to 60 requests per minute.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/blocks` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blocked_tumblelog** | String | The tumblelog to block, specified by any blog identifier | N/A | Only if `post_id` isn't passed |
| **post_id** | String | The anonymous post ID (asks, submissions) to block | N/A | Only if `blocked_tumblelog` isn't passed |

#### Response

Returns `201 Created` or an error code.

### `/blocks/bulk` – Block a list of Blogs

Note that this endpoint is rate limited to 60 requests per minute.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/blocks/bulk` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blocked_tumblelogs** | String | Comma-separated list of tumblelogs to block, specified by any blog identifier | N/A | Yes |
| **force** | Bool | Whether to force the block to go through even if it requires canceling a Post+ Subscription | false | No |

#### Response

Returns `200 OK` or an error code.

### `/blocks` – Remove a Block

Note that this endpoint is rate limited to 60 requests per minute.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/blocks` | DELETE | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blocked_tumblelog** | String | The tumblelog whose block to remove, specified by any blog identifier | N/A | Only if `anonymous_only` isn't passed |
| **anonymous_only** | Boolean | When passed without the `blocked_tumblelog` parameter, this will clear all anonymous IP blocks | N/A | Only if `blocked_tumblelog` isn't passed |

- If you pass both params, `anonymous_only` is going to be ignored.
- Passing just `anonymous_only=false` is the equivalent of no params. A `404` response will be returned.

#### Response

Returns `200 OK` if the block on the passed `blocked_tumblelog` parameter was successfully removed.

Returns `200 OK` if all anonymous blocks were successfully removed when `anonymous_only=true` was passed without a `blocked_tumblelog` parameter in the path.

### `/likes` — Retrieve Blog's Likes

This method can be used to retrieve the publicly exposed likes from a blog.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/likes?api_key={key}` | GET | [API Key](#auth) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
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
               "url": "https:\/\/www.davidslog.com",
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

### `/followed_by` — Check If Followed By Blog

This method can be used to check if one of your blogs is followed by another blog.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/followed_by` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter | Type | Description | Required? |
| --------- | ---- | ----------- | --------- |
| **query** | string | The name of the blog that may be following your blog | Yes |

Example usage:

```
GET https://api.tumblr.com/v2/blog/YOUR-BLOG.tumblr.com/followed_by?query=staff
```

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **followed_by** | Boolean | True when the queried blog follows your blog, false otherwise. |

### `/posts` – Retrieve Published Posts

**Examples**

- `https://api.tumblr.com/v2/blog/peacecorps.tumblr.com/posts/text?notes_info=true`
- `https://api.tumblr.com/v2/blog/pitchersandpoets.tumblr.com/posts/photo?tag=new+york+yankees`
- `https://api.tumblr.com/v2/blog/staff.tumblr.com/posts?before=1496289599`

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts[/type]?api_key={key}&[optional-params=]` | GET | [API Key](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **api_key** | String | Your OAuth Consumer Key See Authentication for more details. | N/A | Yes |
| **type** | String | The type of post to return. Specify one of the following: text, quote, link, answer, video, audio, photo, chat | None – return all types | No |
| **id** | Number | A specific post ID. Returns the single post specified or (if not found) a 404 error. | None | No |
| **tag** | String or Array | Limits the response to posts with the specified tag(s), see note below | None | No |
| **limit** | Number | The number of posts to return: 1–20, inclusive | 20 | No |
| **offset** | Number | Post number to start at | 0 (first post) | No |
| **reblog_info** | Boolean | Indicates whether to return reblog information (specify true or false). Returns the various reblogged_ fields. | False | No |
| **notes_info** | Boolean | Indicates whether to return notes information (specify true or false). Returns note count and note metadata. | False | No |
| **filter** | String | Specifies the post format to return, other than HTML: text – Plain text, no HTML; raw – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |
| **before** | Number | Returns posts published earlier than a specified Unix timestamp, in seconds. | False | No |
| **npf** | Boolean | Returns posts' content in [NPF format](npf-spec.md) instead of the legacy format. | False | No |

Note that `tag` can either be a string or an array of strings:

```
Fetch posts using this one tag:
/v2/blog/{blog-identifier}/posts?tag=something
 or fetch posts using all of these tags:
/v2/blog/{blog-identifier}/posts?tag[0]=something&tag[1]=example
```

When given an array of tags, posts will be returned that have _all_ of the provided tags (so in that example, posts with both "something" and "example" used on them). Also, a maximum of four tags can be provided for this use.

#### Response

Each response includes a `blog` object that is the equivalent of an `/info` [response](#blog-info). Posts are returned as an array attached to the `posts` field.

**Important note:** Post content can be in two formats: legacy or Neue Post Format (NPF). By default, posts returned from this endpoint (and any other endpoint that returns posts) will be in the legacy post-type-based content formats described below. NPF-created posts from the official Tumblr mobile apps will be returned as text/regular posts to maintain backwards compatibility. Note that this "backwards compatibility" from NPF to HTML is _best effort_. To help transition to an NPF-only world, you can pass along the `npf=true` query parameter to force _all posts_ returned here to be in Neue Post Format (also described below). Eventually, all posts will be NPF, so we strongly encourage leveraging NPF JSON via `npf=true` for post content when consuming posts via the API.

**Fields available for all Post types:**

| Response Field            | Type             | Description                                                                                    | Notes                                                         |
|---------------------------|------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------|
| **blog_name**             | String           | The short name used to uniquely identify a blog                                                |                                                               |
| **id**                    | Number           | The post's unique ID                                                                           |                                                               |
| **id_string**             | String           | The post's unique ID as a String                                                               | For clients that don't support 64-bit integers                |
| **genesis_post_id**       | String           | The post's unique "genesis" ID† as a String                                                    | Only available to the post owner in certain circumstances     |
| **post_url**              | String           | The location of the post                                                                       |                                                               |
| **parent_post_url**       | String           | The URL of the parent post, if this is a reblog                                                | Only available if the post is a reblog                        |
| **type**                  | String           | The type of post                                                                               | See the `type` request parameter                              |
| **timestamp**             | Number           | The time of the post, in seconds since the epoch                                               |                                                               |
| **date**                  | String           | The GMT date and time of the post, as a string                                                 |                                                               |
| **format**                | String           | The post format: `html` or `markdown`                                                          |                                                               |
| **reblog_key**            | String           | The key used to reblog this post                                                               | See the `/post/reblog` method                                 |
| **tags**                  | Array of strings | Tags applied to the post                                                                       |                                                               |
| **bookmarklet**           | Boolean          | Indicates whether the post was created via the Tumblr bookmarklet                              | Exists only if true                                           |
| **mobile**                | Boolean          | Indicates whether the post was created via mobile/email publishing                             | Exists only if true                                           |
| **source_url**            | String           | The URL for the source of the content (for quotes, reblogs, etc.)                              | Exists only if there's a content source                       |
| **source_title**          | String           | The title of the source site                                                                   | Exists only if there's a content source                       |
| **liked**                 | Boolean          | Indicates if a user has already liked a post or not                                            | Exists only if the request is fully authenticated with OAuth. |
| **state**                 | String           | Indicates the current state of the post                                                        | States are published, queued, draft and private               |
| **is_blocks_post_format** | Boolean          | Indicates whether the post is stored in the [Neue Post Format](#neue-post-format-objects)      |                                                               |
| **muted**                 | Boolean          | Indicates whether push notifications and activity items are muted for this post by its author. | Only available to the post owner in certain circumstances     |
| **mute_end_timestamp**    | Number           | See note below.                                                                                | Only available to the post owner in certain circumstances     |
| **total_posts**           | Number           | The total number of post available for this request, useful for paginating through results     |                                                               |

† The "genesis" ID for a post is only available and different than its current ID if that post had been drafted, queued, or scheduled, and is now published. In which case, the "genesis" ID will be the original post ID generated when drafting, queuing, or scheduling that post. You cannot use this ID to look up the post after it has been published, but it can be useful for tracking a post from its pre- to post-published state.

If `muted: true` and `mute_end_timestamp: 0`, then the post is muted forever. Otherwise, `mute_end_timestamp` is a unix timestamp of when the mute will end. If `muted: false`, then `mute_end_timestamp` will be `0`, but doesn't matter since the post is not muted.

#### Neue Post Format (NPF) Posts

Posts that have `type: blocks` and/or `is_blocks_post_format: true` will have three fields for the post's content:

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **content** | Array | The content of the post. |
| **layout** | Array | The layout of the post content. |
| **trail** | Array | The reblog trail items, if any. |

The specification for what objects you can find in these fields is [documented here](npf-spec.md).

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
            "id_string": "3507845453",
            "post_url": "https:\/\/citriccomics.tumblr.com\/post\/3507845453",
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
            "body": "<p><img src=\"https:\/\/media.tumblr.com\
               /tumblr_lh6x8d7LBB1qa6gy3.jpg\"\/><a href=\"http:\/\
               /citriccomics.com\/blog\/?p=487\" target=\"_blank\">TO READ
               THE REST CLICK HERE<\/a><br\/>\n\nMilky Dog was inspired by
               something <a href=\"http:\/\/gunadie.com\/naomi\"
               target=\"_blank\">Naomi Gee<\/a> wrote on twitter, I really
               liked the hash tag <a href=\"https:\/\/twitter.com\/
               search?q=%23MILKYDOG\" target=\"_blank\">#milkydog<\/a>
               and quickly came up with a little comic about it. You can
               (and should) follow Naomi on twitter <a href=\"https:\/\
               /twitter.com\/ngun\" target=\"_blank\">@ngun<\/a> I'm on
               twitter as well <a href=\"https:\/\/twitter.com\
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
            "id_string": "7431599279",
            "post_url": "https:\/\/derekg.org\/post\/7431599279",
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
                        "url": "https:\/\/derekg.org\/photo\/1280\/7431599279\/1\/
                           tumblr_lo36wbWqqq1qanqww"
                     },
                     {
                        "width": 500,
                        "height": 282,
                        "url": "https:\/\/30.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_500.jpg"
                     },
                     {
                        "width": 400,
                        "height": 225,
                        "url": "https:\/\/29.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_400.jpg"
                     },
                     {
                        "width": 250,
                        "height": 141,
                        "url": "https:\/\/26.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_250.jpg"
                     },
                     {
                        "width": 100,
                        "height": 56,
                        "url": "https:\/\/24.media.tumblr.com\/
                           tumblr_lo36wbWqqq1qanqwwo1_100.jpg"
                     },
                     {
                        "width": 75,
                        "height": 75,
                        "url": "https:\/\/30.media.tumblr.com\/
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
            "id_string": "4742980381",
            "post_url": "https:\/\/museumsandstuff.tumblr.com\/post\/4742980381",
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
| **link_author** | String | The author of the article the link points to |
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
            "id_string": "688472164",
            "post_url": "https:\/\/travellingcameraclub.com\/post\/688472164",
            "type": "link",
            "date": "2010-06-11 23:17:08 GMT",
            "timestamp": 1276298228,
            "format": "html",
            "reblog_key": "RWhJJWia",
            "tags": [],
            "note_count": 9,
            "title": "Esther Aarts Illustration | News and Blog: Faq: How do
               you make those marker doodles?",
            "url": "https:\/\/blog.estadiezijn.nl\/post\/
               459075181\/faq-how-do-you-make-those-marker-doodles",
            "link_author": "Ester Aarts",
            "excerpt": "How I make marker doodles",
            "publisher": "blog.estadiezijn.nl",
            "photos": [
               {
                  "caption": "",
                  "original_size":
                     {
                        "width": 500,
                        "height": 500,
                        "url": "https:\/\/40.media.tumblr.com
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
            "id_string": "5845345725",
            "post_url": "https:\/\/www.davidslog.com\/5845345725\/if-youre-okay-with-it-destroying-tumblr",
            "type": "chat",
            "date": "2011-05-25 22:32:00 GMT",
            "timestamp": 1306362720,
            "format": "html",
            "reblog_key": "wlxAsElf",
            "tags": [],
            "note_count": 114,
            "title": null,
            "body": "David: https://staff.tumblr.com/post/5845153911
                [draft] Topherchris: are you freaking serious\r\n
                David: If you're okay with it - I'd love to :)\r\n
                Topherchris: i'm okay with it, if you're okay with
                it destroying tumblr",
            "dialogue": [
                {
                   "label": "David:",
                   "name": "David",
                   "phrase": "https://staff.tumblr.com/post/5845153911 [draft]\r",
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
            "id_string": "5578378101",
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
            "id_string": "6522991678",
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
            "caption": "<p><a href=\"https:\/\/foreverneilyoung.tumblr.com\/
               post\/6522738445\" target=\"_blank\">foreverneilyoung<\/a>:
               <\/p>\n<blockquote>\n<p><a href=\"https:\/\/watchmojo.tumblr.com\/
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
            "id_string": "7504154594",
            "post_url": "https://www.davidslog.com/7504154594",
            "type": "answer",
            "date": "2011-07-11 20:24:14 GMT",
            "timestamp": 1310415854,
            "format": "html",
            "reblog_key": "HNvqLd5G",
            "tags": [],
            "asking_name": "aperfectfacade",
            "asking_url": "https://aperfectfacade.tumblr.com/",
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

### `/posts/queue` — Retrieve Queued Posts

Gives you a list of the currently queued posts for the specified blog.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/queue` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **offset** | String | Post number to start at | 0 (first post) | No |
| **limit** | Number | The number of results to return: 1–20, inclusive | 20 | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |

### `/posts/queue/reorder` — Reorder Queued Posts

This allows you to reorder a post within the queue, moving it after an existing queued post, or to the top.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/queue/reorder` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Body Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **post_id** | String/Integer | Post ID to move | N/A | Yes |
| **insert_after** | String/Integer | Which post ID to move it after, or 0 to make it the first post | 0 | No |

### `/posts/queue/shuffle` - Shuffle Queued Posts

This randomly shuffles the queue for the specified blog.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/queue/shuffle` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

### `/posts/draft` — Retrieve Draft Posts

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/draft` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **offset** | String | Post number to start at | 0 (first post) | No |
| **filter** | String | Specifies the post format to return, other than HTML: `text` – Plain text, no HTML; `raw` – As entered by the user (no post-processing); if the user writes in Markdown, the Markdown will be returned rather than HTML | None (HTML) | No |

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **id** | Number | The ID of the submitted post |
| **id_string** | String | The ID of the submitted post, in String format for clients that do not support 64-bit integers |
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

### `/notifications` — Retrieve Blog's Activity Feed

Retrieve the activity items for a specific blog, in reverse chronological order.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/notifications` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Query Parameters

| Parameter         | Type     | Description                                                          | Default      | Required? |
|-------------------|----------|----------------------------------------------------------------------|--------------|-----------|
| **before**        | Integer  | Unix epoch timestamp that begins the page, defaults to request time. | Request time | No        |
| **types**         | String[] | An array of one or more types to filter by, or none if you want all  | None         | No        |
| **rollups**       | Boolean  | Whether to "roll up" similar activity items into single items        | `true`       | No        |
| **omit_post_ids** | String[] | An array of one or more of your own Post IDs to filter out           | None         | No        |

Available "types" include:

| Type | Description |
| ---- | ----------- |
| **like** | A like on your post |
| **reply** | A reply on your post |
| **follow** | A new follower |
| **mention_in_reply** | A mention of your blog in a reply |
| **mention_in_post** | A mention of your blog in a post |
| **reblog_naked** | A reblog of your post, without commentary |
| **reblog_with_content** | A reblog of your post, with commentary |
| **ask** | A new ask received |
| **answered_ask** | An answered ask that you had sent |
| **new_group_blog_member** | A new member of your group blog |
| **post_attribution** | Someone using your post content in their post |
| **post_flagged** | A post of yours being flagged |
| **post_appeal_accepted** | An appeal accepted |
| **post_appeal_rejected** | An appeal rejected |
| **what_you_missed** | A post we think you missed |
| **conversational_note** | A conversational note (reply, reblog with comment) on a post you're watching |

#### Response

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **notifications** | Array | An array of activity item objects, see below. |
| **_links** | Object | A pagination links object with a `next` key to use, to load more activity items. |

Note that different activity item objects vary field schema by activity item type. Some common fields include:

- `id` -- The ID of the activity item.
- `type` -- The type of activity item, from the list above.
- `timestamp` -- A unix epoch timestamp of when the event happened.
- `unread` -- A boolean indicating whether this is new/unread as of the last activity read time.
- `target_post_id` -- If the activity has to do with one of your blog's posts, this will be its ID.
- `from_tumblelog_name` -- If the activity is coming from another blog, like a Like or Reblog, this will be its name.
- `post_id` -- For activity like Reblogs and Replies, this will be the relevant post's ID.
- `post_tags` -- An array of used in the reblog, if any.
- `added_text` -- For reblogs with comment, this will be a summary of the added content.
- `reply_text` -- For replies, this will be the text of the reply.

### `/post` — Create a New Blog Post (Legacy)

These legacy posting flows are still available, but we encourage you to use the [Neue Post Format creation route](#posts---createreblog-a-post-neue-post-format).

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

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
| **description** | String | A user-supplied description, HTML allowed | None | No |
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
| **data** | String (URL-encoded binary contents) | A video file, limited to 500MB and 10 minutes | None | Yes, either embed or data |

#### Response

Returns `201: Created` or an error code.

### `/post/edit` – Edit a Blog Post (Legacy)

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/post/edit` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

These parameters are in addition to the common parameters listed under [`/post`](#post--create-a-new-blog-post-legacy).

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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

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
- `"unapproved"` means the post is a new submission.

If omitted, the state parameter on a new post defaults to `"published"`.

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts` | POST | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

### Request Parameters

| Parameter                  | Type    | Description                                                                                                                                                          | Default                                             | Required?          |
|----------------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|--------------------|
| **content**                | Array   | An array of [NPF content blocks](npf-spec.md) to be used to make the post; in a reblog, this is any content you want to add.                                         | `[]`                                                | Yes, for new posts |
| **layout**                 | Array   | An array of [NPF layout objects](npf-spec.md) to be used to lay out the post content.                                                                                | `[]`                                                | No                 |
| **state**                  | String  | The initial [state of the new post](#note-about-post-states), such as "published" or "queued".                                                                       | `"published"`                                       | No                 |
| **publish_on**             | String  | The exact _future_ date and time (ISO 8601 format) to publish the post, if desired. This parameter will be ignored unless the `state` parameter is `"queue"`.        | Now                                                 | No                 |
| **date**                   | String  | The exact date and time (ISO 8601 format) in the past to _backdate_ the post, if desired. This backdating does not apply to when the post shows up in the Dashboard. | Now                                                 | No                 |
| **tags**                   | String  | A comma-separated list of tags to associate with the post.                                                                                                           | None                                                | No                 |
| **source_url**             | String  | A source attribution for the post content.                                                                                                                           | None                                                | No                 |
| **send_to_twitter**        | Boolean | Whether or not to share this via any connected Twitter account on post publish. Defaults to the blog's global setting.                                               | `false`                                             | No                 |
| **is_private**             | Boolean | Whether this should be a private answer, if this is an answer.                                                                                                       | `false`                                             | No                 |
| **slug**                   | String  | A custom URL slug to use in the post's permalink URL                                                                                                                 | Automatically generated based on the post's content | No                 |
| **interactability_reblog** | String  | Who can interact with this when reblogging; `'everyone'` or `'noone'`                                                                                                | `'everyone'`                                        | No                 |

**If the post being created is a reblog, all of the above parameters are expected, along with:**

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **parent_tumblelog_uuid** | String | The unique public identifier of the Tumblelog that's being reblogged from. | N/A | Yes |
| **parent_post_id** | Integer | The unique public post ID being reblogged. | N/A | Yes |
| **reblog_key** | String | The unique per-post hash validating that this is a genuine reblog action. | N/A | Yes |
| **hide_trail** | Boolean | Whether or not to hide the reblog trail with this new post. Defaults to false. | `false` | No |
| **exclude_trail_items** | Array | Instead of **hide_trail**, use this to specify an array of specific reblog trail item indexes to _exclude_ from your reblog. | `[]` | No |

Note that when making a reblog, the `trail` from the post you're reblogging is not required, and will be ignored if given. Any `content` you provide when making a reblog will be added at the end of the reblog trail, in your post.

#### User Uploaded Media

In order to support user uploaded media (photos, videos, etc.), the creation and edit routes also support a multi-part form request where the first part contains the JSON body and subsequent parts contain media data. The `Content-Type: multipart/form-data` header must be used in this case.

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

The error codes here are the HTTP statuses that'll be returned in error states, the error subcodes are the specific subcodes also returned in some situations. The response should have an `errors` array at the top level alongside `meta`, containing what error(s) occurred.

- `400 Bad Request` when the request has not provided the required data.
    - `400.8001` when an NPF JSON parameter is invalid or a bad format.
    - `400.8002` when the reblog parent post and/or blog is invalid or cannot be found or cannot be interacted with.
    - `400.8005` when the uploaded media is an invalid format we cannot accept.
    - `400.8016` when there is something invalid about the format of the answer content or layout.
- `401 Unauthorized` when the requester is an unauthorized client.
- `403 Forbidden` when the requester is not allowed to reblog the parent post they specified.
    - `403.8004` when the requesting user cannot upload more media today.
    - `403.8008` when trying to upload a video in reblog content, which is not allowed.
    - `403.8010` when there is a video upload still transcoding, so you can't upload another video yet.
    - `403.8011` when the requesting user cannot upload more videos today.
    - `403.8022` when the blog's queue limit has been reached.
    - `403.8023` when the blog's daily posting limit has been reached.
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

This route can fetch a post, either legacy or NPF, and return it in either in the legacy or NPF format. The specification for NPF is [here](npf-spec.md). Note that NPF to legacy conversion is not valid yet, so the available conversions are: NPF -> NPF, legacy -> legacy, and legacy -> NPF.

The intention of this route is to fetch a post for editing in either the NPF or legacy format.

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}` | GET | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |
| **post-id** | String | Post ID. See the [Post Identifiers](#post-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **post_format** | String | The format to serve the post as, either `npf` or `legacy`. | `npf`| No |

### Response

This returns a `200 OK` on successful post fetching, or an error code.

The response JSON object will contain:

| Response Field             | Type    | Description                                                                                                                                                                                                  |
|----------------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **object_type**            | String  | The timeline object type, always `post`.                                                                                                                                                                     |
| **type**                   | String  | The post type. If formatting as NPF, the `type` will be `blocks`; if formatting as legacy, the `type` will be one of the original legacy types (`text`, `photo`, `quote`, `chat`, `link`, `video`, `audio`). |
| **id**                     | String  | The post ID, intentionally a string instead of an integer, for 32bit device compatibility.                                                                                                                   |
| **tumblelog_uuid**         | String  | The posting blog's unique identifier.                                                                                                                                                                        |
| **parent_post_id**         | String  | The parent post ID, if the post being fetched is a reblog.                                                                                                                                                   |
| **parent_tumblelog_uuid**  | String  | The parent posting blog's UUID, if the post being fetched is a reblog.                                                                                                                                       |
| **reblog_key**             | String  | The reblog key needed as an additional authentication token.                                                                                                                                                 |
| **trail**                  | Array   | Array of trail items, if the post being fetched is a reblog.                                                                                                                                                 |
| **content**                | Array   | Array of the content blocks of the post itself.                                                                                                                                                              |
| **layout**                 | Array   | Array of the post's layout objects.                                                                                                                                                                          |
| **queued_state**           | String  | If the post's state is `queued`, this field will indicate whether it's actually a `scheduled` post.                                                                                                          |
| **scheduled_publish_time** | Integer | If the post's state is `queued`, this will provide the scheduled publishing timestamp.                                                                                                                       |
| **publish_on**             | String  | If the `queued_state` is "scheduled", this is the timestamp of when the post is scheduled to publish, in ISO 8601 format.                                                                                    |
| **interactability_reblog** | String  | Who can interact with this when reblogging; `'everyone'` or `'noone'`                                                                                                                                        |

Note about queued/scheduled posts: the `queued_state` field will only exist for a `state: queued` post, and will indicate either `queued` or `scheduled`. The `scheduled_publish_time` field will tell you when the post is currently scheduled for publishing, as a Unix epoch timestamp. These fields are read-only; to change post's state or scheduled publish time, please use the `state` and `publish_on` fields when editing the post.

#### Errors and Error Subcodes

See the notes from the NPF Post creation route for info about this.

### `/posts/{post-id}` - Editing a Post (Neue Post Format)

This route allows you to edit posts using the Neue Post Format. Note that you can only edit posts in NPF if they were originally created in NPF, or are legacy text posts. The specification for NPF is [here](npf-spec.md).

### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}` | PUT | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |
| **post-id** | String | Post ID. See the [Post Identifiers](#post-identifiers) section for more details. | N/A | Yes |

### Request Parameters

For editing a post, all of the request parameters from the NPF Post Creation route are expected (depending on if it's an original post or reblog), along with the Post's ID in the query path. If you are editing a scheduled post, make sure to include its `publish_on` value.

Note that the `exclude_trail_items` parameter will edit the reblog's current trail at the time of editing, not the trail that existed when the reblog was created. So if you had created a reblog and excluded trail items at creation, then are editing it to exclude trail items again, you are now editing that reblog's trail, not the parent post's trail. Basically: you can't "bring back" trail items without reblogging the parent post all over again.

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

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to delete | N/A | Yes |

#### Response

Returns `200: OK` (successfully deleted) or an error code.

### `/posts/{post-id}/mute` – Muting a Post's Notifications

Muting a post's notifications means the author of the post will no longer receive activity items or push notifications about the post, until the post is unmuted, or until the temporary mute expires.

#### Methods

The `POST` request mutes the given post, and the `DELETE` request can unmute a post.

| URI                                                             | HTTP Method | Authentication |
|-----------------------------------------------------------------|-------------| -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}/mute` | POST        | [OAuth](#authentication) |
| `api.tumblr.com/v2/blog/{blog-identifier}/posts/{post-id}/mute` | DELETE      | [OAuth](#authentication) |

#### Request Path Parameters

| Parameter           | Type | Description                                                                                  | Default | Required? |
|---------------------| ---- |----------------------------------------------------------------------------------------------| ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |
| **post-id**         | String | The post ID to mute.                                                                         | N/A | Yes |

#### Body Parameters

To make the mute temporary, you can supply how long you want the mute to last.

| Parameter               | Type    | Description                           | Default     | Required? |
|-------------------------|---------|---------------------------------------|-------------|-----------|
| **mute_length_seconds** | Integer | How many seconds the mute should last | 0 (forever) | No         |

Giving `0` or omitting `mute_length_seconds` entirely means the mute will last until the post is unmuted by the author.

#### Response

Returns `200 OK` on successful muting or unmuting, or an error code.

Returns `400 Bad Request` if the given `mute_length_seconds` is less than 0.

### `/notes` - Get notes for a specific Post

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/blog/{blog-identifier}/notes` | GET | [API Key](#authentication) |

#### Request Path Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **blog-identifier** | String | Any blog identifier. See the [Blog Identifiers](#blog-identifiers) section for more details. | N/A | Yes |

#### Request Parameters

| Parameter | Type | Description | Default | Required? |
| --------- | ---- | ----------- | ------- | --------- |
| **id** | Number | The ID of the post to fetch notes for | N/A | Yes |
| **before_timestamp** | Number | Fetch notes created before this timestamp, for pagination. This is a unix timestamp in seconds precision, but microsecond precision for `conversation` mode. | N/A | No |
| **mode** | String | The response formatting mode, see list below. | `"all"` | No |

The `mode` field can be one of a few things, most of which act as filters for the types of notes returned. All return notes in reverse chronological order.

- `"all"` loads all notes for the post.
- `"likes"` loads only likes for the post.
- `"conversation"` loads only replies and reblogs with added text commentary, with the rest of the notes (likes, reblogs without commentary) in a `rollup_notes` field.
- `"rollup"` loads only like and reblog notes for the post in the `notes` array.
- `"reblogs_with_tags"` loads only the reblog notes for the post, and each note object includes a `tags` array field (which may be empty).

And different modes can cause the response to contain different things.

Example request:

```
GET https://api.tumblr.com/v2/blog/YOUR-BLOG.tumblr.com/notes?id=1234567890000&mode=all&before_timestamp=1234567890
```

#### Response

Returns `200: OK` with an array of `notes`, along with more info, and a `_links` object to load more notes.

| Field | Type | Description |
| ----- | ---- | ----------- |
| **notes** | Array | An array of note objects, which may be formatted differently based on the mode and note type |
| **rollup_notes** | Array | In "conversation" mode, this contains notes not listed in the `notes` array. |
| **total_notes** | Number | The total notes, which can change depending on the mode. |
| **total_likes** | Number | The total likes, when mode is `conversation`. |
| **total_reblogs** | Number | The total reblogs, when mode is `conversation`. |
| **_links** | Object | Contains a `next` link object for pagination. |

Example response:

```JSON
{
    "meta": {
        "status": 200,
        "msg": "OK"
    },
    "response": {
        "notes": [
            {
                "type": "reblog",
                "timestamp": 1595527574,
                "blog_name": "test-blog",
                "blog_uuid": "t:abcd1234",
                "blog_url": "https://test-blog.tumblr.com/",
                "followed": false,
                "avatar_shape": "square",
                "post_id": "1123456688899000",
                "reblog_parent_blog_name": "other-test-blog"
            },
            {
                "type": "like",
                "timestamp": 1595527569,
                "blog_name": "test-blog",
                "blog_uuid": "t:abcd1234",
                "blog_url": "https://test-blog.tumblr.com/",
                "followed": false,
                "avatar_shape": "square"
            }
        ],
        "total_notes": 2
    }
}
```

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
                "url": "https://derekg.org/",
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

### `/user/limits` – Get a User's Limits

Use this method to retrieve information about the various limits for the current user.

#### Method

| URI | HTTP Method | Authentication |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/limits` | GET | [OAuth](#authentication) |

#### Response

**Example**

```JSON
{
    "meta": {
        "status": 200,
        "msg": "OK"
    },
    "response": {
        "user": {
            "blogs": {
               "description": "the number of secondary blogs you can create per day",
               "limit": 10,
               "remaining": 4,
               "reset_at": 1670043600
            },
            "follows": {
                "description": "the number of blogs you can follow per day",
                "limit": 200,
                "remaining": 199,
                "reset_at": 1670043600
            },
            ...
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
| **npf** | Boolean | Returns posts' content in [NPF format](npf-spec.md) instead of the legacy format. | False | No |

#### Response

Dashboard responses include the fields returned in `/posts` [responses](#posts--retrieve-published-posts) (with all the various type-specific fields), but without the blog object. Instead, a `blog_name` field identifies the blog for each post returned.

**Important note:** Post content can be in two formats: legacy or Neue Post Format (NPF). By default, posts returned from this endpoint (and any other endpoint that returns posts) will be in the legacy post-type-based content formats [described here](#posts--retrieve-published-posts). NPF-created posts from the official Tumblr mobile apps will be returned as text/regular posts to maintain backwards compatibility. To help transition to an NPF-only world, you can pass along the `npf=true` query parameter to force _all posts_ returned here to be in Neue Post Format (also [described here](#posts--retrieve-published-posts)).

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
            "id_string": "6828225215",
            "post_url": "https:\/\/links.laughingsquid.com\/post\/6828225215",
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
             "url": "https:\/\/www.davidslog.com",
             "updated": 1308781073,
             "title": "David’s Log",
             "description": "“Mr. Karp is tall and skinny, with unflinching blue eyes and a mop of brown hair. He speaks incredibly fast and in complete paragraphs.” – NY Observer"
          },
          {
             "name": "matthew",
             "url": "https:\/\/matthew.tumblr.com",
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
| url | String | The URL of the blog to follow | None | Must supply `url` or `email` |
| email | String | The email of the blog to follow. A blog is only followable by email if it has the `Let people find your blogs through this address.` setting enabled on [tumblr.com/settings/account](https://www.tumblr.com/settings/account).| None | Must supply `url` or `email` |

#### Response

Returns `200: OK` (blog successfully followed) or a 404 (blog was not found)

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| **blog** | Object | The followed blog info |

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

### `/user/filtered_tags` - Tag Filtering

This endpoint lets you manage the tagged content you'd like covered on your dashboard. See more info about this feature [here](https://tumblr.zendesk.com/hc/articles/115015814708-Tag-filtering). Note that this filtering relies on the content being tagged, but also filters tag recommendations and other tag-based content. For filtering the text content of posts, see [Content Filtering](#userfiltered_content---content-filtering).

#### Methods

All of the endpoints require [OAuth](#authentication) authentication.

| URI | HTTP Method | Description |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/filtered_tags` | GET | Retrieve a list of currently-filtered tag strings. |
| `api.tumblr.com/v2/user/filtered_tags` | POST | Add one or more tags to filter. |
| `api.tumblr.com/v2/user/filtered_tags/{tag}` | DELETE | Remove a tag filter. |

#### Request Parameters

For adding new tag filters, you can add one at a time or many at once via the `POST` body:

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| **filtered_tags** | Array of Strings | One or more than one tag string to add to your list of filters |

Example:

```
POST https://api.tumblr.com/v2/user/filtered_tags
Content-Type: application/json
{ "filtered_tags": [ "something" ] }

POST https://api.tumblr.com/v2/user/filtered_tags
Content-Type: application/json
{ "filtered_tags": [ "something", "technology" ] }
```

For deleting a tag filter, pass along the tag string in the path:

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| **tag** | String | Tag to stop filtering; this is expected to be ["raw" URL encoded](http://www.faqs.org/rfcs/rfc3986.html). |

Example:

```
DELETE https://api.tumblr.com/v2/user/filtered_tags/tag%20here
```

That would delete the tag "tag here".

#### Response

For `GET` requests, the endpoint will return a `200 OK` on success, along with the list of tags:

```json
{
    "meta": {
        "status": 200,
        "msg": "OK"
    },
    "response": {
        "filtered_tags": [
            "technology",
            "something",
            "and something else"
        ]
    }
}
```

For `POST` requests, the endpoint will return a `201 Created` on success, with an empty `response` object.

For `DELETE` requests, the endpoint will return a `200 OK` on success, with an empty `response` object.

#### Errors

- `400 Bad Request` if given an invalid/empty tag to filter or delete.

### `/user/filtered_content` - Content Filtering

This endpoint lets you manage the plain text content you'd like covered on your dashboard, including blog names in the reblog trail. See more info about this feature [here](https://tumblr.zendesk.com/hc/articles/360046752174). For filtering based on tags, see [Tag Filtering](#userfiltered_tags---tag-filtering).

#### Limits

- Each user can have a maximum of 200 filtered strings.
- Each filtered string cannot be more than 250 characters in length.
- Filtered strings can contain any characters that are valid UTF-8.

#### Methods

All of the endpoints require [OAuth](#authentication) authentication.

| URI | HTTP Method | Description |
| --- | ----------- | -------------- |
| `api.tumblr.com/v2/user/filtered_content` | GET | Retrieve a list of currently-filtered content strings. |
| `api.tumblr.com/v2/user/filtered_content` | POST | Add one or more content strings to filter. |
| `api.tumblr.com/v2/user/filtered_content` | DELETE | Remove a content filter string. |

#### Request Parameters

For adding new content filters, you can add one at a time or many at once via the `POST` body:

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| **filtered_content** | String or Array of Strings | One or more than one string to add to your list of filters |

Example:

```
POST https://api.tumblr.com/v2/user/filtered_content
Content-Type: application/x-www-form-urlencoded
filtered_content=something

POST https://api.tumblr.com/v2/user/filtered_content
Content-Type: application/json
{ "filtered_content": [ "something", "technology" ] }
```

For deleting a content filter, pass along the string in the query params:

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| **filtered_content** | String | Content filter string to remove. |

Example:

```
DELETE https://api.tumblr.com/v2/user/filtered_content?filtered_content=something
```

#### Response

For `GET` requests, the endpoint will return a `200 OK` on success, along with the list of strings:

```json
{
    "meta": {
        "status": 200,
        "msg": "OK"
    },
    "response": {
        "filtered_content": [
            "technology",
            "something",
            "and something else"
        ]
    }
}
```

For `POST` requests, the endpoint will return a `201 Created` on success, with an empty `response` object.

For `DELETE` requests, the endpoint will return a `200 OK` on success, with an empty `response` object.

#### Errors

- `400 Bad Request` if given an invalid/empty string to filter or delete, or if given an already-filtered string.
- `403 Forbidden` if at the maximum number of allowed filters.

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

