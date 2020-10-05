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

> Unique to Toxic: {'page', 'know', 'like', 'article', 'fat', 'pig', 'block', 'edit', 'hate'}

> Unique to Severe: {'yourselfgo', 'fucker', 'shut', 'bitch', 'rape', 'piece', 'damn', 'faggot', 'kill'}

> Common words by greater %:
 {'cunt': 'severe_toxic(1.74%)/toxic(0.58%)', 'nigger': 'severe_toxic(1.28%)/toxic(1.15%)', 'gay': 'severe_toxic(0.84%)/toxic(0.48%)', 'wikipedia': 'severe_toxic(1.15%)/toxic(0.84%)', 'shit': 'severe_toxic(3.31%)/toxic(1.14%)', 'suck': 'severe_toxic(3.58%)/toxic(1.05%)', 'die': 'severe_toxic(1.96%)/toxic(0.73%)', 'ass': 'severe_toxic(3.13%)/toxic(0.81%)', 'fucking': 'severe_toxic(1.46%)/toxic(0.72%)', 'fuck': 'severe_toxic(8.75%)/toxic(2.47%)', 'u': 'severe_toxic(2.51%)/toxic(0.75%)'}
