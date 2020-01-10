# Tumblr Neue Post Format (NPF) Specification

## Table of Contents

- [Overview](#overview)
- [Post Consumption vs Post Creation](#post-consumption-vs-post-creation)
- [A Note About Backwards Compatibility](#a-note-about-backwards-compatibility)
- [Post Identification](#post-identification)
- [Content Blocks](#content-blocks)
    - [Media Objects](#media-objects)
    - [Content Block Type: Text](#content-block-type-text)
        - [Special Note: Leading and Trailing Empty Text Blocks](#special-note-leading-and-trailing-empty-text-blocks)
        - [Text Block Basic Subtypes](#text-block-basic-subtypes)
        - [Text Block Subtype: List Item](#text-block-subtype-list-item)
        - [Inline Formatting within a Text Block](#inline-formatting-within-a-text-block)
        - [Inline Format Types: Bold, Italic, Strikethrough](#inline-format-types-bold-italic-strikethrough)
        - [Inline Format Type: Link](#inline-format-type-link)
        - [Inline Format Type: Mention](#inline-format-type-mention)
        - [Inline Format Type: Color](#inline-format-type-color)
        - [Inline Format Type: Size](#inline-format-type-size)
    - [Content Block Type: Image](#content-block-type-image)
        - [GIF Posters](#gif-posters)
        - [Image Attribution](#image-attribution)
    - [Content Block Type: Link](#content-block-type-link)
    - [Content Block Type: Audio](#content-block-type-audio)
    - [Content Block Type: Video](#content-block-type-video)
    - [Future Content Block Types](#future-content-block-types)
- [Layout Blocks](#layout-blocks)
    - [Layout Block Type: Basic Rows](#layout-block-type-basic-rows)
    - [Layout Block Type: Rows with Display Modes](#layout-block-type-rows-with-display-modes)
        - [Layout Block Display Mode: Carousel](#layout-block-display-mode-carousel)
    - [Layout Block Type: Condensed](#layout-block-type-condensed)
    - [Layout Block Type: Ask](#layout-block-type-ask)
- [Reblog Trail](#reblog-trail)
    - ["Broken" Trail Items](#broken-trail-items)
- [Attributions](#attributions)
    - [Attribution Type: Post](#attribution-type-post)
    - [Attribution Type: Link](#attribution-type-link)
    - [Attribution Type: Blog](#attribution-type-blog)
    - [Attribution Type: App](#attribution-type-app)
- [Mapping NPF Post Content to Legacy Post Types](#mapping-npf-post-content-to-legacy-post-types)

## Overview

This document describes the JSON object definitions that clients should expect when communicating with the Tumblr API when a Post is rendered using the Neue Post Format (NPF). 

Furthermore, this is an evolving spec: **this does not include all future possibilities**. We will add new specs to this document as they are designed and built. Any specification in this document should be considered either in production, in active development, or soon to be in development.

## Post Consumption vs Post Creation

The display specification outlined here can be used as a reference both when _consuming_ Posts in NPF as well as when _creating_ Posts in NPF. Any fields that are not required (and some are not allowed) during Post creation are called out at the appropriate place in this document.

## A Note About Backwards Compatibility

Backwards compatibility for new content block types, layout types, etc, should be determined at the server level. Clients should always be sending version information and the server will determine which logic to use when rendering content for you.

However, clients must have a fallback "This content is not supported, please upgrade your app" type of display in the case that they encounter information that they cannot display.

## Post Identification

To uniquely identify the Post, all Post JSON objects contain an `id` integer and a `blog` object:

```JSON
{
    "id": 1234567891234567,
    "blog": {
        "Standard API Short Blog Info object": "goes here"
    }
}
```

Every Post has an `id` which is a unique unsigned 64bit integer. The `blog` object is a standard API-formatted "short blog info" object.

_This document does not cover the full specification of that Post JSON object._ There are many more fields present in the Post JSON object.

## Content Blocks

A **content block** is one discreet unit of content; it is an abstract concept that has concrete definitions depending on the "type" of block, such as a "text", "image", "video", or "audio" Content Blocks. A lot of these content blocks could be thought of the same way HTML tags were originally conceived. However, there is no guarantee that all content blocks can be cleanly converted to and from HTML.

A Post's content is made up of one or more of these content blocks, each of which can represent text or media. At the base of all Posts in NPF is the `content` field, which holds an array of these new blocks:

```JSON
{
    "id": 1234,
    "blog": {
        "Standard API Short Blog Info object": "goes here"
    },
    "content": []
}
```

> Note: For the rest of the examples in this document, the root level `id` and `blog` fields will be omitted.

Each block within a Post is represented by an object in that "content" array and must have at least a `type` field:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "Hello world!"
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
type | string | yes | The type of the content block, which is required by all blocks.

Additional fields will be present based on the `type` of the block; the following sections outline what those fields are and which are required.

By default, these content blocks are intended to stack vertically in the order they are presented in the list:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "Hello world!"
        },
        {
            "type": "text",
            "text": "This is a second paragraph below the first."
        }
    ]
}
```

Laying out blocks in any format other than a vertical stack is covered in the [Layout Blocks](#layout-blocks) section.

### Media Objects

Many content blocks and their components include **media objects** which link directly to media assets. These media objects share a common JSON object format:

```JSON
{
    "url": "https://69.media.tumblr.com/path/to/image.jpg",
    "type": "image/jpg",
    "width": 540,
    "height": 405
}
```

Media objects are used for image blocks, all kinds of posters (GIF, video, etc), native audio, native video, and some trusted third-party content. All media objects returned from the API should contain `type` and `url`, and any video and image media objects should always contain a `width` and `height`.

Property | Type | Default | Required | Description
-------- | ---- | ------- | -------- | -----------
url | string | _N/A_ | yes | The canonical URL of the media asset
type | string | see description | no | The MIME type of the media asset, or a best approximation will be made based on the given URL
width | integer | 540 | no | The width of the media asset, if that makes sense (for images and videos, but not for audio)
height | integer | 405 | no | The height of the media asset, if that makes sense (for images and videos, but not for audio)
original_dimensions_missing | boolean | `false` | no | For display purposes, this indicates whether the dimensions are defaults
cropped | boolean | `false` | no | This indicates whether this media object is a cropped version of the original media
has_original_dimensions | bool | `false` | no | This indicates whether this media object has the same dimensions as the original media

If the original dimensions of the media are not known, a boolean flag `original_dimensions_missing` with a value of `true` will also be included in the media object. In this scenario, `width` and `height` will be assigned default dimensional values of 540 and 405 respectively. Please note that this field should only be available when _consuming_ an NPF Post, it is not allowed during Post creation.

### Embed Iframe Objects

Embed iframe objects are used for constructing video iframes.

Property | Type | Default | Required | Description
-------- | ---- | ------- | -------- | -----------
url | string | _N/A_ | yes | A URL used for constructing and embeddable video iframe
width | integer | 540 | yes | The width of the video iframe
height | integer | 405 | yes | The height of the video iframe

### Content Block Type: Text

A text block represents a single paragraph of text. At its simplest, it simply wraps a plaintext string. All `text` type blocks must contain a `text` field, like the example used before:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "Hello world!"
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
text | string | yes | The text to use inside this block
subtype | string | no | The subtype of text block, covered in the [Text Block Subtypes](#text-block-subtypes) subsection

#### Special Note: Leading and Trailing Empty Text Blocks

Any empty (as in literally an empty string such as `""`) text blocks at the leading or trailing sides of a `content` array will be trimmed. For example, submitting a Post with the content:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": ""
        },
        {
            "type": "text",
            "text": "ello!"
        },
        {
            "type": "text",
            "text": ""
        },
        {
            "type": "text",
            "text": "my name is cyle!"
        },
        {
            "type": "text",
            "text": ""
        },
        {
            "type": "text",
            "text": ""
        }
    ]
}
```

... will have its empty text blocks trimmed and saved as:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "ello!"
        },
        {
            "type": "text",
            "text": ""
        },
        {
            "type": "text",
            "text": "my name is cyle!"
        }
    ]
}
```

#### Text Block Basic Subtypes

Text blocks can also have a `subtype` field that specifies a semantic meaning to the text block, which can also be used by the clients to render the entire block of text differently.

The following subtypes are supported for text blocks:

* `heading1` - Intended for Post headings.
* `heading2` - Intended for section subheadings.
* `quirky` - Tumblr Official clients display this with a large cursive font.
* `quote` - Intended for short quotations, official Tumblr clients display this with a large serif font.
* `indented` - Intended for longer quotations or photo captions, official Tumblr clients indent this text block.
* `chat` - Intended to mimic the behavior of the Chat Post type, official Tumblr clients display this with a monospace font.
* `ordered-list-item` - Intended to be an ordered list item prefixed by a number, see next section.
* `unordered-list-item` - Intended to be an unordered list item prefixed with a bullet, see next section.

An example Post with two content blocks, one being a regular paragraph and the next being a "quote" block:

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "It's like Ben Franklin always said:"
        },
        {
            "type": "text",
            "subtype": "quote",
            "text": "Genius without education is like silver in the mine.",
        }
    ]
}
```

Another Post, with a header of size 1 and two normal paragraphs:

```JSON
{
    "content": [
        {
            "type": "text",
            "subtype": "heading1",
            "text": "New Post Forms Manifesto"
        },
        {
            "type": "text",
            "text": "There comes a moment in every company's life that they must redefine the rules..."
        },
        {
            "type": "text",
            "text": "We can choose to embrace this moment courageously, or we may choose to cower in fear."
        }
    ]
}
```

Another Post, with a header size 2 and chat blocks (which also feature inline formatting, explained in a different section):

```JSON
{
    "content": [
        {
            "type": "text",
            "subtype": "heading2",
            "text": "what a great conversation"
        },
        {
            "type": "text",
            "subtype": "chat",
            "text": "cyle: ello",
            "formatting": [
                {
                    "start": 0,
                    "end": 5,
                    "type": "bold"
                }
            ]
        },
        {
            "type": "text",
            "subtype": "chat",
            "text": "oli: i'm oli",
            "formatting": [
                {
                    "start": 0,
                    "end": 3,
                    "type": "bold"
                }
            ]
        }
    ]
}
```

#### Text Block Subtype: List Item

You can specify that a text block is a list item by using either `ordered-list-item` or `unordered-list-item` for any text block's `subtype`, like so:

```json
{
    "content": [
        {
            "type": "text",
            "subtype": "heading1",
            "text": "Sward's Shopping List"
        },
        {
            "type": "text",
            "subtype": "ordered-list-item",
            "text": "Sword"
        },
        {
            "type": "text",
            "subtype": "ordered-list-item",
            "text": "Candy"
        },
        {
            "type": "text",
            "text": "But especially don't forget:"
        }
        {
            "type": "text",
            "subtype": "unordered-list-item",
            "text": "Death, which is uncountable on this list."
        }
    ]
}
```

Note that NPF **does not allow nested lists or non-text-based lists** at this time.

#### Inline Formatting within a Text Block

In addition to subtypes at the Text block level, the text within a Text block can have inline styles like bold, italic, external links, blog mentions, and colors.

Inline formatting ranges (`start` and `end`) are zero-indexed and count each character as `1`. Ranges are inclusive at the start and exclusive at the end.

A single unicode character is also treated as one character in this indexing, despite possibly being multiple bytes. For example, `ø` and `🌳` are both counted as single characters in NPF. However, a composite emoji like `👨‍👨‍👦 `is five characters, as it is [made up of five unicode codepoints](http://emojipedia.org/family-man-man-boy/).

Property | Type | Required | Description
-------- | ---- | -------- | -----------
start | integer | yes | The starting index of the formatting range (inclusive)
end | integer | yes | The ending index of the formatting range (exclusive)
type | string | yes | The type of formatting range

Other keys may also be present to specify certain `type`-specific options for a range.

**Overlapping ranges of the same `type` should be combined or separated into non-overlapping ranges.** Overlapping ranges of different types should all be applied to the text, although it is up to the Tumblr API to reconcile the ranges as needed to display correctly, just like when nesting HTML.

For example, suppose the following text with two overlapping ranges:

```JSON
{
    "type": "text",
    "text": "supercalifragilisticexpialidocious",
    "formatting": [
        {
            "start": 0,
            "end": 20,
            "type": "bold"
        },
        {
            "start": 9,
            "end": 34,
            "type": "italic"
        }
    ]
}
```

This would be rendered in HTML as the following:

```HTML
<b>supercali<i>fragilistic</i></b><i>expialidocious</i>
```

#### Inline Format Types: Bold, Italic, Strikethrough

The basic inline formatting types that require no additional information are `bold`, `italic`, and `strikethrough`. They are used as described above:

```JSON
{
    "type": "text",
    "text": "some bold and italic text",
    "formatting": [
        {
            "start": 5,
            "end": 9,
            "type": "bold"
        },
        {
            "start": 14,
            "end": 20,
            "type": "italic"
        }
    ]
}
```

#### Inline Format Type: Link

The most basic inline format type that requires extra information is the `link` type, which requires a `url` field:

```JSON
{
    "type": "text",
    "text": "Found this link for you",
    "formatting": [
        {
            "start": 6,
            "end": 10,
            "type": "link",
            "url": "https://www.nasa.gov"
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | yes | The link's URL!

Please note that when consuming URLs through the Tumblr API in NPF, these URLs may be converted to URLs that redirect through Tumblr's servers, for security and spam prevention.

#### Inline Format Type: Mention

The `mention` type includes the known blog's `BlogInfo` object behind a `blog` key:

```JSON
{
    "type": "text",
    "text": "Shout out to @david",
    "formatting": [
        {
            "start": 13,
            "end": 19,
            "type": "mention",
            "blog": {
                "uuid": "t:123456abcdf",
                "name": "david",
                "url": "https://davidslog.com/",
            }
        }
    ]
}
```

But for Post creation, only the `UUID` field is required.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
blog | object | yes | An object with a `uuid` field, which is the mentioned blog's UUID.

#### Inline Format Type: Color

The `color` type contains the information about the text color of the current text block. It requires extra information which is the `hex` value of the color and it includes the `#` prefix:

```JSON
{
    "type": "text",
    "text": "Celebrate Pride Month",
    "formatting": [
        {
            "start": 10,
            "end": 15,
            "type": "color",
            "hex": "#ff492f"
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
hex | string | yes | The color to use, in standard hex format, with leading #.

### Content Block Type: Image

For image blocks, the only required field is a `media` array, which contains objects per-image-size. Each image media object contains fields for `type`, `url`, `width`, and `height`; see the [Media Objects](#media-objects) section for more details.

We encourage you to include a string in the optional `alt_text` field that describes the image, for use by accessibility tools such as screen readers.

```JSON
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_1280.jpg",
                    "width": 1280,
                    "height": 1073
                },
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_540.jpg",
                    "width": 540,
                    "height": 400
                },
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_250.jpg",
                    "width": 250,
                    "height": 150
                }
            ],
            "alt_text": "Sonic the Hedgehog and friends"
        }
    ]
}
```

Optionally, an image block may contain a `feedback_token` field, which is a string. This is a unique identifier for the image,
referenced by our Recommendation logging to track how users are using certain images, like GIFs from GIF Search.

```JSON
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/gif",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_250.gif",
                    "width": 250,
                    "height": 200
                }
            ],
            "feedback_token": "abcdef123456"
        }
    ]
}
```

Image blocks may contain a `colors` field which provides the dominant colors stored for the image in the Post metadata. The value is an object with string fields where the key starts with a c and ends with a number, eg. `c0`, `c1`, `c9`. The values are a string of three or six characters representing an RGB hex value. These values are typically used for creating a placeholder gradient while images are loading. The colors represented for an image are the same for every size of that image.

```JSON
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_1280.jpg",
                    "width": 1280,
                    "height": 1073
                }
            ],
            "colors": {
                "c0": "a24615",
                "c1": "ff7c00"
            }
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
media | array<[Media Object](#media-objects)> | yes | An array of Media Objects which represent different available sizes of this image asset.
colors | object | no | Colors used in the image.
feedback_token | string | no | A feedback token to use when this image block is a GIF Search result.
poster | [Media Object](#media-objects) | no | For GIFs, this is a single-frame "poster"; see the [GIF Posters](#gif-posters) section.
attribution | [Attribution Object](#attributions) | no | See [the Attributions section](#attributions) for details about these objects.
alt_text | string | no | Text used to describe the image, for screen readers. 200 character maximum.

#### GIF Posters

In the case of GIFs, the image objects may include a `poster` Media Object that contains a single-frame image of the same size to use in place of the GIF on display for low-bandwidth consumers:

```JSON
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/gif",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.gif",
                    "width": 500,
                    "height": 400,
                    "poster": {
                        "type": "image/jpeg",
                        "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                        "width": 500,
                        "height": 400
                    }
                }
            ]
        }
    ]
}
```

#### Image Attribution

In the case of GIF search results and individual images with a content source, the content block may include an `attribution` object with more fields for those cases. See [the Attributions section](#attributions) below for details about these objects.

### Content Block Type: Link

In addition to the inline link format, there is a standalone Link content block, which contains metadata about the link.

All `link` type blocks must have at least a `url` field. `title`, `description`, `author`, `site_name`, and `poster` are all optional, depending on their availability. If only a `url` is given with no other fields, we will attempt to fetch that link's OpenGraph data during post creation, which may populate the other fields. The `title`, `description`, `author`, and `site_name` fields are limited to 140 characters when fetching a link's OpenGraph data server-side. The `site_name` field will either have the site name pulled from OpenGraph data or will have the host name as depicted in the example response below. A `display_url` field is given for display only. The `poster` field is expected to be an array of image MediaObjects with alternate sizes.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | yes | The URL to use for the link block.
title | string | no | The title of where the link goes.
description | string | no | The description of where the link goes.
author | string | no | The author of the link's content.
site_name | string | no | The name of the site being linked to.
display_url | string | no | Supplied on NPF Post consumption, ignored during NPF Post creation.
poster | [Media Object](#media-objects) | no | An image media object to use as a "poster" for the link.

When you are creating a link block, you do not need to supply the `display_url` field. It will be ignored by the backend on Post creation. Creating a new link block looks like this:

```JSON
{
    "content": {
        "type": "link",
        "url": "https://www.nytimes.com/2017/06/15/us/politics/secrecy-surrounding-senate-health-bill-raises-alarms-in-both-parties.html",
        "title": "Secrecy Surrounding Senate Health Bill Raises Alarms in Both Parties",
        "description": "Senate leaders are writing legislation to repeal and replace the Affordable Care Act without a single hearing on the bill and without an open drafting session.",
        "author": "Thomas Kaplan and Robert Pear",
        "poster": [
            {
                "url": "https://static01.nyt.com/images/2017/06/15/us/politics/15dchealth-2/15dchealth-2-facebookJumbo.jpg",
                "type": "image/jpeg",
                "width": 1050,
                "height": 549
            }
        ]
    }
}
```

On display of a link block via our API, the `url` field will be a `t.umblr.com` redirect, for potential spam prevention and security purposes, and `display_url` will show the original link, which is what we encourage clients to show in the UI:

```JSON
{
    "content": {
        "type": "link",
        "url": "http://t.umblr.com/redirect?stuff-here",
        "display_url": "https://www.nytimes.com/2017/06/15/us/politics/secrecy-surrounding-senate-health-bill-raises-alarms-in-both-parties.html",
        "title": "Secrecy Surrounding Senate Health Bill Raises Alarms in Both Parties",
        "description": "Senate leaders are writing legislation to repeal and replace the Affordable Care Act without a single hearing on the bill and without an open drafting session.",
        "author": "Thomas Kaplan and Robert Pear",
        "site_name": "nytimes.com",
        "poster": [
            {
                "url": "https://static01.nyt.com/images/2017/06/15/us/politics/15dchealth-2/15dchealth-2-facebookJumbo.jpg",
                "type": "image/jpeg",
                "width": 1050,
                "height": 549
            }
        ]
    }
}
```

### Content Block Type: Audio

An `audio` block represents a playable track. At a minimum, the `provider` field must be present, and either the `media` field or `url` field must be present. The `provider` field will indicate which embed provider is being used, whether it's `tumblr` for Tumblr native audio content, or a trusted third party like `spotify` or `bandcamp`. Optionally, an `audio` block may include the track's `title`, `artist`, and/or `album`.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | maybe | The URL to use for the audio block, if no `media` is present.
media | [Media Object](#media-objects) | maybe | The [Media Object](#media-objects) to use for the audio block, if no `url` is present.
provider | string | no | The provider of the audio source, whether it's `tumblr` for native audio or a trusted third party.
title | string | no | The title of the audio asset.
artist | string | no | The artist of the audio asset.
album | string | no | The album from which the audio asset originated.
poster | [Media Object](#media-objects) | no | An image media object to use as a "poster" for the audio track, usually album art.
embed_html | string | no | HTML code that could be used to embed this audio track into a webpage.
embed_url | string | no | A URL to the embeddable content to use as an iframe.
metadata | object | no | Optional provider-specific metadata about the audio track.
attribution | [Attribution Object](#attributions) | no | Optional attribution information about where the audio track came from.

An `audio` block may have a canonical `url` specified which, when visited in a web browser, will allow the track to be played. If the track can be embedded on a web page, then an `audio` block will have an `embed_html` field present and, optionally, the `embed_url`.

If the track is playable using the client's native player (such as an iOS or Android application), then an `audio` block will have a `media` object present. This media object contains fields for `type` and `url`; see the [Media Objects](#media-objects) section for more information.

Optionally, an `audio` block may include a `poster` array of image objects to display. Optionally, depending on the `provider`, there may be additional metadata inside a `metadata` object.

Optionally, the content block may contain an `attribution` object with more information about the app or blog it came from. See [the section on Attribution](#attributions) for more info.

Based on the provider, the waterfall the client should attempt to follow looks like:

1. If there is a `media` object present, use the client's native player to play it.
1. If there is a client-side SDK for the given provider, use the given `metadata` object with the client-side SDK.
1. If there is an `embed_html` and/or `embed_url` field present, render `embed_html` or show `embed_url` in an iframe.
1. If all else fails, just show a link to the given `url`.

A native `tumblr`-provider track:

```JSON
{
    "content": [
        {
            "type": "audio",
            "provider": "tumblr",
            "title": "Track Title",
            "artist": "Track Artist",
            "album": "Track Album",
            "media": {
                "type": "audio/mp3",
                "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1.mp3"
            },
            "poster": [
                {
                    "type": "image/jpeg",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                    "width": 500,
                    "height": 400
                }
            ]
        }
    ]
}
```

A SoundCloud track:

```JSON
{
    "content": [
        {
            "type": "audio",
            "provider": "soundcloud",
            "title": "Mouth Sounds",
            "artist": "neilcic",
            "url": "https://soundcloud.com/neilcic/mouth-sounds",
            "embed_html": "<iframe width="100%" height="450" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/146805680&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false&amp;visual=true"></iframe>",
            "embed_url": "https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/146805680&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false&amp;visual=true",
            "media": {
                "type": "audio/mp3",
                "url": "https://soundcloud.com/neilcic/mouth-sounds.mp3"
            },
            "poster": [
                {
                    "type": "image/jpeg",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                    "width": 500,
                    "height": 400
                }
            ]
        }
    ]
}
```

### Content Block Type: Video

A `video` block represents a playable video. At a minimum, the `provider` field must be present, and either the `media` field or `url` field must be present. The `provider` field will indicate which embed provider is being used, whether it's `tumblr` for Tumblr native video content, or a trusted third party like `youtube` or `vimeo`.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | maybe | The URL to use for the video block, if no `media` is present.
media | [Media Object](#media-objects) | maybe | The [Media Object](#media-objects) to use for the video block, if no `url` is present.
provider | string | no | The provider of the video, whether it's `tumblr` for native video or a trusted third party.
embed_html | string | no | HTML code that could be used to embed this video into a webpage.
embed_iframe | [Embed Iframe Object](#embed-iframe-objects) | no | An embed iframe object used for constructing video iframes.
embed_url | string | no | A URL to the embeddable content to use as an iframe.
poster | [Media Object](#media-objects) | no | An image media object to use as a "poster" for the video, usually a single frame.
metadata | object | no | Optional provider-specific metadata about the video.
attribution | [Attribution Object](#attributions) | no | Optional attribution information about where the video came from.
can_autoplay_on_cellular | boolean | no | Whether this video can be played on a cellular connection.

A `video` block may have a canonical `url` specified which, when visited in a web browser, will allow the video to be played. If the video can be embedded on a web page, then the block will have an `embed_html` field present and, optionally, the `embed_url`.

If the video is playable using the client's native player (such as an iOS or Android application), then the block will have a `media` object present. This media object contains fields for `type` and `url`; see the [Media Objects](#media-objects) section for more information.

Optionally, a `video` block may include a `poster` array of image objects to display. Optionally, depending on the `provider`, there may be additional metadata inside a `metadata` object.

Optionally, the content block may contain an `attribution` object with more information about the app or blog it came from. See [the section on Attribution](#attributions) for more info.

Based on the provider, the waterfall the client should attempt to follow looks like:

1. If there is a `media` object present, simply use the client's native player.
1. If there is a client-side SDK for the given provider, use the given `metadata` object with the client-side SDK.
1. If there is an `embed_html` render it.
1. If there is an `embed_iframe` use it to construct an iframe.
1. If there is an `embed_url` use it to construct an iframe.
1. If all else fails, just show a link to `url`.

A native video:

```json
{
    "content": [
        {
            "type": "video",
            "media": {
                "type": "video/mp4",
                "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1.mp4",
                "height": 640,
                "width": 480,
                "hd": false
            },
            "poster": [
                {
                    "type": "image/jpeg",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                    "width": 500,
                    "height": 400
                }
            ],
            "filmstrip": [
                {
                    "type": "image/jpeg",
                    "url": "https://66.media.tumblr.com/previews/tumblr_nshp8oVOnV1rg0s9xo1_filmstrip_frame1.jpg",
                    "width": 200,
                    "height": 125
                },
                {
                    "type": "image/jpeg",
                    "url": "https://66.media.tumblr.com/previews/tumblr_nshp8oVOnV1rg0s9xo1_filmstrip_frame2.jpg",
                    "width": 200,
                    "height": 125
                }
            ],
            "can_autoplay_on_cellular": true
        }
    ]
}
```

A YouTube video, which always has an `id` field in the `metadata` object:

```JSON
{
    "content": [
        {
            "type": "video",
            "provider": "youtube",
            "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
            "embed_html": "<iframe width="560" height="315" src="https://www.youtube.com/embed/dQw4w9WgXcQ" frameborder="0" allowfullscreen></iframe>",
            "embed_url": "https://www.youtube.com/embed/dQw4w9WgXcQ",
            "metadata": {
                "id": "dQw4w9WgXcQ"
            },
            "poster": [
                {
                    "type": "image/jpeg",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                    "width": 500,
                    "height": 400
                }
            ]
        }
    ]
}
```

### Future Content Block Types

New content block types will be added here as needed!

## Layout Blocks

To lay out the blocks of a Post in a way that's different than the default vertical stack, you can use the optional `layout` block alongside the `content` block array. The `layout` block holds an array of layouts. Each layout object requires a `type` field, just like content blocks.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
type | string | yes | The type of layout block; current types should be one of `"rows"`, `"ask"`, or `"condensed"`.

### Layout Default Behavior Recommendations

While it's entirely up to the client to determine how they want to render content blocks based on the given data, we recommend the following approach.

- If no layouts are given in the `layout` array, the content blocks should be rendered in a vertical stack as if there is a `rows` layout with each block in a row in sequential order, i.e. `[[0], [1], [2], [3], ...]`. This should be the starting assumption for rendering content blocks.
- There should only ever be one instance of each type of layout in the `layout` array. For example, there should never be two `rows` layouts, or two `ask` layouts, but there can be one of each. Having more than one layout of the same type is not supported in the official Tumblr clients and the resulting content's appearance may be unexpected.
- If a `rows` layout is present at content creation, it should contain references to **all content block indices**, even if only a subset of the content blocks are being arranged in a grid. For example, if you are placing two images in a single row and one text block in its own row above them, the `rows` layout should contain references to _all three blocks_, even if a content block is by itself in a row: `[[0], [1, 2]]` instead of only `[[1, 2]]`.

### Layout Block Type: Basic Rows

The most basic type of layout block is `"rows"`, which allows you to organize content blocks in rows, with variable elements per row.

```json
{
    "content": [
        {
            "type": "image",
            "media": [
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_1280.jpg",
                    "width": 1280,
                    "height": 1073
                }
            ]
        },
        {
            "type": "image",
            "media": [
                {
                    "type": "image/jpeg",
                    "url": "http://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_1280.jpg",
                    "width": 1280,
                    "height": 1073
                }
            ]
        },
        {
            "type": "text",
            "text": "This is a paragraph underneath two images."
        }
    ],
    "layout": [
        {
            "type": "rows",
            "rows": [
                [0, 1],
                [2]
            ]
        }
    ]
}
```

Each `rows` layout object requires an array of arrays under the `rows` key, each one representing a different row to be rendered with the given content blocks.

In the above example, the two `image` type content blocks are laid out into one row with each image taking up half of the row's total width. The width of each element within the row is determined by however many blocks are in the row. The elements in each `rows` array signifies which content block is placed in the given row position, provided as the index within the `content` array.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
rows | array | maybe | This is an array of the rows and block indices per row, for basic row layouts.
display | array | maybe | This is an array of display objects per row, see [the next section](#layout-block-type-rows-with-display-modes).

Another example, with three paragraphs of text and two photosets between them (media info omitted for easier reading):

```json
{
    "content": [
        {
            "type": "text",
            "text": "Cool pics of Tambler HQ"
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "text",
            "text": "Everybody loves working at tambler."
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "text",
            "text": "and that's all she wrote, folks!"
        }
    ],
    "layout": [
        {
            "type": "rows",
            "rows": [
                [0],
                [1, 2],
                [3],
                [4, 5, 6],
                [7]
            ]
        }
    ]
}
```

In this example you can see that there are five rows, and the client should use the row contents to determine the display width of the content inside. The second row has blocks `1` and `2`, each taking up 50% of the width since there are two blocks in the row, and the fourth row of three blocks, at indexes `4`, `5`, and `6`. Since there are three blocks in that row, each block width will be ~33% of the total width.

Note that the order of the blocks in the `row` arrays are important: the content blocks will be ordered in their row based on their position in the rows arrays. However, blocks _can_ be out-of-order in the `row` array versus in the `content` array:

```json
{
    "content": [
        {
            "type": "image",
            "media": []
        },
        {
            "type": "image",
            "media": []
        },
        {
            "type": "image",
            "media": []
        }
    ],
    "layout": [
        {
            "type": "rows",
            "rows": [
                [2, 0, 1]
            ]
        }
    ]
}
```

### Layout Block Type: Rows with Display Modes

Alternatively, layout objects with the type `"rows"` can have a `display` array instead of the `rows` array. To facilitate richer `rows`-type layout information, `display` is an array of dictionaries containing both the array of `blocks` to be used in the row as well as an optional `mode` dictionary with a specified `type`. The default display mode is `weighted`. The display mode type **does not** need to be specified when creating a Post with NPF content.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
blocks | array | yes | This is an array of block indices to use in this row.
mode | object | no | To specify a display mode other than `weighted`, add an object here with a `type`.

Therefore, the following `display` version...

```json
"layout": [
    {
        "type": "rows",
        "display": [
            {"blocks": [0]}, // Single block, full-width
            {"blocks": [1, 2]}, // Two blocks, each weighted at 50% (e.g. a photoset)
            {"blocks": [3]},
            {"blocks": [4, 5, 6], "mode": {"type": "carousel"}}, // Carousel with three full-width blocks (see "Carousel Display Mode" below)
            {"blocks": [7]}
        ]
    }
]
```

... is compatible with the below "basic" rows version...

```json
"layout": [
    {
        "type": "rows",
        "rows": [
            [0],
            [1, 2],
            [3],
            [4, 5, 6],
            [7]
        ]
    }
]
```

... but doesn't include the display `mode` information about the second to last row.

#### Layout Block Display Mode: Carousel

When using the `display` type of `rows` layout, a `carousel` display mode signifies that the client should use a horizontally paging view where each block occupies 100% of the width of the screen (a "page") and scrolling snaps to the center of a "page".

In the above example, the second `display` row will default to `weighted` and will appear with each of the two items taking up 50% of the width. However, the fourth row should display as a `carousel`, as that `mode` has been specified.

**Note:** Currently only certain Tumblr content creators can create layouts with the `carousel` display mode, but all Tumblr consumers can see it. The API will return a `403 Forbidden` response for ineligible consumers trying to create a carousel.

### Layout Block Type: Condensed

Another type of layout is the `condensed` layout. The `condensed` layout describes how the content should be truncated given a Post with a legacy "read more" signifier. It contains a `truncate_after` property that specifies the last block that should be displayed in the truncated view of the post.

If any blocks exist _after_ the `condensed` layout's `truncate_after` index, they should be ignored when displaying the truncated view of the Post.

```json
"layout": [
    {
        "type": "condensed",
        "truncate_after": 1
    }
]
```

The `truncate_after` property will replace the `blocks` property in future versions of the Tumblr API. The `blocks` property consists of a `blocks` integer array that specifies the block indices that should be displayed as the truncated view of the Post.

During the transition period, API responses will contain both the `truncate_after` and `blocks` properties, as shown below:

```json
"layout": [
    {
        "type": "condensed",
        "truncate_after": 3,
        "blocks": [0, 1, 2, 3]
    }
]
```

Posts may be created or updated using either the `truncate_after` or `blocks` fields, but use of `truncate_after` is recommended.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
truncate_after | int | maybe | The last block to display before the Read More signifier. Required if `blocks` is not supplied.
blocks | array | maybe | This is an array of block indices that are a part of the truncated version of the Post. Required if `truncate_after` is not supplied. Must be sequential, not empty, and begin with 0.

Note that there are certain contexts where the `condensed` layout should not be displayed (namely a permalink revealing the full Post), but the server will return this `condensed` layout for all contexts; this means it is up to the client to determine whether to actually use it. It is expected that the client should use it in every context (a dashboard, via search, etc) except viewing the blog or Post directly.

### Layout Block Type: Ask

When a Post has Ask/Answer content in it, we use a layout object of type `"ask"` to specify which block indices are part of the "ask", as well as what blog (if any) should be attributed as sending the ask. Any blocks not specified in the `blocks` array should be considered a part of the ask's "answer" content. Clients are expected to visually separate these two groups of blocks.

In the below example, blocks at index 0 and 1 are a part of @cyle's ask. Any remaining blocks are a part of the posting blog's answer. If no `attribution` object exists, then the ask is anonymous.

```JSON
"layout": [
    {
        "type": "ask",
        "blocks": [0, 1],
        "attribution": {
            "type": "blog",
            "url": "https://cyle.tumblr.com",
            "blog": {
                "Standard API Short Blog Info object": "goes here, for @cyle's blog"
            }
        }
    }
]
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
blocks | array | yes | This is an array of block indices that are a part of the ask content of the Post.
attribution | [Attribution Object](#attributions) | no | If the ask is not anonymous, this will include information about the blog that submitted the ask.

See also the [Attribution Type: Blog](#attribution-type-blog) section for more information.

## Reblog Trail

Alongside the `content` and `layout` keys at the Post object's root level, there can be an optional `trail` key that holds a reblog trail if the Post is a reblog and the user kept the trail attached. The contents of the `trail` array are the previous Posts in the reblog trail, in order of oldest (the root Post) to the newest (the parent Post).

For example:

```json
{
    "trail": [
        {
            "post": {
                "id": "1234",
            },
            "blog": {
                "Standard API Short Blog Info object": "goes here"
            },
            "content": [
                {
                    "type": "text",
                    "text": "this is the root Post"
                }
            ],
            "layout": []
        },
        {
            "post": {
                "id": "3456",
            },
            "blog": {
                "Standard API Short Blog Info object": "goes here"
            },
            "content": [
                {
                    "type": "text",
                    "text": "this is the parent Post"
                },
                {
                    "type": "text",
                    "text": "this is another text block in the parent Post"
                },
            ],
            "layout": [
                {
                    "type": "rows",
                    "rows": [
                        [1, 0]
                    ]
                }
            ]
        }
    ],
    "content": [
        {
            "type": "text",
            "text": "lol, this is the content i am adding in my reblog of the parent Post"
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
post | object | no | An object with information about the Post in the reblog trail; contains at least an `id` field. This won't be available for ["broken" trail items](#broken-trail-items).
blog | object | no | An object with information about the Post's blog in the reblog trail; contains at least a `uuid` field. This won't be available for ["broken" trail items](#broken-trail-items).
content | array | yes | The content of the Post in the trail.
layout | array | yes | The layout to use for the content of the Post in the trail.
broken_blog_name | string | no | The name of the blog from a broken trail item; see ["broken" trail items](#broken-trail-items).

### "Broken" Trail Items

When converting an old legacy reblog trail to NPF, there are sometimes "broken" parts of the trail, meaning that the original Post cannot be found (i.e. deleted), or the blog itself is broken in some way (i.e. deleted, suspended, etc).

In these cases, the `trail` array will contain a "broken" trail item which has _no Post ID_, but a `broken_blog_name` string (since that's all we have), and the content and layout of the trail item.

```JSON
"trail": [
    {
        "broken_blog_name": "old-broken-blog",
        "content": [
            {
                "type": "text",
                "text": "this is the root Post, which is broken"
            }
        ],
        "layout": []
    },
    {
        "broken_blog_name": "another-broken-blog",
        "content": [
            {
                "type": "text",
                "text": "this is the parent Post, which is also broken"
            },
            {
                "type": "text",
                "text": "this is another text block in the broken parent Post"
            },
        ],
        "layout": []
    }
]
```

## Attributions

Content blocks and layout blocks can have an `attribution` object containing a `link`, `blog`, `post`, or `app` type attribution. Like most objects in NPF, attributions only require the `type` field, and other fields are required based on the given `type`.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
type | string | yes | The type of the attribution; current valid values are `"link"`, `"blog"`, `"post"`, or `"app"`.

### Attribution Type: Post

In the case of GIF search results, the image content block will include a `post` type `attribution` object.

For example:

```JSON
{
    "type": "image",
    "media": [
        {
            "type": "image/gif",
            "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.gif",
            "width": 500,
            "height": 400
        }
    ],
    "attribution": {
        "type": "post",
        "url": "http://www.davidslog.com/153957802620/five-years-of-working-with-this-awesome-girl",
        "post": {
            "id": 1234567890
        },
        "blog": {
            "uuid": "t:123456abcdf",
            "name": "david",
            "url": "https://davidslog.com/"
        }
    }
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | yes | The URL of the Post to be attributed.
post | object | yes | A Post object with at least an `id` field.
blog | object | yes | A Tumblelog object with at least a `uuid` field.

### Attribution Type: Link

In the case of an image with a content source, the image content block will include a link `attribution` object.

For example:

```JSON
{
    "type": "image",
    "media": [
        {
            "type": "image/jpg",
            "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
            "width": 1280,
            "height": 800
        }
    ],
    "attribution": {
        "type": "link",
        "url": "http://shahkashani.com/"
    }
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | yes | The URL to be attributed for the content.

### Attribution Type: Blog

To attribute something to a specific blog on Tumblr, you can use the `blog` type attribution object.

Property | Type | Required | Description
-------- | ---- | -------- | -----------
blog | object | yes | A Tumblelog object with at least a `uuid` field.

For example, ask/answer posts will have layout block that has an array of blocks in the ask and a blog `attribution` object that points to the blog making the ask. In the case of an anonymous ask, no `attribution` object is given.

The `layout` object will have a type of `ask` and an array of `blocks` that indicate which block indexes in the `content` array should be contained within the ask, with a potential `blog` type attribution object. All other blocks in the `content` array should be considered a part of the "answer" to the ask and belong to the posting blog.

```JSON
{
    "content": [
        {
            "type": "text",
            "text": "This is an ask to @cyle from @randerson"
        },
        {
            "type": "text",
            "text": "This is another block in an ask to @cyle from @randerson"
        },
        {
            "type": "text",
            "text": "This is my response to the ask from @randerson!"
        },
    ],
    "layout": [
        {
            "type": "ask",
            "blocks": [0, 1],
            "attribution": {
                "type": "blog",
                "url": "https://randerson.tumblr.com",
                "blog": {
                    "Standard API Short Blog Info object": "goes here, for @randerson's blog"
                }
            }
        }
    ]
}
```

### Attribution Type: App

Block types can have an attribution for what third-party app created them, or were shared from, such as when a user uses an Instagram or Soundcloud embed in their post. We attribute the app in the NPF spec for that block with information about what text (and possibly logo) to use:

```JSON
{
    "content": [
        {
            "type": "video",
            "provider": "instagram",
            "url": "https://www.instagram.com/p/BVZyxTklQWX/",
            "embed_html": "...",
            "media": {
                "url": "https://scontent.cdninstagram.com/t50.2886-16/19229730_166472833892337_5147282940048179200_n.mp4",
                "width": 480,
                "height": 480
            },
            "poster": [
                {
                    "type": "image/jpeg",
                    "url": "https://69.media.tumblr.com/b06fe71cc4ab47e93749df060ff54a90/tumblr_nshp8oVOnV1rg0s9xo1_500.jpg",
                    "width": 500,
                    "height": 400
                }
            ],
            "attribution": {
                "type": "app",
                "url": "https://www.instagram.com/p/BVZyxTklQWX/",
                "app_name": "Instagram",
                "display_text": "tibbythecorgi - Very Cute", // or "Listen on Bandcamp" for audio
                "logo": {
                    "url": "https://scontent.cdninstagram.com/path/to/logo.jpg",
                    "type": "image/jpeg",
                    "width": 64,
                    "height": 64
                }
            }
        }
    ]
}
```

Property | Type | Required | Description
-------- | ---- | -------- | -----------
url | string | yes | The canonical URL to the source content in the third-party app.
app_name | string | no | The name of the application to be attributed.
display_text | string | no | Any display text that the client should use with the attribution.
logo | [Media Object](#media-objects) | no | A specific logo [Media Object](#media-objects) that the client should use with the third-party app attribution.

The `display_text` value may differ for audio and video content:

- Attributions for video content will try to be an artist/author name and title, if available.
- Attributions for audio content will be "Listen on [provider name]" (ex. "Listen on Bandcamp") for all audio providers except SoundCloud, in which case it is only "Listen on" and expected to be appended with the SoundCloud logo by the client.

## Mapping NPF Post Content to Legacy Post Types

### NPF-to-Legacy-Type Waterfall for Original Posts

By default, all new original posts created via NPF are considered legacy `regular`/`text` type posts. The semantic of a single "post type" no longer makes sense for NPF posts that can have a variety of content block types rather than emphasizing a single content type.

However, for backwards compatibility, an original NPF post can be mapped back to a "determined legacy post type" using the following rules, evaluated in this order:

1. If it has an `ask` layout object, it's considered an "answer" post (which is also known as a `note` type post).
1. If it has a video block, it's considered a `video` post.
1. If it has an image block, it's considered a `photo` post.
1. If it has an audio block, it's considered an `audio` post.
1. If it has a text block with the `quote` subtype, it's considered a `quote` post.
1. If it has more than 1 text block with the `chat` subtype, it's considered a `chat`/`conversation` post.
1. If it has a link block, it's considered a `link` post.
1. Otherwise, it's considered a `text`/`regular` post.

### Legacy Post Types of NPF Reblogs

Reblogs of posts, whether done through NPF or legacy flows, retain both the original posts’ type and determined legacy type. This means that an NPF reblog is not necessarily `type: regular` as it always is for NPF original posts (i.e. when it is reblogging a legacy non-text post).

### Changing Legacy Post Type on Edit

If a user edits an NPF post, the determined type may change and be presented differently. For example, if a post has photo, video, and text blocks, and then the user edits it to remove the video block, its determined type will go from `video` to `photo`. This will have downstream effects for its reblogs, since reblogs use their root post's type and determined type.

### Post Types in a Purely NPF World

Someday all official Tumblr apps and services will parse the NPF JSON directly to determine a Post's type(s) when needed, rather than relying on the Post having a single type. With this in mind, for search and rendering purposes an NPF Post could potentially have _several_ legacy types. In the past, a Post on Tumblr has been considered an "atomic unit", alongside other encapsulated things such as a Blog or a Like. The most sematically accurate way to think about "Post types" in a purely NPF world is considering the Content Block as the "atomic unit" instead of the entire Post. A Post can have _several different types_ of content, rather than a Post assuming it only has one type of content.
