# Normalization

## What do we expect?

It is interesting to how many tweets and bookmarks a paper recieves, but just the raw number leaves us wanting more.  How many tweets is a lot?  How many "Likes" do papers like this usually receive?

Let's see what it means to figure out the expected impact for "papers like this":

First, expected impact depends on the *age of the paper*.  Older papers have had longer to accumulate impact: an older paper is likely to have more citations than a younger paper.  

Second, especially for some metrics, expected impact depends on the *absolute year of publication*.  Because papers often get the most social media attention soon after they are published, papers published in years when a social tool is very popular recieve more attention on that tool than papers published before or after the tool was popular.  For example, papers published in years when twitter is popular recieve more tweets than papers published in the 1980s.

Third, expected impact depends on the *size of the field*.  The more people there are who read papers like this, the more people there are who might "Like" it.

Fourth, expected impact depends on the *tool adoption patterns* of a subdiscipline.  Papers in a field with has a strong Mendeley community will receive more Menedeley readers than papers pusblished in a field that prefers Zotero.

Finally, expected impact levels depends on what we mean by papers *like this*.  How do we define the relevant reference set?  Other papers in this journal?  Papers with the same indexing terms?  Funded under the same program?  By investigators I consider my competition?  

There are other variables too.  For example, a paper published in a journal that tweets all its new publications will get a twitter boost, and an Open Access paper might receive more Shares than one behind a paywall.    

Establishing a clear and robust baseline won't be easy, given all of this compexity!  That said, let's start.  Stay tuned for our plans...

## Percentiles

In the last post we talked about the need to supplement raw counts with background information about expected impact.  How should this background information be communicated?  How can raw counts be presented in the context of this background?

Our favourite approach: percentiles.   

Try it on for size: Paper ABC is in the 88th percentile of CiteULike bookmarks, relative to other papers like it.  That tells you something, doesn't it?  It has a lot of bookmarks, but there are some papers with more.  Simple, succinct, intuitive, and applicable to any type of metric.

