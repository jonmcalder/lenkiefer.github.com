---
layout: post
title: "10 ways to visualize mortgage rates "
author: "Len Kiefer"
date: "2016-12-08"
summary: "R statistics forecasting house prices housing"
group: navigation
theme :
  name : lentheme
---
# Introduction

IN ORDER TO HELP PEOPLE UNDERSTAND WHAT'S GOING ON with the economy, housing and mortgage markets I spend a great deal of time thinking about interest rates.  Interest rats, specifically mortgage rates are very important to housing and mortgage markets.  In my [professional life](https://www.linkedin.com/in/leonard-kiefer-51175331) I work on tracking trends in housing and mortgage markets, and that includes mortgage rates.  I create a lot of visualizations of mortgage rates.

In this post I'm going to share with you 10 of my favorate ways to visualize mortgage rates and give you [R](https://www.r-project.org/) code to do it. 

## The data

The data I'm going to use are estimates of weekly U.S. average 30-year fixed mortgage rates from the Primary Mortgage Market Survey from Freddie Mac. These data can be easily downloaded from the St. Louis Fred database [here](http://bit.ly/2hli7Sh).

I have the data saved in a simple text file with a column for data, the mortgage rate, and helper columns week, month, and year, where week is the week number starting with the first week of the year.

Let's load the data and take a peak.



{% highlight r %}
#load libraries
library(data.table, warn.conflicts = FALSE, quietly=TRUE)
library(ggplot2, warn.conflicts = FALSE, quietly=TRUE)
library(dplyr, warn.conflicts = FALSE, quietly=TRUE)
library(zoo, warn.conflicts = FALSE, quietly=TRUE)
library(ggrepel, warn.conflicts = FALSE, quietly=TRUE)
library(ggthemes, warn.conflicts = FALSE, quietly=TRUE)
library(scales, warn.conflicts = FALSE, quietly=TRUE)
library(tidyr, warn.conflicts = FALSE, quietly=TRUE)
library(zoo,warn.conflicts=F,quietly=T)
library(purrr,warn.conflicts=F,quietly=T)
library(xts,warn.conflicts=F,quietly=T)
library(lubridate,warn.conflicts=F,quietly=T)
library(viridis,warn.conflicts = F,quietly = F) #for the colorz
library("htmlTable")
#load data from text file
pmms30yr <- fread("data/pmms30yr.txt")
#set up date variable
pmms30yr$date<-as.Date(pmms30yr$date, format="%m/%d/%Y")



# make tables for viewing formatting dates with purr %>% operations
htmlTable(head(pmms30yr %>% map_if(is.Date, as.character,format="%b %d,%Y") %>% map_if(is.numeric, round,3) %>%as.data.frame() ,10), col.rgroup = c("none", "#F7F7F7"),caption="30-year Fixed Mortgage Rate (%)",
          tfoot="Source: Freddie Mac Primary Mortgage Market Survey")
{% endhighlight %}

