---
layout: post
title: "Treemapify those pies!"
author: "Len Kiefer"
date: "2017-04-22"
summary: "R statistics rstats mortgage rates dataviz"
group: navigation
theme :
  name : lentheme
---


TIME FOR ANOTHER DATAVIZ REMIX.  Saw on Twitter that [@hrbrmstr](https://twitter.com/hrbrmstr) posted a remix of a [Wall Street Journal](https://www.wsj.com/articles/brick-and-mortar-stores-are-shuttering-at-a-record-pace-1492818818) visualization over at [rud.is](https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/).  

The original WSJ article used pies of various size to compare recent store closings.

As we usually do in this space, we'll use [R](https://www.r-project.org/) to create our plots.  

Let's mix things up and go remix the remix.

## Pies

But first let's consider the original.

I'm not going to copy the original from the WSJ (click the link above to check out the story), but I am going to make my own pie version.  I can't believe I spent as much time as I did working with pies in [ggplot2](http://ggplot2.tidyverse.org/reference/coord_polar.html). I wasn't quite able to replicate the original, but the code below follows the spirit of the original.


{% highlight r %}
################################################################################
### Initial data stuff from https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/
################################################################################

library(hrbrthemes)
library(tidyverse)

read.table(text='store,closing,total
"Radio Shack",550,1500
"Payless",400,2600
"Rue21",400,1100
"The Limited",250,250
"bebe",180,180
"Wet Seal",170,170
"Crocs",160,560
"JCPenny",138,1000
"American Apparel",110,110
"Kmart",109,735
"hhgregg",88,220
"Sears",41,695', sep=",", header=TRUE, stringsAsFactors=FALSE) %>% 
  as_tibble() %>% 
  mutate(remaining = total - closing,
         gone = round((closing/total) * 100)/100,
         stay = 1-gone,
         rem_lab = ifelse(remaining == 0, "", scales::comma(remaining))) %>% 
  arrange(desc(stay)) %>% 
  mutate(store=factor(store, levels=store)) -> closing_df

update_geom_font_defaults(font_rc)

################################################################################
### break original
################################################################################

################################################################################
# Len's stuff
# reorganize the data a bit
# df 1,2, and 3 are relics of failed pie stuff not reproduced here
################################################################################

closing_df %>% select(store,remaining,closing) %>% gather( type,value,c(2:3)) -> df3
closing_df %>% select(store,gone,stay) %>% gather( type2,pct,c(2:3)) -> df4
df5<-left_join(df3,df4,by="store")

################################################################################
# Pie charts!  sorta
################################################################################

ggplot(df5, aes(x=factor(1), y = pct, fill = type2, width = value)) +
  geom_bar(position="fill", stat="identity") + 
  geom_col()+
  facet_wrap(~store)+ 
  coord_polar("y")+
  theme_void()+
  scale_fill_ipsum(name=NULL,labels=c("closing","remaining")) +
  theme(plot.caption=element_text(hjust=0,size=8),
        legend.position="top",
        legend.direction="horizontal")+
  labs(title="Selected 2017 store closings (estimated)",
       subtitle="Smaller specialty chains such as Bebe and American Apparel are closing their stores,\nwhile larger chains such as J.C. Penny and Sears are scaling back their footprint.",
       caption="@lenkiefer based on https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/\nitself a remix of https://www.wsj.com/articles/brick-and-mortar-stores-are-shuttering-at-a-record-pace-1492818818")
{% endhighlight %}

![plot of chunk 4-22-2017-mypie](/img/Rfig/4-22-2017-mypie-1.svg)

I couldn't fill in the donut holes, but this chart shows both the percent closing and the percent remaining. The radius of the pie/donut represents the absolute number of stores. As is usual with pies (see [this essay](https://www.perceptualedge.com/articles/visual_business_intelligence/save_the_pies_for_dessert.pdf) for some discussion of the challenges with pie charts), making comparisons is difficult.

# First remix
## via hrbrmstr

[@hrbrmstr](https://twitter.com/hrbrmstr) remixed the original chart and posted some ggplot code. It's short enough to reproduce here:


{% highlight r %}
################################################################################
### Back to code from https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/
################################################################################
ggplot(closing_df) +
  geom_segment(aes(0, store, xend=gone, yend=store, color="Closing"), size=8) +
  geom_segment(aes(gone, store, xend=gone+stay, yend=store, color="Remaining"), size=8) +
  geom_text(aes(x=0, y=store, label=closing), color="white", hjust=0, nudge_x=0.01) +
  geom_text(aes(x=1, y=store, label=rem_lab), color="white", hjust=1, nudge_x=-0.01) +
  scale_x_percent() +
  scale_color_ipsum(name=NULL) +
  labs(x=NULL, y=NULL, 
       title="Selected 2017 Store closings (estimated)",
       subtitle="Smaller specialty chains such as Bebe and American Apparel are closing their stores,\nwhile lareger chains such as J.C. Penny and Sears are scaling back their footprint.") +
  theme_ipsum_rc(grid="X") +
  theme(axis.text.x=element_text(hjust=c(0, 0.5, 0.5, 0.5, 1))) +
  theme(legend.position=c(0.875, 1.025)) +
  theme(legend.direction="horizontal")
{% endhighlight %}

![plot of chunk 04-22-2017-hrbrmstr-original](/img/Rfig/04-22-2017-hrbrmstr-original-1.svg)

This is a pretty good remix, but it's always worth considering alternatives.

While the graphic does list the number of stores closing and remaining on the labels, it doesn't give you any visual sense of the number of stores.  For example, JC Penny is closing 138 stores, but it is keeping 862 stores open. American Apparel is closing all 110 stores. The 138 JC Penny stores represents smaller share of the JC Penny footprint but in absolute number is greater than the American Apparel closings.

# Remix the remix

We could also encode the number of stores in either a [mosaic plot](https://en.wikipedia.org/wiki/Mosaic_plot) or [treemap](https://en.wikipedia.org/wiki/Treemapping). I decided to try out a treemap, but saw later that a commentator [left a mosaic plot version](https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/)(check the comments).

Making a treemap is not trivial, but once again the internet comes through.  On github wilkox shared a library called [treemapify](https://github.com/wilkox/treemapify) that lets you create treemaps in R (*ggplot2 geoms for drawing treemaps*).

Turns out it's super simple to create a treemap with the treemapify library.  Let's do it:


{% highlight r %}
################################################################################
# Treemapify!!!!!
# need viridis (for colors) and scales (for labels) libraries 
# run this to install treemapify:
# library(devtools)
# install_github("wilkox/ggfittext")
# install_github("wilkox/treemapify")
################################################################################

library(treemapify)
ggplot(closing_df, aes(
  area = total,
  fill = gone,
  label = paste(store,"\n",closing," out of ",total) )) +
  geom_treemap() +
  geom_treemap_text(
    colour = "white",
    place = "center",
    reflow = T
  )+  viridis::scale_fill_viridis(label=scales::percent,name="% gone\n",option="C",end=0.85)+
   theme(plot.caption=element_text(hjust=0),
         plot.title=element_text(face="bold"))+
  labs(title="Selected 2017 store closings (estimated)",
       subtitle="Smaller specialty chains such as Bebe and American Apparel are closing their stores,\nwhile larger chains such as J.C. Penny and Sears are scaling back their footprint.",
    caption="@lenkiefer based on https://rud.is/b/2017/04/21/shuttering-pies-with-retiring-stores/\nitself a remix of https://www.wsj.com/articles/brick-and-mortar-stores-are-shuttering-at-a-record-pace-1492818818")
{% endhighlight %}

![plot of chunk 04-22-2017-treemapify](/img/Rfig/04-22-2017-treemapify-1.svg)

### Summary

Overall, I'm not sure the treemap is better than the original remix. The simple bar chart focuses on the reduction in stores (as a percent of total), while the treemap shows multiple dimensions and could be confusing.  

Of course, the data is so small that we can actually show each data point in the treemap or the bar chart. For larger datasets this wouldn't work and we would have to decide what was most important.

But it's pretty good to know we can use treemaps with ggplot2 should we need them.
