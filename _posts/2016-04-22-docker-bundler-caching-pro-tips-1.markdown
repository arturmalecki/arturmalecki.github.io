---
layout: post
title:  "Docker bundler caching - Pro Tips #1"
date:   2016-04-22 18:00:00
categories: Docker
---

This time I would like to share with some small tip how to improve docker file for Rails application.

If you are using <code class="EnlighterJSRAW" data-enlighter-language="null">Dockerfile</code> similar to this one <a href="https://github.com/arturmalecki/rails_5_docker/blob/v1.0/Dockerfile">Dockerfile</a> for sure you noticed that every time you change code, but not a <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile</code>, and then you build an image, docker has to install all gems. It can be quite annoying and eat a lot of your time. This is happening because command which is copping whole app to image:
<pre class="EnlighterJSRAW" data-enlighter-language="null">ADD . /var/www/app</pre>
is before:
<pre class="EnlighterJSRAW" data-enlighter-language="null">RUN bundle install</pre>
Docker is caching all the steps on building process, but when you change something which will affect certain step, docker will rebuild all steps from one which was change to the last one. This is why gems are install everything you change something in application.

There is very simple fix for this issue. Before we copy entire app to <code class="EnlighterJSRAW" data-enlighter-language="null">/var/www/app</code> we have to copy <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile</code> and <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile.lock</code> for example to <code class="EnlighterJSRAW" data-enlighter-language="null">/tmp</code> folder and run <code class="EnlighterJSRAW" data-enlighter-language="null">bundle install</code> before we copy whole app. In this case docker on first run will install all gems and cash this operation. Then if you change something in your code which is not related to <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile</code> or <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile.lock</code> it will use cache for <code class="EnlighterJSRAW" data-enlighter-language="null">bundle install</code> command.

This is how <code class="EnlighterJSRAW" data-enlighter-language="null">Dockerfile</code> should looks like:
<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM ruby:2.3.0

RUN apt-get update &amp;&amp; apt-get -y install nodejs

RUN gem install bundler

ADD Gemfile /tmp
ADD Gemfile.lock /tmp
WORKDIR /tmp
RUN bundle install

ADD . /var/www/app
WORKDIR /var/www/app

EXPOSE 3000

CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]
</pre>
After first run of <code class="EnlighterJSRAW" data-enlighter-language="null">docker build .</code> you should get output similar to this on:
<pre class="EnlighterJSRAW" data-enlighter-language="null">▶ docker build .
Sending build context to Docker daemon 194.6 kB
Step 1 : FROM ruby:2.3.0
 ---&gt; 7ca70eb2dfea
Step 2 : RUN apt-get update &amp;&amp; apt-get -y install nodejs
 ---&gt; Running in 7fe2dc396d88
 ...
 ---&gt; 7aee2168c9dc
Removing intermediate container 7fe2dc396d88
Step 3 : RUN gem install bundler
 ---&gt; Running in 5a33a0cd7f95
Successfully installed bundler-1.11.2
1 gem installed
 ---&gt; b386e495b87a
Removing intermediate container 5a33a0cd7f95
Step 4 : ADD Gemfile /tmp
 ---&gt; 0ddea04cda43
Removing intermediate container 5f9740abbbf6
Step 5 : ADD Gemfile.lock /tmp
 ---&gt; 9cf9426fb288
Removing intermediate container 7c768b0c2894
Step 6 : WORKDIR /tmp
 ---&gt; Running in 6a9ebeb9f58b
 ---&gt; 23307ed8ff5b
Removing intermediate container 6a9ebeb9f58b
Step 7 : RUN bundle install
 ---&gt; Running in 7855e0693da4
 ...
 ---&gt; a1a8588d493a
Removing intermediate container 7855e0693da4
Step 8 : ADD . /var/www/app
 ---&gt; e93a819c6b73
Removing intermediate container 42a6140c1f5d
Step 9 : WORKDIR /var/www/app
 ---&gt; Running in 055b21029fa8
 ---&gt; 246d4b7a2b33
Removing intermediate container 055b21029fa8
Step 10 : EXPOSE 3000
 ---&gt; Running in 54fb4d98df1c
 ---&gt; b04c74446436
Removing intermediate container 54fb4d98df1c
Step 11 : CMD bundle exec rails s -b 0.0.0.0
 ---&gt; Running in 550e1094211e
 ---&gt; 561b5e75c239
Removing intermediate container 550e1094211e
Successfully built 561b5e75c239</pre>
Step 4,5,6,7 are one which you should notice.

Next if you change something in code, but not in  <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile</code> or <code class="EnlighterJSRAW" data-enlighter-language="null">Gemfile.lock</code>, and build an image, you should get something like that:
<pre class="EnlighterJSRAW" data-enlighter-language="null">▶ docker build .
Sending build context to Docker daemon 194.6 kB
Step 1 : FROM ruby:2.3.0
 ---&gt; 7ca70eb2dfea
Step 2 : RUN apt-get update &amp;&amp; apt-get -y install nodejs
 ---&gt; Using cache
 ---&gt; 7aee2168c9dc
Step 3 : RUN gem install bundler
 ---&gt; Using cache
 ---&gt; b386e495b87a
Step 4 : ADD Gemfile /tmp
 ---&gt; Using cache
 ---&gt; 0ddea04cda43
Step 5 : ADD Gemfile.lock /tmp
 ---&gt; Using cache
 ---&gt; 9cf9426fb288
Step 6 : WORKDIR /tmp
 ---&gt; Using cache
 ---&gt; 23307ed8ff5b
Step 7 : RUN bundle install
 ---&gt; Using cache
 ---&gt; a1a8588d493a
Step 8 : ADD . /var/www/app
 ---&gt; 22594da73162
Removing intermediate container 5db5ca639f50
Step 9 : WORKDIR /var/www/app
 ---&gt; Running in ca9ab7f434a2
 ---&gt; d762cabc0c49
Removing intermediate container ca9ab7f434a2
Step 10 : EXPOSE 3000
 ---&gt; Running in 4ea24b7af505
 ---&gt; 092d4aa8ffbc
Removing intermediate container 4ea24b7af505
Step 11 : CMD bundle exec rails s -b 0.0.0.0
 ---&gt; Running in 08a9270508c8
 ---&gt; 5e3c48d014c7
Removing intermediate container 08a9270508c8
Successfully built 5e3c48d014c7</pre>
Now you can see that in step 7 docker is using cache for bundler. Also you should notice that after code change everything from step 8 to the end is build once again.

You can find working example <a href="https://github.com/arturmalecki/rails_5_docker/tree/v1.1">here</a>.

As you can see this is very simple trick and  can save you a lot of time. This is all for this quick article. With this one I would like to open a new set of posts containing simple tips and tricks for docker.

See you!
