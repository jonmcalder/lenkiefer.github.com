---
layout: post
title: "Mortgage rates after dark"
author: "Len Kiefer"
date: "2017-04-14"
summary: "R statistics rstats mortgage rates dataviz"
group: navigation
theme :
  name : lentheme
---

TONIGHT WE VISUALIZE MORTGAGE RATES AFTER DARK.  Last year I shared [10 amazing ways to visualize mortgage rates]({% post_url 2016-12-08-10-ways-to-visualize-rates %})  (and [more ways]({% post_url 2016-12-15-more-amazing-mortgage-viz %}) and [even more ways]({% post_url 2016-12-18-more-ways-to-visualize-rates %})).  In this post I have one more DATA VISUALIZATION (dataviz) for you.

I was putting together a presentation using [remark.js](https://github.com/gnab/remark) via the [xaringan](https://github.com/yihui/xaringan) R package (see my discussion of how to do this [in this post]({% post_url 2017-02-04-hello-ninja %})) and decided to try a dark theme.  The code below, modifies our mortgage rate code to make this graph:

<img src="{{ site.url }}/img/charts_apr_14_2017/rate_04_13_2017 dark2.gif" alt="pmms after dark"/>


# R Code

We're using the weekly average 30-year fixed mortgage rate as before (you can get from St Louis Federal Reserve Economic Database [here](https://fred.stlouisfed.org/series/MORTGAGE30US)). 

In order to make our graph, I need to create my own theme, called *theme_dark2* (ggplot2 has its own [theme_dark](http://ggplot2.tidyverse.org/reference/ggtheme.html), but I want a different one).  I created my theme by modifying [this theme](https://jonlefcheck.net/2013/03/11/black-theme-for-ggplot2-2/).



## The data

The data I'm going to use are estimates of weekly U.S. average 30-year fixed mortgage rates from the [Primary Mortgage Market Survey](http://www.freddiemac.com/pmms/index.html) from Freddie Mac. These data can be easily downloaded from the St. Louis Fred database [here](http://bit.ly/2hli7Sh).

I have the data saved in a simple text file with a column for data, the mortgage rate, and helper columns week, month, and year, where week is the week number starting with the first week of the year.

Let's load the data and create our theme function.



{% highlight r %}
#load libraries
library(data.table, warn.conflicts = FALSE, quietly=TRUE)
library(tidyverse, warn.conflicts = FALSE, quietly=TRUE)
library(ggthemes, warn.conflicts = FALSE, quietly=TRUE)
library(scales, warn.conflicts = FALSE, quietly=TRUE)
library(extrafont, warn.conflicts = FALSE, quietly=TRUE)
library(gridExtra, warn.conflicts = FALSE, quietly=TRUE)
require(animation, warn.conflicts = FALSE, quietly=TRUE)

##### Make theme:  ######################################################
# modifed based on: https://gist.github.com/jslefche/eff85ef06b4705e6efbc
#########################################################################

theme_dark2 = function(base_size = 12, base_family = "Courier New") {
  
  theme_grey(base_size = base_size, base_family = base_family) %+replace%
    
    theme(
      # Specify axis options
      axis.line = element_blank(),  
      axis.text.x = element_text(size = base_size*0.8, color = "white", lineheight = 0.9),  
      axis.text.y = element_text(size = base_size*0.8, color = "white", lineheight = 0.9),  
      axis.ticks = element_line(color = "white", size  =  0.2),  
      axis.title.x = element_text(size = base_size, color = "white", margin = margin(0, 10, 0, 0)),  
      axis.title.y = element_text(size = base_size, color = "white", angle = 90, margin = margin(0, 10, 0, 0)),  
      axis.ticks.length = unit(0.3, "lines"),   
      # Specify legend options
      legend.background = element_rect(color = NA, fill = " gray10"),  
      legend.key = element_rect(color = "white",  fill = " gray10"),  
      legend.key.size = unit(1.2, "lines"),  
      legend.key.height = NULL,  
      legend.key.width = NULL,      
      legend.text = element_text(size = base_size*0.8, color = "white"),  
      legend.title = element_text(size = base_size*0.8, face = "bold", hjust = 0, color = "white"),  
      legend.position = "right",  
      legend.text.align = NULL,  
      legend.title.align = NULL,  
      legend.direction = "vertical",  
      legend.box = NULL, 
      # Specify panel options
      panel.background = element_rect(fill = " gray10", color  =  NA),  
      panel.border = element_rect(fill = NA, color = "white"),  
      panel.grid.major = element_line(color = "grey35"),  
      panel.grid.minor = element_line(color = "grey20"),  
      panel.spacing = unit(0.5, "lines"),   
      # Specify facetting options
      strip.background = element_rect(fill = "grey30", color = "grey10"),  
      strip.text.x = element_text(size = base_size*0.8, color = "white"),  
      strip.text.y = element_text(size = base_size*0.8, color = "white",angle = -90),  
      # Specify plot options
      plot.background = element_rect(color = " gray10", fill = " gray10"),  
      plot.title = element_text(size = base_size*1.2, color = "white",hjust=0,lineheight=1.25,
                                margin=margin(2,2,2,2)),  
      plot.subtitle = element_text(size = base_size*1, color = "white",hjust=0,  margin=margin(2,2,2,2)),  
      plot.caption = element_text(size = base_size*0.8, color = "white",hjust=0),  
      plot.margin = unit(rep(1, 4), "lines")
      
    )
  
}

########################################################################
{% endhighlight %}

Now armed with our theme, we can make our plot.

Try first a static plot.


{% highlight r %}
dt<- read_excel('data/rates.xlsx',sheet= 'rates')
dt$date<-as.Date(dt$date, format="%m/%d/%Y")

    ggplot(data=filter(dt,date>="2016-03-10"), 
           aes(x=date,y=rate30,label=rate30))+
    geom_line(size=1.05,color="#00B0F0")+theme_minimal()+
    theme(legend.position="none")+
    scale_x_date(date_breaks="2 month", date_labels="%b-%Y")+
    scale_y_continuous(limits=c(3,4.4),breaks=seq(3,4.4,.1),sec.axis=dup_axis())+
    geom_text(data=tail(dt,1),
              nudge_x=3,nudge_y=0,hjust=-0.1,size=3,color="#00B0F0")+
    geom_point(data=tail(dt,1),size=3,color="#00B0F0",alpha=0.75)+
    labs(x="", y="",
         title="30-year Fixed Mortgage Rate (%)",
         subtitle="weekly average rates",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    theme_dark2()+

    coord_cartesian(xlim=c(as.Date("2016-03-10"),as.Date("2017-05-15")), y=c(3.4,4.4))
{% endhighlight %}

![plot of chunk load-data-rates-apr-14-2017-plot1](/img/Rfig/load-data-rates-apr-14-2017-plot1-1.svg)
    
### Animate it

We can complete the gif by running this code:
    

{% highlight r %}
# to make life easier (or at least more compatible with my earlier code!)
# convert to data.table()
dt<-data.table(dt)

# Get dates
dlist<-unique(dt[date>="2016-03-10",]$date)
N<-length(dlist)

# loop:
oopt = ani.options(interval = 0.2)
saveGIF({for (i in 1:N) {
  g<-
    ggplot(data=dt[date>="2016-03-10" & date<=dlist[i]], 
           aes(x=date,y=rate,label=rate))+
    geom_line(size=1.05,color="#00B0F0")+theme_minimal()+
    theme(legend.position="none")+
    scale_x_date(date_breaks="2 month", date_labels="%b-%Y")+
    scale_y_continuous(limits=c(3,4.4),breaks=seq(3,4.4,.1),sec.axis=dup_axis())+
    geom_text(data=tail(dt[date==dlist[i]],1),
              nudge_x=3,nudge_y=0,hjust=-0.1,size=3,color="#00B0F0")+
    geom_point(data=tail(dt[date==dlist[i]],1),size=3,color="#00B0F0",alpha=0.75)+
    labs(x="", y="",
         title="30-year Fixed Mortgage Rate (%)",
         subtitle="weekly average rates",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    theme_dark2()+

    coord_cartesian(xlim=c(as.Date("2016-03-10"),as.Date("2017-05-15")), y=c(3.4,4.4))
  
  print(g)
  print(paste(i,"out of",N))
  ani.pause()
}
  
  for (i2 in 1:8) {
    print(g)
    ani.pause()
  }
},movie.name="rate_04_13_2017 dark2.gif",ani.width = 650, ani.height = 400)
{% endhighlight %}
