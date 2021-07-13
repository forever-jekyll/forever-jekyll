# [Forever Jekyll - A simple, elegant & full featured Jekyll theme](https://forever-jekyll.github.io)
Well, as the name suggests, `Forever Jekyll` is a theme-template-boilerplate for `Jekyll`. Primary goal of `Forever Jekyll` is to make the task of building a personal website with `Jekyll` so easy that even a person with non-computing background should be able to do it with ease.  

You can check-out this animated GIF if you wish to have a sneak peek at `Forever Jekyll` before continuing,  

:trumpet::trumpet: [**Forever Jekyll GIF**](https://i.imgur.com/a6r9sIE.gif) :trumpet::trumpet:  

## Introduction
In an era of internet everyone should have a personal website or a blog. Their own personal space on `WWW`. It shouldn't be too difficult to own a personal web space in 21st century, right? Yes, right but apparently it is not that easy. To be honest it is difficult and confusing. It shouldn't be, but is. `Blogger` has long been dead. `Weebly` and `Wix` are not up to the quality mark to say the least. `WordPress` and `ghost` are not free. Site generators like `Jekyll`, `Hugo`, `Grav`, `Gatsby`, `Pelican` are great but they all require a fair amount of expertise in programming language(s) and a very good knowledge of computers and operating system(s), plus you'll have to host the created site somewhere on your own. This is one of the primary reasons why so many people with a dream of starting a blog or creating their own website end-up leaving their dreams unfulfilled. Now picture is not entirely black, there are a few options and solutions out there that can help an average person create his/her own blog/site without too many hassles and for free. Free as in Beer and free as in Freedom too. `Forever Jekyll` is one of them.  

As the name suggests `Forever Jekyll` is based on static site generator `Jekyll` but it eliminates almost all of the up front setup. No need to install/setup things like `Ruby`, `RubyGems` and no knowledge of `HTML`, `JavaScript` and `CSS` is required. All you'll need to create a blog or a site with `Forever Jekyll` is a **computer**, a **browser** and of-course an **internet connection**. Believe me that's all you'll need and in only a few minutes you'll have your own personal website up and running.  

Before continuing I'm sure you would want to know what `Forever Jekyll` has to offer beside the aforementioned ease of getting started. Here are some of its notable features,  
- Simple, clean and distraction free layout.  
- Responsive theme design.  
- Mobile optimized theme.  
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
  `- title: Facebook
    url: https://facebook.com/your.facebook.username
    icon: fa fa-facebook-square`

One more example. To add Your `reddit` account you need to add this to `footer_links:`,  
  `- title: reddit
    url: https://www.reddit.com/user/your.reddit.username
    icon: fa fa-reddit-square`  
    
Save the file by clicking on `Commit changes` button in the bottom left.  

Time to add your avatar and favicons to your site.  

Avatar first. Select an image that you want to use as your site avatar. Scale it down to **70x70** and save it as  `avatar.png` file. There are many tools out there to do so and if you don't know any I'd suggest `mtPaint`.  
Next in your repository navigate to the folder `assets` and then open the folder `image`. Click on the `avatar.png` file and delete it by clicking on `trashcan icon` at the top right and save the action by clicking on `Commit changes` button at the bottom left.  
Go back to `image` folder and in the top right part click on `Add file` -> `Upload files` and from there upload your own `avatar.png` file. Don't forget to save it by clicking on `Commit changes` button.  

Okay now it's time for favicons. A favicon is a site icon. You'll need an image in `png` format for this. Once you are ready with your image, open the following website and use it to generate the favicons,  

[https://favycon.vercel.app/](https://favycon.vercel.app/)  

This site will let you download the generated favicons in a zip file. Unzip that file and then in your `GitHub` repository go to `assets` -> `image` folder and upload all the extracted image files via `Add file` -> `Upload files`. Don't upload any `xml` or `json` file please.  

In your repository there is a file named `about.md`. Open it and add some content related to your site/blog. You can write a short description about yourself, your background, about your website. Possibilities are endless. Try and make this page interesting. Adding a picture is always a good idea. You can also share ways to contact you on this page if you wish. Once you have finished writing a killer introduction to you and your site, don't forget to save the changes by clicking on `Commit changes` button at the bottom left.  
