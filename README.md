# [Forever Jekyll - A simple, elegant & full featured Jekyll theme](https://forever-jekyll.github.io)
`Forever Jekyll` is a theme-template-boilerplate for `Jekyll`. Primary goal of `Forever Jekyll` is to make the task of building a personal website with `Jekyll` so easy that even a person with non-computing background should be able to do it with ease.  

## Introduction
In an era of internet everyone should have a personal website or a blog. Their own personal space on `WWW`. It shouldn't be too difficult to own a personal web space in 21st century, right? Yes, right but apparently it is not that easy. To be honest it is difficult and confusing. It shouldn't be, but is. `Blogger` has long been dead. `Weebly` and `Wix` are not up to the quality mark to say the least. `WordPress` and `ghost` are not free. Site generators like `Jekyll`, `Hugo`, `Grav`, `Gatsby`, `Pelican` are great but they all require a fair amount of expertise in programming language(s) and a very good knowledge of computers and operating system(s), plus you'll have to host the created site somewhere on your own. This is one of the primary reasons why so many people with a dream of starting a blog or creating their own website end-up leaving their dreams unfulfilled. Now picture is not entirely black, there are a few options and solutions out there that can help an average person create his/her own blog/site without too many hassles and for free. Free as in Beer and free as in Freedom too. `Forever Jekyll` is one of them.  

As the name suggests `Forever Jekyll` is based on static site generator `Jekyll` but it eliminates almost all of the up front setup. No need to install/setup things like `Ruby`, `RubyGems` and no knowledge of `HTML`, `JavaScript` and `CSS` is required. All you'll need to create a blog or a site with `Forever Jekyll` is a **computer**, a **browser** and of-course an **internet connection**. Believe me that's all you'll need and in only a few minutes you'll have your own personal website up and running.  

Before continuing I'm sure you would want to know what `Forever Jekyll` has to offer beside the aforementioned ease of getting started. Here are some of its notable features,  
- Simple, clean and distraction free layout.  
- Responsive theme design.  
- Mobile optimized theme.  
- Dark mode toggle
- Good looking and readable font stack.  
- Font Awesome icon set.  
- Search engine optimization.  
- Sass/SCSS preprocessor support.  
- Privacy friendly commenting system (optional).  
- Privacy friendly analytics system (optional).  
- Syntax highlighting.  
- Multimedia content (Video, Audio, Images, Playlists, Maps) embedding.  
- Lightbox for images and videos.  
- Math typesetting.  
- Diagrams and charts.  
- Social sharing buttons for over 10 social networks.  
- Page navigation (pagination).  
- Post navigation.  
- Post categories.  
- Post read time.  
- Site search.  
- RSS feed.  
- Emojis.  
- And last but not the least free hosting support out-of-the-box.  

Not convinced yet about `Forever Jekyll`? Here is a **live demo** of `Forever Jekyll`,  

