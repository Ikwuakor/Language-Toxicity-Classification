# Language Toxicity Classification
Classifying abusive language in online forums

<div>
    <img style="vertical-align:middle" src="https://user-images.githubusercontent.com/42311832/95642493-2514e380-0a66-11eb-9d21-e2761dac4546.png" alt="warning sign" width="80" height="80" >
    <span style=""> Warning: the following project contains language that some might find offensive.</span>
</div>
___
> Many online content providers depend upon an engaged and active online community, oftentimes for the creation of content itself, and need ways to ensure their site is an attractive destination for repeat web traffic. One of the ways they often achieve this is through an engaging online forum. However, when abusive community members are allowed to make a forum feel toxic, many of the other members can be turned off by this behavior and begin to limit, if not completely eliminate, their level of engagement in the community. This can lead to a reduction in content, along with the inherent resulting revenue losses, and a reduction in forum vibrancy. This can potentially lead to an overall stale web experience, inspiring others to abstain as well.

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
    ----------------------------------------------------------------------------------------------------
    severe_toxic:
              precision    recall  f1-score   support
              
           0       0.99      1.00      0.99     29684
           1       0.62      0.10      0.17       316

    accuracy                           0.99     30000
    Class Balance: 0.989/0.011
    ----------------------------------------------------------------------------------------------------
    obscene:
              precision    recall  f1-score   support

           0       0.97      1.00      0.99     28422
           1       0.92      0.54      0.68      1578

    accuracy                           0.97     30000
    Class Balance: 0.947/0.053
    ----------------------------------------------------------------------------------------------------
    threat:
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29904
           1       0.00      0.00      0.00        96

    accuracy                           1.00     30000
    Class Balance: 0.997/0.003
    ----------------------------------------------------------------------------------------------------
    insult:
              precision    recall  f1-score   support

           0       0.97      1.00      0.98     28523
           1       0.82      0.44      0.57      1477

    accuracy                           0.97     30000
    Class Balance: 0.951/0.049
    ----------------------------------------------------------------------------------------------------
    identity_hate:
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29749
           1       0.60      0.05      0.09       251

    accuracy                           0.99     30000
    Class Balance: 0.992/0.008

With the `toxic` class being the predominant classification, we'll use it as the primary class for comparison between the various models, then we'll use the remaining classes for complimentary examination. Beginning with the Naive Bayes algorithm, we can see that the model improves accuracy to 93%, from the majority class baseline of 90%. So there is _some_ improvement, as this model would outperform a 'skill-less' model. It has a high precision but low recall, resulting in a low f1 score.

When we examine the other classes, there are even more stark class imbalances with majority classes as high as 99% in some cases. With these classes, accuracy scores are near meaningless. Therefore, we will be comparing f1 scores so that we can capture the precision and recall characteristics. The `toxic`, `obscene` and `insult` classes have more flagged observations, and thus appear to be capturikng more signal, judging from the f1 scores. The `severe_toxic`, `threat`  and `identity_hate` classes have so few flagged observations Naive Bayes doesn't seem to be capturing much signal with f1 scores of 0.17, 0.0 and 0.09, respectively.

  no. | model | algorithm |stopwords | pos | precision | recall | f1 | accuracy
 --- | --- | --- | --- |--- |--- |--- |--- | ---
