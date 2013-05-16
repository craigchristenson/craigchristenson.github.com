---
layout: post
title: "Decoding JSON to a Python Object"
category: posts
---

I recently had the opportunity to work on a Python project that involved pulling data from a JSON API. Python's JSON module makes serializing and deserializing a breeze but for this project I also wanted to be able to call some custom methods on the dictionary returned after deserializing. The solution turned out to be pretty strait forward so I thought I would share what I came up with in case it is useful to anyone with a similar issue.

Create a new parent class which can get and set the attributes from the deserialized JSON dictionary.

{% highlight python %}
class MyObject(dict):
    def __init__(self, dict_):
        super(MyObject, self).__init__(dict_)
        for key in self:
            item = self[key]
            if isinstance(item, list):
                for id, it in enumerate(item):
                    if isinstance(it, dict):
                        item[id] = self.__class__(it)
            elif isinstance(item, dict):
                self[key] = self.__class__(item)

    def __getattr__(self, key):
        return self[key]
{% endhighlight %}

Then just create a subclass where you will call your custom methods for that object.

{% highlight python %}
class Tweets(MyObject):
    def __init__(self, dict_):
        super(self.__class__, self).__init__(dict_)

{% endhighlight %}

Now you can just initialize a new instance of the subclass with the deserialized JSON dictionary and all of your custom methods will be available.

### Example using Twitter's JSON API
{% highlight python %}
import urllib
import urllib2
import json


class MyObject(dict):
    def __init__(self, dict_):
        super(MyObject, self).__init__(dict_)
        for key in self:
            item = self[key]
            if isinstance(item, list):
                for id, it in enumerate(item):
                    if isinstance(it, dict):
                        item[id] = self.__class__(it)
            elif isinstance(item, dict):
                self[key] = self.__class__(item)

    def __getattr__(self, key):
        return self[key]


class Tweets(MyObject):
    def __init__(self, dict_):
        super(self.__class__, self).__init__(dict_)

    @classmethod
    def search(cls):
        req = urllib2.Request('http://search.twitter.com/search.json?q=%40twitterapi')
        result = urllib2.urlopen(req).read()
        return Tweets(json.loads(result))

    def get_tweets(self):
        tweet_data = dict()
        for tweet in self.results:
            tweet_data[tweet.created_at] = tweet.text
        return tweet_data


# get all tweet data
query = Tweets.search()

# create a new dictionary using the tweet time and text
tweets = query.get_tweets()

# pretty print 
print(json.dumps(tweets, indent=4))
{% endhighlight %}

### Result
{% highlight json %}
{
    "Thu, 16 May 2013 21:14:02 +0000": "Download GrassHeads - Once In A LifeTime ft. Gill.mp3 | Limelinx http://t.co/gQ91Lib6EN via @twitterapi", 
    "Thu, 16 May 2013 21:17:14 +0000": "RT @SaskiaWaterland: Een bloemlezing van VPRO's seksuele hoogtepunten: de vpro.nl seksavond - VPRO http://t.co/aavuKweaAn via @twitterapi", 
    "Thu, 16 May 2013 21:14:44 +0000": "Download Tom Arts - Dont Blame Me.mp3 | Limelinx http://t.co/CeQIl9PW0W via @twitterapi", 
    "Thu, 16 May 2013 21:15:33 +0000": "Whatcom Locavore: Women becoming a major force in sustainable agriculture http://t.co/FGB8psGTGJ via @twitterapi", 
    "Thu, 16 May 2013 21:09:02 +0000": "YANGON, Myanmar: Myanmar says Shan rebels attack oil company office | World | http://t.co/C9KZPeHEQF http://t.co/y4HFNRaMSt via @twitterapi", 
    "Thu, 16 May 2013 21:16:52 +0000": "RALEIGH: Raleigh police get homicide victim\u2019s deleted Facebook page in seeking leads | Crime | http://t.co/pIJZsxrAPs via @twitterapi", 
    "Thu, 16 May 2013 21:15:23 +0000": "RT @TheOnlyPilot: Download Nick - Chopped and Screwed.mp3 | Limelinx http://t.co/XTK87Hy7cL via @twitterapi", 
    "Thu, 16 May 2013 21:13:10 +0000": "Kad ortak odvali nesto, a smeh jaci od tebe | Smesne reakcije @VladoGeorgiev http://t.co/h7bQHrKhmz via @twitterapi", 
    "Thu, 16 May 2013 21:09:04 +0000": ":)) Adam Lambert's fans land him an O Awards nomination: Pressparty http://t.co/dEjjs28gWa via @twitterapi", 
    "Thu, 16 May 2013 21:18:30 +0000": "\u00bb Rene Rancourt Visits Barstool HQ\u2019s To Get Us Ready for B\u2019s Vs. Rangers Barstool Sports: Boston via @twitterapi", 
    "Thu, 16 May 2013 21:09:28 +0000": "Download Nick - Chopped and Screwed.mp3 | Limelinx http://t.co/XTK87Hy7cL via @twitterapi", 
    "Thu, 16 May 2013 21:16:27 +0000": "Yaa check m\u011b Out Go Download Transformer- Akrshome Foe.mp3 | Limelinx http://t.co/zvvWy20HuG via @twitterapi", 
    "Thu, 16 May 2013 21:13:57 +0000": "(1) Valores Habbid - F\u00f3rum \u00b7 Habbid http://t.co/JL02isdniq via @twitterapi", 
    "Thu, 16 May 2013 21:14:17 +0000": "Washington, D.C.: B Reactor park bill passes Senate committee | Hanford news | http://t.co/9Kcvgf3PrY http://t.co/1SDhZPLT8A via @twitterapi", 
    "Thu, 16 May 2013 21:10:13 +0000": "RT @GlamPimpLinda: Adam Lambert's fans land him an O Awards nomination: Pressparty http://t.co/xgJODYkYPK via @twitterapi"
}
{% endhighlight %}

Hopefully this little tutorial helps if you run into the same issue.