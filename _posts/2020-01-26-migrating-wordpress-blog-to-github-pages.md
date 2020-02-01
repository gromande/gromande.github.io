---
layout: post
title:  "Migrating from Wordpress to GitHub Pages"
date:   2020-01-26
tags: wordpress github
---
At the beginning of the new year, I decided to migrate my blog from Wordpress to [Github Pages](https://pages.github.com/). Even though the process was pretty simple, thanks to a number of open-source tools, there were a couple of hurdles along the way, so I decided to create this step-by-step guide.

## Setting Up the project
GitHub Pages runs on top of the GitHub platform and is powered by [Jekyll](https://jekyllrb.com/), a templating engine that translates [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) to static HTML.

Start by creating a GitHub repository named `<username>.github.io` where `<username>` is your GitHub username. In my case the repository is [gromande.github.io](https://github.com/gromande/gromande.github.io):

![image]({{ site.baseurl }}/assets/images/github-pages-repo.png)

Clone the repository to your computer:

{% highlight shell %}
git clone https://github.com/<username>/<username>.github.io.git
{% endhighlight %}

Before creating the Jekyll site, you need to install [Ruby](https://www.ruby-lang.org/en/documentation/installation/) along with the following gems:
- Bundler
- Jekyll (at the time of this post the latest version supported by GitHub pages was 3.8.4. Check out the list of supported versions [here](https://pages.github.com/versions/))

Once Ruby is install on you computer, run the following commands to install the gems:

{% highlight shell %}
gem install bundler
gem install jekyll -v 3.8.5
{% endhighlight %}

You should now be able to create your Jekyll site with [Bundler](https://bundler.io/):

{% highlight shell %}
bundle init
bundle add jekyll -v 3.8.5 --skip-install
bundle exec jekyll new --force --skip-bundle .
{% endhighlight %}

Jekyll allows you to run your site locally, which comes in very handy if you need to check how your posts look before making them public:

{% highlight shell %}
bundle install
bundle exec jekyll serve
{% endhighlight %}

Open your browser and go to [http://localhost:4000](http://localhost:4000) to see the default homepage:

![image]({{ site.baseurl }}/assets/images/jekyll-default-site.png)

You can customize the site's title, contact information, and social media links by updating the corresponding properties on the `_config.yml` YAML file:

{% highlight yaml %}
title: Your awesome title
email: your-email@example.com
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this
  line in _config.yml. It will appear in your document head meta (for
  Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: jekyllrb
github_username:  jekyll
{% endhighlight %}

I also added the following property show an excerpt of my posts on the home page:

{% highlight yaml %}
show_excerpts: true
{% endhighlight %}

And changed the default post URLs to only include the title:

{% highlight yaml %}
permalink: /:title/
{% endhighlight %}

Some of this configuration parameters are theme specific. Check out the [default theme](https://github.com/jekyll/minima) for a full list of available settings.

Now is also a good time to put your Markdown skills to test and add your bio to the `about.md` page.

## Exporting Workdpress Posts
Fortunately, some people have gone down this path before and have created some useful tools to make the migration much easier.

For the most part, the steps below are described in this [post](https://www.deadlyfingers.net/code/migrating-from-wordpress-to-github-pages) in more detail.

Log into you Wordpress admin console and go to the following URL to export your Wordpress site:

[https://your-domain/wp-admin/export.php](https://<your-domain>/wp-admin/export.php)

Select "All Content" and download the export file to your computer:

![image]({{ site.baseurl }}/assets/images/wordpress-export.png)

The export file is in XML format and contains references to images and other assets. We can use the `jekyll-import` Ruby gem to convert the exported files into HTML and download all images to the `assets` directory.

Install the following Gems:

{% highlight shell %}
gem install jekyll-import
gem install hpricot
gem install open_uri_redirections
gem install reverse_markdown
{% endhighlight %}

And run this command:

{% highlight shell %}
ruby -rubygems -e 'require "jekyll-import";
JekyllImport::Importers::WordpressDotCom.run({
  "source" => "<path-to-XML-export>",
  "no_fetch_images" => false,
  "assets_folder" => "assets/images"
})'
{% endhighlight %}

Lastly, use the following [Ruby script](https://gist.github.com/deadlyfingers/2023c61cbac83bb613393f262693cdf4) to convert the HTML content to Markdown:

{% highlight shell %}
ruby ./wordpress-html-to-md.rb "_posts"
{% endhighlight shell %}

All you have to do now is copy the entire `assets` folder to the root of your Jekyll project and move all the `.md` files generate by the scrip from the `_posts` folder into the `_posts` folder of your Jekyll's site.

Re-run you site locally and verify that all your posts are being listed on the home page:

![image]({{ site.baseurl }}/assets/images/jekyll-posts.png)

## Final Touches
From here, you can start customizing your site to your own needs. There are plenty of resources online so I am not going to cover customizations on this post. However, here are a couple of things you might want to do to "clean up" your posts.

If you open one of the Markdown files you'll see a bunch of metadata fields at the top of the page:

```
---
layout: post
title: 'Network Security - Part 1: Setting Up Your Environment'
date: 2017-12-16 22:50:19.000000000 -06:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- InfoSec
- Network Security
tags:
- pfsense
- ubuntu
- virtualbox
meta:
  _edit_last: '1'
  _oembed_26e7131f86c1664c79f1dc834e460dbc: "{{unknown}}"
  _wpas_done_all: '1'
author:
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/network-security-part-1-setting-up-your-environment/"
---
```

The following fields are not needed and can be removed from all your posts:
- type
- parent_id
- published
- password
- status
- meta
- author

In addition, some posts might contain absolute URLs referencing other posts. Absolute URLs are discouraged since they make your site less portable. Run the following `sed` command to replace absolute URLs with relative URLs. Replace `<your-domain>` with the domain for your site:

{% highlight shell %}
sed -i.bak 's/https:\/\/<your-domain>/{{ site.baseurl }}/g' *.md
{% endhighlight %}
