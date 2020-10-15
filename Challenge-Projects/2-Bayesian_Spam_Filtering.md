# Challenge Project &ndash; Bayesian Spam Filtering

## Overview

This project will let you explore a useful application of Bayes' Rule: **e-mail spam filtering**. Along the way, you'll get some practice working with statistical machine
learning models.

By the end of this project you will be familiar with:

- The concept of a predictive classification model

- Formulating a classification model as a conditional probability problem

- Using Bayes' Rule to derive an expression that can be used for making predictions

- How the naïve Bayesian classifier works

- How to use naïve Bayes to classify texts

- Implementing a classifier using Python's scikit-learn library

## Spam Filtering as a Learning Problem

Suppose you want to build a system that filters unwanted e-mail or text messages ("spam") from legitimate messages (sometimes called "ham", though I hate saying that). You
could consider creating a **rule-based system**, which uses a series of conditional checks on each new message &ndash; content, grammar, sender's address, etc. &ndash; to determine
whether it should be accepted or rejected.

You would quickly run into a problem, though: coming up with a set of rules that's complex enough to identify spam messages but still reasonable to code is really hard. Such a
system would also be hard to maintain over time, because every change in spammers' behavior would require you to create and implement new sets of rules to correctly classify
the new messages.

Instead of trying to manually build a rule-based system, let's approach spam filtering as a **learning problem**. If we had a large data set of spam messages (which is not 
difficult to obtain) and a set of legitimate messages, we could **learn a model from the data** that succeeds at distinguishing between the two classes. Given a new 
message, our model should be able to predict, with high accuracy, whether that message is more likely to be spam or legitimate.

This formulation of spam filtering is an example of a **supervised learning problem**:

- There is a set of **classes**, which we'll denote as *C*. Here, the set of classes is *C* = {*spam*, *not spam*}. Many classification problems have this type of
yes/no or true/false structure, where there are two classes of interest, but it's possible to create models that differentiate between more than two classes.

- There is a **training data set**, consisting of labeled example messages for which we know the correct classification.

- There is a **model**, which is capable of discriminating between the two classes. Our goal is to **fit** the model using the given training data, so that it makes
accurate predictions on new data that will appear in the future. 

  Most machine learning models are like abstract and general pieces of applied statistics and linear algebra; 
every model can be used to solve many different types of classification tasks, although some models may be better suited for some problems that others. Examples
of popular machine learning models include the naïve Bayesian classifier, which we'll discuss in more detail below, deep learning neural networks, decision trees, and
support vector machines.

- There is usually a **testing data set**, which is used to evaluate the trained model. The testing data set consists of labeled examples with known classifications,
but isn't used in the training process. If the model is good, it should produce accurate results on the testing set. The testing set is used to ensure that the model
doesn't **overfit** the training data, which would result in a model that doesn't generalize well beyond the training examples.

## Bayesian Classification

### Classification as a Conditional Probability

Very good. We've now committed to learning a model using a training data set that can discriminate between spam and non-spam messages.

The Bayesian approach considers classification as a **probability problem**. Suppose we're considering a message *m*. We'd like to use the **words in *m*** as our **features**
to determine if *m* is spam or not. We could, potenially, expand our list of features to include things other than just the contents of the message, but we won't worry about 
that in these examples.

Intuitively, there are some words that are likely to occur in spam message but not in legitimate messages. I'll let you think about what some of those words are.

Consider two conditional probabilities:

```
P(m is spam | words in m)
```

```
P(m is NOT spam | words in m)
```

If we could calculate these two probabilities for the message *m*, we could classify *m* based on whichever probability is highest. For example, if our model deems it more 
likely that *m* is spam than not-spam, we should send *m* straight to the junk folder.

Here's the problem: **how do we calculate these two probabilities**?

To begin, let's think about the reversed conditonal probability:

```
P(words in m | spam)
```

This probability expresses the likelihood that a message picked from the universe of all spam messages contains the words we observed in message *m*. **We can estimate
this from the training data**. The training data contains a large number of example spam messages, so we can look at the words they contain and build a model that
estimates how likely a given message is if it's really spam.

Similarly, we can construct a model that estimates `P(words in m | not spam)` using the messages in the training data that we know are legitimate.

### Intuition