0 | Logistic Regression Normalized | doc2vec | True | False | 0.81 | 0.54 | 0.65 | 0.94
1 | Logistic Regression Normalized | doc2vec | True | True | 0.8 | 0.52 | 0.63 | 0.94
2 | Logistic Regression Normalized | doc2vec | False | False | 0.82 | 0.49 | 0.62 | 0.94
3 | Logistic Regression Normalized | doc2vec | False | True | 0.81 | 0.46 | 0.59 | 0.94
4 | Logistic Regression non-Normalized | doc2vec | True | False | 0.76 | 0.47 | 0.58 | 0.93
5 | Logistic Regression non-Normalized | doc2vec | True | True | 0.75 | 0.43 | 0.55 | 0.93
6 | Logistic Regression non-Normalized | doc2vec | False | False | 0.78 | 0.37 | 0.5 | 0.93
7 | Logistic Regression non-Normalized | doc2vec | False | True | 0.75 | 0.34 | 0.47 | 0.93
8 | K-Means non-Normalized | doc2vec | False | True | 0.1 | 0.95 | 0.18 | 0.16
9 | K-Means Normalized | doc2vec | True | True | 0.1 | 0.53 | 0.16 | 0.47
10 | K-Means Normalized | doc2vec | False | True | 0.1 | 0.52 | 0.16 | 0.47
11 | K-Means Normalized | doc2vec | False | False | 0.1 | 0.41 | 0.16 | 0.59
12 | K-Means Normalized | doc2vec | True | False | 0.09 | 0.45 | 0.15 | 0.52
13 | K-Means non-Normalized | doc2vec | True | False | 0.07 | 0.41 | 0.11 | 0.57
14 | K-Means non-Normalized | doc2vec | True | True | 0.06 | 0.23 | 0.08 | 0.69
15 | K-Means non-Normalized | doc2vec | False | False | 0.06 | 0.04 | 0.05 | 0.84

Clearly the Normalized, Logistic Regression is superior to the K-means classifier and slightly edge out the non-Normalized Logistic Regression. The inclusion of stopwords is a clear boost in sentiment retention. The inclusion of parts of speech (pos) seems to underperform the exclusion of parts of speech, albeit not by much. The Logistic Regression, Normalized	doc2vec	model, with stopwords=True and pos=False, was the top scorer in all categories except recall, where it scored second, and precision where it score a close second. Next, we'll examine whether lemmas can provide additional sentiment retention, keeping stopwords throughout and, again, comparing with and without parts of speech.

no. | model | algorithm | stopwords | pos | precision | recall | f1 | accuracy
 --- | --- | --- | --- |--- |--- |--- |--- | ---
0 | Lemmatized Log Reg Normalized | doc2vec | True | False | 0.82 | 0.55 | 0.66 | 0.94
1 | Logistic Regression Normalized | doc2vec | True | False | 0.81 | 0.54 | 0.65 | 0.94
2 | Logistic Regression Normalized | doc2vec | True | True | 0.8 | 0.52 | 0.63 | 0.94
3 | Lemmatized Log Reg Normalized | doc2vec | True | True | 0.81 | 0.52 | 0.63 | 0.94
4 | Logistic Regression Normalized | doc2vec | False | False | 0.82 | 0.49 | 0.62 | 0.94

Lemmatization has earned slight improvements for f-1, recall and precision scores while maintaining its accuracy score. Again, the exclusion of parts of speech tags slightly underperformed the model with its inclusion. We'll now run a Logistic Regression random search to tune hyperparameters.

no. | C | max_iter | class_weight | solver | d2v_params | train_precision | test_precision | train_recall | test_recall | train_acc | test_acc | train_f1 | test_f1
 --- | --- | --- | --- |--- |--- |--- |--- | ---| --- |--- |--- |--- |--- 
0 | 1000 | 1000 | nan | sag | {'dm': 0, 'vector_size': 300, 'window': 3, 'dm_mean': 1, 'iter': 50} | 0.830483 | 0.820856 | 0.660165 | 0.647679 | 0.954114 | 0.9532 | 0.735594 | 0.724057
1 | 5 | 2000 | nan | sag | {'dm': 0, 'vector_size': 300, 'window': 3, 'dm_mean': 1, 'iter': 50} | 0.835995 | 0.828204 | 0.657506 | 0.633966 | 0.954414 | 0.952833 | 0.736085 | 0.718184
2 | 5 | 2000 | nan | saga | {'dm': 0, 'vector_size': 300, 'window': 3, 'dm_mean': 1, 'iter': 50} | 0.835995 | 0.828204 | 0.657506 | 0.633966 | 0.954414 | 0.952833 | 0.736085 | 0.718184
3 | 100000 | 2000 | nan | newton-cg | {'dm': 0, 'vector_size': 300, 'window': 3, 'dm_mean': 1, 'iter': 50} | 0.832091 | 0.822808 | 0.659722 | 0.636779 | 0.954229 | 0.952567 | 0.735949 | 0.717939
4 | 1 | 3000 | nan | sag | {'dm': 0, 'vector_size': 300, 'window': 3, 'dm_mean': 1, 'iter': 50} | 0.843574 | 0.835905 | 0.643026 | 0.628692 | 0.953957 | 0.9531 | 0.729773 | 0.71764

