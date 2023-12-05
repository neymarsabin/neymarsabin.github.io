+++
layout = 'post'
title = 'Jekyll And Emacs Org-Mode For Blogging'
date = 2016-10-23T12:00:00+05:45
+++

From  [jekyll website](https://jekyllrb.com/docs/home) 

> Jekyll is a simple, blog-aware, static site generator. It takes a template directory containing raw text files in various formats, runs it through a converter (like Markdown) and our Liquid renderer, and spits out a complete, ready-to-publish static website suitable for serving with your favorite web server. Jekyll also happens to be the engine behind GitHub Pages, which means you can use Jekyll to host your project’s page, blog, or website from GitHub’s servers for free.   
<!--more-->
I think Jekyll is one of the most suitable tool for publishing websites. It is easy to maintain and easy to deploy in github as it is the  engine behind github pages. 


# 

I am not fan of markdown. I use **EMACS** for coding and other stuffs , i use EMACS almost everytime. So, why not publish 
my blog using EMACS! And here it comes the well known **EMACS org-mode** (some people hate it.. <del>:frowning:</del> ). 
**Org-Mode** is a great tool by Carsten Dominik in 2003. Org mode is for keeping notes, maintaining todo lists, planning 
projects, and authoring contents documents and fast and effective plain-text system. It provides features like publishing plain text files into various others like html,pdf, latex and more&#x2026; 

Let's start a jekyll project in a folder called **myBlog**. 

Basic Jekyll structure looks something like this&#x2026;. 

    myBlog
    |- jekyll
    		├── _config.yml
    		├── _drafts
    		|   ├── begin-with-the-crazy-ideas.md
    		|   └── on-simplicity-in-technology.md
    		├── _includes
    		|   ├── footer.html
    		|   └── header.html
    		├── _layouts
    		|   ├── default.html
    		|   └── post.html
    		├── _posts
    		|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
    		|   └── 2009-04-26-barcamp-boston-4-roundup.md
    		├── _data
    		|   └── members.yml
    		├── _site
    		├── .jekyll-metadata
    		└── index.html

For more info [look here](http://jekyllrb.com/docs/structure). 

Anything in the **\_posts** directory will be considered as our blog post.Name of that file should be in the following format:-

    yy-mm-dd-name-of-the-post.md 

Create a folder **org** inside **myBlog** which will hold our org-mode files.
The final directory should look like this:- 

    myBlog
    |- jekyll
    		├── _config.yml
    		├── _drafts
    		|   ├── begin-with-the-crazy-ideas.md
    		|   └── on-simplicity-in-technology.md
    		├── _includes
    		|   ├── footer.html
    		|   └── header.html
    		├── _layouts
    		|   ├── default.html
    		|   └── post.html
    		├── _posts
    		|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
    		|   └── 2009-04-26-barcamp-boston-4-roundup.md
    		├── _data
    		|   └── members.yml
    		├── _site
    		├── .jekyll-metadata
    		└── index.html
    |- org
    		|_ _posts
    			 |-- 2016-10-21-my-first-org-mode-blog.org

> You should setup the directory structure of your org files to mirror that of the Jekyll project. Then when you export your org files as html the files will end up in the correct place in your Jekyll project. I usually place the directory containing my org files in the directory about the Jekyll project directory to make sure that Jekyll doesn't consider .org files to be part of its project. 


# 

The final step is to write some emacs-lisp to configure org html export.Here's the section of my **.emacs** :- 

    (setq org-publish-project-alist
    			'(
    
    	("org-myBlog"
    					;; Path to your org files.
    					:base-directory "/path/to/myBlog/org/"
    					:base-extension "org"
    
    					;; Path to your Jekyll project.
    					:publishing-directory "/path/to/myBlog/jekyll/"
    					:recursive t
    					:publishing-function org-html-publish-to-html
    					:headline-levels 4 
    					:html-extension "html"
    					:body-only t ;; Only export section between <body> </body>
    		)
    
    
    		("org-static-myBlog"
    					:base-directory "/path/to/myBlog/org/"
    					:base-extension "css\\|js\\|png\\|jpg\\|gif\\|pdf\\|mp3\\|ogg\\|swf\\|php"
    					:publishing-directory "/path/to/myBlog/"
    					:recursive t
    					:publishing-function org-publish-attachment)
    
    		("blog" :components ("org-myBlog" "org-static-myBlog"))
    
    ))

I write my blog posts inside **myBlog/org/<sub>posts</sub>/** directory and to export i do `M-x org-publish-project RET blog`.
If everything is correct and directory structure is maintained as above your blog post should be published.
