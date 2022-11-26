---
title: "Waterfall vs. iterative development"
date: 2009-05-18
draft: false
---

The waterfall method of development is inherently broken.  Applications evolve, requirements evolve, clients change their mind, and the web moves forward at an ever-accelerating pace.

Why were developers forced into using waterfall development in the past?

# Definitions

- **Waterfall development** completes the project-wide work-products of each discipline in a single step before moving on to the next discipline in the next step. Business value is delivered all at once, and only at the very end of the project.

- **Iterative development** slices the deliverable business value (system functionality) into iterations. In each iteration a slice of functionality is delivered through cross-discipline work, starting from the model/requirements through to the testing/deployment.

(From [Wikipedia: Iterative and incremental development](http://en.wikipedia.org/wiki/Iterative_and_incremental_development))

# Waterfall development: physical examples

In some cases, falling down the boulder-strewn and dangerous waterfall is both inevitable and unavoidable.  For example:

- Toyota builds a car.  This particular car happens to be a Corolla, and encounters minimal problems.  Hooray!  Compare, however, with a Ford Windstar.  Malfunctions and repairs galore.  Can Ford fix globally these issues as time goes on?  Nope.

- Intel releases a CPU with a little-known bug in the floating-point arithmetic unit.  99.99% of the time, the CPU spits out the correct answer.  But can Intel fix it?  Not unless they announce a recall and send out repaired versions of all chips sold.

- In May 1990, Windows 3.0 is released and shipped to customers worldwide.  It is installed on millions of computers, and is not updated until the release of Windows 3.1 in March 1992.

# A compromise: the advent of the internet

The internet allows software a major advantage over physical entities such as those described above.  It enables numerous different ways of streamlining updates directly to the end-user:

- Users can easily download updated versions of their software themselves.  This usually doesn't happen.

- Software can pop up annoying messages reminding users to update their software.  This is annoying.  If you're lucky, it can also make the update as easy as clicking a button.

- Package management systems. Here's where the money is. Linux distributions with powerful package management software and rich repositories can typically be updated by the user with a single command.

Thus releasing software more often becomes possible.

# The ideal solution: web applications

Web applications are ideal because they do not require versions at all.  The application can simply be updated on the server side, gradually adding features as needed.

Of course, it remains a good idea to use source control and attribute version numbers to your application "releases" behind the scenes.  It simply allows for these releases to become live for all users of the application instantaneously.

Thus web applications provide the ultimate freedom in releasing software whenever needed.  This allows for advantageous iterative approaches to development, including the **agile** and **scrum** methods being buzzed about lately.

# Live programming

The software on which this web application runs, Squeak, is unconventional (but not unique) in that it can be updated while it is running.  This will be investigated in an upcoming post.
