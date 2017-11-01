---
layout: post
title:  "Smallest Rails Docker Image"
date:   2016-07-29 18:00:00
categories: Docker
redirect_from:
  - /index.php/2016/07/29/smallest-rails-docker-image/
tags: [docker,docker image,rails docker,small rails image,small docker image,rails alpine,docker alpine,ruby alpine,ruby,rails]
---

This is quick post about how to get smallest image for Rails 5 application.

As a base I've used <code class="EnlighterJSRAW" data-enlighter-language="null">alpine</code> linux with ruby:<a href="https://hub.docker.com/_/ruby/"> https://hub.docker.com/_/ruby/</a>

And this is Dockerfile:
<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM ruby:2.3-alpine

# Update information about packages
RUN apk update

# Install bash in case you want to provide some work inside containe
# For example:
#     docker run -i -t name_of_your_image bash
RUN apk add bash

# g++ and make for gems building
RUN apk add g++ make

# Timezone data - required by Rails
RUN apk add tzdata

# Sqlite developer library
RUN apk add sqlite-dev

# Javscript runtime
RUN apk add nodejs

# Install bundler
RUN gem install bundler

# Install all gems and cache them
ADD ./Gemfile /tmp
ADD ./Gemfile.lock /tmp
WORKDIR /tmp
RUN bundle install

# Copy application to image and set work dir
ADD . /usr/src/app
WORKDIR /usr/src/app

EXPOSE 3000

CMD bundle exec rails s -b 0.0.0.0</pre>
After:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker build -t rails_alpine .</pre>
I've got:
<pre class="EnlighterJSRAW" data-enlighter-language="null">REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
rails_alpine            latest              324b967d3f40        About an hour ago   403.4 MB</pre>
For compare purpose this is image build for the same project but using ruby default image:
<pre class="EnlighterJSRAW" data-enlighter-language="null">REPOSITORY              TAG                 IMAGE ID            CREATED              SIZE
rails_common            latest              8eab0254b555        About a minute ago   862.1 MB</pre>
If you are short on disk space on your server then it can be a good idea to use ruby alpine base image.

See you next time!

&nbsp;
