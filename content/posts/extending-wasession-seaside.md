---
title: "Extending WASession in Seaside"
date: 2009-06-23
draft: false
---

One of the goals I had with this blog was to make the maintenance and updates dead simple.  Certainly, I could open up the Squeak image every time I want to add a post, but I wanted to be able to edit and create posts directly in the browser.

# What is WASession?

A WASession instance is created when "a user accesses an application for the first time and is persistent as long as the user is interacting with it."  You who is reading this post have an instance of WASession stored in the Squeak image behind this website right now!

The beauty of this system is that you can store information about *who* is viewing the blog in the WASession instance, by subclassing it.  I extended WASession, and simply added an instance variable called `admin`, along with appropriate setter and accessor methods:

- **`WASession`**
  - **`JKSession`** extends **`WASession`**
    - instance variables:
      - `admin`
    - methods:
      - `isAdmin`
      - `setAdmin`
      - `unsetAdmin`
      - `initialize` (runs `self unsetAdmin`)

# Logging in as an administrator

I wanted to be able to do what most people would expect on modern CMS's today: go to a private `/admin` URL and log in to receive special privileges.  Try it, and see what happens!  (I won't put a link so that the ubiquitous Google spider will not find it.)

On the Seaside of things, this involved extending my main **JKLayout** application class, pointing `/admin` to that class with password protection, and then in the initialization method, calling the session's `setAdmin` method.

- **`JKLayout`**
  - **`JKAdminLayout`** extends **`JKLayout`**
    - methods:
      - `initialize` (runs `self session setAdmin`)

To install both of these applications at their respective locations, I ran these commands in a workspace:

```squeak
"registers the application for normal users"
JKLayout registerAsApplication: index

"registers the application for administrators"
"note: prompts you for a username and password"
JKAdminLayout registerAsAuthenticatedApplication: admin
```

# Administrator privileges

Now, whenever I want to have hidden content or extra links that only an administrator can see, it is as simple as: `self session isAdmin ifTrue: [ ... ]`, which can be called within any WAComponent instance.

On my own blog, this allowed me to add links to create, edit and delete posts, also creating the appropriate forms to which these links point.  In addition, I show all unpublished content and mark it as such, allowing me to gradually write draft posts.

# Different user, different content

Similar techniques can be used for sites that have multiple levels of accounts.  For example, you might have an e-commerce website in which the administrator needs to be able to add and edit items for sale, with user accounts with the ability to purchase said items.

Now go forth and code your subclassed WASessions!