<table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr><td colspan='6' style='text-align: left;'>
30-year Fixed Mortgage Rate (%)</td></tr>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>date</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>rate</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>year</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>month</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>week</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>Apr 02,1971</td>
<td style='text-align: center;'>7.33</td>
<td style='text-align: center;'>1971</td>
<td style='text-align: center;'>4</td>
<td style='text-align: center;'>14</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>Apr 09,1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>7.31</td>
<td style='background-color: #f7f7f7; text-align: center;'>1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>15</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>Apr 16,1971</td>
<td style='text-align: center;'>7.31</td>
<td style='text-align: center;'>1971</td>
<td style='text-align: center;'>4</td>
<td style='text-align: center;'>16</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>Apr 23,1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>7.31</td>
<td style='background-color: #f7f7f7; text-align: center;'>1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>17</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>Apr 30,1971</td>
<td style='text-align: center;'>7.29</td>
<td style='text-align: center;'>1971</td>
<td style='text-align: center;'>4</td>
<td style='text-align: center;'>18</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>6</td>
<td style='background-color: #f7f7f7; text-align: center;'>May 07,1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>7.38</td>
<td style='background-color: #f7f7f7; text-align: center;'>1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>5</td>
<td style='background-color: #f7f7f7; text-align: center;'>19</td>
</tr>
<tr>
<td style='text-align: left;'>7</td>
<td style='text-align: center;'>May 14,1971</td>
<td style='text-align: center;'>7.42</td>
<td style='text-align: center;'>1971</td>
<td style='text-align: center;'>5</td>
<td style='text-align: center;'>20</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>8</td>
<td style='background-color: #f7f7f7; text-align: center;'>May 21,1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>7.44</td>
<td style='background-color: #f7f7f7; text-align: center;'>1971</td>
<td style='background-color: #f7f7f7; text-align: center;'>5</td>
<td style='background-color: #f7f7f7; text-align: center;'>21</td>
</tr>
<tr>
<td style='text-align: left;'>9</td>
<td style='text-align: center;'>May 28,1971</td>
<td style='text-align: center;'>7.46</td>
<td style='text-align: center;'>1971</td>
<td style='text-align: center;'>5</td>
<td style='text-align: center;'>22</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: left;'>10</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Jun 04,1971</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>7.52</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>1971</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>6</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>23</td>
</tr>
</tbody>
<tfoot><tr><td colspan='6'>
Source: Freddie Mac Primary Mortgage Market Survey</td></tr></tfoot>
</table>

The data are weekly observations on mortgage rates running from April 2, 1971 through December 8, 2016.  Now let's take these series and make 10 different visualizations. We'll start simple, and build up to more complex visualizations

### A note on data manipulations

I'm going to be using the [data.table()](https://cran.r-project.org/web/packages/data.table/index.html) package from R.  I've found this package very helpful for doing the types of data manipulations I most frequently need.  Check the comments in the code below for specific callouts.

# 1: A simple line chart

Let's start simple with a line chart.  We'll also add some styling including a reference line at the last monthly observation and a dot at the last point.  We'll start the data in 2001.



