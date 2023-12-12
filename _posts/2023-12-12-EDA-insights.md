---
layout: post
title:  "Exploring the Past: Insights from a Viking Artifacts EDA Introduction"
author: Aly Milne
description: Find insights from Viking artifact data
image: "/assets/images/DALLE/viking_graph.png"
---

After scraping and cleaning my data, I had a few questions:
1. Where were these artifacts found?
2. What durable materials did Scandinavians use in the Viking Age?
3. What types of weapons did Vikings use in raiding?
4. When were these artifacts uncovered?

## Dataset Overview
The dataset comprised various attributes of Viking artifacts, including their types, materials, and discovery locations. The primary focus was on artifacts uncovered from different archaeological sites, offering a glimpse into the Vikings' way of life and their interactions with the world.

## Key Insights
### 1. Material Analysis
I started off by analyzing the materials for both datasets. This part of the EDA provided insights into the resources available to the Vikings and their technological capabilities. The prevalence of certain materials, such as iron, wood, and bronze, highlighted their skills in metallurgy and craftsmanship.

From the graphs, I gleaned that silver is most common for trade items and iron is most common for war items. This makes sense as Vikings were known to trade 'hacksilver' which were bracelets and other trinkets measured by weight.

![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/trade_materials.png)

### 2. Geographical Spread
Another intriguing aspect of the EDA was mapping the locations where these artifacts were found. This exercise illustrated the vast geographical spread of Viking influence, and this was hardly all of the artifacts discovered! The distribution of artifacts across various regions pointed to the extensive trade networks and exploratory ventures of the Vikings.

Most trade objects found in Sweden, which makes sense since hacksilver was a Scandinavian tradition and other peoples might not have seen the value in it.

The plot of war objects was much more interesting. Most war objects were found in Sweden, but some were found all the way in Portugal/Spain!

![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/war_locations.png)

### 3. Time Series Analysis
The EDA also included a time series analysis of the artifacts, plotting their discovery over time. This analysis shed light on the periods of peak Viking activity and the evolution of their culture and technology over time.

There was a spike in war artifacts and trade artifatcs found around 1880. I'd be interested to see if there was one particular dig that uncovered a lot.

![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/war_years.png)
![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/trade_years.png)

### 4. Comparative Analysis of Dig Sites
Since I saw the spike around 1880, I looked back into my data to see if there were any fields that might indicate a single fortuitous dig. The best I could find was 'Fornlämning' which I believe indicated dig site. I decided to plot the dig site names to the number of objects found there and add color detail for the year found.

![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/war_dig_sites.png)
![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/trade_dig_sites.png)

It looks like for both groups there were big digs at location L2017:1904 in 1879 and 1881.

### 5. Weapon Types
Lastly, I took a look at the names of the objects in the war dataset to see what kind of weapon seemed most popular.

![Figure]({{site.url}}/{{site.baseurl}}/assets/images/Viking_EDA/weapons.png)

I have no clue whats a pilspet is, so I had to look it up. Turns out, it's an arrowhead! I'm not sure why it didn't get translated, but this makes sense since many arrows are required for a single bow.

This was a surprisingly fun insight as I most often picture Vikings with axes and swords.

## Conclusion
Even though there were many insights I could have learned from a textbook (ie. iron for weapons and silver for trading), there were some fun details I learned about Vikings doing this EDA! I really appreciate having the skill set necessary to do my own analysis instead of relying on the stereotypes of Vikings and what others can tell me.

## Future Work
Moving forward, I would love to see if I can pinpoint exact digs for these artifacts.