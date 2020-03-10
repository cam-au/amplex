---
layout: post
title:  "First peak at multigraph"
date:   2020-03-10 12:00:00
author: Adela
categories: Digital-pondering
published: true
---

### Climbing the Mountain


Some while ago, I saw [this comic](https://devhumor.com/media/trying-to-learn-any-programming-language-100) about the never-ending story of learning programming. The never-ending story applies to any collaborative discovery endeavour. At SDAM, we have been agonizing over what can we actually do with all the large data, once we aggregated it. It’s a question that’s been giving us anxiety, because until you harvest, streamline and wrangle your data, you cannot even start answering it. Just like a potter who collects clay from an unknown source without knowing whether it will be good for anything. You knead and levigate the cold, hard mass and only as it warms up and softens with your effort, you may gauge its qualities.  Is it plastic and malleable enough to make a small sculpture with fine detail or coarse and bulky to make a big chunky dinosaur?  Playing with large datasets is similar. You spend hours poking and probing its features in order to learn what it can be used for and how much effort it’ll likely take.    
So far we have been following Vojtech’s python scripts in slicing and subsetting the EDH dataset and discovering what problems we need to overcome. The choice is wide; there are the incomparable diversity of chronological intervals ascribed to individual inscriptions, that we will need to refine. While working on these preliminaries, we have all been anxious about what will we do next, once we figure out a way to make chronology more tractable?
Who will think of the right question and who will make it answerable with our toolkit of python and R?
We saw a little bit of the possible world today after running through as of yet imperfect dataset with the *multigraph* package in R. We used a similarity function to assess the correspondence among inscription attributes and create a network of these similarities. The revelation to me, a newbie to networks, was that with multigraph we can choose which columns we wish to pass to R for assessment, and then display the result either as a collated network with links relating to different attributes, or filter out individual network components (for each attribute we are evaluating) and assess them in their own right.  Once we implement a good (transparent and quantifiable) measure of temporal intervals and a bit of control over coordinates, we are a few steps away from analysing the diffusion of epigraphic habits over time.
Maybe this sounds simple, but I learnt something today.