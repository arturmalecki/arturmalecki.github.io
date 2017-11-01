---
layout: post
title:  "Rails 5 and Mysql meet Docker Compose"
date:   2016-07-26 18:00:00
categories: Docker
redirect_from:
  - /index.php/2016/07/26/rails-5-and-mysql-meet-docker-compose/
tags: [docker,docker compose,docker rails,docker compose rails,rails 5,mysql,rails container]
---

Docker compose is awesome and you have to give it a try!

<a href="https://github.com/arturmalecki/rails_5_mysql_docker_compose">https://github.com/arturmalecki/rails_5_mysql_docker_compose</a>
<h2><strong>What is docker compose?</strong></h2>
From <a href="https://docs.docker.com/compose/">https://docs.docker.com/compose/</a> "<em>Compose is a tool for defining and running multi-container Docker applications." </em>Basically this is very nice wrapper around docker cli with some extra features like running multi-container applications.
<h2><strong>Docker file for rails 5</strong></h2>
Let say that we have this docker file for our rails 5 application:
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

CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]</pre>
I've wrote more about docker file for rails in those posts: <a href="http://www.devcherry.com/index.php/2016/03/05/rails-5-on-docker/">Rails 5 On Docker</a> and <a href="http://www.devcherry.com/index.php/2016/04/22/avoid-installing-gems-on-each-docker-image-build-pro-tips-1/">Pro Tipis for Docker</a>
<h2><strong>Configure database</strong></h2>
In this example we put database in different Docker container. This means that we have pass database credentials to <code class="EnlighterJSRAW" data-enlighter-language="null">database.yml</code> Docker encourage to use environment variables for keeping and passing configuration. And of course we want to keep standards:
<pre class="EnlighterJSRAW" data-enlighter-language="null">development:
  url: &lt;%= ENV['DATABASE_URL'] %&gt;</pre>
In most Rails project database configuration is moved to <code class="EnlighterJSRAW" data-enlighter-language="null">database.yml.example</code> or something simillar. On this point workflow will change a little bit. Right now database configuration will be kept in environment variable. And we can keep our <code class="EnlighterJSRAW" data-enlighter-language="null">database.yml</code> in repository.
<h2><strong>Putting everything together</strong></h2>
This is the nice part:
<pre class="EnlighterJSRAW" data-enlighter-language="null">app:
  build: .
  environment:
    RAILS_ENV: development
    DATABASE_URL: mysql2://root:root@mysql/rails
  ports:
  - "13000:3000"
  volumes:
  - ".:/var/www/app"
  links:
  - "mysql:mysql"
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: rails</pre>
All options are explained here: <a href="https://docs.docker.com/compose/compose-file/">https://docs.docker.com/compose/compose-file/</a>

Here I will just explain main things. First of all there are two sections: <code class="EnlighterJSRAW" data-enlighter-language="null">app</code> and <code class="EnlighterJSRAW" data-enlighter-language="null">mysql</code> - those are containers. There can be build using local dockerfile: <code class="EnlighterJSRAW" data-enlighter-language="null">build: .</code> or image: <code class="EnlighterJSRAW" data-enlighter-language="null">image: mysql</code> All other options are very simillar to docker cli commands. There are things like port mapping, volume mounting, linking containers or passing environment variables.

Now this command:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose up</pre>
will build images and run containers.

Also you use <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose stop</code> to stop containers, <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose rm</code> to remove images, <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose build</code> to build images only. Full list of commands for compose can be found here: <a href="https://docs.docker.com/compose/reference/">https://docs.docker.com/compose/reference/</a>
<h2><strong>Workflow - Development</strong></h2>
Yes, you can develop your application using only docker compose. This brings a lot of benefits. You don't have to worry about dependencies, you can use linux, osx or windows without any problems, less tech members of your team don't have to struggle to run application. How it can be done? It's very simple. Notice this one in <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose.yml</code> file:
<pre class="EnlighterJSRAW" data-enlighter-language="null">volumes:
- ".:/var/www/app"</pre>
This entry tells docker to use your current location (your project) and mount inside container. And... that is it! Easy as that! Now you can work on your local directory and docker container will use your dir to run applicaiton. There is one little disadvantages, each time you change you Gemfile you have to rebuild image. But come on, how often you are doing it?
<h2><strong>Workflow - Tests</strong></h2>
Of course you can use docker compose to run your test as well. But to do it we have to change docker compose file a little bit. I really recommend to create a new one: <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose.test.yml</code>
<pre class="EnlighterJSRAW" data-enlighter-language="null">app:
  build: .
  environment:
    RAILS_ENV: test
    DATABASE_URL: mysql2://root:root@mysql/railstest
  links:
  - "mysql:mysql"
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: railstest</pre>
Please notice that there is no <code class="EnlighterJSRAW" data-enlighter-language="null">volumes</code> or <code class="EnlighterJSRAW" data-enlighter-language="null">ports</code> options. This is because to run tests we don't need them. Also environment variable <code class="EnlighterJSRAW" data-enlighter-language="null">RAILS_ENV</code> is set to <code class="EnlighterJSRAW" data-enlighter-language="null">true</code> .

To run test we have to use <code class="EnlighterJSRAW" data-enlighter-language="null">run</code> command:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose -f docker-compose.test.yml run app rake test</pre>
First we have to pass test docker compose file <code class="EnlighterJSRAW" data-enlighter-language="null">-f docker-compose.test.yml</code>, specify on what container we want to run command and run rake command to trigger tests. This is very useful when you can use Docker on your CI tools. No more problem with dependencies!
<h2><strong>How to run commands in container</strong></h2>
As you already noticed, running commands like <code class="EnlighterJSRAW" data-enlighter-language="null">rake db:migrate</code> can be done via <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose run</code>

First choose on which container you want to run your command and do this
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose run app rake db:migrate</pre>
This is it!
<h2><strong>Workflow - production</strong></h2>
Of course you can use docker compose in production. You can find more information here <a href="https://docs.docker.com/compose/production/">https://docs.docker.com/compose/production/</a>

There is couple of things that you have to think of, like where to store production credentials and how to pass them to docker-compose or take care for mysql container storage persist. I will try to cover this in more details in next blog posts.
<h2><strong>Sum up</strong></h2>
As you can see, docker with compose is a great way of dealing with of levels of application development to production usage. Hope you find this post inspiring!
