---
title: IntelliJ IDEA&#58; Beyond The Basics
author: Hamza Belmellouki
categories: [IntelliJ]
tags: [IntelliJ]
comments: true
---
IntelliJ IDEA Ultimate is the most powerful IDE for JVM developers in the market by now. It has support for various JVM frameworks, complex refactorings, Integration with VCS, and many more.

Java developers spend a tremendous amount of time in front of their IDEs. Unfortunately, I've noticed that developers don't take advantage of IDEA's powerful features.

In this blog, I'll talk about some tricks that I use my day to day job. And show you some best practices that can boost your productivity 😉.

# UI-Less mode

Jetbrains developer advocates recommend working with IntelliJ without using the UI (tool window buttons, toolbar, etc.). Always remember that all shortcuts are configurable. So make sure to take your time to customize the IDE according to your need and preferences.

Up till now, most developers I've met are using IntelliJ with UI enabled.  
![](https://i.imgur.com/G9gx6EM.png)

However, it isn't recommended to do it that way! If you want to take your IDEA's skills to the next level, you have to get rid of those UIs. They represent distraction to the developer, preventing you from focusing on what matters: Your code!

Now you can remove these distractions by going into your **Main Menu \| View \| Appearance** and uncheck toolbar, tool window bar, navigation bar. What's left are tabs. Please get rid of them by right-click into the tab and choose **Configure Editor Tabs...** and on the appearance section, set **tab placement** to **None**. The resulting IDEA's UI would be:

![](https://i.imgur.com/kpltSyQ.png)

Fantastic! I've been working with IDEA for almost two years using this mode. At first, it may sound overwhelming the number of shortcuts/actions you'll have to learn. Learning IDEA's shortcuts is like driving. When you know how to drive a car, you don't think about the process of driving. It comes naturally. But when driving for the first time you do. So be patient on yourself.

## Running code

How do you run code in IDEA? Typically, you would go to **Edit Configuration** in the toolbar and set up your run configuration. Once you do that, you build and run your code by clicking the run button at the toolbar. Using the mouse to run your code isn't professional. Instead, I prefer to do it the following: You can set up multiple run configurations using a keyboard shortcut and run the runnable class directly from IDEA so IDEA can create the run configs using its defaults. Then, whenever you want to run your project. Invoke the run popup (using a shortcut) and choose the appropriate config: 
<img src="https://media.giphy.com/media/l36D9ivFJOvc2c9yAD/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

## Abbreviate Actions

Sometimes actions can be hard to remember and take more time to type compared to an abbreviation. Fortunately, in IDEA you can set up an abbreviation for an action, and when you invoke **Find Action** you can just type the name of the abbreviation, and it'll invoke the action that is associated with it. First, you can set an abbreviation like so:

<img src="https://media.giphy.com/media/KHPOeQYUTJBNLIJmhK/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

After setting your action abbreviation. Call it using **Find by Action**. Type your abbreviation and boom your action will appear in two keystrokes:

<img src="https://media.giphy.com/media/KDtmbLZypP3Erz0TF8/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

## Navigation

In IntelliJ, you don't need tabs. Tabs are inefficient in navigation. Instead, IDEA provides you powerful ways to navigate in your project. You can navigate using "**Recent Files**" action. This action shows all the recent files you've visited. A double shortcut would show only recent files that have been changed:

<img src="https://media.giphy.com/media/gGqn826NbrurZthsCE/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

**Recent Locations** action is also a great way to navigate/search for recent portions of code:

<img src="https://media.giphy.com/media/Y4DC7WiZrfq8CVUXFH/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

## File Preview

What if you want to preview a file without opening it? **Quick Definition** action can do just that. You invoke it from multiple contexts: 

* In the editor itself
* In the project tool window
* In **Go To Class** action popup
* Etc...

This GIF showcase how I use **Quick Definition** action to preview a specific file:

<img src="https://media.giphy.com/media/cNZ2UStMMqCOL9euJC/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

# VCS

I could write a bunch of articles talking about VCS support in IDEA. But in this article, I'd like to highlight on an unknown feature of IntelliJ, which IntelliJ's VCS Shelve. Shelving commits is similar to stashing commits in git, but it's more powerful. **I would prefer to shelve changes instead of stashing them if I am not sharing my changes elsewhere.**

Stashing is a git feature and doesn't give you the option to select specific files or changes inside a file. Shelving can do that, but this is an IDE-specific feature, not a git feature:

<img src="https://media.giphy.com/media/LpFJWC5yiBXtIxjyMd/giphy.gif" alt="grab-landing-page" style="zoom:150%;" />

As you can see, I am able to choose to specify which files/lines to include on my shelve. Note that I can't do that with stashing.

Beware using shelves in the IDE may limit the portability of your patches because those changes are not stored in a .git folder. But this isn't important since we use remote branches to showcase work. 

# Learn More

If you want to measure your productivity and see usage statistics for IntelliJ IDEA features, make sure to check out **Productivity Guide** by going to the main menu -> help -> **Productivity Guide**. It should display a popup containing all usage statistics of IDEA's features: 
![](https://i.imgur.com/dgdo7L3.png)

Also, Checkout [Key Promoter X](https://plugins.jetbrains.com/plugin/9792-key-promoter-x) plugin if you want to avoid using the mouse.

Finally, the [manual guide](https://www.jetbrains.com/help/idea/2020.2/getting-started.html#contact-support) is the most comprehensive resource you can find online. So, make sure to skim through the manual if you want to know what other things you can do with IDEA.



# Wrap Up

In this blog, I've shown you how you can take your IDEA's skills to the next level by using a UI-Less mode as Jetbrains developer advocates recommend. We also saw how to navigate and work with VCS in IDEA.

If you have any feedback about my blogs. Please, don't hesitate to reach out to me or just say a "hello" on twitter: [@HamzaLovesJava](https://twitter.com/HamzaLovesJava).