If you're not sure about this part, consider a message that contains the words **FREE HERBAL VIAGRA**. This is a popular topic for spammers, so if we examined the universe of 
all spam messages, we'd expect these words to show up frequently and we'd reasonably expect that `P(FREE HERBAL VIAGRA | spam)` is a relatively high probability. 
If we were to examine the universe of all legitimate non-spam messages, we would not expect to see **FREE HERBAL VIAGRA** show up frequently, so `P(FREE HERBAL VIAGRA | not spam)` should be small.

### Bayes' Rule

**This is the perfect setup for Bayes' Rule**. We have a conditional probability of interest that's hard to calculate directly, but we can reason about the reversed conditional
probability.

Unleash the math!

But first, let's simplify the notation a little bit. Let *c* denote a class of interest, either *spam* or *not spam* in this example. Let **m** denote the contents of the
message, where we're using bold notation to indicate that we're thinking about the contents as a vector of words.

Direct application of Bayes' Rules yields:

<img src="https://render.githubusercontent.com/render/math?math=P(c \, | \, \textbf{m}) = \frac{P(\textbf{m} \, |  \, c) P(c)}{P(\textbf{m})}" width="20%">

The left-hand side is the classification probability we're interested in: the probability of observing class *c* given the contents of the message. The right side
contains three terms.

- The first is the conditional probability we considered a moment ago: *P*(**m** | *c*), which we interpret as the probability of observing message **m** if it really belongs
to class *c*. In a moment, we'll talk about how to calculate these from the training data.

- *P*(*c*) is the unconditional probability of observing class *c*, independent of any information about the message. In our problem, this is the fraction all messages that
are spam or not spam. In Bayesian statistics, this is called the **prior** 
probability. If you have reason to believe that one class is more likely than another, the prior probability allows you to incorporate this information into the model.  

  For example, suppose that we believe 80% of all e-mail traffic is spam and only 20% is legitimate, which is consistent with research estimates. Using these probabilities for 
*P*(spam) and *P*(not spam) would have the effect of making it more likely for us to classify messages as spam and require stronger eveidence of legitimacy to mark a message
as non-spam.  

  In practice, we could use pre-existing evidence to set these values, estimate them from the training set, or assume that all classes are equally likely, which
is equivalent to assuming no prior evidence about the class distribution.

- The denominator, *P*(**m**), is the unconditional probability of observing a message with contents **m**, across the universe of all spam and non-spam messages. Notice:
this does not depend on *c*. Therefore, the value of *P*(**m**) will be the same for both classes, so **we can ignore it in our calculations**.

## Example

This is all pretty abstract, so let's look at how this plays out in a **small** example.

Suppose we have a universe of only four messages, two spam and two non-spam.

| Message contents           | Class label |
| -------------------------- | ----------- |
| watch free anime downloads | spam        |
| see you at the house       | not spam    |
| do you want takeout        | not spam    |
| sell your house now        | spam        |

Is the message "you want to watch anime at my house" more likely to be spam or not spam?

Using the Bayesian formulation, we need to calculate two probabilities:

```
P(spam | "you want to watch anime at my house")
```

```
P(not spam | "you want to watch anime at my house")
```

We'll take the larger probability to be the correct classification.

From the previous model, we know that

```
P(spam | "you want to watch anime at my house") = P("you want to watch anime at my house" | spam) P(spam)
```

```
P(not spam | "you want to watch anime at my house") = P("you want to watch anime at my house" | not spam) P(not spam)
```

These formulas are the numerator Bayes' Rule, where we've removed the denominator because it's the same for both classes.

### Prior Probabilities

First, let's consider the prior probabilities. Because we have an equal number of training examples in each class, we could reasonably decide that

```
P(spam) = P(not spam)
```

which means that the priors will not affect our classification decision, and can be dropped from further calculation. 
If we felt it was important to weight one class as more likely than the other, we could change the prior probabilities to do so.

### The Naïve Bayes Model

We now need to consider the likelihood of the message conditioned on each class, and to do it we're going to make a very strong simplifying assumption: **Assume that the likelihood of each word in a message is independent of all of the other words**. This is a strong assumption because it ignores all word context, sentence structure, and grammar.

If all of the words are independent, then the likelihood of the entire message is the product of the individual word likelihoods

