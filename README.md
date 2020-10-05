# Language Toxicity Classification
Classifying abusive language in online forums
___
>Many online content providers depend upon an engaged and active online community, oftentimes for the creation of content itself, and need ways to ensure their site is an attractive destination for repeat web traffic. One of the ways they often achieve this is through an engaging online forum. However, when abusive community members are allowed to make a forum feel toxic, many of the other members can be turned off by this behavior and begin to limit, if not completely eliminate, their level of engagement in the community. This can lead to a reduction in content, along with the inherent resulting revenue losses, and a reduction in forum vibrancy. This can potentially lead to an overall stale web experience, inspiring others to abstain as well.

I’ll be using data from Kaggle’s Toxic Comment Classification Challenge in which a research initiative founded by Jigsaw and Google, known as the Conversation AI team, held a competition to determine if negative online behaviors, specifically comments that are rude, disrespectful or otherwise likely to make someone leave a discussion, could be identified and removed, thus improving online conversations. The challenge is to designate six different categories of abusive comments so that users interested in moderating an online discussion forum could allow degrees, or types, of profanity/abuse while eliminating others. The six designations are *toxic*, *severe toxic*, *obscene*, *threat*, *insult*, and *identity hate*.

The `severe_toxic` designation is the only flag that's completely dependent upon the `toxic` designation. The other designations only have about 5.3-7.7% non-`toxic` correlated observations. Since all but the `severe_toxic` flag have separate observations from the `toxic` designation, we will create a new `abusive` column that will tally when any one of the six designations have been flagged. We will then plot the designations vs. overall abusiveness then vs. overall abusiveness and non-abusiveness.