Percentiles were also a favourite in the "normalization" breakout group at [altmetrics12](http://altmetrics.org/altmetrics12/) and have already popped up as a [total-impact feature request](https://totalimpact.uservoice.com/forums/166950-general/suggestions/3028257-show-percentile-metrics-as-well-as-counts).  They have been explored scientometrics for journal impact metrics, including in a [recent preprint by Leydesdorff and Bornmann](http://arxiv.org/pdf/1103.5241.pdf).10.1002/asi.21609

As it turns out, actually implementing percentiles for altmetrics isn't quite as simple as it sounds.  We have to make a few decisions about how to handle ties, and zeros, and sampling... stay tuned for our next post...


## Percentiles, the tricky bits

Percentiles seem so easy!  And they are, except for their tricky bits.  

Our first clue that percentiles have tricky bits is that there is no standard definition for what percentile means.  When you get an 800/800 on your SAT test, the testing board announces you are in the 98th percentile (or whatever) because 2% of test-takers got an 800... their definition of percentile is *the percentage of tests with scores less than yours.*  A different choice would be to declare that 800/800 is the 100th percentile, representing *the percentage with tests with scores less than or equal to yours.*  Total-impact will choose the second definition because it seems more intuitive.

What to do with ties?  Imagine there were only 10 SAT takers, and they scored one 400, eight 600s and one 700.  What is the percentile for those who scored 600?  They are right in the middle of the pack, so it seems that they are in the 50th percentile.  But this could be misleading... their scores are more accurately represented by a range, in this case the 20th-80th percentile.  Altmetrics has to deal with a lot of ties: there are a lot of papers that recieve only one tweet, for example.  We will deal with ties by representing them as a range.  When we need to collapse this to a single number (for graphing, for example) we'll use the midpoint of the range.

Finally, what to do with zeros?  Impact metrics have many zeros: many papers have never been tweeted.  Our range solution works well.  If your paper hasn't been tweeted, but neither have 80% of papers in your field, then your percentile range for tweets would be 0-80th.  Summarizing this result as a single number using the midpoint of the range seems counterintutive though.  40th percentile, even though nothing has happened?  So in the case of zeros, when we need to summarize as a single number, we'll use 0.

Given these definitions, let's take the for a test drive on Mendeley Readership percentiles for a hypothetical set of 100 papers, ordered by increasing readership:

10 papers with 0 readers (0-10th)
40 papers with 1 reader (11-50th)
10 papers with 2 readers (51-60th)
9 papers with 3 readers (61-69th)
1 papers with 5 readers (70th)
29 papers with 10 readers (71-99th)
1 paper with 42 readers (100th)

If someone came to us with a new paper that had 0 Mendeley Readers, given the background above we would assign it to the 0-10th percentile, and graph it as a 0.  A new paper with 1 Reader would be in the 10st-50th percentile, graphed as 30th.  A new paper with 5 Readers is easy: 70st percentile.  If we got a paper with 8 readers we'd see it falls between examples we've seen before, so we'd interpolate, see it lies between the 70th and 71st percentile, and report it as 70th percentile.  If we get something that got more readers than anything we'd ever seen before we'd give it a 100+ percentile.

Does this map to what you'd expect?  Our goal is to communicate accurate data as simply and intuitively as possible.  Let us know what you think!

## Choosing reference sets

We're going to take a short break from thinking about percentile calculations to think about reference sets.  In the previous post we assumed we knew which 100 papers we wanted to use as a baseline for stats.  Where would we get such a list?  What papers do we want in our Mendeley readership percentile calculations?  How can we find them?

The answer is, of course, that it depends.  It depends on who we are and what story we are trying to tell with our numbers.  In the future, perhaps we could indeed make this very flexible and allow users to specify custom reference sets by query terms or doi lists.

For now, though, let's start simply, with just a few standard reference sets to get going.  Standard reference sets should be:
* meaningful and interesting
* easily interpreted
* not too high impact nor too low impact, so gradations can be apparant
* applicable to a wide variety of papers
* amenable to large-scale collection
* if large, available as a random sample

For practical reasons we start with the last three points.  We need to collect our reference samples with automated tools.  Unfortunately, few open scholarly indexes allow queries by scholarly discipline or subdiscipline... but there is one stellar exception.  PubMed.  If only all of research had a PubMed!  PubMed's eUtils API lets us query by MeSH indexing term, journal title, funder name, all sorts of things.  It returns lists of PMIDs that match our queries.  This isn't a random sample out-of-the-box, but we can fix that ([see code!](https://github.com/total-impact/total-impact-core/blob/master/extras/random_pmids.py)).  

What query should we use to derive our reference set?  After thinking hard about the first three points and doing some experimentation, we've decided to start with "NIH-funded research".  It is pretty broad, so it is roughly applicable to a wide variety of papers.  People have a good sense for what it represents, and knowing that a metric is in the Xth percentile of NIH-funded research is a meaningful statistic.

There is of course one huge downside to PubMed-inspired reference sets: it focuses on a single domain.  Biomedicine is a huge and important domain, so that's good, but leaving out other domains is sad.  We'll definitely be keeping an eye on other solutions to derive easy reference sets (a PubSci and PubSocialSci anyone?  Or hopefully Mendeley will include query by subdiscipline in its api soon?).  That said, focus is a good thing, so we're going to run with this and learn as much as we can.


## Percentiles: confidence

The final step on our percentile journey involves some lovely math.  In our last post we thought about what we'd use as a reference set, and decided to run with "NIH funded research".  It is impractical at the present time to run all of NIH funded research papers through total-impact (!), so we are going to have to sample the papers.  W'll be able to generalize from our sample back to all of NIH-funded research if we calculate confidence intervals for our percentiles.

Confidence intervals for our percentiles, eh?  Sound hard to calculate?  They aren't, as it turns out, they are gloriously easy.  You see, another name for percentiles is "Order Statistics", and statisticians have been studying order statistics for a long time.  They've derived a really neat thing.  As you might expect, if you have 100 samples, the best estimate for the 50th percentile is the 50th value if you order all the values from highest to lowest.  Now want to know the 95% confidence intervals?  They are simply the values in your list at spot 40 and spot 60 (in R, [this can be calculated](http://stats.stackexchange.com/questions/33226/explanation-for-confidence-interval-for-quantile-with-small-data-set) with qbinom(c(.025,.975), 100, 0.5)).  The 95% confidence intervals for the 75th percentile are the values in your list at spot 66 and spot 83 (in R, qbinom(c(.025,.975), 100, 0.75)).  The fact that they are always at known places in the ordered list of values makes them very easy to look up!

Now we have to look at this from the other direction: rather than figuring out the confidence interval for a given percentile, we want to know the confidence ranges for new data point.  

```{r}
set.seed(42)
x = rnorm(100, 15, 5)
interval_indexes_50th = qbinom(c(.025,.975), length(x), 0.5)
interval_50th = sort(x)[interval_indexes_50th]
interval_50th
interval_indexes_60th = qbinom(c(.025,.975), length(x), 0.6)
interval_60th = sort(x)[interval_indexes_60th]
interval_60th
```

So with 95% confidence (only 5% of the time would our sample have been so unexpected that this wouldn't be true), a 16 could be in either the 50th or 60th percentile in the underlying population.  In fact, if we experiment a bit we can determine that a 16 might be as low as the 43rd percentile or as high as the 62nd percentile in the population represented by this sample.  In total-impact, if the raw value was 16 for this metric, given these reference sample values we'd report the percentile range as 43-62nd percent.

Isn't that cool???  (yes I'm kind of a math geek.)  

The only remaining issue is how we handle ties in our reference set.  Total-impact reports the lower confidence interval bound as the lower CI for the lowest postition held by the tie, and the upper CI as the upper CI for the highest position held by the tie.

* GIVE AN EXAMPLE OF THIS *

* ALSO LINK TO THE LOOKUP TABLES FOR THOSE WHO WANT IT? *


## Refsets - some data!



```{r}
refsets <- read.csv("http://localhost:5001/collections/reference-sets-histograms", header=T)

library(reshape)
a = melt(refsets, id.vars=c('genre', 'year', 'refset', 'metric_name'))
a = transform(a, nth=100 - as.numeric(gsub("X|th", "", levels(variable)[variable])))

library(ggplot2)


for (i in 1:5) {
  refset_lists = c("nih", "pubmed", "plosone", "nature", "science")
  myrefset = refset_lists[i]
  g = ggplot(subset(a, metric_name=="pubmed:pmc_citations"&refset==myrefset), aes(x=nth, y=1+value)) + geom_bar(width=1, stat="identity", position="identity") + scale_y_log10(myrefset) + facet_grid(year~.)
  quartz()
  print(g)
}

sort(summary(a$metric_name), dec=TRUE)
"pubmed:pmc_citations"
"wikipedia:mentions"
"mendeley:readers"
"citeulike:bookmarks"

for (myyear in 2005:2010) {
  g = ggplot(subset(a, metric_name=="wikipedia:mentions"&year==myyear), aes(x=nth, y=1+value)) + geom_bar(width=1, stat="identity", position="identity") + scale_y_log10(myyear) + facet_grid(refset~.)
  quartz()
  print(g)
}


ggplot(subset(a, refset=="nih"), aes(value, color=factor(year), fill=factor(year))) + 
geom_density(alpha=0.2) +
scale_x_log10() + 
facet_grid(metric_name~.) 


ggplot(subset(a, metric_name=="mendeley:readers"), aes(value, color=factor(year), fill=factor(year))) + 
geom_density(alpha=0.2) +
scale_x_log10() + 
facet_grid(year~.)

scale_fill_manual(name="",
                    breaks=c("0", "1"),
                    labels=c("data NOT available", "data available"), 
                    values=cbgRaw) + 
theme_bw(base_size=16)

```

You can also embed plots, for example:

```{r fig.width=7, fig.height=6}
plot(cars)
```

