---
layout: post
title:  "Swagger integration with Rails"
date:   2016-10-08 18:00:00
categories: Docker
---

Documentation is very important part of developing an application. Especially if you have to work with more then one. Nowadays when microservices approach is so common is even more important. This is why I would like to write quick blog post how to use swagger-ui with rails. I will cover two cases. When you want to use swagger ui as part of application/service or as separate service.
<h2>TL;DR</h2>
You can find full example here: <a href="https://github.com/arturmalecki/rails_swagger">https://github.com/arturmalecki/rails_swagger</a>
<h2>swagger-ui</h2>
First of all, what is swagger-ui:
<p style="padding-left: 30px;">Swagger UI is a dependency-free collection of HTML, Javascript, and CSS assets that dynamically generate beautiful documentation and sandbox from a Swagger-compliant API. Because Swagger UI has no dependencies, you can host it in any server environment, or on your local machine.</p>
More information you can find on site: <a href="http://swagger.io/swagger-ui/">http://swagger.io/swagger-ui/</a>
<h2>Integration swagger-ui with Rails</h2>
Basically there are two way to do it. First is just to add swagger-ui to your project and use it from there. Other option is to serve swagger-ui as separate service and pass configuration file there.

Lets start from first solution.

In your rails project run this command:
<pre class="EnlighterJSRAW" data-enlighter-language="null">git clone git@github.com:swagger-api/swagger-ui.git public/swagger-ui</pre>
This will add swagger-ui as git submodule.

Then you have to tell rails to redirect all calls for <code class="EnlighterJSRAW" data-enlighter-language="null">/swagger-ui</code> endpoints to swagger-ui application.
<pre class="EnlighterJSRAW" data-enlighter-language="null">get '/swagger-ui', to: redirect('swagger-ui/dist/index.html?url=%2Fswagger.yaml')</pre>
<code class="EnlighterJSRAW" data-enlighter-language="null">url </code> is a location of <code class="EnlighterJSRAW" data-enlighter-language="null">swagger.yml</code> where you should have specification of your endpoints: <a href="http://swagger.io/specification/">http://swagger.io/specification/</a>

And this is basically it. If your <code class="EnlighterJSRAW" data-enlighter-language="null">swagger.yml</code> is correct then you should have your swagger-ui on this location: <code class="EnlighterJSRAW" data-enlighter-language="null">/swagger-ui</code>

This solution is really good, you don't have to deal with CORS issues (more about them later). But also each of your applications and services will need to have it. Which is not so bad if you have not to many of them.

Second option is to put swagger-ui as separate service. In this example we will use docker. This solution is allows you to have only one instance of swagger-ui. Which is great. We want to stay DRY. But there is small disadvantages. Swagger-ui is doing all calls to endpoints via AJAX. Which means that you will have CORS problem. You can find more informations how to solve it here: <a href="https://github.com/swagger-api/swagger-ui/blob/master/README.md#cors-support">https://github.com/swagger-api/swagger-ui/blob/master/README.md#cors-support</a>

In my case I'm using rails application with this gem <a href="https://github.com/cyu/rack-cors">https://github.com/cyu/rack-cors</a> which allows me to do cross domain AJAX requests.

For this example I will use docker.

This is example of simple <code class="EnlighterJSRAW" data-enlighter-language="null">Dockerfile</code> for Rails applicaiton:
{% highlight docker %}
FROM ruby:2.3.0

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
{% endhighlight %}
On <a href="https://hub.docker.com">https://hub.docker.com</a> there is image for swagger-ui <a href="https://hub.docker.com/r/schickling/swagger-ui/">https://hub.docker.com/r/schickling/swagger-ui/</a>

Now we only need to put everything together in <code class="EnlighterJSRAW" data-enlighter-language="null">docker-compose.yml</code>
{% highlight docker %}
version: '2'
services:
  web:
    build: .
    ports:
     - "4000:3000"
    volumes:
     - .:/var/www/app
    depends_on:
     - swagger_ui
  swagger_ui:
    image: schickling/swagger-ui
    ports:
     - "4001:80"
    environment:
     - API_URL=http://localhost:4000/swagger.yaml
{% endhighlight %}
Build your containers:
<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose up</pre>
and open http://localhost:4001

As you can see, integration with swagger-ui is very simple. Hope you find this post very well :)
