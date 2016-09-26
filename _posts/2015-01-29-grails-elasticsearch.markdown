---
layout: post
title:  "Elasticsearch with Grails"
date:   2016-09-26 11:00:00
categories: Grails Elasticsearch
---

Elasticsearch is based on lucene which is a search engine and provides range of features, specially famous for its text based searching. Elasticsearch provides domain class mapping to indexes, and add methods to domain for searching. You also have an option to create, update or delete index as soon as there is a change in the domain.

Elasticsearch is currently being used by github, soundcloud and many more. More detail from <a href="http://www.elasticsearch.org/case-study/">here</a>.

This gives a <a href="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS0wetPnhOhTtRanRjXcwrke4veL6MTwdLZDjsiRH5-TZpj63awow">summary</a> of how cluster, nodes and shards are linked:


For a list of elasticsearch terminologies, see <a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html">this</a>.  

So, lets quickly create a grails project for Events searching:

Note: Make sure you have latest version of grails (for this example I used grails v3.19), for more information checkout gvmtools.net and jdk is installed with 1.7 version (or above)

>$ grails create-app GrailsES --profile=rest-api

>$ cd GrailsES  

Create a domain, controller and service:

>$ grails create-domain-resource com.sufyan.demo.Event

>$ grails create-restful-controller com.sufyan.demo.Event

>$ grails create-service com.sufyan.demo.Event

Add elasticsearch plugin by adding its dependency in build.gradle
{% highlight xml %}
    compile "org.grails.plugins:elasticsearch:1.0.0"
{% endhighlight %}

or if you are using Grails v2.* or older, add dependency in  BuildConfig.groovy
{% highlight xml %}
plugins {
	runtime ":elasticsearch:1.0.0" 
}
{% endhighlight %}

Elasticsearch grails plugin, by default, perform auto indexing of domains which are annotated with `searchable` class property set to true as soon as the domain is save or update.

Update Event domain to have some properties:
{% highlight java %}
class Event {
   String title
   String description

   static searchable = {
       title index:'analyzed', boost:4
       description index:'analyzed', boost:3
   }
}
{% endhighlight %}

Here I have marked title and description properties to be `analyzed`, that is, the values stored in index will be tokenized, so if title has `Technology Expo Pakistan`, tokens will be `technology`, `expo`, `pakistan`. Boost has been given to give some priority to those fields in index while performing search.

Update `application.yml` to define datastore you are using in your system, thats because elasticsearch hooks into GORM’s storage events 
{% highlight ruby %}
elasticSearch:
   datastoreImpl: hibernateDatastore
{% endhighlight %}

For older grails versions, update `Config.groovy`:
{% highlight ruby %}
elasticsearch {
   datastoreImpl = 'hibernateDatastore'
}
{% endhighlight %}

Cool... so far

Lets write an service to index events in EventService:
{% highlight java %}
def add(String title, String description) {
   Event event = new Event(title:title, description:description)
   event.save(flush:true)

   def resp = ['success':true]
   render resp as JSON
}
{% endhighlight %}


Add some events to index via Bootstrap.groovy:
{% highlight java %}
def init = { servletContext ->
    eventService.addEvent("Expo Pakistan", "2 days expo of Pakistan, all stuffs...at expo center")
    eventService.addEvent("New Year Even", "Enjoy new year night at beach club")
    eventService.addEvent("Children's Day", "Kids play, activities, sports, fun, family")
    eventService.addEvent("Education Expo", "3 days Eduction expo, college, schools at expo center...")
    eventService.addEvent("Arena sports area", "All year open, indoor activities, bowling, snooker, ice skating")
    eventService.addEvent("All you can eat", "All you can eat offer is back in this ramadan at Pizza Hut")
}
{% endhighlight %}

As we are using gradle build system, lets compile with gradle to download dependencies:

>$ gradle classes

More gradle task can be found <a href="http://docs.grails.org/latest/guide/commandLine.html#gradleTasks">here</a>.

And now lets run the app to index some events:

>$ gradle bootRun

Lets write some searching code. There are two ways to search, you can use searchable(indexed) Domain class or `ElasticsearchService`. Both provides search methods. There are multiple overloaded search methods, the easiest one to use is by clouser. More detail can be found in <a href="http://noamt.github.io/elasticsearch-grails-plugin/guide/searching.html#queryStrings">plugin documentation</a>.  
You can even use a QueryBuilder to construct query to do the search.

Add an action to search index, update `EventController` with:
{% highlight java %}
    def search(String query) {
        def events = elasticSearchService.search( [:],
                {
                    query_string(fields: ["title", "description"],
                            query: query)
                }, null as Closure)
        render events as JSON
    }
{% endhighlight %}

You have successfully written a search query over your events index, Congrats! This will perform a search on title and description fields that we indexed, and return the result as JSON.

Lets perform search:

>$ http://localhost:8080/event/searchWithDistance?query=ice

Here is the response:
{% highlight JSON %}
{
	"total": 1,
	"searchResults": [
		{
			"id": 5,
			"description": "All year open, indoor activities, bowling, snooker, ice skating",
			"title": "Arena sports area"
		}
	]
}
{% endhighlight %}

To read index, along with many useful data, you can follow this, its a plugin that connects with elasticsearch server: 
https://github.com/mobz/elasticsearch-head


<h3>Helpful links:</h3>
<a href="http://www.elasticsearchtutorial.com/elasticsearch-in-5-minutes.html">5 Minutes with elasticsearch</a>
<a href="http://noamt.github.io/elasticsearch-grails-plugin/">Grails plugin Documentation</a>

You can download the project from <a href="https://github.com/sufyanshoaib/GrailsElasticsearch">github</a>.

<h3>Cons of using ElasticSearch Plugin:</h3>
<ul>
<li>There isnt any option yet to get the words suggestions</li>
<li>If you want to sort by some field and filter by distance and get to know the distance of each result, its not possible via the query, but you can iterate and calcualte distance against each result.</li>
</ul>

I will try to get it hosted somewhere and add more stuffs like Term highlighting etc