:trumpet::trumpet: [**Live Demo of Forever Jekyll**](https://forever-jekyll.github.io) :trumpet::trumpet:  

Hopefully that live demo has convinced you, hasn't it? Great! Continue with me...  

## Getting started
In the top-right corner of this page, click `Fork`.  
`GitHub` will prompt you to sign in. If you don't have a `GitHub` account you can easily create one from that page.  
Next rename your newly forked repository to `yourgithubusername.github.io`. This is an important step.  
For example your `GitHub` user name is `elaine-thompson` then name of your repository will be,  
`elaine-thompson.github.io`  

Another example. If your `GitHub` user name is `bolt-usain` then name of your repository will be,  
`bolt-usain.github.io`  
That's it. Your site will be available a few seconds later at `https://yourgithubusername.github.io` - if not, give it a few minutes as `GitHub` suggests and it'll appear soon.  

## Customizing your site
Now that your site is up and running it's time to customize it and truly make it yours.  

Click on `_config.yml` file in your repository and next click on the `pencil icon` on the right hand corner.  
First thing you'd want to change in this file is the name of your website. Edit the following line with your desired site name,  
```
name: Forever Jekyll
```  

Next, it's time to change the description of your website. To do so edit this line,  
```
description: A simple, elegant and full featured Jekyll theme
```
And finally change the url to your sites url,  
```
url: https://forever-jekyll.github.io
```  

You can add links to some or all of your social networks, online accounts, emails, even phone numbers if you wish to the footer at the bottom of your site.  
To do so you need to edit the `footer_links:` section in the file. Remove the ones used by the theme first and then add some of your accounts.  
For example you can add a link to your `Facebook` account like this,  
  ```
  - title: Facebook  
    url: https://facebook.com/your.facebook.username  
    icon: fa fa-facebook-square  
```  

One more example. To add Your `reddit` account you need to add this to `footer_links:`,  
```
- title: reddit  
  url: https://www.reddit.com/user/your.reddit.username  
  icon: fa fa-reddit-square  
```  

Save the file by clicking on `Commit changes` button in the bottom left.  

Time to add your avatar and favicons to your site.  

Avatar first. Select an image that you want to use as your site avatar. Scale it down to **70x70** and save it as  `avatar.png` file. There are many tools out there to do so and if you don't know any I'd suggest `mtPaint`.  
Next in your repository navigate to the folder `assets` and then open the folder `image`. Click on the `avatar.png` file and delete it by clicking on `trashcan icon` at the top right and save the action by clicking on `Commit changes` button at the bottom left.  
Go back to `image` folder and in the top right part click on `Add file` -> `Upload files` and from there upload your own `avatar.png` file. Don't forget to save it by clicking on `Commit changes` button.  

Okay now it's time for favicons. A favicon is a site icon. You'll need an image in `png` format for this. Once you are ready with your image, open the following website and use it to generate the favicons,  

[https://favycon.vercel.app/](https://favycon.vercel.app/)  

This site will let you download the generated favicons in a zip file. Unzip that file and then in your `GitHub` repository go to `assets` -> `image` folder and upload all the extracted image files via `Add file` -> `Upload files`. Don't upload any `xml` or `json` files please.  

In your repository there is a file named `about.md`. Open it and add some content related to your site/blog. You can write a short description about yourself, your background, about your website. Possibilities are endless. Try and make this page interesting. Adding a picture is always a good idea. You can also share ways to contact you on this page if you wish. Once you have finished writing a killer introduction to you and your site, don't forget to save the changes by clicking on `Commit changes` button at the bottom left.  

## Enabling comments
Comments are not enabled by default in `Forever Jekyll` but this functionality can easily be added.  
`Forever Jekyll` uses `Cactus Comments`. It is a privacy respecting federated comment system built on Matrix protocol.  
`Cactus Comments` is completely free and open source.  
First you need to register your site with `Cactus Comments`. To do so, click on the following link,  

[chat with @cactusbot:cactus.chat](https://matrix.to/#/@cactusbot:cactus.chat)  

Clicking on that link will take you to the `Matrix` page and you'll be prompted to choose an app to continue.  
Select `Element` and click on `continue`.  
On the next page click on `Continue in your browser`.  
On the next page click on `Send a Direct Message`. It'll prompt you to either `Sign In` or to `Create Account`.  
After signing in to your account send following text as a direct message,  
```
register <YOUR-SITE-NAME>
```  
For example,  
```
register elaine-thompson.github.io
```  
50% of work done.  

Right click on the following link and select `Save link as...` option and save the file with the name `default.html`.  

[default.html](https://github.com/forever-jekyll/propaganda/raw/a6a98c79f871fb51fc18ba2fc6f3364ce33c4d62/default-with-comments-.html)  

Right click on the following link and again select `Save link as...` option and save the file with the name `post.html`.  

[post.html](https://github.com/forever-jekyll/propaganda/raw/main/post-with-comments.html)  

Save the following link with the name `_cactus.scss` just like we did earlier.  

[_cactus.scss](https://github.com/forever-jekyll/propaganda/raw/main/_cactus.scss)  

Now Navigate back to your site repository and open the folder `_layouts`.  
Upload the `default.html` file via `Add file` -> `Upload files` option.  
Open this file `default.html` in your site repository and go to the line number **120**,  
```
siteName: "elaine-thompson.github.io",
```  
Replace `elaine-thompson.github.io` with the site name you just registered on `Cactus Comments`.  
Don't forget to save the changes by clicking on `Commit Changes` button.  

Next upload the `post.html` file to the folder `_layouts` via `Add file` -> `Upload files` option.  
In your site repository go to folder `_sass` and upload `_cactus.scss` file via `Add file` -> `Upload files` option.  
And finally open the file `style.scss` in your site repository. Add the following line at the very end of the file,  
```
@import "cactus";
```  
Save the changes by clicking on `Commit Changes` button.  

That's it. Kudos! You have successfully enabled commenting system on your site.  

## Enabling web analytics
Like comments web analytics is not enabled by default in `Forever Jekyll` but this can easily be added.  
`Forever Jekyll` uses `GoatCounter`. It is an open source and privacy friendly web analytics platform.  
`GoatCounter` is free for non-commercial use. To sign up for `GoatCounter` click open the following link,  

[Sign up for GoatCounter](https://www.goatcounter.com/signup)  

Next navigate back to your site repository and open the file `default.html` in the folder `_layouts`.  
Right at the end of the file you'll see following closing tags,  
```
</body>
</html>
```  
Add the following snippet right before those closing tags,  
```
<script data-goatcounter="https://MYCODE.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```  
Of-course you'll have to replace `MYCODE` in the snippet above with your actual `GoatCounter` code acquired during the sign up.  
Save the changes by clicking on `Commit Changes` button.  

Well-done! You have successfully enabled an ethical and privacy friendly web analytics system for your site.   

## Support

If you like `Forever Jekyll`, please consider supporting it, even the smallest contribution goes a long way:  

[![Support via PayPal](https://cdn.jsdelivr.net/gh/twolfson/paypal-github-button@1.0.0/dist/button.svg)](https://paypal.me/hakerdefo)  
[!["Buy Me A Coffee"](https://user-images.githubusercontent.com/1376749/120938564-50c59780-c6e1-11eb-814f-22a0399623c5.png)](https://www.buymeacoffee.com/hakerdefo)  
[![Support via Liberapay](https://liberapay.com/assets/widgets/donate.svg)](https://liberapay.com/hakerdefo/donate)  

## Writing posts
The `_posts` folder is where your blog posts live.  
Posts in `Jekyll` are text files and `Jekyll` provides everything you need to turn it into a blog.  
`Jekyll` posts are typically written in `Markdown` but you can also write in `HTML` if you wish.  
All post files must begin with front matter. What's a front matter?  
A front matter is typically used to set a layout or other meta data.  
But at this point in time all that technical stuff is strictly not necessary for you to remember.  
All you need to remember is, you'll have to begin every new post with following snippet,  
```  
---
layout: post
title: Title of your post
categories: [category 1, category 2]
---
```  
`title:` in the above snippet is obviously the title of your post.  
`categories:` A blog category can either be a word or a group of words separated by a comma.  
Choose them wisely and use them to give a gist of the post or post's content.  
`Categories` can play a crucial role in organizing your site’s posts properly. It is also considered a good SEO practice.  
`Categories` makes it easy for your potential web visitors to navigate around your site and offers a good user experience.  

Now coming to actual writing. `Markdown` doesn't require any special application. You can write in `Markdown` everywhere.  
Here is a very good resource to get you started with `Markdown`=> [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).  
But for someone who is new to `Markdown` I would recommend any of these three editors,  
 - [Abricotine](https://abricotine.brrd.fr/)  
 - [Mark Text](https://marktext.app/)  
 - [Zettlr](https://www.zettlr.com/)  

After you have completed your post, you'll have to save the post file in `_posts` folder in your repository.  
`Jekyll` is very particular about the post filename format. Every post filename must follow this format,  
```
YEAR-MONTH-DAY-title.md
```  
An example,  
```
2011-12-31-new-years-eve-is-awesome.md
```  
One more example,  
```
2012-09-12-how-to-write-a-blog.md
```  
A bit odd but not that difficult, right?!?  

I've included a sample post in `_posts` directory to get you started. Feel free to delete it when you are ready with your first post.  

## Conclusion
I'm pretty sure that this should be good enough for almost everyone to get started with `Forever Jekyll` in particular and `Blogging` in general.  
But I'm also aware that I must have missed a few things in this README and there will be a few unexplained things left behind.  
In such a case feel free to [start a discussion](https://github.com/forever-jekyll/forever-jekyll/discussions/). Everything related to `ForeverJekyll` is welcome.  

> Your personal website or blog might not change the world but it will certainly change you.  
~ Me  

> It is necessary to write, if the days are not to slip emptily by. How else, indeed, to clap the net over the butterfly of the moment? for the moment passes, it is forgotten; the mood is gone; life itself is gone.  
~ Vita Sackville-West  
