---
layout: archive
title: Wise Widget
---



### Insight Data Science
As a fellow in the Insight Data Science program, I had the opportunity to create a project as a consultant for a local startup. This company provides tools, deployed as widgets on online news articles. The company strives to match the widgets to articles based on the content of news articles. Identifying content in text is a challenging process, and their existing method failed to confidently identify the article content about 25% of the time. In these cases a default widget is deployed, which is often entirely unrelated to the article content and therefore may not be relevant to the reader. This company asked me to help them improve the matching process between article content and widgets, and also to identify content areas that might represent opportunities for growth.

### Project Goals

1. Create a classification pipeline to match articles with widgets based on content relevance
2. Identify areas of potential growth for the creation of new widgets

### Scoping the Project

#### The Data: 

For this project I was given 60,000 URLs and corresponding text in a MySQL database. For every URL, I was able to identify which widget was deployed, and whether this was based on a content match or if it was a default deployment. Each widget corresponds to a content vertical, and currently the company addresses 10 content verticals.

#### Goal 1: 

To address the first goal of the project, I planned to employ supervised machine learning techniques. I decided to train a classifier on articles that had been matched to a widget. I then planned to apply the trained classifier on those articles that had been previously unable to be matched. The expectation in attempting to classify these unmatched articles is that a portion of articles will be able to be classified into the existing verticals, however another portion will not fit into the existing labeling schema. The latter portion of articles represents content verticals that are not currently addressed by the existing widgets and will become relevant in addressing my second goal. 

In order to improve on the company's current approach, it was important to carefully consider feature selection. The current matching process employed a bag of words approach, relying on counts of key words thought to be reflective of a content vertical. 

There are two limitations to the current approach
	* The mapping of key words to verticals is based on human assumption, and may not be entirely accurate or may neglect important information
	* In the case of this company, all of the content verticals are related and so there is crossover between key words and content verticals

Keeping these limitations in mind, I chose to use a topic modeling approach to reduce the dimensionality of the text data while maximizing the informational quality of the features.

[Topic modeling](https://www.cs.princeton.edu/~blei/papers/Blei2012.pdf "Topic Modeling Reference") is an alorithmic approach to identifying themes (topics) in a corpus of texts, which results in representing documents within the corpus as distributions of topics. Importantly, this approach does not assume that a given document only contains one content theme. In the case of news articles, any one article may touch on a number of topics (economics, politics, foreign affairs), but still may be qualitatively different from an article that contains overlapping topics. Thus, this approach lends itself well to the problem that I am trying to address.



#### Goal 2:

By addressing the content common to these articles, I will be able to address the second goal of identifying opportunities for growth.
