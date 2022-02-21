+++
title = "Restarting my blog"
date = 2022-02-20
+++

Everyone in tech is getting in on the blog business these days. It's all the rage!

This is really a blog post about choice overload and resisting complexity.

# Picking a technology

There are way too many blog technologies out there. I'm someone who easily gets distracted by choice, so I've been putting this off for a while.

## Wordpress

Back in high school, I put together a wordpress site for my Mom's art portfolio. The editor in wordpress is great, but the hosting story isn't. It's being hosted in a virtual private server she pays a small yearly fee for.

The hosting needs maintenance from time to time - she forgot to install updates once, and then the admin page got hacked and filled up with advertisements for "male enhancement" pills. That was not fun to clean up. I always worry about the state of the server. Is it being patched? Will it have downtime? Will she get hacked again?

The load times are also not great. The server isn't particularly fast and it can't be near everyone at once. There are multiple round trips to pull in CSS and javascript stuff from the theme. It just seems like *a lot* to display a simple portfolio. I'm afraid the next time she asks for help with it, I'll have to spend an hour just figuring out what's going on before I can start to fix things.

I won't even link the website here until I feel more confident it's secured.

## Write the HTML yourself

[My friend and colleague Robert Ruark keeps an excellent blog about his electronics projects](http://robruark.com/). He started with an HTML template and hand-writes the HTML for each page. This is not a bad approach and it seems to work well for him. 

I slightly prefer markdown over HTML because I think it's more distraction free, so I went looking for somethign that was more or less like this but with markdown support. There were a few other nice-to-have features missing
* Automatically generating menus based on my post categories
* Scaling down images to web-size

Still, this ended up being my #2 option.

## Content Generators

When I was applying to jobs out of college, I wrote a portfolio with Jekyll hosted on Github Pages. This was pretty reasonable, but I picked a theme with a mess of javascript and CSS and lots of custom shortcodes. I had a strong vision for what the portfolio should look like before I started. I sorted through all of the themes and hacked on one until it matched my vision. 

In hindsight this was a huge waste of effort. I spent hours debugging an issue where none of the links worked in local previews because they went to the public site instead. I had custom shortcodes to do post background images. That was a pain. The theme had some kind of javascript to add smooth scrolling to browsers that didn't natively do it. I *hate* anything that messes with a browser's basic functionality, but I couldn't figure out how to turn it off.

Someone who knows more about Jekyll than I do would tell me that these are dumb problems. They're probably right, but I still don't want to put any mental energy into thinking about them.

## Going with Zola

At this point I was pretty sold on the idea of [JamStack](https://jamstack.org/) - static HTML, generated from markdown & hosted on a CDN that I have to do zero maintenance on.

The options I considered here were Jekyll, Hugo, and Zola. 

I tried all three to see how much overhead was involved in setting up a simple blog. In the end
* Jekyll left a bad taste in my mouth from my college portfolio website. Also, maintaining a ruby environment on my computer just to edit a website isn't something I want to do.
* Hugo was unopinionated, so even getting categories on my blog was different for every theme. I liked the idea but it was trying to be too much of a generalist and putting decisions on me that I didn't want to make.
* Zola had fewer moving parts than Hugo and got me to my minimum viable product in the least amount of time. Maybe there are downsides, but I don't see any relevant ones right now.

I can't put my finger on why I liked Zola over Hugo, but it just seemed to work with less shennanigans for the kind of simple blog I wanted to build. I only needed to commit two files + grab a theme to have my first post (config.toml and the post body)

I timed how long it took to get a proof of concept serving from my computer. Zola won at only 8 minutes from starting the docs to having what I wanted.

### Choosing a theme

This one was pretty simple - I picked a few that looked nice from the list and then went through their demo pages with web inspector. Whichever one had the least amount of round trips + resources to load won. I chose [Anpu](https://www.getzola.org/themes/anpu/) because it only had 3 files to download: My content, a small theme CSS, and a font. No javascript, yay! 

I'm not a frontend developer, but the generated HTML was readable enough I felt confident I'd be able to debug if something broke.

### Porting my old posts

My college projects were a mixed bag. I went through the posts on my old website and kept the ones that looked interesting. They're marked with the "college" tag. To port them from Jekyll to Zola I copied the markdown and rewrote the header section.

# Publishing

I decided to use CloudFlare to host my content, because it seems to be popular on hackernews right now and the [setup guide](https://developers.cloudflare.com/pages/framework-guides/deploy-a-zola-site) worked on the first try.

Not everything has to have a great engineering rationale. Some parts just have to be there and work.

When you get choice fatigued, remember that popular options are often popular for a reason.
Limit yourself to the top 3-4 and pick the one that's simplest.