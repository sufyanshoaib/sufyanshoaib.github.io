---
layout: post
title:  "Elasticsearch with Grails"
date:   2015-01-29 18:30:00
categories: Grails Elasticsearch
---

Elasticsearch is based on lucene which is a search engine and provides range of features, specially famous for its text based searching. Elasticsearch provides domain class mapping to indexes, and add methods to domain for searching. You also have an option to create, update or delete index as soon as there is a change in the domain.

Elasticsearch is currently being used by github, soundcloud and many more. More detail from <a href="http://www.elasticsearch.org/case-study/">here</a>.

This gives a <a href="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS0wetPnhOhTtRanRjXcwrke4veL6MTwdLZDjsiRH5-TZpj63awow">summary</a> of how cluster, nodes and shards are linked:


For a list of elasticsearch terminologies, see <a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html">this</a>.  

So, lets quickly create a grails project for Events searching:

Note: Make sure you have latest version of grails, for more information checkout gvmtools.net and jdk is installed with 1.7 version (or above)

>``$> grails CreateApp GrailsES``  
>``$> cd GrailsES``  


Create a domain, controller and service:
`$> grails CreateDomainClass com.sufyan.demo.Event`
`$> grails CreateController com.sufyan.demo.Event`
`$> grails CreateService com.sufyan.demo.Event`

Modify BuildConfig.groovy to add elasticsearch plugin dependency
{% highlight xml %}
plugins {
	runtime ":elasticsearch:0.0.3.8" 
}
{% endhighlight %}

And incase you want to integrate with intellij IDEA IDE,
`$>grails integrate-with --intellij`

Perform compile so that grails download any dependencies:
`$>grails compile`

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

Here I have marked title and description properties to be `analyzed` that is the values stored in index will be tokenized, so if title has `Technology Expo Pakistan`, tokens will be technology, expo, pakistan. Boost has been given to give some priority to those fields in index while performing search.

Update `Config.groovy` to define the datastore you are using in your project, this is because, elasticsearch hooks into GORM’s storage events

{% highlight ruby %}
elasticsearch {
   datastoreImpl = 'hibernateDatastore'
}
{% endhighlight %}

Cool... so far

Lets write a method to add events:
{% highlight java %}
def add(String title, String description) {
   Event event = new Event(title:title, description:description)
   event.save(flush:true)

   def resp = ['success':true]
   render resp as JSON
}
{% endhighlight %}

Either add events from bootstrap.groovy or call the action to add event(s):
<a href="http://localhost:8080/GrailsES/event/add?title=New%20Year%20Eve&description=Enjoy%20new%20year%20eve%20with%20friends%20at%20Beach%20Club">Event 1</a>
<a href="http://localhost:8090/GrailsES/event/add?title=Valentine%27s%20Day&description=Enjoy%20Valetines%20day%20with%20your%20wife">Event 2</a>

or from Bootstrap:
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

Lets write some searching. There are two ways to search, you can use searchable(indexed) Domain class or ElasticsearchService. Both provides search methods. There are multiple overloaded search methods, the easiest one to use is by clouser. More detail can be seen here from <a href="http://noamt.github.io/elasticsearch-grails-plugin/guide/searching.html#queryStrings">plugin documentation</a>.  
You can even use a QueryBuilder to construct query do the search.

{% highlight java %}
{% endhighlight %}

To read index, along with many useful data, you can follow this, its a plugin that connects with elasticsearch server: 
https://github.com/mobz/elasticsearch-head
But if that sound bit too much todo, browse at: 
`http://localhost:9200/<package>/<domain-name>/<index-id>`
Details from <a href="https://github.com/mobz/elasticsearch-head">here</a>

<h3>Helpful links:</h3>
<a href="http://www.elasticsearchtutorial.com/elasticsearch-in-5-minutes.html">5 Minutes with elasticsearch</a>
<a href="http://noamt.github.io/elasticsearch-grails-plugin/">Grails plugin Documentation</a>

<h3>Cons:</h3>
You can get a working copy of this from <a href="https://github.com/sufyanshoaib/GrailsElasticsearch">github</a>.

<h3>Cons:</h3>
<ul>
<li>There isnt any option yet to get the words suggestions</li>
<li>If you want to sort by some field and filter by distance and get to know the distance of each result, its not possible via the query, but you can iterate and calcualte distance against each result.</li>
</ul>

I will try to get it hosted somewhere and add more stuffs like Term highlighting etc
