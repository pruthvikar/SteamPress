# SteamPress

[![Language](https://img.shields.io/badge/Swift-3-brightgreen.svg)](http://swift.org)
[![Build Status](https://travis-ci.org/brokenhandsio/SteamPress.svg?branch=master)](https://travis-ci.org/brokenhandsio/SteamPress)
[![codecov](https://codecov.io/gh/brokenhandsio/SteamPress/branch/master/graph/badge.svg)](https://codecov.io/gh/brokenhandsio/SteamPress)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/brokenhandsio/SteamPress/master/LICENSE)


SteamPress is a Swift blogging engine for use with the Vapor Framework to deploy blogs to sites that run on top of Vapor. It uses [Fluent](https://github.com/vapor/fluent) so will work with any database that has a Fluent Driver. It also incorporates [LeafMarkdown](https://github.com/brokenhandsio/LeafMarkdown) allowing you to write your posts in Markdown and then use Leaf to render the markdown.

The blog can either be used as the root of your website (i.e. appearing at https://www.acme.org) or in a subpath (i.e. https://www.acme.org/blog/).

There is an example of how it can work in a site (and what it requires in terms of Leaf templates and the parameters is passes to them) at https://github.com/brokenhandsio/SteamPressExample.

## Features:

* Blog entries with Markdown
* Multiple user accounts
* Tags on blog posts
* Snippet for posts
* Draft Posts
* Works with any Fluent driver
* Protected Admin route for creating blog posts
* Pagination on the main blog page
* Slug URLs for SEO optimisation and easy linking to posts
* Support for comments via Disqus
* Open Graph and Twitter Card support

# How to Use

## Setup

It's just a single line! Well almost... First add it to your `Package.swift` dependencies:

```swift
dependencies: [
    ...,
    .Package(url: "https://github.com/brokenhandsio/SteamPress", majorVersion: 0)
]
```

Next import it in the file where you are setting up your `Droplet` with:

```swift
import SteamPress
```

Finally, add the provider!

```swift
try drop.addProvider(SteamPress.Provider.self)
```

This will look for a config file called `steampress.json` that looks like:

```json
{
    "postsPerPage": 5,
    "blogPath": "blog"
}
```

The `blogPath` line is optional, if you want your blog to be at the root path of your site, just remove that line.

### Manual initialisation

You can also initialise the Provider manually, by creating it as so:

```swift
let steampress = SteamPress.Provider(postsPerPage: 5)
drop.addProvider(steampress)
```

This will initialise it as the root path of your site. If you wish to have it in a subdirectory, initialise it with:

```swift
let steampress = SteamPress.Provider(postsPerPage: 5, blogPath: "blog")
drop.addProvider(steampress)
```

### Bootstrap Versions

By default, the paginator used by SteamPress is expecting to use Bootstrap 4. You can configure it to use Bootstrap 3 by either adding it to the configuration file or the manual initialisation. To add to a config file, in your `steampress.json`, add:

```json
{
    "postsPerPage": 5,
    "blogPath": "blog",
    "paginator": {
        "useBootstrap4": false
    }
}
```

To manually iniatialise, set up the Provider like so:

```swift
let steampress = SteamPress.Provider(postsPerPage: 5, blogPath: "blog", useBootstrap4: false)
```

### Disabling Routes

You can disable the routes for authors pages and tags pages (both individual and all) by adding the option in your configuration file. To disable all of the authors pages, in your `steampress.json` add: 

```json
{
    "enableAuthorsPages": false
}
```

To disable all of the tags pages, set:

```json
{
    "enableTagsPages": false
}
```

Both of these settings can also be configured if manually setting up the Provider:

```swift
let steampress = SteamPress.Provider(postsPerPage:5, enableAuthorsPages: true, enableTagsPages: true)
```

## Logging In

When you first visit the login page of the admin section of the blog it will create a user for you to use for login, with the username `admin`. The password will be printed out to the console and you will be required to reset your password when you first login. It is recommended you do this as soon as your site is up and running.

## Comments

SteamPress currently supports using [Disqus](https://disqus.com) for the comments engine. To use Disqus, just add a config file `disqus.json` to your site that looks like:

```swift
{
    "disqusName": "NAME_OF_YOUR_DISQUS_SITE" // This can be found from your Disqus admin panel
}
```

This will pass it through to the Leaf templates for the Blog index (`blog.leaf`), blog posts (`blogpost.leaf`), author page (`profile.leaf`) and tag page (`tag.leaf`) so you can include it if needs be. If you want to manually set up comments you can do this yourself and just include the necessary files for your provider. This is mainly to provide easy configuration for the [Platform site](https://github.com/brokenhandsio/SteamPressExample).

## Open Graph Twitter Card Support

SteamPress supports both Open Graph and Twitter Cards. The Blog Post `all` Context (see below) will pass in the created date and last edited date (if applicable) in ISO 8601 format for Open Graph article support, under the parameters `create_date_iso8601` and `last_edited_date_iso8601`.

The Blog Post page will also be passed a number of other useful parameters for Open Graph and Twitter Cards. See the `blogpost.leaf` section below.

The Twitter handle of the site can be configured with a `twitter.json` config file (or injected in) with a property `siteHandle` (the site's twitter handle without the `@`). If set, this will be injected into the public pages as described below. This is for the `twitter:site` tag for Twitter Cards

# Expected Leaf Templates

SteamPress expects there to be a number of Leaf template files in the correct location in `Resources/Views`. All these files should be in a `blog` directory, with the admin template files being in an `admin` directory. For an example of how it SteamPress works with the leaf templates, see the [Example SteamPress site](https://github.com/brokenhandsio/SteamPressExample).

The basic structure of your `Resources/View` directory should be:

* `blog`
  * `blog.leaf` - the main index page
  * `blogpost.leaf` - the page for a single blog post
  * `tag.leaf` - the page for a tag
  * `profile.leaf` - the page for a user profile
  * `tags.leaf` - the page for displaying all of the tags
  * `authors.leaf` - the page for displaying all of the authors
  * `admin`
    * `createPost.leaf` - the page for creating and editing a blog post
    * `createUser.leaf` - the page for creating and editing a user
    * `index.leaf` - the index page for the Admin site
    * `login.leaf` - the login page for the Admin site
    * `resetPassword.leaf` - the page for resetting your password

## Main Blog Site

### `blog.leaf`

This is the index page of the blog. The parameters it will receive are:

* `posts` - a Node containing data about the posts and metadata for the paginator. You can access the posts by calling the `.data` object on it, which is an array of blog posts if there are any, in date descending order. The posts will be made with a `longSnippet` context (see below)
* `tags` - an array of tags if there are any
* `authors` - an array of the authors if there are any
* `user` - the currently logged in user if a user is currently logged in
* `disqus_name` - the name of your Disqus site if configured
* `blog_index_page` - a boolean saying we are on the index page of the blog - useful for navbars
* `site_twitter_handle` - the Twitter handle for the site if configured
* `uri` - the URI of the page - useful for Open Graph


### `blogpost.leaf`

This is the page for viewing a single entire blog post. The parameters set are:

* `post` - the full current post, with the `all` context
* `author` - the author of the post
* `blog_post_page` - a boolean saying we are on the blog post page
* `user` - the currently logged in user if a user is currently logged in
* `disqus_name` - the name of your Disqus site if configured
* `post_uri` - The URI of the post
* `post_uri_encoded` - A URL-query encoded for of the URI for passing to Share buttons
* `site_uri`: The URI of the root site - this is useful for creating links to author pages for `article:author` Open Graph support
* `post_description` - The HTML of the short snippet of the post on a single line with all HTML tags stripped out for the `description` tags
* `post_image` - The first image in the blog post if one is there. Useful for OpenGraph and Twitter Cards
* `post_image_alt` - The alt text of the first image if it exists. Useful for Twitter Cards
* `site_twitter_handle` - the Twitter handle for the site if configured

### `tag.leaf`

This is the page for a tag. A blog post can be tagged with many tags and a tag can be tagged on many blog posts. This page is generally used for viewing all posts under that tag. The parameters are:

* `tag` - the tag
* `posts` - a Node containing data about the posts and metadata for the paginator. You can access the posts by calling the `.data` object on it, which is an array of blog posts if there are any, in date descending order. The posts will be made with a `longSnippet` context (see below)
* `tag_page` - a boolean saying we are on the tag page
* `user` - the currently logged in user if a user is currently logged in
* `disqus_name` - the name of your Disqus site if configured
* `site_twitter_handle` - the Twitter handle for the site if configured
* `uri` - the URI of the page - useful for Open Graph

### `profile.leaf`

This is the page for viewing a profile of a user. This is generally used for viewing all posts written by a user, as well as some information about them. This template is also used by the Admin section for viewing a 'My Profile' page when logged in. The parameters it can have set are:

* `author` - the user the page is for
* `my_profile` - a boolean set to true if we are viewing the my profile page
* `profile_page` - a boolean set to to true if we are viewing the profile page
* `posts` - all the posts the user has written if they have written any in `shortSnippet` form
* `user` - the currently logged in user if a user is currently logged in
* `disqus_name` - the name of your Disqus site if configured
* `site_twitter_handle` - the Twitter handle for the site if configured
* `uri` - the URI of the page - useful for Open Graph

### `tags.leaf`

This is the page for viewing all of the tags on the blog. This provides some more navigation points for the blog as well as providing a page in case the user strips off the tag from the Tag's URL. The parameters that can be passed to it are:

* `tags` - an array of all the tags on the blog, in `withPostCount` context (see below) sorted by post count
* `user` - the currently logged in user if a user is currently logged in
* `site_twitter_handle` - the Twitter handle for the site if configured
* `uri` - the URI of the page - useful for Open Graph

### `authors.leaf`

This is the page for viewing all of the authors on the blog. It provides a useful page for user's to see everyone who has contributed to the site.

* `authors` - an array of all the `BlogUser`s on the blog, in `withPostCount` context (see below) sorted by post count
* `user` - the currently logged in user if a user is currently logged in
* `site_twitter_handle` - the Twitter handle for the site if configured
* `uri` - the URI of the page - useful for Open Graph

## Admin Site

### `index.leaf`

This is the main Admin page for the blog where you can create and edit users and posts. The parameters for this page are:

* `users` - all the users for the site
* `published_posts` - all the posts that have been published if there are any, with the `all` Context
* `draft_posts` - all the draft posts that have been saved but not published, if there are any, with the `all` Context
* `errors` - any error messages for errors that have occurred when trying to delete posts or users (for instance trying to delete yourself or the last user)
* `blog_admin_page` - a boolean set to true, useful for navigation

### `login.leaf`

This is the page for logging in to the admin section of the blog. The parameters are:

* `username_error` - a boolean set if there was an issue with the username
* `password_error` - a boolean set if there was an error with the password (note that we do not pass any password submitted back to any pages if there was an error for security reasons)
* `username_supplied` - the username supplied (if any) when originally submitting the login for and there (useful for pre-populating the form)
* `errors` - an array of error messages if there were any errors logging in

### `resetPassword.leaf`

This is the page you will be redirected to if you need to reset your password. The parameters are:

* `errors` - an array of errors if there were any errors resetting your password
* `password_error` - a boolean set if there was an error with the password (for instance it was blank)
* `confirm_password_error` - a boolean set if there was an error with the password confirm (for instance it was blank)

### `createPost.leaf`

This is the page for creating a new blog post, or editing an existing one. The parameters for this page are:

* `title_error` - a boolean set to true if there was an error with the title
* `contents_error` - a boolean set to true if there was an error with the blog contents
* `errors` - an array of error messages if there were any errors creating or editing the blog post
* `title_supplied` - the title of the blog post to edit, or the post that failed to be created
* `contents_supplied` - the contents of the blog post to edit, or the post that failed to be created
* `tags_supplied` - an array of all of the tags that have been specified for the blog post
* `editing` - a boolean set to true if we are currently editing the a blog post rather than creating a new one
* `post` - the post object we are currently editing
* `create_blog_post_page` - a boolean set to true, useful for the navbar etc

### `createLogin.leaf`

This is the page for creating a new user, or editing an existing one. The parameters are:

* `name_error` - a boolean set if there was an error with the name
* `username_error` - a boolean set if there was an error with the username
* `errors` - an array of error messages if there were any errors editing or creating the user
* `name_supplied` - the name of the user we are editing or that we failed to create
* `username_supplied` - the username of the user we are editing or that we failed to create
* `password_error` - a boolean set to true if there was an error with the password
* `confirm_password_error` - a boolean set to true if there was an error with the password confirm
* `reset_password_on_login_supplied` - a boolean set to true if the edit/create submission was asked to reset the users password on next login (this is only supplied in an error so you can pre-populate the form for a user to correct without losing any information)
* `editing` - a boolean set to true if we are editing a user
* `user_id` - this is the ID of the user if we are editing one. This allows you to send the edit user `POST` back to the correct route


## `POST` Routes

There are a number of `POST` routes to the Admin site for creating and editing user etc that are required otherwise you will receive errors.

This section needs to be filled out, but you can view the Controllers in the code to work out what they should be, or see the [Example Site](https://github.com/brokenhandsio/SteamPressExample).

# Contexts

## Blog Post

The blog post has a number of `Context`s you can pass to the `makeNode()` function to provide more information when getting a `BlogPost`. Currently there are three contexts supported:

* `.shortSnippet` - this will return the post with an `id`, `title`, `author_name`, `author_username`, `slug_url`, `created_date` (Human readable) and `short_snippet`
* `.longSnippet` - this will return the post with an `id`, `title`, `author_name`, `author_username`, `slug_url`, `created_date` (Human readable) and `long_snippet`. It will also include all of the tags in a `tags` object if there are any associated with that post
* `.all` - this returns the post with all information, including both snippet lengths, including author names and human readable dates, as well as both dates in ISO 8601 format under the parameter names `created_date_iso8601` and `last_edited_date_iso8601`.

If no `Context` is supplied to the `makeNode()` call you will get:

* `id`
* `title`
* `contents`
* `bloguser_id` - The ID of the Author of the post
* `created` - The time the post was created as a `Double`
* `slug_url`
* `published` - Whether the post has been published or not

## Blog User

The blog user has a `withPostCount` `BlogUserContext` available to pass into the `makeNode()` function that provides an extra `post_count` parameter to the user node, which contains the number of posts that author has written.

## Blog Tag

The blog user has a `withPostCount` `BlogTagContext` available to pass into the `makeNode()` function that provides an extra `post_count` parameter to the tag node, which contains the number of posts tagged with that tag.

# Snippets

SteamPress supports two type of snippets for blog posts - short and long. Short snippets will provide the first paragraph or so of the blog post, whereas long snippets will show several paragraphs (such as for use on the main blog page, when listing all of the posts)

## Usage

You can pass in a `BlogPostContext` to the `makeNode()` call to provide more information when getting `BlogPost` objects, as shown above.

You can also call them directly on a `BlogPost` object (such as from a `Query()`):

```swift
// These both return the some of the contents of a blog post (as a String)
let shortSnippet = post.shortSnippet()
let longSnippet = post.longSnippet()
```

# Leaf Markdown

LeafMarkdown allows you to render markdown as HTML in your Leaf files. To use, just simply use:

```
#markdown(myObject.markdownContent)
```

This will convert the `Node` object `myObject`'s `markdownContent` to HTML (you pass in `myObject` as a parameter to your Leaf view). It uses CommonMark under the hood, but for more details, see the [LeafMarkdown repo](https://github.com/brokenhandsio/LeafMarkdown).

# API

SteamPress also contains an API for accessing certain things that may be useful. The current endpoints are:

* `/<blog-path>/api/tags/` - returns all the tags that have been saved in JSON

# Known issues

* When the admin user is created when first accessing the login screen, sometimes two are created so you need to use the first password displayed. You can then delete the second Admin user in the Admin pane.

# Roadmap

I anticipate SteamPress staying on a version 0 for some time, whilst some of the biggest features are implemented. Semantic versioning makes life a little difficult for this as any new endpoints will require a breaking change for you to add Leaf templates! However, I will aim to stabilise this as quickly as possible, and any breaking changes will be announced in the [releases](https://github.com/brokenhandsio/SteamPress/releases) page.

On the roadmap we have:

* Sitemap/RSS feed for SEO
* AMP/Facebook instant articles endpoints for posts
* Searching through the blog
* Saving state when logging in - if you go to a page (e.g. edit post) but need to be logged in, it would be great if you could head back to that page once logged in. Also, if you have edited a post and your session expires before you post it, wouldn't it be great if it remembered everything!
