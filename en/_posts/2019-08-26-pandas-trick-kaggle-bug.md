---
layout: post
title:  "A pandas trick from a Kaggle bug"
date:   2019-08-26 09:00:00 -0300
categories: data-science
lang: en
lang-ref: pandas-trick-kaggle-bug
---

It's been a while since I started thinking about setting up a [Kaggle](https://kaggle.com) account. Actually I had already registered an account three years ago but I've never actually done anything there.

In case you're not familiar with Kaggle, it has been known mostly for hosting various [competitions](https://www.kaggle.com/competitions). These competitions cover different areas, from image recognition to health and sales. In a nutshell, they give you the data, the problem to be tackled and through data science, AI and machine learning you try to get to the desired results. I was never big on joining these. For one I never felt I was truly quite ready. Moreover, generally speaking, competitions were never my thing.

However, Kaggle has launched some very interesting [courses](https://www.kaggle.com/learn/overview) lately, covering from [pandas](https://www.kaggle.com/learn/pandas) to [machine learning explainability](https://www.kaggle.com/learn/machine-learning-explainability). I, for one, am a person that learns best through practical, hands-on exercises, so I'm in love with this new feature. One of these new courses is the [Intro to SQL](https://www.kaggle.com/learn/intro-to-sql). This course is part of the [SQL Summer Camp](https://www.kaggle.com/sql-summer-camp).

It is a very introductory course, so it shouldn't be hard to go through it if you're familiar with SQL. However, as part of a Summer camp for beginners, I find this incredibly useful. There is even an accompanying [YouTube playlist](https://www.youtube.com/watch?v=jYQoQfFzJRw&list=PLqFaTIg4myu9neIs_wfWzgeOkKbiImXB6) to guide you through it.

The courses are composed of some text covering a bit of the theory followed by some practice on the topic being explored. The practice session is carried out in a [kernel/notebook](https://www.kaggle.com/docs/kernels) and it's very nicely put together. It has a nifty feature for checking whether you got the correct results or not, and it can even give you hints on how to better your answer.

However, during [one of the exercises](https://www.kaggle.com/dansbecker/as-with) in the Intro do SQL, in which I had to gather the total number of taxi trips from each year from a [bigquery](https://cloud.google.com/bigquery/) [dataset](https://console.cloud.google.com/marketplace/details/city-of-chicago-public-data/chicago-taxi-trips), I ran into a bug.

The query was very straightforward.
```
rides_per_year_query = """
    SELECT
        EXTRACT(YEAR FROM trip_start_timestamp) as year,
        COUNT(1) as num_trips
    FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
    GROUP BY year
    ORDER BY year DESC
                        """
```

I thought my query was perfectly fine and seemed to give me the correct answer. However, Kaggle's question checking function went nuts and returned me [this error trace](https://pastebin.com/rePWBm5G). I was very confused, so I peeked at the solution and, to my surprise, it only differed from mine in that it had no `DESC` ordering.

I decided to report this weird behavior in their [forum](https://www.kaggle.com/product-feedback/105149). Not much time went by and another user commented in my post with a simple and clever explanation to what had just happened.

[Alexander Savchuck](https://www.kaggle.com/amsavchuk) created a [kernel](https://www.kaggle.com/amsavchuk/learntools-checking-in-sql-summer-camp) for going through his reasoning. You can take a look at what he found there, but in case you're still a bit lost with all this info on the Kaggle courses, kernels and question checking, I'll go into it in a bit detail over here.

So basically the question checking function in the Intro to SQL course for this question works by checking three things. First it checks if you got the correct column names. That's simple and obviously very important. Then it checks the length of the dataframe generated from your query. Finally, it compares the value of one of the rows from your dataframe to that of the expected answer. That should be enough to guarantee you have the right answer. But then, the bug seemed to arise when trying to retrieve the users' answer in the following way

`submitted_number = int(results[results["year"]==year_to_check]["num_trips"][0])`

(you can see how this is done in detail in [Kaggle's Github](https://github.com/Kaggle/learntools/tree/master/learntools); more specifically, I'm looking at [this](https://github.com/Kaggle/learntools/blob/master/learntools/sql/ex5.py) code.)

The problem with checking the answer like this, as you can see by the last line of the error trace (`KeyError: 0`) is that pandas does not handle indexing quite the same way python handles a list or an array.

One might look at this piece of code and imagine that by adding `[0]` to the end of the dataframe it will be retrieving the first element/row of the dataset. But that's not what it does. Adding `[0]` to your pandas transformation will make it behave as a `loc` rather than `iloc`.

Ok, I'll back up a bit. What does `loc` and `iloc` do? [It's actually quite simple](https://stackoverflow.com/questions/31593201/how-are-iloc-ix-and-loc-different). `dataframe.iloc[0]` will return the first row in your dataframe. That's roughly the same as doing `array[0]` for retrieving the first element of an array. But `dataframe[0]` will be translated to `dataframe.loc[0]`, which will give you the row whose **label** is `0`. So `loc` has nothing to do with the row position, only its label.

This makes the error clearer: since I sorted my query in a descending order, from 2019 to 2013, my dataframe looked like this

```
year  num_trips
0  2019    9843414
1  2018   20732088
2  2017   24988003
3  2016   31759339
4  2015   32385875
5  2014   37395436
6  2013   27217716
```

while the expected result was

```
   year  num_trips
0  2013   27217716
1  2014   37395436
2  2015   32385875
3  2016   31759339
4  2017   24988003
5  2018   20732088
6  2019    9843414
```

So if I try to run

`submitted_number_row = results[results["year"]==year_to_check]["num_trips"]`

with `year_to_check = 2013` in my dataframe (sorted with `DESC`) the result would be

```
6    27217716
Name: num_trips, dtype: int64
```

(which is a pandas series).

Trying to do `submitted_number = int(submitted_number_row[0])` would raise the aforementioned error. Because this would be translated to `submitted_number = int(submitted_number_row.loc[0])` and, as shown previously, pandas would be looking for the label `0`, while the only row present has the label `6`.

If I executed the same query (`submitted_number_row = results[results["year"]==year_to_check]["num_trips"]`) in the dataframe with the default ordering, I'd get as result

```
0    27217716
Name: num_trips, dtype: int64
```

and in this case no error would be raised, since the label `0` is actually present this time.

That's what that huge error trace was all about! As it showed us in the last line, pandas simply could not find the label `0` in the given dataframe (`KeyError: 0`).

So both Alexander and [Alexis Cook](https://www.kaggle.com/alexisbcook), Kaggle team member responsible for this course, came up with a nice fix to this issue in [that post](https://www.kaggle.com/product-feedback/105149).

Alexander simply suggested using `iloc[0]` explicitly for getting back the first element of the dataframe correctly. As in

```
submitted_number =
  int(results[results["year"]==year_to_check]["num_trips"].iloc[0])
```

Alexis opted for using `.values` instead, with some minor changes to the original code

```
submitted_number =
  int(results.loc[results["year"]==year_to_check]["num_trips"].values)
```

In the end both options solve the issue.

To me, this was a nice reminder that you can always learn from mistakes. And not only your own, but also from others' as well! And I'm in no way mocking Kaggle. They handled the issue pretty quickly and I praise them for the new courses and all the features they developed to make this a much appreciated addition to the site.

Anyhow, the message stays true. Learn from every opportunity, specially through mistakes.

"Learn from the mistakes of others. You can never live long enough to make them all yourself." [Groucho Marx](https://pinterest.com/pin/533535887077279336/), apparently. Or maybe [Eleanor Roosevelt](https://pinterest.com/pin/378020962462200152/). Well, someone here is mistaken.
