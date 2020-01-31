---
layout: post
title:  "Lemmatization and POStagging of Ancient Greek Texts"
date:   2020-01-31 12:00:00
author: Vojtech
categories: Digital-pondering
published: true
---

### Introduction

In the second half of January, while I was working with Vojtěch Linka on a quantitative text analysis of the concept of pain in *Corpus Aristotelicum* and *Corpus Hippocraticum*, we realized that the data which I regularly use  for this type of tasks  (i.e. [Lemmatized Ancient Greek](https://github.com/gcelano/LemmatizedAncientGreekXML) dataset, LAG) are substantially incomplete. We noticed this fact once we were working with Aristotelian texts (tlg0086), as we realized that the dataset completely misses such works as *Ethica Nicomachea* and *Ethica Eudemia* - works really crucial for our task at hands!

This was quite surprising finding, since from the LAG documentation I had an impression that it combines  ***all***  Greek texts currently  available in the Perseus Digital Library with all texts from the First1KGreek project. However, this is not the case. First1KGreek explicitly focuses  upon "texts that do not already exist in the Perseus Digital Library" (see [here]([https://opengreekandlatin.github.io/First1KGreek/](https://opengreekandlatin.github.io/First1KGreek/))). But it appears that in the case of Aristotle it includes only works from the First1KGreek corpus, while there are some other Aristotle's works in the PerseusDL not included in First1KGreek . 

### Compiling a new dataset of all publicly available ancient Greek texts 

This led me to an urgent need to develop our own comprehensive dataset of ancient Greek text combining PerseusDL and First1KGreek. Just to combine these two dataset was quite straightforward. I developed my own code being able to parse xml files for these (or any similar) datasets directly from GitHub (i.e. without locally cleaning/downloaded the repo). 

My code works like this: For a chosen GitHub repository, it firstly recursively extracts paths for all files it contains (by browsing a repo folder by folder, using a loop over a PyGitHub package object). Subsequently, it uses the returned list of paths to parse all xml files.

### Lemmatization and POStagging using CLTK 

But what I really need is not only a dataset of all machine-readible texts, but a lemmatized and POStagged dataset, organized in a similar way as LAG (i.e. for each wordform containing its lemmatization and part of speech parsing).

To proceed with this task, I firstly turned to [CLTK](http://cltk.org) and its Greek module, since I knew that it includes a number of useful functions for both lemmatization and POStagging. However, after some testing, I realized that the results are unsufficient. 

For instance, once we apply the cltk function `lemmatizer.lemmatize()` upon all words in Aristotle's works and calculate word frequencies, we can identify there is more than 5,000 instances of the term ζεύς.  This is in striking contrast with TLG, which reports only 48 instances of the same lemma for Aristle. This discrepancy is  caused   by misidentification of the term διά , which in most instances  means simply "through". In this particular case, the problem might be solved by making a stopwords filtering before the lemmatization, but this is not an universal solution. 

Using cltk for POStagging revealed even more complicated. After doing some preliminary cleaning of all Aristotle work, the cltk greek tagger was still able to parse only 56% of words. Perhaps I could obtain a bit better results by some parametrization of these functions, but I guess the improvement would not be substantial. But first of all, it would require to make a much more detailed inspection of the  code behind these cltk functions, which is much more granular and complex than I am used to work with. 

### Lemmatization and POStagging with Morpheus

With these unsufficient results, I decided to develop my own functions for lemmatization, postagging and automatic translation. The main tool for this was the Morpheus Dictionary, available as a xml file for instance from [here](https://github.com/gcelano/LemmatizedAncientGreekXML/blob/master/Morpheus/MorpheusUnicode.xml.zip). This file contains 958,476 unique tokens. An entry for each token has the following structure: 

```xml
 <t>
    <i>1631</i> 			# word-form-id
    <f>ἀγακλέᾰς</f>		# word-form
    <b>αγακλεᾰς</b>		# word-form-without-diacritics
    <l>ἀγακλεής</l>		# lemma
    <e>αγακλεης</e>		# lemma-without-diacritics
    <p>a-p---ma-</p>	# morphological-analysis
    <d>76</d>					# lemma-id
    <s>very glorious, famous</s> 	# translation
    <a>doric aeolic</a> 					# dialect 
  </t>
```

I parsed this structure into two python dictionaries:

The first dictionary (`morpheus_dict`) has as its keys all unique word forms, both with and without diacritics. It has 706,506 key-value pairs. For each key, a value is a list of all possible tokens. In most cases, there is just one element in the list. Thus:

```
morpheus_dict["λόγος"]
>>> [{'a': None,
  'b': 'λογος',
  'd': '19869',
  'e': 'λογος',
  'f': 'λόγος',
  'i': '542404',
  'l': 'λόγος',
  'p': 'n-s---mn-',
  's': 'the word'}]
```

This simple navigation is then used in a series of functions drawing upon it.

The second dictionary (`morpheus_by_lemma`) is much more simple, since it contains only information associated with each individual lemma (and not every wordform). It is especially useful for getting a simple translation of a lemmatized word:

```
morpheus_by_lemma["λόγος"]
>>> [{'a': None, 'd': 19869, 'l': 'λόγος', 's': 'the word'}]
```

### Functions for lemmatization, POStagging and translation of ancient Greek

On the top is the function `get_lemmatized_sentences(string, all_lemmata=False, filter_by_postag=None, involve_unknown=False)`.

As an input, it receives a raw Greek text of any kind and extent.  Such input is  processed by a series of subsequent functions embedded within each other:

(1) `get_sentences()` splits the string into sentences by common sentence separators.

(2) `lemmatize_string(sentence)`  first calls `tokenize_string()`, which makes a basic cleaning and stopwords filtering for the sentence, and returns a list of words. Subsequently, each word from the tokenized sentence is sent either to `return_first_lemma()` or to `return_all_unique_lemmata()`, on the basis of the value of the parameter `all_lemmata=` (set to `False` by default). 

(4) `return_all_unique_lemmata()`goes to the `morpheus_dict` values and returns all unique lemmata.

(5) Parameter `filter_by_postag=` (default `None`) enables to sub-select  chosen word types from the tokens, on the basis of first character in the tag "p" . Thus, to choose only  nouns, adjectives, and verbs, you can set  `filter_by_postag=["n", "a", "v"].`

Next to the lemmatization, there is also a series of functions for translations, like `return_all_unique_translations(word, filter_by_postag=None, involve_unknown=False)`, useful for any wordform, and `lemma_translator(word)`, where we already have a lemma.

As a next step, I will put these functions into a package. Perhaps some of my functions could also be implemented into the cltk package. Hopefully I will soon update this post with a link to a repo containing its code and refs.





