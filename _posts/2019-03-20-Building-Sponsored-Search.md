---
layout: post
title: Building Sponsored Search (Yield Optimization Framework - Paid Search)
comments: true
useMath: true
---

I recently joined Snagajob, which is the leading part-time job search platform in the US. Those who are familiar with 
[Indeed](http://www.indeed.com) or [LinkedIn](http://www.linkedin.com) - Jobs (feature), [Snagajob](http://www.snagajob.com) is similar but the primary focus is on part-time jobs. 
[Snagajob](http://www.snagajob.com) also have the shifts concepts where restaurants work with Snagajob to set up shifts schedule. 
This post is primarily going to cover how I implemented the Sponsored Search at [Snagajob](http://www.snagajob.com) 
which lead the company to move in the direction of a two-sided marketplace (Job Advertisers vs Job Seekers). In the next few articles 
I plan to cover how I went about on implementing [Snagajob](http://www.snagajob.com)'s first ever Yield Optimization Framework and what I learned on Sponsored Search. 
Before this, I had only worked on query expansions, and recall improvement in search engines and had very little knowledge on 
building a sponsored search engine with the ability to run campaigns. As a scientist, this was a fantastic challenge and 
I hope the shed some light into Sponsored search.  

```Let's cover the basics first. What is Sponsored Search / Yield Optimization Framework?``` 

Search Engines are not new to us. Search engines have a very colorful history. Starting from Lycos, AltaVista, 
Yahoo Search, Ask, Microsoft Search, and Google. Google may be the most successful Search Engine out of all but 
every search engine played a key role in getting us this far on search. Regardless, search is still not a solved problem, 
if it was we wouldn't be working on the search. 

The goal of the search engine is to bring the most relevant items to the query that is sent to the search engine in an 
orderly fashion. This is what we call a rank list. But how do we do that? Additionally one could ask few more questions, 
how do you define relevance (Search Relevance)? Search Relevance defines the calculation of the strength of the 
relationship between the search query and the results. Going back to our original question ```what is 
Sponsored Search?```

When a user types a keyword the search engine carries out a task on finding all the relevant documents it has in its 
index (more like inverted index - What is an inverted index? Hold on to that question too? ) and send it back to the user 
in an ordered list. There are multiple ways to order/rank these documents but the ultimate goal of ranking these documents 
is to have the most relevant documents show up at the top of the ranked list and order the rest in decreasing order of relevancy 
to the search query user entered. To calculate the relevance and order them in a decreasing fashion, there must exist a utility 
function. Most basic utility function of relevance measurement is [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 
(Term Frequency - Inverse Document Frequency). But when you run for-profit search engines there exists a duality problem where 
the search engine needs to be optimized for relevancy while optimizing for revenue. Sponsored Search is a search engine that 
has the capability of tuning the results with the target of increasing revenue without decreasing relevancy. But what many 
don't realize is if you have a good practice of improving the relevancy the increased revenue side needs less work than many think. 

Our goal is to implement a utility function that captures multiple signals of search behavior of users and use that to 
fine-tune the order of the list of these relevant documents. Let's dive into the ingredients of the utility function. 

### Click(s)
Formally we call it as [implicit feedback](https://en.wikipedia.org/wiki/Relevance_feedback) is a signal that
we use to understand the preference of a user to an item. In a search engine setting, we encapsulate this preference in 
the context of the query that the user submits to the search engine. The human judgments are noisy therefore this signal 
can be noisy. But there are basic remedies that we can apply to clean up.

Instead of just taking the clicked items filtering by time spent on a clicked link/item is a good indicator of the 
preference or adding another layer to collect the clicks that followed up with "added to basket", "applied jobs" 
(in a job search engine) from search are good techniques to filter the noisy clicks.

### Impression(s) 
How often an item (in this case a vacancy) shown. An impression is counted each time the item is shown on a search result page.

### Clicks to Click-Through Rate (CTR)
Once you identify how you collect the clicks (with the above filtering mechanism) these clicks need to be converted 
into a click-through rate. CTR is the ratio of how often people who see a listed item end up clicking it. 
Click-through-rate (CTR) can be used to gauge how well your keywords and ads, and free listings, are performing. 

CTR for Given Item can be calculated: 

```python
CTR = Total Number of Clicks / Total Number of Impressions
```
```i.e: if an item gets 10 clicks and it appeared in 100 searched results, the CTR is 5/100 = 0.05```

In the above example, we are calculating the average because we are not restricting per specific keyword CTR per item. 
For simplicity, we are only going to focus on the average CTR per item. 

In a sponsored search setting we have content owners paying for the search engine to obtain more clicks/traffic. 
To get more clicks these items should appear at the top of the rank because users only pay attention to the most 
relevant items at the top of the search results. This means we need to combine CTR with an additional element. 

```What would that be?```

### CPC (Cost-Per-Click) 
The amount that you pay for each click on your item. It's hard to discuss CPC without discussing 
campaigns and bidding but for this article let's just say there exists a campaign management system where content owners 
(job posting owners) can increase the amount they want to pay as they wish based on the applications/traffic they get to 
see on their job postings.  The other side of the CPC is RPC (Revenue Per Click) this is how search engines or ad 
publishers look at the amount that advertisers are paying for a click. 

### Snagajob search engine setup

At Snagajob, we were using Elastic Search as our search engine technology and on top of elastic search, we had Learning 
to Rank (LTR) for ranking optimization (Stay tuned for Learning to Rank article). In a nutshell, LTR is a machine learning 
technique that generates a function to predict the relative ranking of a document given query and the relevant documents. 
Before implementing the Yield Optimization Framework [Snagajob](http://www.snagajob.com) had the following setup: 

![Pre Yield Setup](/assets/preYieldSetup.png)

This is two-stage ranking setup where the BM25 algorithm and LTR in combination provide a rank list based on the 
documents and the query. But this ranking is not influenced by any of the above features such as ```CPC / RPC``` and CTR. 
Therefore we introduced Yield Optimization as the third stage ranker of the system which pays attention to CPC, CTR, 
and few other features of the document. 

![Post Yield Setup](/assets/postYieldSetup.png)

As a search engine, we need to make sure we balance the relevance while improving the revenue. Let's take the above 
features CTR, CPC and build a function to re-rank the search results that we obtain from the second stage of the search engine. 

To setup a ranking booster for the sponsored search I introduced the ranking score of an ad/job posting is equal to 
its bid multiplied by its estimated CTR, and introduce the following weighting scheme:

![Yield Optimization Function](/assets/yieldEQ.png)

In addition to ``CTR`` and ```RPC``` I also build couple of features based off of the document. Think of these as characteristics 
of the document as wells availability of the document - ```D1, D1```. 

*To be continued*