# The Best Doc2vec Recipe

> + Lemmatized documents, normalized vectors
> + Doc2Vec(dm=0, vector_size=300, window=3, dm_mean=1, iter=50)
> + LogisticRegression(C=1000, max_iter=1000, class_weight=None, solver='sag')

With the above pipeline, we've managed to maximize our preferred criteria, `test_f1`. With a test accuracy of 0.9532, we've captured over 50% of the remaining accuracy that was up for grabs when given a baseline accuracy of nearly 90% for all non-abusive comments. Our recall score represents our minority class, and we are capturing almost 65% of the `toxic` comments with over 82% of the observations predicted to be `toxic` actually being toxic.

We now extend our optimal model to all of the classes.

    Log_loss score for toxic is -0.12608957538569543
              precision    recall  f1-score   support

           0       0.96      0.99      0.97     27156
           1       0.84      0.63      0.72      2844

    accuracy                           0.95     30000
    macro avg       0.90      0.81      0.85     30000
    weighted avg       0.95      0.95      0.95     30000
    ----------------------------------------------------------------------------------------------------
    Log_loss score for severe_toxic is -0.027014954122679874
              precision    recall  f1-score   support

           0       0.99      1.00      0.99     29684
           1       0.49      0.17      0.25       316

    accuracy                           0.99     30000
    macro avg       0.74      0.58      0.62     30000
    weighted avg       0.99      0.99      0.99     30000
    ----------------------------------------------------------------------------------------------------
    Log_loss score for obscene is -0.06876860305325452
              precision    recall  f1-score   support

           0       0.98      0.99      0.99     28422
           1       0.85      0.66      0.74      1578

    accuracy                           0.98     30000
    macro avg       0.92      0.83      0.86     30000
    weighted avg       0.97      0.98      0.97     30000
    ----------------------------------------------------------------------------------------------------
    Log_loss score for threat is -0.010428913922936826
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29904
           1       0.90      0.09      0.17        96

    accuracy                           1.00     30000
    macro avg       0.95      0.55      0.58     30000
    weighted avg       1.00      1.00      1.00     30000
    ----------------------------------------------------------------------------------------------------
    Log_loss score for insult is -0.08016496842596106
              precision    recall  f1-score   support

           0       0.98      0.99      0.98     28523
           1       0.76      0.55      0.64      1477

    accuracy                           0.97     30000
    macro avg       0.87      0.77      0.81     30000
    weighted avg       0.97      0.97      0.97     30000
    ----------------------------------------------------------------------------------------------------
    Log_loss score for identity_hate is -0.025609311483858456
               precision    recall  f1-score   support

           0       0.99      1.00      1.00     29749
           1       0.70      0.15      0.24       251

    accuracy                           0.99     30000
    macro avg       0.85      0.57      0.62     30000
    weighted avg       0.99      0.99      0.99     30000

Although still low in some cases, the f1 score for the classes, other than `toxic`, have made improvements with the Doc2vec model:

+ toxic: 0.72 (up from 0.64 for Naive Bayes)
+ severe_toxic: 0.25 (up from 0.17)
+ obscene: 0.74 (up from 0.68)
+ threat: 0.17 (up from 0.00)
+ insult: 0.64 (up from 0.57)
+ identity_hate: 0.24 (up from 0.09)


# TF-IDF

> TfidfVectorizer(analyzer='word', binary=False, decode_error='strict',
                    dtype=<class 'numpy.float64'>, encoding='utf-8',
                input='content', lowercase=True, max_df=0.9, max_features=None,
                min_df=100, ngram_range=(1, 1), norm='l2', preprocessor=None,
                smooth_idf=True, stop_words='english', strip_accents=None,
                sublinear_tf=True, token_pattern='\\w{2,}', tokenizer=None,
                use_idf=True, vocabulary=None)

