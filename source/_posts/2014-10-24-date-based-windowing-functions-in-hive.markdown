---
layout: post
title: "Date-based windowing functions in Hive"
date: 2014-10-24 11:16
comments: true
categories: hive hadoop data-science big-data
published: true
---

At my current gig, when migrating our data warehouse to Hive, I had a bit of trouble tracking down documentation on date-based windowing function support in Hive. I had to piece together information from a few sources, so I figured this might be useful to others. At the time of this post, the Hive [wiki page on windowing functions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+WindowingAndAnalytics "Windowing functions") makes no mention of range-based windows, but I did find so elsewhere. There are a couple Hive [jira](https://issues.apache.org/jira/browse/HIVE-4197) [tickets](https://issues.apache.org/jira/browse/HIVE-4112) which mention support for range-based windows. Unfortunately, neither discuss date-based ranges which is my use case. A bit more digging turned up a valuable [interview](http://www.qubole.com/big-data-analytics-using-window-functions) with one of the Hortonworks committers where he sheds a bit more light on the topic:

{% blockquote Harish Butani http://www.qubole.com/big-data-analytics-using-window-functions %}
The window ranges today are numeric constants, whereas, in the spec, you can have arbitrary numerical expressions. And in Hive, you don’t have date intervals. So if you want to do date-based windows, you pretty much have to break down the date into components, and treat each component as a int.
{% endblockquote %}

What this tells us is we need to convert the date to a numeric constant when using it in a range window. Let's walk through an example.

Suppose we have the following transaction data:

user|store       |purchase_date|amount
:----:|:------------|:-------------:|------:
42|petes coffee|2014-01-01|1
42|petes coffee|2014-01-02|3
42|petes coffee|2014-01-03|2
42|petes coffee|2014-01-04|4
42|petes coffee|2014-01-05|1
42|petes coffee|2014-01-06|1
42|petes coffee|2014-01-07|1
44|petes coffee|2014-01-01|1
44|petes coffee|2014-01-02|1
44|petes coffee|2014-01-03|1
44|petes coffee|2014-01-04|1
44|petes coffee|2014-01-05|5
44|petes coffee|2014-01-06|5
44|petes coffee|2014-01-07|5


<p/>
Now suppose we want to answer the question: What is the purchase amount for each transaction and what is the N-day purchase amount total leading up to that transaction? The following query could answer that question using a range-based window.


{% codeblock Date-based Windowing lang:sql %}
select *, 
 sum(amount) over 
   (partition by user order by unix_timestamp(purchase_date) RANGE 172800 PRECEDING) as 3_day_window
 from transactions 
 order by user, purchase_date asc;
{% endcodeblock %}

A few things to note about the query. First, we want to do this on a user-basis, so we partition by user. Next, we must use *unix_timestamp* to convert the date to a numeric constant in seconds. This will be the value in seconds since the [unix epoch](https://en.wikipedia.org/wiki/Unix_time#Encoding_time_as_a_number). Finally, the *RANGE X PRECEDING* says, go backwards from the current row until the value of the column used in the *order by* clause changes by *X* amount or we hit the partition boundary. In this example I use 172800 which is 2 days in seconds plus the current row, giving me a 3 day window.

Here is the result set:

row_num|user|store       |purchase_date|amount|3_day_total
:--|:----:|:------------|:-------------:|------:|----:
1|42|petes coffee|2014-01-01|1|1
2|42|petes coffee|2014-01-02|3|4
3|42|petes coffee|2014-01-03|2|6
4|42|petes coffee|2014-01-04|4|9
5|42|petes coffee|2014-01-05|1|7
6|42|petes coffee|2014-01-06|1|6
7|42|petes coffee|2014-01-07|1|3
8|44|petes coffee|2014-01-01|1|1
9|44|petes coffee|2014-01-02|1|2
10|44|petes coffee|2014-01-03|1|3
11|44|petes coffee|2014-01-04|1|3
12|44|petes coffee|2014-01-05|5|7
13|44|petes coffee|2014-01-06|5|11
14|44|petes coffee|2014-01-07|5|15


<p/>

Let's explain some of the results.

* Row 9 - The 3_day_total is 2 because only row 8 is included. Row 7 is for the previous user, which is the partition boundary, so the range traversal ceases.
* Row 10 - Here we have a full 3 day range since it does not bump up against the partition boundary. The n-day total is 1+1+1 for the dates 2014-01-01, 2014-01-02 and 2014-01-03.
* Row 14 - This proves that the window is sliding along with the current row. The n-day total is 5+5+5 for the dates 2014-01-05, 2014-01-06 and 2014-01-07.

Hope this is useful to someone!
