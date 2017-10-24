---
layout: post
title:  "Rails, Grafana And InfluxDB"
date:   2015-12-21 18:00:00
---

What is Grafana. It is "The leading graph and dashboard builder for visualizing time series metrics." and it is open source. <a href="http://grafana.org/" target="_blank">http://grafana.org/</a>

For sake of this article I will make my examples using Ruby on Rails framework (<a href="http://rubyonrails.org/" target="_blank">http://rubyonrails.org/</a>). What I want to show you is how we can get timings from each request, push them to Grafana and visualize them.

Grafana needs data sources from where data will be consumed <a href="http://docs.grafana.org/datasources/overview/" target="_blank">http://docs.grafana.org/datasources/overview/</a>. In this particular example I will use InfluxDB (<a href="https://influxdb.com/" target="_blank">https://influxdb.com/</a>).

First of all we have to make sure that InfluxDB is installed on our computer. I encourage to do it via docker <a href="https://hub.docker.com/r/tutum/influxdb/" target="_blank">https://hub.docker.com/r/tutum/influxdb/</a>

Than we have to install Grafana also I'm using docker for this <a href="https://hub.docker.com/r/grafana/grafana/" target="_blank">https://hub.docker.com/r/grafana/grafana/</a>

Of course you can install InfluxDB and Grafana without docker.

Next think is to setup InfluxDB in Rails app.

First add influxdb gem (Gemfile):
<pre class="EnlighterJSRAW" data-enlighter-language="ruby">gem 'influxdb'</pre>
Next we need little class which will make a connection to InfluxDB and save data to it (lib/metrics.rb):
<pre class="EnlighterJSRAW" data-enlighter-language="ruby">class Metrics
  def write_data_points(options)
    client.write_points(data_points(options))
  rescue =&gt; e
    Rails.logger.error "Error while using Metrics: #{e}"
    Rails.logger.error e.backtrace
  end

  private

  def client
    @client ||= InfluxDB::Client.new(
      host:     ENV['INFLUXDB_HOST'],
      port:     ENV['INFLUXDB_PORT'],
      database: ENV['INFLUXDB_DATABASE'],
      user:     ENV['INFLUXDB_USER'],
      password: ENV['INFLUXDB_PASSWORD']
    )
  end

  def data_points(options)
    [
      {
        series: 'endpoint_stats',
        values: {
          endpoint: options[:endpoint],
          duration: options[:duration],
          view_runtime: options[:view_runtime] || 0
        }
      }
    ]
  end
end
</pre>
After that we have to figured out how to get request response times, it took me a while to find how to do it. But on the end I've found that I can use Active Support Instrumentation <a href="http://edgeguides.rubyonrails.org/active_support_instrumentation.html" target="_blank">http://edgeguides.rubyonrails.org/active_support_instrumentation.html</a>

Thanks to this I can get all needed data about request. What we need to do is just put this code in to initializers (config/initializers/metrics.rb):
<pre class="EnlighterJSRAW" data-enlighter-language="ruby">ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event  = ActiveSupport::Notifications::Event.new(*args)
  endpoint = "#{event.payload[:method]}_#{event.payload[:controller]}##{event.payload[:action]}"

  Metrics.new.write_data_points(
    endpoint: endpoint,
    duration: event.duration,
    view_runtime: event.payload[:view_runtime]
  )
end
</pre>
And that's it. Next step is to go to Grafana and add some metrics. There is a lot of tutorials how to do it: <a href="http://docs.grafana.org/datasources/influxdb/" target="_blank">http://docs.grafana.org/datasources/influxdb/</a>

You can find simple rails application with whole setup here:

<a href="https://github.com/arturmalecki/grafana_influxdb_rails" target="_blank">https://github.com/arturmalecki/grafana_influxdb_rails</a>

This solution is very easy to setup and it is open source!

Sources:

<a href="http://davidanguita.name/articles/simple-data-visualization-stack-with-docker-influxdb-and-grafana/" target="_blank">http://davidanguita.name/articles/simple-data-visualization-stack-with-docker-influxdb-and-grafana/</a>
