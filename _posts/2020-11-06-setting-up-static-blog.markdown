---
layout: post
title:  "Setting up Static Blog with Jekyll!"
date:   2020-11-06 13:03:33 +0800
categories: guides
---


<p>Compilation of useful links for installation - there's already comprehensive documentation out there so this is just aggregating them.</p>
<p>What you'll need:</p>
<ol>
<li>Linux distro (I was using Windows Linux Subsystem) : <a href="https://docs.microsoft.com/en-gb/windows/wsl/install-win10" rel="nofollow">https://docs.microsoft.com/en-gb/windows/wsl/install-win10</a></li>
</ol>
<p>-- Make sure to enable Virtualization in BIOS (check Task Manager &gt; Performance &gt; CPU &gt; Virtualization Enabled)</p>
<p>-- Make sure to update all linux stuff</p>
<ol start="2">
<li>Follow the installation steps here: <a href="https://jekyllrb.com/docs/installation/windows/" rel="nofollow">https://jekyllrb.com/docs/installation/windows/</a></li>
</ol>
<p>-- Make sure to install and update gem</p>
<p>-- Don't forget to install minima theme</p>
<p>{% highlight shell %}
gem install minima
{% endhighlight %}</p>
<ol start="3">
<li>Setup your GitHub repository: <a href="https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll">https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll</a></li>
</ol>
<p>-- The installation step proposed in the link did not work for me, so try below if you have the same error below</p>
<p>{% highlight shell %}
$ jekyll 4.1.1 new .
fatal: 'jekyll 4.1.1' could not be found. You may need to install the jekyll-4.1.1 gem or a related gem to be able to use this subcommand.</p>
<p>$ jekyll new .
{% endhighlight %}</p>
<ol start="4">
<li>Commit and push to GitHub</li>
</ol>
<p>-- I made a mistake here on not pulling from origin first hence it took me a while to commit my changes</p>
<p>-- If you have 2FA enabled, don't forget to generate developer tokens and use that as your accounts' password</p>
<ol start="5">
<li>Setup your subdomain : <a href="https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-custom-domains-and-github-pages">https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-custom-domains-and-github-pages</a></li>
</ol>
<p>-- Basically just update CNAME record and point it to yourrepository.github.io</p>
<p>-- Enable HTTPS if you want to (recommended)</p>
<ol start="6">
<li>Update your ABOUT page and CONFIG (site name, headers, etc.)</li>
</ol>
<p>-- _config.yml, about.markdown</p>
<ol start="7">
<li>Enjoy!</li>
</ol>
