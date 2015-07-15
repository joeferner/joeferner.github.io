---
layout: post
title: "Linux Command Line: Parsing HTML with w3m and awk"
---

I needed to generate some fake data to simulate transactions. I wanted some valid merchant names
to make the data look reasonable. After failing to search the internt for a nice CSV containing merchant names
I settled on this [Top 100 Retailers Chart 2011](https://nrf.com/resources/top-retailers-list/top-100-retailers-2011). Unfortunatly when you copy and paste the table you get a run together mess.

```
1 Wal-Mart Bentonville, Ark. $307,736,000 0.6% $421,886,000 72.9% 4,358 1.3% 2 Kroger Cincinnati $78,326,000 6.4% $78,326,000 100.0% 3,609 -0.4% 3 Target Minneapolis $65,815,000 3.8% $65,815,000 100.0% 1,750 0.6% 4 Walgreen Deerfield, Ill. $61,240,000 6.3% $63,038,000 97.1% 7,456 8.1% 5 The Home Depot Atlanta $60,194,000 2.2% $68,000,000 88.5% 1,966 0.0% - See more at: https://nrf.com/resources/top-retailers-list/top-100-retailers-2011#sthash.RUUwpfm0.dpuf
```

## Download using curl

This is the easiest part and most Linux systems will have this installed by default.

{% highlight bash %}
curl -s https://nrf.com/resources/top-retailers-list/top-100-retailers-2011
{% endhighlight %}

`-s` will tell `curl` to be silent with it's messages. The output will go to standard output.

## Normalize the HTML

Before we extract content from the HTML we need it to be normalized. To do this we can use `hxnormalize` by w3.org in
their [HTML-XML-utils package](http://www.w3.org/Tools/HTML-XML-utils/).

{% highlight bash %}
... | hxnormalize -x
{% endhighlight %}

`-x` will tell `hxnormalize` to output XHTML.

## Extract the table we care about

Now we need only the content we care about. [HTML-XML-utils package](http://www.w3.org/Tools/HTML-XML-utils/) has
a tool for this as well `hxselect`.

{% highlight bash %}
... | hxselect 'table.views-table'
{% endhighlight %}

`'table.views-table'` tells `hxselect` to extract all table with a CSS selector of `views-table`.

## Format the HTML

[w3m](http://w3m.sourceforge.net/) is a command line text based web browser. It can also just dump formatted HTML to standard out which is what I used it for.

{% highlight bash %}
... | w3m -dump -cols 2000 -T 'text/html'
{% endhighlight %}

`-dump` tells `w3m` to write it's output to standard out as opposed to a scrollable viewer. `-cols 2000` ensures
we don't have wrapping of the lines which would make parsing more tedious. `-T 'text/html` tells `w3m` that the
input should be treated as HTML.

## Grab the columns we we want

Finally we need to grab only the first column. `awk` will help with that.

{% highlight bash %}
... | awk 'BEGIN{FIELDWIDTHS="5 29"}{gsub(/^[ \t]+/, "", $2); gsub(/[ \t]+$/, "", $2); print $2}'
{% endhighlight %}

Lets break down the `awk` script a little. `BEGIN{...}` is used to run something before we start processing data.
In this case `FIELDWIDTHS="5 29"` tells `awk` that the first 5 columns are field 1 and the next 29 columns are field
2 and the remaining columns are field 3.

The second part of the `awk` script is what will run on each line. The first two `gsub` statements will trim the
start and end of the respectively. Finally `print $2` will print the 2nd column which in our case is the company
name.

## All together

Here is the final full command to run.

{% highlight bash %}
curl -s https://nrf.com/resources/top-retailers-list/top-100-retailers-2011 \
  | hxnormalize -x \
  | hxselect -s '\n' 'table.views-table' \
  | w3m -dump -cols 2000 -T 'text/html' \
  | awk 'BEGIN{FIELDWIDTHS="5 29"}{gsub(/^[ \t]+/, "", $2); gsub(/[ \t]+$/, "", $2); print $2}'
{% endhighlight %}
