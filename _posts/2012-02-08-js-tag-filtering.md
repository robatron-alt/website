---
layout: post
title:  JavaScript Tag Filtering
tags:   meta old-site

summary: >
    A new addition to my little blog system: simple, JavaScript-based tag
    filtering. It works like this: When the blog-list page is rendered, each
    blog article snippet is wrapped in a div with class names equal to the
    article's tags.

cover_img_filename: ssh-tunnel-whiteboard-diagram.jpg
---

A new addition to my little blog system: simple JavaScript-based tag filtering.
It works like this: When the [blog-list page][blog-list-page] is rendered,
each blog article snippet is wrapped in a `div` with class names equal to the
article's tags:

<pre class='prettyprint lang-html'>
    <div class='tag1 tag2 tag3'>
        <!-- article snippet -->
    </div>
</pre>

[blog-list-page]: https://github.com/robatron/robmd.net/blob/f33250244660ba25f05c291352f6f901dee9af1f/templates/blog-list.html

A list of all of the tags in every article are also rendered in the sidebar,
and JavaScript [click events are attached to each one][click-events]. When a
tag is clicked, the associated click event is fired which hides all of the
article snippets not belonging to that tag using a simple [jQuery][] select and
hide:

<pre class='prettyprint'>
    $(".blog-item").each(function(index){
        if( $(this).find("." + tag).length ){
            // show the snippets that are tagged with `tag`
            $(this).slideDown("slow");
        } else {
            // hide all the ones that aren't
            $(this).slideUp("slow");
        }
</pre>

[click-events]: https://github.com/robatron/robmd.net/blob/7a013ce1f0087e82419f1491ddf1db03faa645e9/templates/blog-base.html#L96
[jquery]: http://jquery.com

Additionally, when a tag is clicked, it adds a hash of the tag to the URL. The
blog system [checks for hashes][checks] on load of the blog list, and filters
the article snippets immediately if any are found. This makes a tag filter
bookmarkable, e.g., [http://robmd.net/blog/#meta](/blog/#meta)

[checks]: https://github.com/robatron/robmd.net/blob/7a013ce1f0087e82419f1491ddf1db03faa645e9/templates/blog-base.html#L104

<br>

## A wok-based solution to tag filtering?

[Mike][] is working on a [wok][] a feature currently dubbed "vary-on" where a
page could be generated multiple times per a variable. For example, this could
be used to create a set of pages for every tag. Current discussion can be found
[here][vary-on-issue].

[mike]: https://github.com/mythmon
[wok]: https://github.com/mythmon/wok
[vary-on-issue]: https://github.com/mythmon/wok/issues/55
