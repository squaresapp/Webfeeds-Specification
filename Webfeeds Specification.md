# Webfeeds Specification



## Abstract

Webfeeds are a way of using HTML, CSS, and the existing HTTP protocol in order to achieve self-hosted media syndication using nothing other than a boring static HTTP file server. Webfeeds introduce almost nothing that is new. This is **by-design**. 

RSS is/was a dumpster fire of biblical proportions. But it showed to the world that self-hosted media syndication is possible without complex infrastructure. Webfeeds build upon this idea. In fact, Webfeeds can be thought of as *RSS done right*. 



### Introduction

A Webfeed is just a series of HTML pages, refered to as *HTML posts*. In an HTML post, there's no `<body>` section. There's just an optional `<head>` section followed by a series of top-level `<section>` elements. Webfeed pages render without issue in a browser.

Each top-level `<section>` element is rendered as a separate screen whose display height is equal to one full-length height of the containing browser window. HTML posts therefore present like slides of a presentation. Each `<section>` is self-contained. CSS overflowing is not allowed.

The first `<section>` of an HTML post is called the *lead*. Readers will cherry-pick the lead out of each HTML post, and organize them into a grid. This is how the feed user interface is constructed. Users click on the lead for the post they want to view, to reveal the other sections.

Syndication is done by polling a text file for changes to `Content-Length`. The text file contains a list of URIs that point to each post in the feed. Users subscribe to Webfeeds by adding the URL of the Webfeed text file to their reader, such as `www.example.com/index.txt`. 



### Example HTML Post

```html
<!DOCTYPE html>

<!--
The user's name is extracted from the meta author tag.
This would be displayed in the identity area of the feed reader.
-->
<meta name="author" content="Elon Musk">

<!--
The users's bio is extracted from the meta description tag.
This would also be displayed in the identity area of the feed reader.
-->
<meta name="description" content="Founder of Tesla, SpaceX, and a few other things.">

<!--
Avatars for the feed are rendered within the feed reader
by inspected the standard shortcut icon tag.
-->
<link rel="icon" type="image/png" href="avatar.png">

<!--
CSS files can be drawn in, and the styling is applied to
all sections as authors would expect.
-->
<link rel="stylesheet" type="text/css" href="style.css">

<section>
  <p>
   	This section is the lead. HTML and CSS content in here will be
	  cherry picked and displayed on the user's post grid.
  </p>

</section>

<section>
  <p>
    This is another section that is buried. In order to reveal this,
    the user would need to have clicked on the lead in the grid.
  </p>
</section>
```



### Ignored HTML Structure

In order to streamline the user experience of the reader, and to improve security, Webfeed readers ignore the HTML tags listed below:

- `<frame>`
- `<frameset>`
- `<script>`
- `<iframe>`
- `<portal>`

Additionally any HTML attribute that starts with `on`, such the as `onclick` are stripped before being rendered. JavaScript does not execute within the context of a reader. Some readers may allow white-list of trusted scripts, but won't be part of any official specification. 



### Example Feed Text File

```
# Lines that start with a # sign are considered comments
# Most lines in this URL list will be relative URIs, which
# are relative to the URL of the folder where the feed text
# file was found.

/post-1
/post-2
/post 3

# If the URL is fully-qualified and points to a URL on a
# different domain, this is interpreted by the reader as
# as a repost, and the reader will indicate this within
# it's user interface.

https://www.myfriendsfeed.com/post-a
https://www.myfriendsfeed.com/post-b
```



### Polling For Updates

Webfeed readers periodically issue an HTTP `HEAD` request to the URL of the feed text file. The reader compares the returned HTTP `Content-Length` header values from the last poll to determine if a full re-download of the index is necessary.

Feed readers should avoid using the `Last-Modified` header. Readers should time-stamp posts based on the time when they are discovered, rather than relying on declarations made by the server. These header values are easily modified, and if the reader is using chronological ordering of posts, this could incentivize manipulation.

When a reader detects an update to a feed text file, it will redownloads the entire text file, parses it, and adds any newly discovered URLs to its feed, and removes any posts from the feed that where removed from the text file. Third party feed management software may use this functionality to achieve the ephemeral posting functionality such as that found in SnapChat and Instagram Stories.

Authors are discouraged from excessive-posting. Readers may implement strategies to mitigate abusers, such as merging multiple posts into a single tile on the feed grid, or omitting some posts all together.

