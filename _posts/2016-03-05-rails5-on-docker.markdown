---
layout: post
title:  "Rails 5 on Docker"
date:   2016-03-05 18:00:00
categories: Docker
---

In this short post I will describe how easily you can setup docker on Rails application.

Full example can be found here: <a href="https://github.com/arturmalecki/rails_5_docker" target="_blank">https://github.com/arturmalecki/rails_5_docker</a>

First thing that yo have to know is that docker use environment variables to define the environment on which container will be run. This means that good idea is to use environment variables in Rails application as well. Of course it is possible to use other configuration options but this require additional work and I will try to cover this in next posts.

First let's create Dockerfile and set image for <code class="EnlighterJSRAW" data-enlighter-language="null">ruby 2.3.0</code>  :
<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM ruby:2.3.0</pre>
This will tell docker to use image that have <code class="EnlighterJSRAW" data-enlighter-language="null">ruby 2.3.0</code>  inside. You can find a lot of useful images here <a href="https://hub.docker.com/" target="_blank">https://hub.docker.com/</a>.

For some libs like <code class="EnlighterJSRAW" data-enlighter-language="null">uglifier</code> you need to have <code class="EnlighterJSRAW" data-enlighter-language="null">nodejs</code> and to install it in this image add this:
<pre class="EnlighterJSRAW" data-enlighter-language="null">RUN apt-get update &amp;&amp; apt-get -y install nodejs</pre>
Next you have to tell docker to install bundler:
<pre class="EnlighterJSRAW" data-enlighter-language="null">RUN gem install bundler</pre>
Now is time to copy application in to the image and set working dir to application dir:
<pre class="EnlighterJSRAW" data-enlighter-language="null">ADD . /var/www/app
WORKDIR /var/www/app</pre>
After we have application inside the image, let's install all gems:
<pre class="EnlighterJSRAW" data-enlighter-language="null">RUN bundle install</pre>
Also you have to tell docker which port you want to expose, of course you want to expose application server port which by default is 3000:
<pre class="EnlighterJSRAW" data-enlighter-language="null">EXPOSE 3000</pre>
And what command will should be run to start application server:
<pre class="EnlighterJSRAW" data-enlighter-language="null">CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]</pre>
By default puma server will be binded to <code class="EnlighterJSRAW" data-enlighter-language="null">localhost</code> which will cause issues on port mapping. Basically docker is doing his port mapping on <code class="EnlighterJSRAW" data-enlighter-language="null">0.0.0.0</code> inside container, which is why we have to bind application server to this ip.

This is how entire Dockerfile should looks like:
<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM ruby:2.3.0

RUN apt-get update &amp;&amp; apt-get -y install nodejs

RUN gem install bundler

ADD . /var/www/app
WORKDIR /var/www/app

RUN bundle install

EXPOSE 3000

CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]
</pre>
Now build image:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker build -t rails_5_app .</pre>
And run container:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker run -p 13000:3000 rails_5_app</pre>
Option <code class="EnlighterJSRAW" data-enlighter-language="null">-p</code> is used to map port from your machine with port which we exposed inside container, in this case 3000.

Now you should be able to hit <code class="EnlighterJSRAW" data-enlighter-language="null">http://localhost:13000</code> and see welcome page from Rails.

This short instruction will allow you to run your first docker container with rails app inside. Of course there is a lot of room for improvements. I will try to describe them in my next posts. Hope it will be helpful for you!

Thanks!