{% highlight r %}
g1<-
  ggplot(data=pmms30yr[date>'2000-12-31'],  
       #We're going to subset the data to be after Dec 31, 2000.  
       #We could use filter function, but instead I'm using the data.table() shorthand
        aes(x=date,y=rate,label=rate))+geom_line()+theme_minimal()+
    #set date breaks at 1 year, format as 2000 ("%Y"), 00 would be "%y"
    scale_x_date(date_breaks="1 year", date_labels="%Y")+  
  #add text, marker and reference line for last observation
  geom_text(data=tail(pmms30yr,1),nudge_y=.25,color="red")+
  #use nudge_y to lift label above point
  geom_point(data=tail(pmms30yr,1),size=2,color="red",alpha=0.75)+
  geom_hline(yintercept=tail(pmms30yr,1)$rate,linetype=2,alpha=0.82,color="red")+
  labs(x="", y="",
       title="30-year Fixed Mortgage Rate (%)",
       subtitle="weekly rates since 2001",
       caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
  theme(plot.title=element_text(size=14))+
  theme(axis.text=element_text(size=8))+
  theme(plot.caption=element_text(hjust=0))
g1
{% endhighlight %}

![plot of chunk rate-viz1-dec-08-2016,](/img/Rfig/rate-viz1-dec-08-2016,-1.svg)

# 2: Line chart comparing weeks by year

This next chart is a variation on the line chart.  Instead of using date for the *x* axis, we use the week of the year and plot a separate line for recent years (2013, 2014, 2015 and 2016). By comparing the lines at any point on the x axis, we can see where rates were one or more years ago on this week.


{% highlight r %}
i<-max(pmms30yr[year==2016]$week)  #figure out the maximum week number in 2016

g2<-
  ggplot(data=pmms30yr[year>2012 & week<=i], 
           aes(x=week,y=rate,label=paste("   ",year),
               linetype=as.factor(year),
               color=as.factor(year)))+
    geom_line(size=1.05)+theme_minimal()+
    theme(legend.position="none")+
    scale_x_continuous(limits=c(0,54),breaks=seq(0,55,5))+
    scale_y_continuous(limits=c(3.25,4.75),breaks=seq(3.25,5,0.25))+
    geom_text(data=pmms30yr[year>2012 & week==i],nudge_x=2)+
    geom_point(data=pmms30yr[year>2012 & week==i],size=3.5,alpha=0.75)+
    labs(x="Week Number", y="Mortgage Rate (%)",
         title="30-year Fixed Mortgage Rate by Week of Year",
         subtitle="comparing 2013, 2014, 2015 and 2016",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
  theme(plot.title=element_text(size=14))+
  theme(axis.text=element_text(size=8))+
  theme(plot.caption=element_text(hjust=0))
g2
{% endhighlight %}

![plot of chunk rate-viz2-dec-08-2016,](/img/Rfig/rate-viz2-dec-08-2016,-1.svg)

# 3: Area chart showing year-over-year changes

In this next chart we're going to compute a rolling 52-week difference. We also want to shade in the area between the line difference colors based on whether or not rates are up or down.  Shading between two lines in ggplot is tricky, so I'm going to actually create two series, one for positive 52-week changes and another for negative 52-week changes.


{% highlight r %}
pmms30yr[,d52:=rate-shift(rate,52)]
pmms30yr[,d52.up:=ifelse(d52>0,d52,0)]      #if diff >0, diff in rates, else 0
pmms30yr[,d52.down:=ifelse(d52<0,d52,0)]    #if diff <0, diff in rates, else 0

g3<-
  ggplot(data=pmms30yr[date>'1979-12-31'],  
       #We're going to subset the data to be after Dec 31, 2000.  
       #We could use filter function, but instead I'm using the data.table() shorthand
        aes(x=date,y=d52,label=rate))+geom_line()+theme_minimal()+
    #set date breaks at 1 year, format as 2000 ("%Y"), 00 would be "%y"
    scale_x_date(breaks=seq(as.Date("1980-01-01"),as.Date("2020-01-01"),"5 years"), date_labels="%Y")+  
  #add text, marker and reference line for last observation
  geom_area(aes(y=d52.down),fill=viridis(5)[4],alpha=0.5)+  #shade green if down
  geom_area(aes(y=d52.up),fill=viridis(5)[2],alpha=0.5)+    # shade blue if up
  labs(x="", y="",
       title="30-year Fixed Mortgage Rate",
       subtitle="52-week change in percentage points",
       caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
  theme(plot.title=element_text(size=14))+
  theme(axis.text=element_text(size=8))+
  theme(plot.caption=element_text(hjust=0))
g3
{% endhighlight %}

![plot of chunk rate-viz3-dec-08-2016,](/img/Rfig/rate-viz3-dec-08-2016,-1.svg)

# 4: Combo line with rugplot

We can enhance this plot by adding a [marginal rug plot](http://docs.ggplot2.org/0.9.2.1/geom_rug.html) to the bottom of the chart indicating whether or not rates are up or down for that week.


{% highlight r %}
g4<-g3+
  geom_rug(data=pmms30yr[year(date)>1979 & d52<0,],
           aes(y=d52.down),color=viridis(5)[4],alpha=0.5,sides="b")+  # Only put rug at bottom
  geom_rug(data=pmms30yr[year(date)>1979 & d52>0,],
           aes(y=d52.up),color=viridis(5)[2],alpha=0.5,sides="b")     # Only put rug at bottom
g4
{% endhighlight %}

![plot of chunk rate-viz4-dec-08-2016,](/img/Rfig/rate-viz4-dec-08-2016,-1.svg)

# 5: Step function

We can use a step function to compare the annual average to the weekly values. What we'll do in this plot is compute the annual average (using data.table()) and plot it as a step function on top of the line chart from 1.


{% highlight r %}
#compute monthly and annual averages:
pmms30yr[,rate.y:=mean(rate),by=c("year")]

g5<-ggplot(data=pmms30yr[date>'2000-12-31'],  
       #We're going to subset the data to be after Dec 31, 2000.  
       #We could use filter function, but instead I'm using the data.table() shorthand
        aes(x=date,y=rate,label=rate))+
  geom_line(alpha=0.75)+
  
  theme_minimal()+
    #set date breaks at 1 year, format as 2000 ("%Y"), 00 would be "%y"
    scale_x_date(date_breaks="1 year", date_labels="%Y")+  
  #add annual average as step function
  geom_step(aes(y=rate.y),color=viridis(5)[2],size=1.1)+
  labs(x="", y="",title="30-year Fixed Mortgage Rate (%)",
       subtitle="weekly rates since 2001",
       caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey, blue line annual average")+
  theme(plot.title=element_text(size=14))+
  theme(axis.text=element_text(size=8))+
  theme(plot.caption=element_text(hjust=0))
g5
{% endhighlight %}

![plot of chunk rate-viz5-dec-08-2016,](/img/Rfig/rate-viz5-dec-08-2016,-1.svg)

# 6: Pie Chart

We can also make a Pie chart:


{% highlight r %}
g6<-
  ggplot(pmms30yr[year==2015], aes(x="", y=rate, fill=as.character(date,"%b")))+
  geom_bar(width = 1, stat = "identity") + coord_polar("y", start=0)+
  theme_void()+  scale_fill_viridis(name="Month",discrete=T)+
  labs(title="Share of a year")
g6
{% endhighlight %}

![plot of chunk rate-viz6-dec-08-2016,](/img/Rfig/rate-viz6-dec-08-2016,-1.svg)

Ha ha, just kidding, that's awful

# 7: Strip chart

I just tried this chart out today and I really like it.  It's a strip chart that shows the year-over-year percent change in mortgage rates. You can't read the information as accurately as a line chart, but it gives you a much better impression on how rates have been changing.  Let's make it and then discuss more:


{% highlight r %}
g7<-
  ggplot(data=pmms30yr[year>2000,],aes(x=week,y=1,color=d52,fill=d52))+
  geom_col()+
  scale_fill_viridis(name="52-week\nChange (pp)",discrete=F,option="B")+
  scale_color_viridis(name="52-week\nChange (pp)",discrete=F,option="B")+
  theme_minimal()+
  facet_wrap(~year,ncol=4)+
  theme(axis.ticks.y=element_blank(),
        axis.text.y=element_blank(),
        panel.grid.major=element_blank(),
        panel.grid.minor=element_blank(),
        axis.text.x=element_text(size=6))+
    labs(x="", y="",
       title="30-year Fixed Mortgage Rate",
       subtitle="52-week change in percentage points",
       caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
  scale_x_continuous(breaks=c(1,26,52),labels=c("Jan","Jul","Dec"))+
    theme(plot.title=element_text(size=14))+
  theme(axis.text=element_text(size=8))+
  theme(plot.caption=element_text(hjust=0))
g7
{% endhighlight %}

![plot of chunk rate-viz7-dec-08-2016,](/img/Rfig/rate-viz7-dec-08-2016,-1.svg)

While you can't read the values from this chart as clearly as a line chart (or a table), you can quickly get a *feel* for the important trends in the data.  The bright yellow periods are when mortgage rates were rising, while the dark purple corresponds to periods when rates were falling.  It's pretty easy to see that 2001, 2003, 2009 and 2012 were years when rates fell a lot, while 2006, 2013, and 2014 were when rates were rising relative to the previous year.

# 8: Animated Line Chart

These next three charts are animated versions of some the preceding charts.  We'll start with a simple animated line chart from 1.


{% highlight r %}
N<-max(pmms30yr[year==2016]$week)  
oopt = ani.options(interval = 0.15)
saveGIF({for (i in 1:N) {
  g<-
    ggplot(data=pmms30yr[date>='2015-12-31' & week<=i], aes(x=date,y=rate,label=rate))+geom_line()+theme_minimal()+
    theme(legend.position="none")+
    scale_x_date(date_breaks="1 month", date_labels="%b")+
    scale_y_continuous(limits=c(3,4.2),breaks=seq(3,4.2,.1))+
    geom_text(data=pmms30yr[date>='2015-12-31' & week==i],nudge_x=10)+
    geom_point(data=pmms30yr[date>='2015-12-31' & week==i],size=2,color="red",alpha=0.75)+
    labs(x="", y="Mortgage Rate (%)",
         title="30-year Fixed Mortgage Rate (%)",
         subtitle="weekly rates in 2016",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    theme(plot.title=element_text(size=18))+
    theme(plot.caption=element_text(hjust=0,vjust=1,margin=margin(t=10)))+
    theme(plot.margin=unit(c(0.25,0.25,0.25,0.25),"cm"))+
    coord_cartesian(xlim=c(as.Date("2015-12-31"),as.Date("2016-11-30")), y=c(3.3,4.2))
  print(g)
  ani.pause()
}
  
  for (i2 in 1:20) {
    print(g)
    ani.pause()
  }
},movie.name="rate_12_08_2016.gif",ani.width = 650, ani.height = 400)
{% endhighlight %}

<img src="{{ site.url }}/img/charts_dec_8_2016/rate_12_08_2016.gif" alt="pmms gif 2016"/>

# 9: Animated Line Chart 2

We can also construct an animation for the weekly comparison line chart:


{% highlight r %}
oopt = ani.options(interval = 0.15)
saveGIF({for (i in 1:N) {
  g<-
    ggplot(data=pmms30yr[year>2012 & week<=i], 
           aes(alpha=week/i,x=week,y=rate,label=paste(" ",year),color=as.factor(year)))+
    geom_line(size=1.05)+theme_minimal()+
    scale_color_viridis(discrete=T,end=0.9,direction=-1)+
    theme(legend.position="none")+
    scale_x_continuous(limits=c(0,50),breaks=seq(0,50,5))+
    scale_y_continuous(limits=c(3.25,4.75),breaks=seq(3.25,5,0.25))+
    geom_text(data=pmms30yr[year>2012 & week==i],nudge_x=2)+
    geom_point(data=pmms30yr[year>2012 & week==i],size=3.5,alpha=0.75)+
    labs(x="Week Number", y="Mortgage Rate (%)",
         title="30-year Fixed Mortgage Rate by Week of Year",
         subtitle="comparing 2013, 2014, 2015 and 2016",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    theme(plot.title=element_text(size=18))+
    theme(plot.caption=element_text(hjust=0,vjust=1,margin=margin(t=10)))+
    theme(plot.margin=unit(c(0.25,0.25,0.25,0.25),"cm"))
  print(g)
  print(i)
  ani.pause()
}
  for (i2 in 1:20) {
    print(g)
    print(i2)
    ani.pause()
  }
},movie.name="rate_compare_dec_08_2016.gif",ani.width = 500, ani.height = 350)
{% endhighlight %}

<img src="{{ site.url }}/img/charts_dec_8_2016/rate_compare_dec_08_2016.gif" alt="pmms gif weekly"/>

# 10: Animated line chart with annotations

We can also add some annotations and some more detailed camerawork for the animated linechart.

The way we'll do it, is set up a function that takes two input dates, a minimum and a maximum. The data will then get truncated at the min and max dates, allowing us to zoom around the time series history of mortgage rates.

For smoother animations we'll use tweenr.See my earlier [post about tweenr]({% post_url 2016-05-29-improving-R-animated-gifs-with-tweenr %}) for an introduction to tweenr, and more examples [here]({% post_url 2016-05-30-more-tweenr-animations %}) and [here]({% post_url 2016-06-26-week-in-review %}). 


{% highlight r %}
library(tweenr)
DT<-copy(pmms30yr)
myf<-function(dd,dmin=as.Date("2014-12-31"),
              #Variable subt contains annotations in the subtitle frame
              subt="Nothing",
              keepdots="No"){
  DT2<-copy(DT)
  #set max to last value
  DT2[date>dd,rate:=DT[date==dd]$rate]
  DT2[date>dd,date:=dd]
  #set min to first value
  DT2[date<=dmin,rate:=DT[date==dmin]$rate]
  DT2[date<=dmin,date:=dmin]
  DT2[,subt:=label_wrap_gen(100)(subt)]
  DT2$subt<-factor(DT2$subt)
  DT2$keepdots<-factor(keepdots)
  as.data.frame(DT2[, list(date,rate,subt,keepdots)])}


tf <- tween_states(
  list(myf(as.Date("2016-11-03"),as.Date("2015-12-31"),subt="rates fell throughout most of 2016 up to the U.S. general election..."),
       myf(as.Date("2016-12-01"),as.Date("2015-12-31"),subt="...rising rapidly after the election...."),
       myf(as.Date("2016-12-08"),as.Date("2014-10-02"),subt="...rates are up to the highest level since Oct 2014...."),
       myf(as.Date("2016-12-08"),as.Date("2012-12-27"),subt="...having declined after the Taper Talk in 2013...."),
       myf(as.Date("2016-12-08"),as.Date("1971-04-02"),subt="...and after over 30 years of general decline."),
       myf(as.Date("2015-12-31"),as.Date("2015-12-31"),subt="...rates entered 2016 above 4%, but...")
         ),tweenlength= 3, statelength=1, ease=rep('cubic-in-out',2),nframes=100)
tf<-data.table(tf)

oopt = ani.options(interval = 0.15)
saveGIF({for (i in 1:44) {
  g<-
    ggplot(data=pmms30yr[date=='2015-12-31' | (date>='2015-12-31' & week<=i)], aes(x=date,y=rate,label=paste(" ",rate)))+
    geom_line()+theme_minimal()+
    theme(legend.position="none")+
    geom_text(data=pmms30yr[date>='2015-12-31' & week==i],nudge_x=10)+
    geom_point(data=pmms30yr[date>='2015-12-31' & week==i],size=2,color="red",alpha=0.75)+
    labs(x="", y="Mortgage Rate (%)",
         title="30-year Fixed Mortgage Rate (%)",
         subtitle="Rates fell throughout most of 2016 up to the U.S. general election...",
         caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
    theme(plot.title=element_text(size=18))+
    theme(plot.caption=element_text(hjust=0,vjust=1,margin=margin(t=10)))+
    theme(plot.margin=unit(c(0.25,0.25,0.25,0.25),"cm"))+
    coord_cartesian(xlim=c(as.Date("2015-12-31"),as.Date("2016-12-31")), y=c(3.25,4.01))
  print(g)
  ani.pause()
  print(i)
}
  
  for (i in 1:1){
    print(g)
    ani.pause()
    print(i)
  }
  
  for (i in 1:max(tf$.frame)) {
    g<-
      ggplot(data=tf[.frame==i], aes(x=date,y=rate,label=rate))+geom_line()+theme_minimal()+
      theme(legend.position="none")+
      geom_point(data=tf[date==max(tf[.frame==i]$date) & .frame==i,],size=2,color="red",alpha=0.75)+
      labs(x="", y="Mortgage Rate (%)",
           title="30-year Fixed Mortgage Rate (%)",
           subtitle=tf[.frame==i,]$subt,
           caption="@lenkiefer Source: Freddie Mac Primary Mortgage Market Survey")+
      theme(plot.title=element_text(size=18))+
      theme(plot.caption=element_text(hjust=0,vjust=1,margin=margin(t=10)))+
      theme(plot.margin=unit(c(0.25,0.25,0.25,0.25),"cm"))+
      coord_cartesian(xlim=c(min(tf[.frame==i]$date),as.Date("2016-12-31")), y=c(3.25,max(tf[.frame==i]$rate)))
    
    print(g)
    ani.pause()
    print(i)
  }
},movie.name="rate_12_08_2016_annotate.gif",ani.width = 500, ani.height = 350)
{% endhighlight %}

<img src="{{ site.url }}/img/charts_dec_8_2016/rate_12_08_2016_annotate.gif" alt="pmms gif annotate 2016"/>


# More?

I've got several other visualizations I use from time to time. Check back in this space for more.
