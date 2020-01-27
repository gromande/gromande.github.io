---
layout: post
title:  "Migrating My Blog From Wordpress to GitHub Pages"
date:   2020-01-26
tags: wordpress github
---
At the begining of the new year, I decided to migrate my blog from Workdpress to [Github Pages](https://pages.github.com/). Even though the process was pretty simple, thanks to a number of different tools built by the community, there are a couple of things to keep in mind so I decided to create this step-by-step guide.

## Setting Up the project
GitHub Pages runs on top of the GitHub platform and is powered by [Jekyll](https://jekyllrb.com/), a templating engine that converts text written in [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) to static HTML. The first thing you need to do is create a new Jekyll project and store it in a GitHub repository.

Create a GitHub repository named `<username>.github.io` where `<username>` is your GitHub username. In my case the repository is [gromande.github.io](https://github.com/gromande/gromande.github.io):

![image]({{ site.baseurl }}/assets/images/github-pages-repo.png)

Clone the repository to your computer:

{% highlight shell %}
git clone https://github.com/<username>/<username>.github.io.git
{% endhighlight %}

Before creating the Jekyll site, you need to install [Ruby](https://www.ruby-lang.org/en/documentation/installation/) along with the following gems:
- Bundler
- Jekyll (at the time of this post the latest version supported by GitHub pages was 3.8.4. Check out the list of supported versions [here](https://pages.github.com/versions/))

Once Ruby is install on you computer, run the following commands to installed the required gems:

{% highlight shell %}
gem install bundler
gem install jekyll -v 3.8.5
{% endhighlight %}

You should now be able to use [Bundler](https://bundler.io/) to create your Jekyll site:

{% highlight shell %}
bundle init
bundle add jekyll -v 3.8.5 --skip-install
bundle exec jekyll new --force --skip-bundle .
{% endhighlight %}

You can even run your site locally to verify that everything was setup correctly:

{% highlight shell %}
bundle install
bundle exec jekyll serve
{% endhighlight %}

Open your browser and go to [http://localhost:4000](http://localhost:4000) to check the default homepage:

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

In my case, I added the following property show an excerpt of my posts below the title:

{% highlight yaml %}
show_excerpts: true
{% endhighlight %}

And changed the default post URLs to only include the title:

{% highlight yaml %}
permalink: /:title/
{% endhighlight %}

Now is a good time to put your Markdown skills to test by adding your bio to the `about.md` page.

## Exporting Workdpress Posts
Fortunately, some people have gone down this path before and have created some useful tools that makes the process much easier.

I followed the process descrived in this [post](https://www.deadlyfingers.net/code/migrating-from-wordpress-to-github-pages). To make things a little easier for you, here are all the steps to export, convert, and load your posts.

Log into you Wordpress admin console and go to the following URL to export your post:

[https://your-domain/wp-admin/export.php](https://<your-domain>/wp-admin/export.php)

Select "All Content" and download the export file to your computer:

![image]({{ site.baseurl }}/assets/images/wordpress-export.png)

The export file is in XML format and contains references to your images. We can use the `jekyll-import` Ruby gem to convert the exported files into HTML and download all images the `assets` directory.

Install the following Gems:

{% highlight shell %}
gem install jekyll-import
gem install hpricot
gem install open_uri_redirections
gem install reverse_markdown
{% endhighlight %}

And run this command to convert the files:

{% highlight shell %}
ruby -rubygems -e 'require "jekyll-import";
JekyllImport::Importers::WordpressDotCom.run({
  "source" => "<path-to-XML-export>",
  "no_fetch_images" => false,
  "assets_folder" => "assets/images"
})'
{% endhighlight %}

Finally, use the following [Ruby script](https://gist.github.com/deadlyfingers/2023c61cbac83bb613393f262693cdf4) to convert the HTML files inside the `_posts` folder to Markdown:

{% highlight shell %}
ruby ./wordpress-html-to-md.rb "_posts"
{% endhighlight shell %}

All you have to do now is copy the entire `assets` folder to the root of your Jekyll folder and all the `.md` files generate by the scrip from the `_posts` folder into the `_posts` folder of your Jekyll's site.

Re-run you site locally and check you homepage again:

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

In addition, some posts might contain absolute URLs referencing other posts. Absolute URLs are discouraged since they make your site less portable. Run the following sed command to replace absolute URLs with relative URLs where `<your-domain>` is the domain for your site:

{% highlight shell %}
sed -i.bak 's/https:\/\/<your-domain>/{{ site.baseurl }}/g' *.md
{% endhighlight %}
