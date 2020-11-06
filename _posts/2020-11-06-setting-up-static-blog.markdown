---
layout: post
title:  "Setting up Static Blog with Jekyll!"
date:   2020-11-06 13:03:33 +0800
categories: guides
---

Compilation of useful links for installation - there's already comprehensive documentation out there so this is just aggregating them.


What you'll need:


1) Linux distro (I was using Windows Linux Subsystem) : <a href="https://docs.microsoft.com/en-gb/windows/wsl/install-win10" target="_blank">https://docs.microsoft.com/en-gb/windows/wsl/install-win10</a>

-- Make sure to enable Virtualization in BIOS (check Task Manager > Performance > CPU > Virtualization Enabled)

-- Make sure to update all linux stuff


2) Follow the installation steps here: <a href="https://jekyllrb.com/docs/installation/windows/" target="_blank">https://jekyllrb.com/docs/installation/windows/</a>


-- Make sure to install and update gem

-- Don't forget to install minima theme

{% highlight shell %}
gem install minima
{% endhighlight %}


3) Setup your GitHub repository: <a href="https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll" target="_blank">https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll</a>


-- The installation step proposed in the link did not work for me, so try below if you have the same error below

{% highlight shell %}
$ jekyll 4.1.1 new .
fatal: 'jekyll 4.1.1' could not be found. You may need to install the jekyll-4.1.1 gem or a related gem to be able to use this subcommand.

$ jekyll new .
{% endhighlight %}


4) Commit and push to GitHub

-- I made a mistake here on not pulling from origin first hence it took me a while to commit my changes

-- If you have 2FA enabled, don't forget to generate developer tokens and use that as your accounts' password


5) Setup your subdomain : <a href="https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-custom-domains-and-github-pages" target="_blank">https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-custom-domains-and-github-pages</a>

-- Basically just update CNAME record and point it to yourrepository.github.io

-- Enable HTTPS if you want to (recommended)


6) Update your ABOUT page and CONFIG (site name, headers, etc.)

-- _config.yml, about.markdown


7) Enjoy!