```
P("you want to watch anime at my house" | spam) = P("you" | spam) * P("want" | spam) * P("to" | spam) * ... * P("house" | spam)
```

A Bayesian model that assumes independence of the features is called a **naïve Bayesian classifier**, because the assumption of independence is usually a huge simplification
of reality. Nonetheless, naïve Bayes has been shown to be effective in many real-world problems including text classification, so it's a standard technique in the field.

### Estimating Word Likelihoods

Estimating the likelihood of individual words is easy:

```
                  Number of times word occurs in all spam messages
P(word | spam) = --------------------------------------------------
                      Count of all words in all spam messages
``` 

For example, the word "anime" occurs one time in the set of spam messages and there are a total count of eight words in the entire spam data set, so we estimate

```
P("anime" | spam) = 1 / 8 = .125
```

Based on our data set, we expect that 12.5% of all words in spam message should be "anime".

There are two issues to consider before moving on the final calculations.

First, some words &dash; "a", "at", "the", "to", etc. &ndash; are so common they won't yield useful classification information. We can ignore these. More generally, we could
pre-filter all messages to focus on only a subset of key words that we think are useful for classification. This has the advantage of making our feature vectors smaller and
reducing irrelevant information in the model, at the risk that we choose to exclude something that would actually be useful.

The second issue is more complicated: what about words that don't appear in one of the data sets? For example, "anime" does not appear in the non-spam data set, but we don't
want to automatically conclude that `P("anime" | non-spam) = 0`, because that would imply it's impossible for me to receive a non-spam message about anime.

A typical solution to this problem is to assume that every word has some small constant probability of occurring, even if it was never observed in the training data set. Modify the word likelihood formula to be

```
                                  Number of times word occurs in all spam messages + 1
P(word | spam) = ---------------------------------------------------------------------------------------
                      Count of all words in all spam messages + Number of unique words in all messages

```

The numerator is now guaranteed to be at least 1, even a word does not appear in any messages. To compensate for this change, the denominator has been increased to
include the number of unique words in all messages (the **vocabulary** of the training set). Our training set contains 13 unique words after dropping "at", "the", and "do".

Under these new assumptions, we can calculate

```
                      1 + 1
P("anime" | spam) =  -------- ~ .095
                      8 + 13
```

The corresponding non-spam probability is

```
                           0 + 1
P("anime" | not spam) =  -------- ~ .053
                           6 + 13
```

The fancy name for this adjustment is **Laplace smoothing**.

### Results

Here is the table of word likelihoods calculated using the Laplace smoothing strategy.


| word | P(word \| spam)  | P(word \| not spam) |
|------| ----------------- | ------------------- |
| watch | .095 | .053 |
| free | .095 | .053 |
| anime | .095 | .053 |
| downloads | .095 | .053 |
| see | .0476 | .105 |
| you | .0476 | .158 |
| house | .095 | .105 |
| want | .0476 | .105 |
| takeout | .0476 | .105 |
| sell | .095 | .053 |
| your | .095 | .053 |
| now | .095 | .053 |
| my | .0476 | .053 |

For example, "you" appears two times in the non-spam messages and zero times in the spam messages. It's probabilities are therefore:

```
P("you" | non-spam) = (2 + 1) / (6 + 13) ~ .158
```

```
P("you" | spam) = (0 + 1) / (8 + 13) ~ .0476
```

The final step is to calculate the likelihood of the entire message "you want to watch anime at my house".

```
P("want to watch anime at my house" | spam) = P("you" | spam) * P("want" | spam) * P("watch" | spam) * P("anime" | spam) * P("my" | spam) * P("house" | spam)

                                            = .0476 * .0476 * .095 * .095 * .0476 * .095
                                            
                                            = 9.247e-8
```

The corresponding probability for the non-spam case is

```
P("want to watch anime at my house" | spam) = .158 * .105 * .053 * .053 * .053 * .105
                                           
                                            = 2.593e-7

```

Based on these results, we conclude that "you want to watch anime at my house" is **most likely not spam** because the non-spam case yields the higher probability.

Note that We can't truly interpret the output of these calculations as probabilities, because we dropped the denominator in the original Bayes' Rules formlation and 
the prior probability. For our purposes, what matters is simply the fact that one value is larger than the other one.
