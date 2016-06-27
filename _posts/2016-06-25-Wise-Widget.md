---
layout: archive
author_profile: True
title: Wise Widget
---



### Insight Data Science

As a fellow in the Insight Data Science program, I had the opportunity to complete a three week consulting project for a local startup. This company provides tools that are deployed as widgets on online news articles. The company strives to match the widgets to articles based on the article content.

Identifying content in text is a challenging process, and the company’s existing method failed to confidently identify the article content about 25% of the time. In these cases, a default widget would be deployed, which is often entirely unrelated to the article content and therefore may not be relevant to the reader. 

This company asked me to help them improve the matching process between article content and widgets, and also to identify content areas that might represent opportunities for growth.

### Project Goals

1. Create a classification pipeline to match articles with widgets based on content relevance
2. Identify areas of potential growth for the creation of new widgets

### The Data: 

For this project I was given __62,500 urls__ and corresponding text in a MySQL database. For every url, I was able to identify which widget was deployed, and whether this was based on a content match or if it was a default deployment. Each widget corresponds to a content vertical, and currently the company addresses __10 content verticals__.

### Goal 1: 

To address the first goal of the project, I planned to employ supervised machine learning techniques. I decided to train a classifier on articles that had been matched to a widget. I then planned to apply the trained classifier on those articles that had been previously unable to be matched. The expectation in attempting to classify these unmatched articles is that a portion of articles will be able to be classified into the existing verticals, however another portion will not fit into the existing labeling schema. The latter portion of articles represents content verticals that are not currently addressed by the existing widgets and will become relevant in addressing my second goal. 

#### Feature Selection and Topic Modeling

In order to improve on the company's current approach, it was important to carefully consider feature selection. The current matching process relies on counts of key words thought to be reflective of a content vertical. 

There are two limitations to the current approach

* The mapping of key words to verticals is based on human assumption, and may not be entirely accurate or may neglect important information

* In the case of this company, all of the content verticals are related and so there is crossover between key words and content verticals

Keeping these limitations in mind, I chose to use a __topic modeling approach__, specifically __latent Dirichlet allocation (LDA)__, to reduce the dimensionality of the text data while maximizing the informational quality of the features.