![image](https://user-images.githubusercontent.com/42311832/95043000-cd0e6380-0698-11eb-8485-4067c7f55875.png)

There is significant overlap between the classes. With over ten thousand comments flagged as abusive, only the general `toxic` class has a significant amount of uniquely flagged comments, at 3567 observations, with the `threat` class having a mere 13! Also, unique flags account for only 4% of our total documents, at 3998 observations. I anticipate that such class overlap might make detecting noise unique to any particular class a bit problematic.

![image](https://user-images.githubusercontent.com/42311832/95043065-fc24d500-0698-11eb-993a-6ded1c86e8d7.png)

All the percentages vary widely for each category of abuse. Furthermore, non-abusive comments comprise almost 90 percent of our observations. Due to these differences, we might have to deal with class imbalance issues later during our modeling.
> + **Note**: It's important to remember that overlap exists between the abusive designations, hence why the sum of the abusive and non-abusive percentages don't add up to 100.

# Word Frequency Comparisons (Lemmatization)

We can examine which words might prove the most useful in distinguishing one class from another by comparing which words are both more unique and more frequent for each class. There are 15 possible comparisons we could make between the various classes, but below we just examine a few of the similar-sized classes, as well as the two 'toxic' classifications, to get aglimpse at which words might best define a class.

---

    Unique to Toxic: {'page', 'know', 'like', 'article', 'fat', 'pig', 'block', 'edit', 'hate'}
    Unique to Severe: {'yourselfgo', 'fucker', 'shut', 'bitch', 'rape', 'piece', 'damn', 'faggot', 'kill'}
    Common words by greater %:
     {'cunt': 'severe_toxic(1.74%)/toxic(0.58%)', 'nigger': 'severe_toxic(1.28%)/toxic(1.15%)', 'gay': 'severe_toxic(0.84%)/toxic(0.48%)', 'wikipedia': 'severe_toxic(1.15%)/toxic(0.84%)', 'shit': 'severe_toxic(3.31%)/toxic(1.14%)', 'suck': 'severe_toxic(3.58%)/toxic(1.05%)', 'die': 'severe_toxic(1.96%)/toxic(0.73%)', 'ass': 'severe_toxic(3.13%)/toxic(0.81%)', 'fucking': 'severe_toxic(1.46%)/toxic(0.72%)', 'fuck': 'severe_toxic(8.75%)/toxic(2.47%)', 'u': 'severe_toxic(2.51%)/toxic(0.75%)'}
 
 The largest disparities in word usage for the `toxic` and `severe_toxic` classes are for the f-word, with `severe_toxic` comprising 8.75% of it's non-stopword corpus while `toxic` is at a lower 2.47%. The s-word also boast a significant disparity at 3.31% and 1.14% for `severe_toxic` and `toxic` respectively.

---

    Unique to Threat: {'page', 'wales', 'talk', 'ban', 'fool', 'di', 'forever', 'live', 'jim', 'go', 'supertrll', 'block', 'murder', 'pathetic', 'edie'}
    Unique to Identity Hate: {'cunt', 'niggas', 'nigger', 'mexicans', 'gay', 'like', 'bunksteve', 'licker', 'spanish', 'shit', 'fat', 'suck', 'faggot', 'jew', 'hate'}
    Common words by greater %:
     {'die': 'threat(11.02%)/identity_hate(2.68%)', 'fucking': 'identity_hate(1.07%)/threat(0.98%)', 'ass': 'threat(7.86%)/identity_hate(0.66%)', 'fuck': 'identity_hate(2.74%)/threat(1.13%)', 'kill': 'threat(4.62%)/identity_hate(0.64%)'}

There's a large disparity in usage for the word 'die' favoring the `threat` class over the `identity_hate`, 11.02% to 2.68%. the word 'ass' and 'kill' follow next with disparities of 7.86% to 0.66% and 4.62% to 0.64%, both in favor of the `threat` class. There also appears to be more ethnic and racial slurs among the most popular `identity_hate`-related words than in the `threat`-related words.

---

    Unique to Obscene: {'block', 'bullshit'}
    Unique to insult: {'jew', 'moron'}
    Common words by greater %:
     {'know': 'insult(0.64%)/obscene(0.56%)', 'page': 'obscene(0.58%)/insult(0.57%)', 'cunt': 'insult(1.12%)/obscene(1.02%)', 'fuck': 'obscene(4.32%)/insult(3.63%)', 'like': 'insult(0.88%)/obscene(0.77%)', 'nigger': 'insult(1.94%)/obscene(1.74%)', 'wikipedia': 'obscene(0.82%)/insult(0.63%)', 'bitch': 'insult(0.82%)/obscene(0.74%)', 'shit': 'obscene(1.79%)/insult(0.9%)', 'suck': 'obscene(1.64%)/insult(1.63%)', 'fat': 'insult(0.98%)/obscene(0.84%)', 'fucking': 'insult(1.27%)/obscene(1.24%)', 'ass': 'obscene(1.4%)/insult(1.13%)', 'die': 'insult(0.79%)/obscene(0.66%)', 'dick': 'obscene(0.57%)/insult(0.54%)', 'u': 'insult(1.31%)/obscene(1.19%)', 'edit': 'obscene(0.59%)/insult(0.57%)', 'hate': 'insult(1.02%)/obscene(0.49%)'}
 
The `obscene` and `insult` most prevalent common words don't seem to have the same level of disparity as the previous comparisons, with perhaps the word 'hate' having the only somewhat significant disparity, with a prevalence favoring `insult`, 1.02%  to 0.49%. The F-word and S-word are the two words that favor the `obscene` class over the `insult` class, at 4.32% to 3.63% and 1.79% to 0.9%, repectively.

## Naive Bayes

    toxic:              
              precision    recall       f-1   support
              
           0       0.95      1.00      0.97     27156
           1       0.93      0.49      0.64      2844

    accuracy                           0.95     30000
    Class Balance: 0.905/0.095
    
    severe_toxic:
              precision    recall  f1-score   support
              
           0       0.99      1.00      0.99     29684
           1       0.62      0.10      0.17       316

    accuracy                           0.99     30000
    Class Balance: 0.989/0.011
    
    obscene:
              precision    recall  f1-score   support

           0       0.97      1.00      0.99     28422
           1       0.92      0.54      0.68      1578

    accuracy                           0.97     30000
    Class Balance: 0.947/0.053
    
    threat:
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29904
           1       0.00      0.00      0.00        96

    accuracy                           1.00     30000
    Class Balance: 0.997/0.003
    
    insult:
              precision    recall  f1-score   support

           0       0.97      1.00      0.98     28523
           1       0.82      0.44      0.57      1477

    accuracy                           0.97     30000
    Class Balance: 0.951/0.049
    
    identity_hate:
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29749
           1       0.60      0.05      0.09       251

    accuracy                           0.99     30000
    Class Balance: 0.992/0.008

With the `toxic` class being the predominant classification, we'll use it as the primary class for comparison between the various models, then we'll use the remaining classes for complimentary examination. Beginning with the Naive Bayes algorithm, we can see that the model improves accuracy to 93%, from the majority class baseline of 90%. So there is _some_ improvement, as this model would outperform a 'skill-less' model. It has a high precision but low recall, resulting in a low f1 score.

When we examine the other classes, there are even more stark class imbalances with majority classes as high as 99% in some cases. With these classes, accuracy scores are near meaningless. Therefore, we will be comparing f1 scores so that we can capture the precision and recall characteristics. The `toxic`, `obscene` and `insult` classes have more flagged observations, and thus appear to be capturikng more signal, judging from the f1 scores. The `severe_toxic`, `threat`  and `identity_hate` classes have so few flagged observations Naive Bayes doesn't seem to be capturing much signal with f1 scores of 0.17, 0.0 and 0.09, respectively.

  no. | model | algorithm |stopwords | pos | precision | recall | f1 | accuracy
 --- | --- | --- | --- |--- |--- |--- |--- 
0 | Logistic Regression Normalized | doc2vec | True | False | 0.81 | 0.54 | 0.65 | 0.94
 --- | --- | --- | --- |--- |--- |--- |--- 
1 | Logistic Regression Normalized | doc2vec | True | True | 0.8 | 0.52 | 0.63 | 0.94
 --- | --- | --- | --- |--- |--- |--- |--- 
 
 
 
 
 
2 | Logistic Regression Normalized | doc2vec	False	False	0.82	0.49	0.62	0.94
 --- | --- | --- | --- |--- |--- |--- |--- |
3 | Logistic Regression Normalized | doc2vec	False	True	0.81	0.46	0.59	0.94
 --- | --- | --- | --- |--- |--- |--- |--- |
4 | Logistic Regression non-Normalized | doc2vec	True	False	0.76	0.47	0.58	0.93
 --- | --- | --- | --- |--- |--- |--- |--- |
5 | Logistic Regression non-Normalized | doc2vec	True	True	0.75	0.43	0.55	0.93
 --- | --- | --- | --- |--- |--- |--- |--- |
6 | Logistic Regression non-Normalized | doc2vec	False	False	0.78	0.37	0.5	0.93
 --- | --- | --- | --- |--- |--- |--- |--- |
7 | Logistic Regression non-Normalized | doc2vec	False	True	0.75	0.34	0.47	0.93
 --- | --- | --- | --- |--- |--- |--- |--- |
8 | K-Means non-Normalized | doc2vec	False	True	0.1	0.95	0.18	0.16
 --- | --- | --- | --- |--- |--- |--- |--- |
9 | K-Means Normalized | doc2vec	True	True	0.1	0.53	0.16	0.47
 --- | --- | --- | --- |--- |--- |--- |--- |
10 | K-Means Normalized | doc2vec	False	True	0.1	0.52	0.16	0.47
 --- | --- | --- | --- |--- |--- |--- |--- |
11 | K-Means Normalized | doc2vec	False	False	0.1	0.41	0.16	0.59
 --- | --- | --- | --- |--- |--- |--- |--- |
12 | K-Means Normalized | doc2vec	True	False	0.09	0.45	0.15	0.52
 --- | --- | --- | --- |--- |--- |--- |--- |
13 | K-Means non-Normalized | doc2vec	True	False	0.07	0.41	0.11	0.57
 --- | --- | --- | --- |--- |--- |--- |--- |
14 | K-Means non-Normalized | doc2vec	True	True	0.06	0.23	0.08	0.69
 --- | --- | --- | --- |--- |--- |--- |--- |
15 | K-Means non-Normalized | doc2vec	False	False	0.06	0.04	0.05	0.84

Attempt | #1 | #2 | #3 | #4 | #5 | #6 | #7 | #8 | #9 | #10 | #11
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |--- |---
Seconds | 301 | 283 | 290 | 286 | 289 | 285 | 287 | 287 | 272 | 276 | 269
