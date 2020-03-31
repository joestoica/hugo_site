---
title: "Examining the 2016 Presidential Election"
date: 2018-09-19T18:28:13-05:00
tags: ["visualization", "projects"]
---

## A Leaflet map of the 2016 U.S. Election

The United States 2016 presidential election seemed like it would
be an [easy victory for the DNC's Hillary Clinton over the GOP's Donald Trump.](http://www.pewresearch.org/fact-tank/2016/11/09/why-2016-election-polls-missed-their-mark/)

However, Trump came out the victor, depsite [losing the popular vote by three million votes.](https://en.wikipedia.org/wiki/United_States_presidential_election,_2016)
This election has inspired me to dig deeper into the voting data. Through this investigation, I also wanted to become more familiar with choropleth maps in R, and make them interactive using the leaflet package. If you would like to see the code used to make this, click [here.](https://github.com/joestoica/2016-Presidential-Election-Analysis/blob/master/election_analysis.Rmd)

## Examination Through Mapping

{{< iframe src="/map.html">}}

There are three different ways to explore the map. Clicking on an individual county reveals voting data relevant to that specific county. The radio buttons in the bottom left allow the map to be colored in different ways: one by county winner, by diverging vote percent margin, and the number of votes contributed to each candidate. Clinton is mapped to blue counties, and Trump is mapped to red (both following partisan color traditions). It is extremely important to note that the areas on the first map do not represent the population of those counties, but simply who won which county (even though there are more red counties than blue, this does not represent the true voting population. Clinton still won the popular vote). This is a common example of a misleading visualization without the correct context, and is somewhat remedied by the third view of the map.

## Creation of the Map

Making this map was actually quite a challenge for me. As it turns out, I ended up using three different datasets combined into one larger dataset. One dataset had general party election information by county, another had candidate data by county, and the final was a shapefile to use during the actual creation of the map. The major downfall of this data is that some counties were excluded, so if the candidate data set was missing one county, it had to be removed from the other two datasets. This is why Washington D.C. and the entire state of Alaska were removed from the final map.

Once the data was compiled it was smooth sailing. All I had to complete was the palettes, popup, and the map itself. It's worth mentioning that the diverging color palette used in the second and third maps had to be crafted carefully, due to the fact that the color scales darker with larger winning margins. In order to represent the data accurately, the palettes must scale equally in both directions since the two candidates' maximum votes are different sizes. Otherwise, you run the risk of incorectly representing the difference in voting magnitudes.