no. | C | max_iter | class_weight | solver | train_precision | test_precision | train_recall | test_recall | train_acc | test_acc | train_f1 | test_f1
 --- | --- | --- | --- |--- |--- |--- |--- | ---| --- |--- |--- |--- 
25 | 5 | 3000 | None | sag | 0.909903 | 0.872602 | 0.66253 | 0.623769 | 0.961029 | 0.9557 | 0.766758 | 0.727496
6 | 1000 | 1000 | None | lbfgs | 0.896705 | 0.795181 | 0.715721 | 0.649789 | 0.964543 | 0.950933 | 0.796056 | 0.71517
7 | 1000 | 1000 | None | saga | 0.896501 | 0.795181 | 0.715426 | 0.649789 | 0.9645 | 0.950933 | 0.795793 | 0.71517
11 | 1000 | 3000 | None | sag | 0.896705 | 0.795181 | 0.715721 | 0.649789 | 0.964543 | 0.950933 | 0.796056 | 0.71517
26 | 100000 | 3000 | None | saga | 0.896456 | 0.79006	0.717642 | 0.648383 | 0.964686 | 0.950333 | 0.797144 | 0.712244

    Log_loss score for toxic is 1.5335290548836404
              precision    recall  f1-score   support
              
           0       0.96      0.99      0.98     27161
           1       0.87      0.63      0.73      2839

    accuracy                           0.96     30000
    Class Balance: 0.905/0.095
    ----------------------------------------------------------------------------------------------------
    Log_loss score for severe_toxic is 0.3258173632002711
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29698
           1       0.57      0.26      0.36       302

    accuracy                           0.99     30000
    Class Balance: 0.99/0.01
    ----------------------------------------------------------------------------------------------------
    Log_loss score for obscene is 0.7897900452061652
              precision    recall  f1-score   support

           0       0.98      1.00      0.99     28400
           1       0.89      0.65      0.75      1600

    accuracy                           0.98     30000
    Class Balance: 0.947/0.053
    ----------------------------------------------------------------------------------------------------
    Log_loss score for threat is 0.1082218192096932
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29898
           1       0.62      0.20      0.30       102

    accuracy                           1.00     30000
    Class Balance: 0.997/0.003
    ----------------------------------------------------------------------------------------------------
    Log_loss score for insult is 1.070707851996977
              precision    recall  f1-score   support

           0       0.98      0.99      0.98     28527
           1       0.78      0.52      0.62      1473

    accuracy                           0.97     30000
    Class Balance: 0.951/0.049
    ----------------------------------------------------------------------------------------------------
    Log_loss score for identity_hate is 0.27515971821021956
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29734
           1       0.66      0.21      0.32       266

    accuracy                           0.99     30000
    Class Balance: 0.991/0.009

# The Best TF-IDF Recipe
> + TfidfVectorizer(max_df=0.9, min_df=100, ngram_range=(1, 1), smooth_idf=True, token_pattern='\\w{2,}', use_idf=True)
> + LogisticRegression(C=5, class_weight=None, max_iter=3000, solver='sag')

With the above pipeline, we've managed to maximize our preferred criteria, test_f1. The test accuracy has increased to 96%, capturing nearly 60% of the remaining accuracy above the `toxic` baseline of 90%. Our recall scores have fallen slightly to 63% but the precision has risen to 87% for a comparable f1 score of 0.73. Again, we'll compare TF-IDF with the Doc2vec model:
+ toxic: 0.73 (up from 0.72 for Doc2vec/Logistic Regression)
+ severe_toxic: 0.30 (up from 0.25)
+ obscene: 0.77 (up from 0.74)
+ threat: 0.25 ( up from 0.17)
+ insult: 0.64 (same as 0.64 for Doc2vec/Logistic Regression)
+ identity_hate: 0.35 (up from 0.24)

Our TF-IDF model has matched, or outperformed, Doc2vec with every class.

## Targeted Features

