---
layout: post
title: "Nested recursion: Loops within loops within data frames "
author: "Len Kiefer"
date: "2016-12-04"
summary: "R statistics forecasting house prices housing"
group: navigation
theme :
  name : lentheme
---
# Introduction

I HAVE BEEN WATCHING SOME VIDEOS of [Plotcon 2016](https://plotcon.plot.ly/).  All of the videos I've watched are worth watching ([check out the playlist](https://www.youtube.com/playlist?list=PLR7d32uh__xbrzSdfzxxCxCEFQnt7BcOj)), but I was particularly interested in this one from Hadley Wickham:

<iframe width="560" height="315" src="https://www.youtube.com/embed/cU0-NrUxRw4?list=PLR7d32uh__xbrzSdfzxxCxCEFQnt7BcOj" frameborder="0" allowfullscreen></iframe>

Among other things Hadley talks about the idea of nesting data frames, models and model results within a data frame. That idea struck me as something that could be quite useful and not at all something that could lead to explosive increase in the size of data frames or unending loops.

We'll try these ideas out in a simple, stylized forecasting exercise. We'll use [R](https://www.r-project.org/) to explore and model.


## The data

I happen to have some data [handy]({% post_url 2016-12-03-visual-meditations-on-house-prices-part7 %}) that's perfect for this exercise.  It's the [Freddie Mac House Price Index](http://www.freddiemac.com/finance/house_price_index.html) I've been using for my series of [Visual Meditations on House Prices]({% post_url 2016-05-08-visual-meditations-on-house-prices %}).

We're going to use data house prices for the United States, each of the 50 states and the District of Columbia collected in the following file:

1. [*state and national called fmhpi2016q3.txt*]({{ site.url }}/img/charts_nov_3_2016/fmhpi2016q3.txt)

**Important note: Though I'm going to use house prices as an illustrative example, this shouldn't be interpreted as my recommendation for a reasonable way to model house prices in any way.  This is just for fun, and trying out some coding things.**

## The strategy

Here's what we're going to do.  We're going to take our house price data, a dataset with monthly observations on the house price index from January 1975 to September 2016, and construct a simple forecast for house price growth (year-over-year percent change) for each state rolling forward from 1985.  

To make things simple we'll subset the data to just include observations in September of each (the last month available for 2016). That will reduce the number of observations and get rid of overlapping time series observations.

We can apply the techniques outlined in Hadley's Plotcon talk to organize our results in a single data frame.

### The old way

What I would usually do in this situation, is construct a series of loops to iterate over each state and each time period.  Something like this (where *forecast.function*, and *stack.data.function* are some functions that compute forecasts and organize the output data respectively):


{% highlight r %}
for (i: 1:N.states){                                                    # iterate over states
  for (t: 1:T.months){                                                  # iterate over time periods
    newdata<-filter(data, date<=T & state==i)                           # filter data, state=i, date <= T
    forecast.data<-forecast.function(newdata)                           # forecast function
    output.data<-stack.data.function(output.data,forecast.data)         # organize data function
    }
  }
{% endhighlight %}

### Nest with map

Now we can avoid the loops by using the *map* or *map2* functions from the [purrr](https://cran.r-project.org/web/packages/purrr/index.html) package.  Instead of loops, we can take a dataframe with a list of states and dates and then use the *map2* function to store our model results in the same dataframe. Like so:


{% highlight r %}
output.data<- data %>% mutate(mod=map2(state,date,forecast.function))
{% endhighlight %}

Besides being more efficient to write, this will result in a nice structure and we don't have to worry about index numbers and aligning things if we want to add or subtract rows.

# Example using house prices

Let's load packages, import the data from the text file above, and do some data manipulations:



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

#load data from text file
d.state <- fread("data/fmhpi2016q3.txt")
#set up date variable
d.state$date<-as.Date(d.state$date, format="%m/%d/%Y")
#get list of states
state.list<-unique(d.state$state)

# construct variable hpa12 which is 12-month percentage change in house price index
# using data.table for convenience of rolling operations across groups
d.state[,hpa12:=c(rep(NA,12),((1+diff(hpi,12)/hpi))^1)-1,by=state]

#subset so that year >1975 and month ==9
d.state<-d.state[month==9 & year>1975]
{% endhighlight %}

Now that we have our data loaded let's take a peek at the structure:


{% highlight r %}
library("htmlTable")
# make tables for viewing formatting dates with purr %>% operations
htmlTable(head(d.state %>% map_if(is.Date, as.character,format="%b-%Y") %>% map_if(is.numeric, round,3) %>%as.data.frame() ,10), col.rgroup = c("none", "#F7F7F7"),caption="State House Price Data",
          tfoot="Source: Freddie Mac House Price Index")
{% endhighlight %}

<table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr><td colspan='8' style='text-align: left;'>
State House Price Data</td></tr>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>date</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>state</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpi</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>year</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>month</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>type</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpa12</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>Sep-1976</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>41.953</td>
<td style='text-align: center;'>1976</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>State</td>
<td style='text-align: center;'>0.077</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>AL</td>
<td style='background-color: #f7f7f7; text-align: center;'>38.775</td>
<td style='background-color: #f7f7f7; text-align: center;'>1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>9</td>
<td style='background-color: #f7f7f7; text-align: center;'>State</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.073</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>Sep-1976</td>
<td style='text-align: center;'>AR</td>
<td style='text-align: center;'>41.717</td>
<td style='text-align: center;'>1976</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>State</td>
<td style='text-align: center;'>0.084</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>AZ</td>
<td style='background-color: #f7f7f7; text-align: center;'>30.269</td>
<td style='background-color: #f7f7f7; text-align: center;'>1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>9</td>
<td style='background-color: #f7f7f7; text-align: center;'>State</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.039</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>Sep-1976</td>
<td style='text-align: center;'>CA</td>
<td style='text-align: center;'>19.974</td>
<td style='text-align: center;'>1976</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>State</td>
<td style='text-align: center;'>0.162</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>6</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>CO</td>
<td style='background-color: #f7f7f7; text-align: center;'>22.027</td>
<td style='background-color: #f7f7f7; text-align: center;'>1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>9</td>
<td style='background-color: #f7f7f7; text-align: center;'>State</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.066</td>
</tr>
<tr>
<td style='text-align: left;'>7</td>
<td style='text-align: center;'>Sep-1976</td>
<td style='text-align: center;'>CT</td>
<td style='text-align: center;'>27.238</td>
<td style='text-align: center;'>1976</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>State</td>
<td style='text-align: center;'>0.06</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>8</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>DC</td>
<td style='background-color: #f7f7f7; text-align: center;'>22.244</td>
<td style='background-color: #f7f7f7; text-align: center;'>1976</td>
<td style='background-color: #f7f7f7; text-align: center;'>9</td>
<td style='background-color: #f7f7f7; text-align: center;'>State</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.101</td>
</tr>
<tr>
<td style='text-align: left;'>9</td>
<td style='text-align: center;'>Sep-1976</td>
<td style='text-align: center;'>DE</td>
<td style='text-align: center;'>28.837</td>
<td style='text-align: center;'>1976</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>State</td>
<td style='text-align: center;'>0.011</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: left;'>10</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Sep-1976</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>FL</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>34.051</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>1976</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>9</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>State</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>0.036</td>
</tr>
</tbody>
<tfoot><tr><td colspan='8'>
Source: Freddie Mac House Price Index</td></tr></tfoot>
</table>

What we want to do is forecast the house price appreciation ("hpa12": year-over-year percent change in index) at different points in time.

### Working on a single state

Before we get into the nesting business, let's just do it for a single state, for a single month.  Let's extract the history of house price growth (hereafter HPA) in Virginia from 1976 through 2016 in September of each year:


{% highlight r %}
# just plot data for VA

ggplot(data=d.state[state=="VA",],aes(x=date,y=hpa12,label=paste(percent(hpa12))))+
  geom_line(color=viridis(10)[1],size=1.1)+
  geom_point(data=tail(d.state[state=="VA",],1),color=viridis(10)[8],alpha=0.82,size=3)+
  geom_text(data=tail(d.state[state=="VA",],1),color=viridis(10)[8],alpha=0.82,size=3,hjust=0,vjust=-1)+
  scale_y_continuous(label=percent)+
  coord_cartesian(xlim=c(as.Date("1976-01-01"),as.Date("2017-12-31")))+
  theme_minimal()+labs(x="",y="",title="Annual percentage change in house prices",
                       subtitle="Virginia in September",
                       caption="@lenkiefer Source: Freddie Mac House Price Index")+
  theme(plot.caption=element_text(hjust=0))
{% endhighlight %}

![plot of chunk va-plot-recursion-dec4-2016,](/img/Rfig/va-plot-recursion-dec4-2016,-1.svg)

House prices in Virginia are growing by just under 3 percent year-over-year in September 2016, up from the lows of 2009, but below the middle part of last decade and also a bit below the long-run average.

Let's construct a simple times series forecasting model for house prices (see my note above, this is just for illustration) based onthe history of HPA.  There are many models we could use, but one of the simplest is an Autoregressive Model.We can code this pretty easily in R, but working with dates can be tricky.  I've tried a couple things, but [xts](https://cran.r-project.org/web/packages/xts/index.html) works for me.

We use a simple [autoregressive model](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/ar.ols.html) fit to up to two lags of HPA.


{% highlight r %}
  df<-d.state[state=="VA",]                       #just get data for VA
  va.ts<-xts(df$hpa12,df$date)                    #create xts time series of hpa12 with data
  va.out<-ar.ols(va.ts,order.max=2)               #construct ar model
  va.out
{% endhighlight %}



{% highlight text %}
## 
## Call:
## ar.ols(x = va.ts, order.max = 2)
## 
## Coefficients:
##       1        2  
##  0.9019  -0.2672  
## 
## Intercept: -0.0009794 (0.00611) 
## 
## Order selected 2  sigma^2 estimated as  0.001456
{% endhighlight %}

There's quite a bit of persistence in the HPA series, reflected in the coefficients.  We might be concerned about stationarity with this series.  We can use some nice [functions from Rob Hyndman](http://robjhyndman.com/hyndsight/arma-roots/) to check:


{% highlight r %}
source("code/roots.R")  #load AR roots
# see http://robjhyndman.com/hyndsight/arma-roots/ 
# for code
plot.armaroots(arroots(va.out))
{% endhighlight %}

![plot of chunk va-ar-recursion-2-dec4-2016,](/img/Rfig/va-ar-recursion-2-dec4-2016,-1.svg)

Now we can use this simple AR model to forecast house prices.  The code below will stack the predictions to the input data. Then we'll make a simple forecast plot:
  

{% highlight r %}
  va.p<-predict(va.out, n.ahead = 4)              #forecasts from ar model
  va.p.ts<-xts(va.p$pred,
                 seq(max(df$date)+years(1),
                     max(df$date)+years(4),"1 year"))
  va.ts<-rbind(va.ts,va.p.ts)
  f.data<-data.frame(date=index(va.ts), 
                     hpa12=coredata(va.ts))
    f.data$f<-ifelse(f.data$date>max(df$date),"forecast","actual")

    #plot forecast:
    ggplot(data=f.data,aes(x=date,y=hpa12,label=paste(percent(hpa12)),color=f,linetype=f))+
      geom_line(size=1.1)+
      scale_color_viridis(name="",discrete=T,end=0.75,direction=1)+
      geom_point(data=tail(f.data,1),color=viridis(10)[8],alpha=0.82,size=3)+
      geom_text(data=tail(f.data,1),color=viridis(10)[8],alpha=0.82,size=3,hjust=0,vjust=-1)+
      scale_y_continuous(label=percent)+
      coord_cartesian(xlim=c(as.Date("1976-01-01"),as.Date("2021-12-31")))+
      theme_minimal()+labs(x="",y="",title="Annual percentage change in house prices",
                           subtitle="Virginia in September, dotted line forecast from AR(2) model",
                           caption="@lenkiefer Source: Freddie Mac House Price Index")+
      guides(linetype=F)+
      theme(plot.caption=element_text(hjust=0),
            legend.position="top")
{% endhighlight %}

![plot of chunk va-ar-recursion-3-dec4-2016,](/img/Rfig/va-ar-recursion-3-dec4-2016,-1.svg)

Okay things look reasonable. These forecasts are sort of dumb, they don't account for inflation or other factors, but based on the history of house prices they aren't totally outlandish. What's a little more interesting is to consider if we rolled back the clock, what these simple forecasts would have looked like.

Again, just using Virginia as an example, we can go back for each year since 1985, fit a forecasting model on history up to that point in time, and then project forward a few years. We could do it with a loop, but instead we'll use Hadley's approach described in Plotcon and store the results of each estimate as a dataframe nested inside a data frame. This structure will be more compact, and also make plotting with ggplot quite simple, no explicit loops involved.

### Build a function

To get this to work we're going to require a function with two inputs. First, it will take the name of state and second it will take a maximum date. Then it will construct the forecast for that state up to that time (as we did for Virginia above) and stack the forecasts.



{% highlight r %}
fcst.d3<-function(s,dmax) { 
  #subset data based on state s and up to date dmax:
  df<-d.state[state==s & date<=dmax,c("state","date","hpa12"),with=F]
  test.ts<-xts(df$hpa12,df$date)
  test.out<-ar(test.ts,order.max=2)
  test.p<-predict(test.out, n.ahead = 4)
  test.p.ts<-xts(test.p$pred,seq(max(df$date)+years(1),max(df$date)+years(4),"1 year"))
  test.ts<-rbind(tail(test.ts,1),test.p.ts)
  f.data<-data.frame(state=s,date=index(test.ts), hpa12=coredata(test.ts))
  f.data$f<-ifelse(f.data$date>max(df$date),"forecast","actual")
  return(f.data)
}
{% endhighlight %}

Let's use this function by adding a forecast based on data up to September 2005 to our original plot for Virginia:



{% highlight r %}
# Examine forecast output
htmlTable(fcst.d3("VA","2005-09-01") %>% map_if(is.Date, as.character,format="%b-%Y") %>% map_if(is.numeric, round,3) %>%as.data.frame() , col.rgroup = c("none", "#F7F7F7"),caption="Virginia house price forecasts\nfrom simple AR(2) model",
          tfoot="Source: Freddie Mac House Price Index\nAR(2) forecast fit on data through Sep 2005")
{% endhighlight %}

<table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr><td colspan='4' style='text-align: left;'>
Virginia house price forecasts
from simple AR(2) model</td></tr>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>date</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpa12</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>state</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>Sep-2005</td>
<td style='text-align: center;'>0.179</td>
<td style='text-align: center;'>VA</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-2006</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.14</td>
<td style='background-color: #f7f7f7; text-align: center;'>VA</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>Sep-2007</td>
<td style='text-align: center;'>0.114</td>
<td style='text-align: center;'>VA</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep-2008</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.097</td>
<td style='background-color: #f7f7f7; text-align: center;'>VA</td>
</tr>
<tr>
<td style='border-bottom: 2px solid grey; text-align: left;'>5</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>Sep-2009</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>0.086</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>VA</td>
</tr>
</tbody>
<tfoot><tr><td colspan='4'>
Source: Freddie Mac House Price Index<br>
AR(2) forecast fit on data through Sep 2005</td></tr></tfoot>
</table>

{% highlight r %}
# Create plot

ggplot(data=d.state[state=="VA",],aes(x=date,y=hpa12,label=paste(percent(hpa12))))+
  geom_line(color=viridis(10)[1],size=1.1)+
  geom_line(data=fcst.d3("VA","2005-09-01"), #use forecast function data
            linetype=2,color=viridis(10)[4],size=1.1)+
  geom_point(data=tail(d.state[state=="VA",],1),color=viridis(10)[1],alpha=0.82,size=3)+
  geom_text(data=tail(d.state[state=="VA",],1),color=viridis(10)[1],alpha=0.82,size=3,hjust=0,vjust=-1)+
  scale_y_continuous(label=percent)+
  coord_cartesian(xlim=c(as.Date("1976-01-01"),as.Date("2017-12-31")))+
  theme_minimal()+labs(x="",y="",title="Annual percentage change in house prices",
                       subtitle="Virginia in September, dotted line forecast from AR(2) model fit on data through 2005",
                       caption="@lenkiefer Source: Freddie Mac House Price Index")+
  theme(plot.caption=element_text(hjust=0))
{% endhighlight %}

![plot of chunk va-plot-recursion-5-dec4-2016,](/img/Rfig/va-plot-recursion-5-dec4-2016,-1.svg)

Here we can see that based on the history of hpa, a simple model would have expected some mean reversion back towards a long-run average, as depicted by the tentacle extending from the plot.  What would it look like if we added a tentacle for each year?  We could do that through a loop, or use the nesting described in Hadley's talk.  Let's try that:

### Nesting

The function below will allow us to next each prediction in our dataframe.  For now, we'll restrict the data just to Virginia, but soon we'll add in all 50 states plus D.C.  Turns out it just takes a couple lines of code!


{% highlight r %}
fcsts.va<- d.state[year(date)>1984 & state=="VA", c("date","state","hpa12"),with=F] %>% 
           mutate(fcst=map2(state,date,fcst.d3))
head(fcsts.va,5)
{% endhighlight %}



{% highlight text %}
##          date state      hpa12         fcst
## 1: 1985-09-01    VA 0.05019297 <data.frame>
## 2: 1986-09-01    VA 0.06988367 <data.frame>
## 3: 1987-09-01    VA 0.10303571 <data.frame>
## 4: 1988-09-01    VA 0.09598973 <data.frame>
## 5: 1989-09-01    VA 0.07379588 <data.frame>
{% endhighlight %}
Now we have a data frame with a set of data frames (one for each date) corresponding to forecasts beginning at the date and extending 4 years.

Now we can easily construct our tentacle plot. We use the *unnest()* function to expand the data.



{% highlight r %}
p.data<-unnest(fcsts.va,fcst,.sep=".") #unnest the data for plotting
ggplot()+  
  geom_path(aes(x=fcst.date,y=fcst.hpa12,color=factor(year(date))),
                     data=unnest(fcsts.va,fcst,.sep=".")
            ,linetype=2 )    +
  
      geom_path(aes(x=date,y=hpa12),
                data=d.state[year(date)>1984 & state=="VA", c("date","state","hpa12"),with=F]
                ,size=1.05)+
  theme_minimal()+theme(legend.position="none")+  scale_y_continuous(label=percent)+
  labs(x="",y="",title="Annual percentage change in house prices",
                       subtitle="Virginia in September, dotted lines forecast from AR(2) model fit",
                       caption="@lenkiefer Source: Freddie Mac House Price Index\nAR(2) forecast fit on data up to point where solid line and dotted lines diverge")+
  theme(plot.caption=element_text(hjust=0))+

  coord_cartesian(xlim=c(as.Date("1985-01-01"),max(p.data$fcst.date)),ylim=c(-.1,.2))
{% endhighlight %}

![plot of chunk va-plot-recursion-7-dec4-2016,](/img/Rfig/va-plot-recursion-7-dec4-2016,-1.svg)

We can also roll over each state:


{% highlight r %}
fcsts.st<- d.state[year(date)>1984, c("date","state","hpa12"),with=F] %>% 
           mutate(fcst=map2(state,date,fcst.d3))
           
p.data2<-unnest(fcsts.va,fcst,.sep=".")

ggplot()+  
  geom_path(aes(x=fcst.date,y=fcst.hpa12,color=factor(year(date))),
                     data=unnest(filter(fcsts.st,! state %in% c("DC","US.NSA","US.SA")),fcst,.sep=".")
            ,linetype=2,size=.75 )    +
  
      geom_path(aes(x=date,y=hpa12),
                data=d.state[year(date)>1984 & ! state %in% c("DC","US.NSA","US.SA") , c("date","state","hpa12"),with=F]
                ,size=.85)+
  theme_minimal()+theme(legend.position="none")+  scale_y_continuous(label=percent)+
  labs(x="",y="",title="Annual percentage change in house prices",
                       subtitle="Dotted lines forecast from AR(2) model fit",
                       caption="@lenkiefer Source: Freddie Mac House Price Index\nAR(2) forecast fit on data up to point where solid line and dotted lines diverge")+
  theme(plot.caption=element_text(hjust=0))+
  facet_wrap(~state,ncol=5)
{% endhighlight %}

![plot of chunk va-plot-recursion-8-dec4-2016,](/img/Rfig/va-plot-recursion-8-dec4-2016,-1.svg)

And we can make a gif looping through a few key states:

<img src="{{ site.url }}/img/charts_dec_4_2016/hpi tentacles.gif" alt="tentacle gifs"/>

# Going forward

This type of approach could be quite useful for a number of other applications. The *map* functions from *purrr* have a lot of potential uses I'm looking forward to trying out in the future. So far, I haven't blown up my computer with an endless loop. If I can keep it that way, I'll follow up in this space with some other applications.
