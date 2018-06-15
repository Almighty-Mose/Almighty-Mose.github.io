---
layout: post
title:      "Pull Yourself Up By Your Bootstraps"
date:       2018-06-15 23:13:33 +0000
permalink:  pull_yourself_up_by_your_bootstraps
---


Writing clean HTML and CSS code can often be frustrating for a new programmer. Endless tweaking -- a pixel more here, a margin percent less there -- and finding the code for those tweaks, can be time-consuming for a designer without lots of experience.  Custom layouts are often troublesome to work out.

In today's web development world, developers also face a unique problem. Mobile devices -- smartphones, tablets, laptops -- have become increasingly prolific.  These different screen sizes present quite the challenge for the modern developer. How do I ensure that my webpage is clean, easy to navigate, and engagingly displayed across all of these screens, from the smallest phone to the largest computer monitor?

Enter Bootstrap Framework.

Bootstrap Framework creates a webpage skeleton, an empty chassis on which to build a site.  Initially created by Twitter for use by their inside design team, Bootstrap quickly gained fans and spread to the rest of the development world.  The framework makes it easy to organize and lay out content, even across multiple types of devices.

# The Grid.

<iframe src="https://giphy.com/embed/oSYflamt3IEjm" width="360" height="360" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/loop-vaporwave-oSYflamt3IEjm">via GIPHY</a></p>

Bootstrap Framework gives us access to a series of classes and containers with pre-defined breakpoints for content, utilizing a grid system with 12 units of horizontal space. This allows a developer to quickly flesh out a responsive layout without needing to spend lots of time tweaking CSS code. Check out the example below.

```
<div class="container">
  <div class="row">
    <div class="col-sm-4">
      One of three columns
    </div>
    <div class="col-sm-4">
      One of three columns
    </div>
    <div class="col-sm-4">
      One of three columns
    </div>
  </div>
</div>
```

The above code automatically centers it's content within .container, in 3 columns which each take up 4 of the total 12 width units.  The *sm* addition sets the breakpoints for small, medium, large, and extra large devices. Voila! Now our content will be in 3 automatically scaling columns across all screen widths!

The power of this becomes clear when you begin to think about unique column layouts. Bootstrap allows for the creation of up to 12 columns in any amount that adds up to 12. Want 4 columns, but one needs to be larger than the others? 6, 2, 2, 2.  Need a classic 2/3 two column layout? One column with a width of 8, one with a width of 4. Done. No more dividing up percentages, working around padding, etc. Bootstrap makes laying out columns easy!

# Components
A truly great thing about Bootstrap is its expansive library of components, small bits of HTML code for things like nav bars, forms, dropdowns, alerts, etc.  All of these can be accessed on Bootstrap's [Components](https://getbootstrap.com/docs/4.0/components/alerts/) page, to be easily copied and added to a project!

> Bootstrap Framework takes the grunt work out of building a responsive webpage layout. 
 
Its suite of components and backend HTML classes make it easy to put the focus on creating engaging content for site users, making it a highly desirable tool to have in any developer's box.