With the following model, we will create features, using the most common words flagged for each class.

    toxic:
              precision    recall  f1-score   support

           0       0.96      0.99      0.98     27156
           1       0.87      0.64      0.74      2844

    accuracy                           0.96     30000
    Class Balance: 0.905/0.095
    Execution time: 31.41 seconds
    ----------------------------------------------------------------------------------------------------
    severe_toxic:
              precision    recall  f1-score   support

           0       0.99      1.00      0.99     29684
           1       0.48      0.16      0.24       316

    accuracy                           0.99     30000
    Class Balance: 0.989/0.011
    Execution time: 34.91 seconds
    ----------------------------------------------------------------------------------------------------
    obscene:
              precision    recall  f1-score   support

           0       0.98      0.99      0.99     28422
           1       0.88      0.69      0.78      1578

    accuracy                           0.98     30000
    Class Balance: 0.947/0.053
    Execution time: 29.06 seconds
    ----------------------------------------------------------------------------------------------------
    threat:
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29904
           1       0.24      0.05      0.09        96

    accuracy                           1.00     30000
    Class Balance: 0.997/0.003
    Execution time: 26.4 seconds
    ----------------------------------------------------------------------------------------------------
    insult:
              precision    recall  f1-score   support

           0       0.98      0.99      0.98     28523
           1       0.78      0.55      0.64      1477

    accuracy                           0.97     30000
    Class Balance: 0.951/0.049
    Execution time: 34.38 seconds
    ----------------------------------------------------------------------------------------------------
    identity_hate:
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29749
           1       0.51      0.16      0.25       251

    accuracy                           0.99     30000
    Class Balance: 0.992/0.008
    Execution time: 28.93 seconds

# The Best (TFIDF, Targeted Features) Recipe
> + TfidfVectorizer(max_df=0.9, min_df=100, use_idf=True, token_pattern=r'\w{2,}', sublinear_tf=True, vocabulary=bows)
+ LogisticRegression(C=5, class_weight=None, max_iter=3000, solver='sag', random_state=0)

We now compare the TF-IDF model with algorithmically generated features to our TF-IDF, target features model:

+ toxic: 0.74 (up from 0.73 for TF-IDF)
+ severe_toxic: 0.24 (DOWN from 0.30)
+ obscene: 0.78 (up from 0.77)
+ threat: 0.09 (DOWN from 0.25)
+ insult: 0.64 (same as 0.64 for TF-IDF)
+ identity_hate: 0.25 (DOWN from 0.35)

The targeted TF-IDF model has performed worse than the 'out-of-the-box' TF-IDF model.

## Bigrams

> TfidfVectorizer(analyzer='word', binary=False, decode_error='strict',
                dtype=<class 'numpy.float64'>, encoding='utf-8',
                input='content', lowercase=True, max_df=0.9, max_features=3000,
                min_df=2, **ngram_range=(1, 2)**, norm='l2', preprocessor=None,
                smooth_idf=True, stop_words='english', strip_accents=None,
                sublinear_tf=True, token_pattern='\\w{2,}', tokenizer=None,
                use_idf=True, vocabulary=None)

    Log_loss score for toxic is 1.586488565328999
              precision    recall  f1-score   support

           0       0.96      0.99      0.98     27161
           1       0.86      0.61      0.72      2839

    accuracy                           0.95     30000
    Class Balance: 0.905/0.095
    ----------------------------------------------------------------------------------------------------
    Log_loss score for severe_toxic is 0.3154554904025688
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29698
           1       0.61      0.26      0.36       302

    accuracy                           0.99     30000
    Class Balance: 0.99/0.01
    ----------------------------------------------------------------------------------------------------
    Log_loss score for obscene is 0.8013031305906215
              precision    recall  f1-score   support

           0       0.98      1.00      0.99     28400
           1       0.89      0.65      0.75      1600

    accuracy                           0.98     30000
    Class Balance: 0.947/0.053
    ----------------------------------------------------------------------------------------------------
    Log_loss score for threat is 0.11052440430268726
              precision    recall  f1-score   support

           0       1.00      1.00      1.00     29898
           1       0.60      0.18      0.27       102

    accuracy                           1.00     30000
    Class Balance: 0.997/0.003
    ----------------------------------------------------------------------------------------------------
    Log_loss score for insult is 1.084523549127675
              precision    recall  f1-score   support

           0       0.98      0.99      0.98     28527
           1       0.77      0.51      0.62      1473

    accuracy                           0.97     30000
    Class Balance: 0.951/0.049
    ----------------------------------------------------------------------------------------------------
    Log_loss score for identity_hate is 0.28552140443518814
              precision    recall  f1-score   support

           0       0.99      1.00      1.00     29734
           1       0.61      0.19      0.29       266

    accuracy                           0.99     30000
    Class Balance: 0.991/0.009

