---
layout: post
title: "QR code or dataviz?"
author: "Len Kiefer"
date: "2017-03-30"
summary: "R statistics dataviz housing mortgage data"
group: navigation
theme :
  name : lentheme
---
  


TODAY I MADE A KIND OF SILLY DATAVIZ, a tile plot of weekly changes in mortgage rates. A colleague happened by my viz terminal, pointed at my monitor and asked "what is that, a QR code?"

Nope, it was a tile plot. 

We're going to make the  plot with [R](https://www.r-project.org/).  As before, we'll use data we used in our [mortgage rate post]({% post_url 2016-12-08-10-ways-to-visualize-rates %}) to explore weekly average mortgage rates in the United States based on Freddie Mac's [Primary Mortgage Market Survey](http://www.freddiemac.com/pmms/index.html).

# Get started

Let's load our mortgage rates data and make a plot.


{% highlight r %}
########################
####  Load Packages ####
########################

library(data.table)
library(tidyverse)
library(viridis)


########################
####  Load Data ########
########################

#for more on these data see http://lenkiefer.com/2016/12/08/10-ways-to-visualize-rates

# read in data
dt<- read_excel('data/rates.xlsx',sheet= 'rates')

##########################################################

##########################################################
# Add variables to data
##########################################################

dt$date<-as.Date(dt$date, format="%m/%d/%Y")
dt<-data.table(dt) 
dt$year<-year(dt$date) # create year variable

# create a year factor, levels going backwards from 2017 to 1971
dt$yearf<-factor(dt$year,levels=seq(2017,1971,-1))  

# create a weekly indicator by year:

dt=dt[,week:=1:.N,by=year]

# weekly change in rates
dt=dt[,dr:=rate30-shift(rate30,1)]

# indicator rates up, no change, or down
dt=dt[,dri:=ifelse(dr>0,"Up",ifelse(dr==0,"No change","Down"))]


##########################################################
# Function for plotting
##########################################################

tile.plot<-function(d="2017-03-30"){
  g.tile<-  
    ggplot(data=dt[year>1971 & date<=d,],aes(x=week,y=yearf,color=dri,fill=dri))+
    geom_tile(data=dt[year>1971,],alpha=0,color=NA)+
    geom_tile(color="gray")  +
    scale_fill_viridis(name="1-week\nChange",discrete=T,option="D",end=0.85)+
    scale_color_viridis(name="1-week\nChange",discrete=T,option="D")+
    theme_minimal()+labs(x="week of year",y="year",
                       title="Weekly change in 30-year fixed mortgage rates",
                       subtitle="Down, no change, or up?",
                       caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    scale_x_continuous(breaks=seq(0,50,10))+
    theme(legend.position="top",plot.caption=element_text(hjust=0))+
    theme(axis.text.y=element_text(size=8),
          axis.text.x=element_text(size=8))
  return(g.tile)
}
{% endhighlight %}

We can now call our function to create our plot.


{% highlight r %}
tile.plot()
{% endhighlight %}

![plot of chunk mar-30-2017-chart-1](/img/Rfig/mar-30-2017-chart-1-1.svg)

Isn't it beautiful? 

By why stop there, as we've done in the past, why don't we animate it?  The code below animates the plot.


{% highlight r %}
##########################################################
# animate the plot
##########################################################

dlist<-unique(dt[year>1971 & week %in% c(13,26,39,52),]$date)


#dlist<-unique(dt[year>1971 & week %in% c(13,52),]$date)

oopt = ani.options(interval = 0.05)
saveGIF({for (i in 1:length(dlist)) {
  g<-tile.plot(dlist[i])
  print(g)
  print(paste(i,"out of",length(dlist)))  # print a counter
  ani.pause()
}
  for (i2 in 1:10) {
    print(g)
    ani.pause()
  }
},movie.name="tile_rates_03_30_2017.gif",ani.width = 650, ani.height = 800)
{% endhighlight %}

<img src="{{ site.url}}/img/charts_mar_30_2017/tile_rates_03_30_2017.gif">


## What do we get out of it?

This plot isn't that informative. It does answer one question though. Folks had been asking me about seasonality in mortgage rates.  But mortgage rates follow Treasury yields pretty closely and don't have a lot of seasonality.  The lack of a distinctive pattern (all down in winter, all up in the summer say) kind of shows it.

For better mortgage rate charts, check out [my earlier post]({% post_url 2016-12-08-10-ways-to-visualize-rates %}) and [my mortgage rates flexdashboard]({% post_url 2017-01-08-mortgage-rate-viewer %}) .