[Topic modeling](https://www.cs.princeton.edu/~blei/papers/Blei2012.pdf "Topic Modeling Reference") is an algorithmic approach to identifying themes (topics) in a corpus of texts, which results in representing documents within the corpus as distributions of topics. Importantly, this approach does not assume that a given document contains only one content theme. In the case of news articles, any one article may touch on a number of topics (economics, politics, foreign affairs), but still may be qualitatively different from an article that contains overlapping topics. Thus, this approach lends itself well to the problem that I am trying to address. 


###### Conceptual Illustration of Topic Modeling using Latent Dirichlet Allocation

![Conceptual illustration of LDA]({{ emilyhalket.com }}/images/lda_illustration.png)

One challenge of this approach is that the number of topics is assumed to be a known parameter, and so the user must specify the number of topics to be used in the algorithm. There are a number of proposed techniques to address the choice of number of topics. For this project, I chose to vary the number of topics and investigate subsequent topic vocabularies and optimize classifier performance. 

To prepare the article text for topic modeling, I first needed to preprocess the raw text. Using python, I was able to create a preprocessing function to parse the words and remove punctuation and stop words.  I then used [nltk’s WordNetLemmatizer](http://www.nltk.org/api/nltk.stem.html “nltk documentation”) function to return the [lemma](https://en.wikipedia.org/wiki/Lemmatisation “lemmatization reference”) of each word. After preprocessing the text, I used [scikit-learn’s CountVectorizer](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html, “CountVectorizer”) function to convert the lemmatized words within each document into features for the LDA model. I then used [scikit-learn’s latent Dirichlet allocation](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html “sklearn LDA”) function to fit a topic model to my corpus of processed article text.

#### Classification Algorithm

I used the topic distributions for each article as features for the classification algorithm. I chose to train and test the classifier on those articles that had been matched to widgets using the company’s existing approach. By using these existing labels, I was able to make the classification problem a supervised learning problem. A limitation to this choice, however, is that it assumes that there is truth to the existing labeling schema. This is a potential area of improvement for future iterations of this project.

I divided the 45,000 matched articles into training (36,000) and testing (9,000) sets for cross-validation and tested three machine learning classifiers: [naïve bayes]( http://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.GaussianNB.html “naïve bayes”), [logistic regression]( http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html “logistic regression”), and [random forest]( http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html “random forest”). 

I investigated the [precision and recall measures](https://en.wikipedia.org/wiki/Precision_and_recall “precision and recall reference”) for each labelling vertical across the three classification methods and ultimately chose __random forest classification__ due to its superior performance. To avoid overfitting, I limited the tree depth and increased the number of estimators for my random forest algorithm.

I then applied the trained and validated classifier to those articles that were not able to be matched to a widget using the company’s approach. I calculated the probability of classification for each of the 17,500 unmatched articles and selected subsets of these articles with the highest and lowest classification probability. For those articles with the highest probability of classification, I investigated the assigned labeling with respects to the article content as a quality check for the classifier performance. This was a subjective assessment, and to investigate the performance in a more objective manner, I suggest that the company perform user engagement testing to better quantify the performance of my classifier. Focusing on those articles with the lowest classification probability, I addressed the company’s second goal.


### Goal 2:

The second goal of the project was to identify content areas not currently represented in the company’s existing subject verticals, and thus areas for potential growth. Because of the unknown nature of the uncovered content, this goal posed an unsupervised learning problem. Given the limited timeline of the project, I chose to focus my efforts on the unmatched articles that had the lowest probability of classification. Investigating the content of the articles with the lowest probability of classification allowed me to focus on those articles with the lowest certainty of fitting within the existing content verticals and thus with the highest likelihood of representing new subject verticals. 

#### Clustering Unmatched Content

I chose to apply __k-means clustering__ to the topic distributions of approximately 4,000 articles. Varying the number of clusters and examining the subsequent cluster distributions, I ultimately divided the articles into six clusters. In this approach, each cluster represented a content theme common to a subset of articles. I then investigated articles appearing in each cluster to give a qualitative label to each cluster. 


##### Visualization of K-Means Clustering Output
![Visualization of KMeans Clustering]({{ emilyhalket.com }}/images/cluster_image_06262016.png)

Results from the k-means clustering led me to make three suggestions to the company.

* First, two somewhat overlapping clusters (purple and dark blue) represented content related to existing subject verticals, however the style of these articles was qualitatively different than those related articles that were able to be matched to the subject verticals. As opposed to being written in the style of a typical news article, these articles were in a Q&A format. These clusters represented an __area of improvement__ for the classification method. I suggested that the company chose a subset of these articles to label with the appropriate widget and vertical using the existing labeling schema and then retrain the classifier.

* Second, the large cluster (light blue) represented a content area not well represented by the company’s existing set of widgets. There is currently one widget assigned to this vertical, but the size of the cluster suggests that there is __potential for growth__ in this subject vertical. Additionally, the classification failed to confidently assign articles within this cluster to the appropriate cluster, thus suggesting another __area for improvement__. I again suggested that the company label a subset of these articles with the appropriate vertical and retrain the classifier. 

* Finally, there was one cluster (pink) that represented a content theme not addressed by the existing subject verticals. This cluster represented an __area for growth and potential new market__ for the company. 



### Conclusions:

For this consulting project, I set out to help the company improve their article-widget matching algorithm based on content relevance and to identify potential areas for growth. I created a reproducible topic modeling and classification pipeline for the company to incorporate into their existing system. This pipeline represents each article as a distribution of __75 topics__ and these topic weights are used as features for __random forest classification__. Additionally, I identified areas of improvement within existing content verticals as well as within potential for growth in new verticals using __k-means clustering__. 

As the company addresses the areas of improvement identified in my cluster analysis as well as future goals, the topic model and classifier can be adapted, thus improving performance. Given that the overall goal of the company is to drive user engagement with the widgets, I suggested that the company __investigate user engagement in a side by side comparison of my classification model with their existing method to quantify the model performance__. 