# The Best Bigrams, TF-IDF Recipe
> + TfidfVectorizer(max_df=0.9, min_df=100, use_idf=True, token_pattern=r'\w{2,}', sublinear_tf=True, vocabulary=bows)
+ LogisticRegression(C=5, class_weight=None, max_iter=3000, solver='sag', random_state=0)

We now compare the TF-IDF model with algorithmically generated features to our TF-IDF, target features model:

+ toxic: 0.72 (DOWN from 0.73 for TF-IDF)
+ severe_toxic: 0.36 (up from 0.30)
+ obscene: 0.75 (DOWN from 0.77)
+ threat: 0.27 (up from 0.25)
+ insult: 0.62 (DOWN from 0.64)
+ identity_hate: 0.29 (DOWN from 0.35)

The bigrams TF-IDF model has performed worse than the 'out-of-the-box' TF-IDF model for four out of six classes, including the primary class of `toxic`.

# Conclusion
> Without a clear set of directives determining what makes a comment fall into one, or more, of the abusive categories, it's hard to gauge whether we are capturing a set of hard and fast rules with which these comments were originally classified or if we're simply back-engineering the subjective biases of the humans who originally flagged these comments. Are these moderator flagged comments? Community flagged comments? Do some of the flagged comments become toxic only after they borrow context from other comments they were replying to; comments which may have avoided being flagged themselves?

Some of the **false positives** in our model seem to convey language that, given a certain context, could most certainly be classified as toxic.

    --> yo mama yo mama yo mama 

    --> bite me irishguy contextflexed 

    --> the block page says i should add this to my page so there it goes someone fix this and get these power corrupt idiots to stop stalking me

They typically tend to convey dissatisfaction of some kind, with some of them even getting away with use of bad words, albeit not always in a manner meant to insult.

    --> nan bread is a b**** to make how do you do it i can make chicken marsala but not the indan breads 

    --> thanks sorry i didn't know you weren't allowed to write things like that on articles my bad c***

Some of the **false negatives** have contextually harsh language, but which contain words that, on their own, don't necessarily convey toxicity.

    --> baby eaters alright is it true that some of the band's songs translate into lyrics like i'm going to cut open your fetus and eat your baby or other horrific lyrics 

    --> you love the devil and worship him

Other false negatives seem to convey mild dissatisfaction, but nothing that seems obviously toxic. Perhaps some of these abusive comments borrow context from the comments they were replying to? If so, that makes classification that much trickier.

    --> whats the deal you can put crap on my talk page but i cant put crap on yours you're real cool dude

Perhaps this one escaped detection by our model due to a convenient typo.

    --> go and complain me again for personal attack looser i do not care


In the end, whatever it is we're capturing, the TF-IDF, with algorithm generated features, outperformed all our other models across all models (though slightly outperformed by targeted BOWS in the `toxic` class. It also requires the least amount of text modification (no lemmatization) and supplementary functions. With it being the quickest, and easiest model to run, it is the clear winner, performing as well, or better than any of the other models, and in each and every class. Unfortunately, none of the models managed to post impressive results across the board, most likely due to some of the classes having too few instances with which to train a model adequately. A possible solution might be to allow content providers to manually add words to the feature set that they want to trigger an automatic comment flag, either for automatic removal or for human review.

### To see the full notebook, complete with my code, where I created my random search algorithms, and a demo where you can score user input text, click <a href="https://github.com/Ikwuakor/Language-Toxicity-Classification/blob/main/Language%20Toxicity%20Classification.ipynb">here</href>.
