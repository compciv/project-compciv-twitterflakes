# Learning about CSV and Python data analysis with Twitter user info

In-class exploration of Twitter data and what makes a Twitter user a real person. Mostly practice with Python's `csv` parsing and collection sorting/filtering.



<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Data files](#data-files)
- [Getting a project folder set up with the command-line](#getting-a-project-folder-set-up-with-the-command-line)
  - [Creating the directories](#creating-the-directories)
  - [Use curl to download the relevant data files](#use-curl-to-download-the-relevant-data-files)
    - [curl for Macs:](#curl-for-macs)
    - [curl for Windows:](#curl-for-windows)
- [Python tasks](#python-tasks)
  - [How to open a CSV file, read it, and deserialize it](#how-to-open-a-csv-file-read-it-and-deserialize-it)
  - [How to filter the `peeps` list](#how-to-filter-the-peeps-list)
  - [Dealing with numbers that are strings](#dealing-with-numbers-that-are-strings)
  - [Sorting Twitter users](#sorting-twitter-users)
- [Bot-detection and other stories](#bot-detection-and-other-stories)
- [Background](#background)
  - [Examples of auditing/comparison/detection methods for determining Twitter user value](#examples-of-auditingcomparisondetection-methods-for-determining-twitter-user-value)
  - [Sorting a list](#sorting-a-list)
  - [Sorting lists of complex/irregular items](#sorting-lists-of-complexirregular-items)
  - [The importance of sorting](#the-importance-of-sorting)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Data files

The data files for this lesson consist of the Twitter users following and followed by the [@stanforddaily](https://twitter.com/stanforddaily) account, as it happens to be a nice manageable number (less than 100K). I [used a neat command-line tool](https://github.com/sferik/t) (the exact tool isn't terribly important) that made it easy to pull down the data in CSV format.

I uploaded the CSVs to Google Sheets so you could have a user-friendly view of the data, and even try traditional ways of sorting/filtering/analysis (via spreadsheet):

- [stanforddaily-twitter-followings](https://docs.google.com/spreadsheets/d/1YGVvm_aLOt5U756euW0n66z1zXpXsfuTtdmTPWToaMc/edit#gid=0)
- [stanforddaily-twitter-followers](https://docs.google.com/spreadsheets/d/1i5D8ggCY5kyDfJV991NrPlhIjwUKFtWURaQmQxgjLWM/edit#gid=0)

Here are URLs for directly downloading the text files:

- https://compciv.github.io/project-twitterfakes/data/stanforddaily-followings.csv
- https://compciv.github.io/project-twitterfakes/data/stanforddaily-followers.csv



## Getting a project folder set up with the command-line

This is just a quick run-through of how to create folders and download files via the command-line.

Open Terminal/PowerShell


### Creating the directories

```sh
# Create a new directory
$ mkdir ~/Desktop/project-compciv-twitterfakes
$ mkdir ~/Desktop/project-compciv-twitterfakes/data
# change our working directory
$ cd ~/Desktop/project-compciv-twitterfakes
```

### Use curl to download the relevant data files

#### curl for Macs:

```sh
$ curl -o data/stanforddaily-followings.csv \
   https://compciv.github.io/project-twitterfakes/data/stanforddaily-followings.csv

$ curl -o data/stanforddaily-followers.csv \
  https://compciv.github.io/project-twitterfakes/data/stanforddaily-followers.csv
```

#### curl for Windows:

(Run `conda install curl` if `curl.exe` isn't on your system...)

```
$ curl.exe -o data/stanforddaily-followings.csv `
   https://compciv.github.io/project-twitterfakes/data/stanforddaily-followings.csv

$ curl.exe -o data/stanforddaily-followers.csv `
  https://compciv.github.io/project-twitterfakes/data/stanforddaily-followers.csv
```

Optional: try opening these csv files from the command-line:

Macs: 

```
$ open data/stanforddaily-followings.csv
```

Windows:

```
$ start data/stanforddaily-followings.csv
```


## Python tasks

### How to open a CSV file, read it, and deserialize it

Here's one variation, that uses some shortcuts when it comes to the concept of opening files and reading them:

```py
from pathlib import Path
import csv

DATA_DIR = Path('data')
DATA_FILEPATH = Path(DATA_DIR, 'stanforddaily-followings.csv')
# opening the file
rawtxt = DATA_FILEPATH.read_text()
txtlines = rawtxt.splitlines()
# parse the CSV
peeps = list(csv.DictReader(txtlines))
```

Test out what `records` is, how big it is, and what it contains.


### How to filter the `peeps` list

Let's find out how many of these users are verified


```py
verified_peeps = []
for p in peeps:
  if p['Verified'] == 'true':
    verified_peeps.append(p)
```

A more Pythonic way (list comprehensions):

```py
verified_peeps = [p for p in peeps if p['Verified'] == 'true']
```

### Dealing with numbers that are strings

One of the aspects of using CSV to store data objects is that there's really no way to tell what values, if any, are intended to be treated as numbers. 

So deserializing CSV -- converting the text into data objects -- treats everything as a string:

```py
>>>> p = peeps[0]
>>>> p['Followers']
'1838'
>>>> type(p['Followers'])
str
```

Later on, we'll learn how to use [data science libraries such as pandas](https://pandas.pydata.org/), which, like Excel and other spreadsheets, will guess datatypes for our convenience. But for now, let's learn how to do things the old-fashioned way, which includes manual conversion of objects.

Let's try to filter the list for users who have more than 10000 followers. You'll find that the following code throws a **runtime** error because Python won't compare an `int` to a `str`:

```py
pop_peeps = []
for p in peeps:
  if p['Followers'] > 10000:
    pop_peeps.append(p)
```

And this attempt will result in a **data error** (i.e. not one that makes Python choke):


```py
pop_peeps = []
for p in peeps:
  if p['Followers'] > '10000':
    pop_peeps.append(p)
```

Why? See if you can figure out the logic here:


```py
>>>> '12' > '10'
True
>>>> '12' > '20'
False
>>>> '12' > '100'
True
>>>> '12' > '9'
False
>>>> 'r' > 'c'
True
>>>> 'r' > 'ra'
False
>>>> 'r' > 'czzzzzzzz'
True
```

To properly filter with a numerical comparison, we have to convert the value in `p['Followers']` to a number. The easiest way to do that is to invoke the `int()` function, which converts a string of numbers to an actual number datatype for Python purposes:



```py
>>>> int('42')
42
```

```py
pop_peeps = []
for p in peeps:
  if int(p['Followers']) > 10000:
     pop_peeps.append(p)
```


### Sorting Twitter users

Use the `sorted()` method with its `key` argument to get a list of the 5 most followed Twitter users.

Remember that the comparison has to be done by numerical comparison:

```py
def kfoo(m):
  return int(m['Followers'])

mostpop = sorted(peeps, key=kfoo, reverse=True)
topfive = mostpop[0:5]
```


## Bot-detection and other story ideas


- Sorting by a single metric, such as most followers, may not be super interesting. But what if that sort were reversed? What if we combined metrics (via arithmetic)?
- Think about combining filters, such as finding the highest-followed users who are also not verified.
- Comparing and sorting numerical metrics is pretty easy -- and often controversial/annoying -- think of how people are compared/judged by income, age, IQ, SAT scores, height, weight. What are ways we can convert non-numerical values (i.e. strings) into numerically comparable values?


## Background

### Stories

- [The Follower Factory](https://www.nytimes.com/interactive/2018/01/27/technology/social-media-bots.html)
- [Twitter Followers Vanish Amid Inquiries into Fake Accounts](https://www.nytimes.com/interactive/2018/01/31/technology/social-media-bots-investigations.html)
  - https://twitter.com/Rich_Harris/status/958704701826101250
  - https://twitter.com/Rich_Harris/status/957736903482183681
  - https://twitter.com/Rich_Harris/status/957310634378629121 
- [Critic Richard Roeper Suspended by Chicago Sun-Times After Buying Twitter Followers](https://www.hollywoodreporter.com/news/film-critic-richard-roeper-suspended-by-chicago-sun-times-purchasing-twitter-followers-1079969)
- [Twitter has been ignoring its fake account problem for years](https://www.cjr.org/innovations/twitter-fake-follower-accounts.php)
- A story from a year ago: [How to Hack an Election](https://www.bloomberg.com/features/2016-how-to-hack-an-election/)
- An even older story, but one that I feel sheds light on Twitter's constant problems with detecting abusive/bad behavior: [Even ISIS Guys Have Twitter Drama](http://gawker.com/even-isis-guys-have-twitter-drama-1740541455)


#### When fingerprinting/family analysis needs augmentation of real world observations

[Elaine Ou posted a script on Github for doing the data collection and visualization done by the NYT](https://github.com/elaineo/FollowerFactory)

She also wrote a post, [Your Twitter Followers are Probably Bots](https://elaineou.com/2018/01/30/your-twitter-followers-are-probably-bots/), where she pondered whether the NYT was protecting its own employees and allies:

- [@paulkrugman](https://i2.wp.com/elaineou.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-29-at-10.57.56-PM.png?w=1234&ssl=1)
- [AG Schneiderman](https://i1.wp.com/elaineou.com/wp-content/uploads/2018/01/agschneiderman.png?resize=1200%2C723&ssl=1)

Some rebuttal/observations from the NYT's Rich Harris:

- On why AG Schneiderman had large clumps of followers all at once: https://twitter.com/Rich_Harris/status/958425919680663556
- On whether the NYT looked into its own house: https://twitter.com/Rich_Harris/status/958426184769048577

### Questions to have in mind

- What heuristics can be used to efficiently filter out good/bad Twitter users?
- How does a typical Twitter user attract other accounts?
- How does a typical Twitter user make decisions about following other Twitter accounts?
- How does an active/good/valuable Twitter user behave, when it comes to Twitter activity?
- What is the allure of using Twitter follower numbers as a metric?
- What are obvious examples where these heuristics 
- In the NYT Followers Factory story, how were they able to know that certain users had bought fake followers?
- According to the NYT, how do fake-follower factories create fake accounts that are hard to detect?
- Should journalists, and people in similar roles, face punishment for buying into this scam?
- Does it seem like Twitter is doing a diligent job in policing itself. How could we tell or not?
- What techniques are used to impersonate real people?
- Why do people feel pressured to buy fake followers in the first place?
- Explain the reasoning behind the NYT's fingerprinting/"fake familes" graph. Why did they use the metrics (e.g. join date) that they did?


### Examples of auditing/comparison/detection methods for determining Twitter user value

- [TwitterAudit](https://www.twitteraudit.com/StanfordDaily)
- [Twee-Q purported to measure the gender diversity/bias of a Twitter user](https://source.opennews.org/articles/gender-twitter-and-value-taking-things-apart/)
- [BotorNot](http://botornot.co/)
- [Fake Follower Check](https://fakers.statuspeople.com/)


Last but not least, Twitter has an analytics service, which you can visit for your own account here:

https://analytics.twitter.com/

Be sure to check out the Audiences and Tweets views, such as Interests and Gender and household income and education.

Note all the data points they claim to assess about users despite users not explicitly stating -- e.g. 


### Sorting a list

The concept of sorting is relatively understandable. I do think that doing it programmatically feels harder than it should, especially compared to the ease of use of a spreadsheet. But as you start to realize how fine-grained/specific your sorting conditions can be, the "complications" of sorting in Python will make sense.

A primer: http://www.compciv.org/guides/python/fundamentals/sorting-collections-with-sorted/

One quick tip: Basically, always use the `sorted()` method

Sorting a list of individual items (such as numbers and words) is straight-forward:

```py
>>>> mydata = ['b', 'dog', 'apples']
>>>> sorted(mydata)
['apples', 'b', 'dog']

>>>> mydata = [99, 4, -45, 12]
>>>> sorted(mydata)
[-45, 4, 12, 99]
```

### Sorting lists of complex/irregular items


However, when we're dealing with a list of complex, irregular objects, the `sorted()` method will (if we are lucky) raise an error and complain that it doesn't know how to compare the objects. Or, if we are unlucky, it will do something unpredictable.

Look at the individual examples below and see if you don't also see the ambiguity:

```py
>>>> [1, 2, 3] > [2]

>>>> {'name': 'Scott', 'age': 40} > {'name': 'Zelda', 'age': 33}

>>>> 100000 > [1]
```

In these situations, we take advantage of the `sorted()` function's `key` argument, to which we pass a function that can be used by `sorted()` for comparison.

For example, consider 2 lists:
```py
a = [1, 42]
b = [2, 1, 1]
```

It's possible to compare them without getting an error, but is the result what you expect?


```py
>>>> a > b
False
```

What if we wanted to sort `a` and `b` by their number of elements? We would use the `len()` function:

```py
>>>> len(a) > len(b)
False
```

Another person might argue that it's the sum of the elements that should be compared. In that case, we use `sum()`:

```py
>>>> sum(a) > sum(b)
True
```

This is how we tell `sorted()` to apply a function via its `key` argument:

```py
>>>> mydata = [a, b]
>>>> sorted(mydata, key=len)
[[1, 42], [2, 1, 1]]
>>>> sorted(mydata, key=sum)
```

What if we want to sort by a transformation for which there is no Python built-in function? Then we can simply write our own function and name it what we want. This function is to be designed as if it were operating on one member of the list at a time. And it should return what a sorting function would use to rank/sort that member.

For example, let's say I have a list of strings, and I want to sort them in ascending order of number of vowels. Here's a function that returns a vowel count for a given string:

```py
def vfoo(txt):
  vcount = 0
  for t in txt.lower():
    if t in 'aeiou':
       vcount += 1
  return vcount
```

We pass in the name of the function to `sorted()`'s `key` argument:

```py
>>>> names = ['danny', 'aaron', 'zee']
>>>> sorted(names, key=vfoo)
['danny', 'zee', 'aaron']
```

The Python standard library comes with some helpers that make it easier to deal with common situations, such as comparing dict objects by a key value:

```py
>>>> from operator import itemgetter
>>>> mydata = [{'name': 'Aaron', 'age': 42}, {'name': 'Sally', 'age': 29}]
>>>> sorted(mydata, key=itemgetter('age'))
[{'age': 29, 'name': 'Sally'}, {'age': 42, 'name': 'Aaron'}]
```




### The importance of sorting 

Besides filtering, sorting is one of the most important things we can do with data. In life, we don't have time to process all information and all events. We make choices not only on principle, but on *priorities*.

Think of how innately attractive "Top N" lists are. Think of how many contests and competitions are won based on having the most of something or other.

Sorting is an integral part of storytelling. For a typical news story, most journalists do not present a chronological ordering of events. Instead, most journalists are taught the ["inverted pyramid"](https://en.wikipedia.org/wiki/Inverted_pyramid_(journalism)), where the most "newsworthy" facts should be told at the top of the story.

(nevermind what it means for one fact to be more "newsworthy" than the other)

One of the most prominent visual examples of the importance of sorting can be found at the Vietnam War Memorial, as Edward Tufte notes in an essay on lists:

https://www.edwardtufte.com/bboard/q-and-a-fetch-msg?msg_id=0002QF

Given a list of names of fallen veterans, all of whom are to be given equal prominence at the memorial, what is the right thing to sort by? It seems natural to go by alphabetical order, the way many other lists of people are sorted by. 
> ...But when a two-inch thick Defense Department listing of Vietnam casualties was examined, thinking changed. There were over 600 Smiths; 16 people named James Jones had died in Vietnam. Alphabetical listing would make the Memorial look like a telephone book engraved in granite, destroying the sense of unique loss that each name carried...""

Direct link to the page/info about the Vietnam War memorial:
https://en.wikipedia.org/wiki/Inverted_pyramid_(journalism)
