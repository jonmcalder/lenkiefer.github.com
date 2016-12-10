---
layout: post
title: "Global house price trends "
author: "Len Kiefer"
date: "2016-12-10"
summary: "R statistics forecasting house prices housing"
group: navigation
theme :
  name : lentheme
---
# Introduction

THE GREAT THING ABOUT SOCIAL MEDIA is that it helps put me in contact with people from around the world.  I started up a conversation with Twitter user @benmyers29 who shared some recent trends in Canadian housing markets.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">89% of Canadians have more than 25% equity in their homes. Avg equity ratio is 74%. Nearly 40% of Canadians have their homes fully paid off. <a href="https://t.co/mAfkO38Kg2">pic.twitter.com/mAfkO38Kg2</a></p>&mdash; Big Ben Myers (@benmyers29) <a href="https://twitter.com/benmyers29/status/806956597155131393">December 8, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

@benmyers29 pointed out that homeowner equity in Canada was quite high, with only about 1% of Canadian mortgaged residential properties underwater (where the mortgage amount is greater than the property value so homeowner equity is negative) per [this report](http://www.mortgageproscan.ca/en/site/doc/40632) referenced in the tweet.  This was in contrast to a [recent report from CoreLogic](http://www.corelogic.com/blog/authors/molly-boesel/2016/12/borrower-equity-update-third-quarter-2016.aspx#.WEwsyfkrIfl) that estimates more than 6 percent of U.S. mortgaged residential properties were underwater.

Of course, the primary driver of this is going to be house prices, so I thought it would be helpful to look at some trends in international house prices.  In this post I'll share some charts I made (along with [R](https://www.r-project.org/) code) to help visualize the trends.

## Get data on house prices

First we're going to need to gather some data on house prices.  Fortunately, the [Dallas Federal Reserve Bank](http://www.dallasfed.org/index.cfm) has compiled statistics on [international house price trends](http://www.dallasfed.org/institute/houseprice/). Fed researchers have gone through the hard work of collecting data for many countries and harmonizing the series so they are more easily comparable.  Read about their hard work and the details [here.](http://www.dallasfed.org/assets/documents/institute/wpapers/2011/0099.pdf)

The data are available in a convenient spreadsheet ([2016Q2 data](http://www.dallasfed.org/assets/documents/institute/houseprice/hp1602.xlsx)), and they even posted some R code to import it.

# Comparing house price trends

First, let's just look at nominal house price trends by country.  "Aggregate" is a purchase-power parity GDP weighted composition of the countries in the series. Let's take a look:


![plot of chunk fig-global-viz1-12-10-2016](/img/Rfig/fig-global-viz1-12-10-2016-1.svg)


A lot going on... let's just compare Canada to the US:

![plot of chunk fig-global-viz2-12-10-2016](/img/Rfig/fig-global-viz2-12-10-2016-1.svg)

Here we can see that since 2005, US prices are only up slightly, while Canadian prices are almost double (index value of 2000).  That alone goes a long way to explaining why so many more Americans are underwater relative to Canadians.  In fact, given the Canadian house price path, I bet the 1% figure is rounded up.

# Trends in real prices

Comparing nominal price trends can be misleading if inflation differs significantly across countries.  Because mortgage debt is typically not inflation-adjusted, comparing nominal prices is appropriate for thinking about negative equity. But how about if we wanted to know how house prices are evolving relative to other goods (something I've [written about for the U.S. recently]({% post_url 2016-11-30-hpi-gif %}))?

Fortunately the Dallas Fed has included both a real (inflation-adjusted) house price series for each country along with estimates of personal disposable income.  Let's examine trends in these variables. First, let's recreate the panel plot for real house price trends:

![plot of chunk fig-global-viz3-12-10-2016](/img/Rfig/fig-global-viz3-12-10-2016-1.svg)

We can see quite a bit of differences across countries.  Trends in the log-level of the index might not be the best way to visualize these data.  Let's instead look at 3-year percent changes in real house prices by country:

![plot of chunk fig-global-viz4-12-10-2016](/img/Rfig/fig-global-viz4-12-10-2016-1.svg)

Now I think we can better see what's going on across the globe. Many countries saw real house prices decline during the Great Recession, but not all are recovering the same. Let's try an alternative visualization that lets us see more history.

![plot of chunk fig-global-viz5-12-10-2016](/img/Rfig/fig-global-viz5-12-10-2016-1.svg)

This chart and the panel line chart above tells several stories.  S. Korea hit hard by late 90s financial crisis, Japan slows in 90s, US mostly positive until 2007, Germany stable, Australia, and New Zealand not as much.

# Affordability

Real house price trends don't tell us bout affordability, whether or not average households can afford to buy homes.  We also need to take into account incomes and interest rates.  Fortunately the Dallas Fed database gives us estimates of disposable income. Unfortunately it doesn't provide mortgage interest rates. Complicating things further, [typical mortgage terms differ quite a bit by country.](https://cbaweb.sdsu.edu/assets/files/research/Lea/10122_Research_RIHA_Lea_Report.pdf)

Nevertheless, we can get some idea about how housing markets are evolving relative to the rest of the economy by comparing house prices to income.

![plot of chunk fig-global-viz6-12-10-2016](/img/Rfig/fig-global-viz6-12-10-2016-1.svg)

Here we see that since 2005, house prices have outstripped personal income growth in Canada, while the opposite is true for the U.S.  Because 2005 is so close to the peak of the U.S. housing cycle, it might make sense to re-index the variables.  Here we re-index the data so that 1990 Q1 is equal to 100:

![plot of chunk fig-global-viz7-12-10-2016](/img/Rfig/fig-global-viz7-12-10-2016-1.svg)

Now we can create a small multiple for each country in the database:

![plot of chunk fig-global-viz8-12-10-2016](/img/Rfig/fig-global-viz8-12-10-2016-1.svg)

If we normalize the price-to-income ratio to be 100 at 1990 we can see how the ratio has evolved (how much space between the two lines).

![plot of chunk fig-global-viz9-12-10-2016](/img/Rfig/fig-global-viz9-12-10-2016-1.svg)

And a strip chart:

![plot of chunk fig-global-viz10-12-10-2016](/img/Rfig/fig-global-viz10-12-10-2016-1.svg)

# Wrapping up

I've looked at trends in U.S. house prices [see my series]({% post_url 2016-05-08-visual-meditations-on-house-prices %}), but haven't studied international trends as closely. In this post we took a look at some general trends in key housing market indicators from around the world. Let me leave you--for now--with this animated gif:

<img src="{{ site.url }}/img/charts_dec_10_2016/hpi compare international.gif" alt="global hpi gif"/>

# Code for plots

R code for the plots in this article can be found below.  I had to do some adjustments to the Dallas Fed spreadsheet to play nice.  I deleted comments from the worksheets, removed the spacer row between country names and the first data rows, and I added a date column.


{% highlight r %}
#load packages
library(data.table)
library(viridis)
library(tidyverse)
library(lubridate)

#load data files

hpi <- read_excel("data/dallas fed hp1602b.xlsx",sheet="HPI")
hpi$date<-as.Date(hpi$date, format="%m/%d/%Y")

rhpi <- read_excel("data/dallas fed hp1602b.xlsx",sheet="RHPI")
rhpi$date<-as.Date(rhpi$date, format="%m/%d/%Y")

pdi <- read_excel("data/dallas fed hp1602b.xlsx",sheet="PDI")
pdi$date<-as.Date(pdi$date, format="%m/%d/%Y")

rpdi <- read_excel("data/dallas fed hp1602b.xlsx",sheet="RPDI")
rpdi$date<-as.Date(rpdi$date, format="%m/%d/%Y")

# tidy data with gather()
pdi %>% gather(key=country,value=dpi ,-c(date,cycle)) %>% data.table()->pdi.dt
rpdi %>% gather(key=country,value=rpdi ,-c(date,cycle)) %>% data.table()->rpdi.dt
hpi  %>% gather(key=country,value=hpi ,-c(date,cycle)) %>% data.table()->hpi.dt
rhpi  %>% gather(key=country,value=rhpi ,-c(date,cycle)) %>% data.table()->rhpi.dt

# 1: panel plot

ggplot(data=hpi.dt[year(date)>2004 & 
                      !(country %in% c("Aggregate2"))],
       aes(x=date,y=hpi,color=country,label=country))+
  geom_line(size=1.1)+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
  facet_wrap(~country)+
  theme_minimal()+ geom_hline(yintercept=100,linetype=2)+
  scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Comparing house prices",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (2005=100, log scale)")

# 2: compare just Canada and US

ggplot(data=hpi.dt[year(date)>2004 & 
                      (country %in% c("Canada","US"))],
       aes(x=date,y=hpi,color=country,label=country,linetype=country))+
  geom_line(size=1.1)+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
 geom_text(data=hpi.dt[date==dlist[i] &
                            country %in% c("US","Canada")],
              hjust=0,nudge_x=30)+
  theme_minimal()+ geom_hline(yintercept=100,linetype=2)+
  scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Comparing house prices",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (2005=100, log scale)")

# 3: panel plot

ggplot(data=rhpi.dt[year(date)>2004 & 
                      !(country %in% c("Aggregate2"))],
       aes(x=date,y=rhpi,color=country,label=country))+
  geom_line(size=1.1)+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
  facet_wrap(~country)+
  theme_minimal()+ geom_hline(yintercept=100,linetype=2)+
  scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Comparing real house prices",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (2005=100, log scale)")


#compute 3 year (12-quarter) percent changes
rhpi.dt[,rhpa3:=(rhpi-shift(rhpi,12))/shift(rhpi,12),by=country]

# 4: real house price growth panel

ggplot(data=rhpi.dt[year(date)>2004 & 
                      !(country %in% c("Aggregate2"))],
       aes(x=date,y=rhpa3,color=country,label=country))+
  geom_line(size=1.1)+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
  facet_wrap(~country)+
  geom_hline(yintercept=0,linetype=2)+
  scale_y_continuous(labels=percent)+ 
  theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Real house price growth",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Percent change over 3 years in seasonally adjusted real index")

# 5: real hpa strip

ggplot(data=rhpi.dt[year(date)>2004 & 
                      !(country %in% c("Aggregate2"))],
       aes(x=date,y=rhpa3,color=country,label=country))+
  geom_line(size=1.1)+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
  facet_wrap(~country)+
  geom_hline(yintercept=0,linetype=2)+
  scale_y_continuous(labels=percent)+ 
  theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Real house price growth",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Percent change over 3 years in seasonally adjusted real index")

# 6:  Canada vs US real hpi and real income 1

ggplot(data=rhpi.dt[year(date)>2004 & 
                      (country %in% c("US","Canada"))],
       aes(x=date,y=rhpi,color=country,label=country))+
  geom_line(size=1.1,color=fivethirtyeight_pal()(2)[1])+
  scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"2 year"),
               date_labels="%y",limits=c(dlist[121],dlist[166]+years(1)))+
  facet_wrap(~country)+
  geom_line(size=1.1,linetype=2,color=fivethirtyeight_pal()(2)[2],
            data=rpdi.dt[year(date)>2004 &
                                      (country %in% c("US","Canada"))],aes(y=rpdi))  +
  geom_text(data=rhpi.dt[date==max(rhpi.dt$date,na.rm=T) & country %in% c("US","Canada")],
              hjust=1,nudge_x=-30,label="House Price",
              color=fivethirtyeight_pal()(2)[1],fontface="bold")+
  geom_text(data=rpdi.dt[date==max(rpdi.dt$date,na.rm=T) &
                           country %in% c("US","Canada")],
            hjust=1,nudge_x=-30,nudge_y=.01,label="Income",aes(y=rpdi),
            color=fivethirtyeight_pal()(2)[2],fontface="bold")+
  geom_hline(yintercept=100,linetype=2)+
  scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Real house prices and disposable incomes",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (2005=100, log scale)")

# 7:  Canada vs US real hpi and real income 2

# re-index data so 1990Q1 = 1, first append 1990 values:
rhpi.dt[, rhpi.90:=sum(ifelse(date==as.Date("1990-03-01"),rhpi,0),na.rm=T),by=country]
rpdi.dt[, rpdi.90:=sum(ifelse(date==as.Date("1990-03-01"),rpdi,0),na.rm=T),by=country]

ggplot(data=rhpi.dt[year(date)>1989 & 
                      (country %in% c("US","Canada"))],
       aes(x=date,y=100*rhpi/rhpi.90,color=country,label=country))+
  geom_line(size=1.1,color=fivethirtyeight_pal()(2)[1])+
  scale_x_date(date_breaks="5 year", date_labels="%y")+
  facet_wrap(~country)+
  geom_line(size=1.1,linetype=2,color=fivethirtyeight_pal()(2)[2],
            data=rpdi.dt[year(date)>1989 &
                                      (country %in% c("US","Canada"))],aes(y=100*rpdi/rpdi.90))  +
  geom_text(data=rhpi.dt[date==max(rhpi.dt$date,na.rm=T) & country %in% c("Canada")],
              hjust=1,nudge_x=-30,label="House Price",
              color=fivethirtyeight_pal()(2)[1],fontface="bold")+
  geom_text(data=rpdi.dt[date==max(rpdi.dt$date,na.rm=T) &
                           country %in% c("Canada")],
            hjust=1,nudge_x=-30,nudge_y=-20,label="Income",aes(y=rpdi*100/rpdi.90),
            color=fivethirtyeight_pal()(2)[2],fontface="bold")+
  geom_hline(yintercept=100,linetype=2)+
  #scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ 
  theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Real house prices and disposable incomes",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (1990Q1=100, log scale)")

# 8:  Panel real hpi and real income 2

ggplot(data=rhpi.dt[year(date)>1989 & country !="Croatia"], #drop Croatia as it is extreme outlier
       aes(x=date,y=100*rhpi/rhpi.90,color=country,label=country))+
  geom_line(size=1.1,color=fivethirtyeight_pal()(2)[1])+
  scale_x_date(date_breaks="5 year", date_labels="%y")+
  facet_wrap(~country)+
  geom_line(size=1.1,linetype=2,color=fivethirtyeight_pal()(2)[2],
            data=rpdi.dt[year(date)>1989& country !="Croatia" ],aes(y=100*rpdi/rpdi.90))  +
  geom_text(data=rhpi.dt[date==max(rhpi.dt$date,na.rm=T) & country %in% c("Australia")],
              hjust=1,nudge_x=-30,label="House Price",size=3,
              color=fivethirtyeight_pal()(2)[1],fontface="bold",aes(y=300))+
  geom_text(data=rpdi.dt[date==max(rpdi.dt$date,na.rm=T) &
                           country %in% c("Aggregate")],size=3,
            hjust=1,nudge_x=-30,label="Income",aes(y=200),
            color=fivethirtyeight_pal()(2)[2],fontface="bold")+
  geom_hline(yintercept=100,linetype=2)+
  scale_y_log10()+
  #scale_y_log10(breaks=seq(75,200,25),limits=c(60,225))+ 
  theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Real house prices and disposable incomes",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Seasonally adjusted index (1990Q1=100, log scale)")

# 9:  Panel of house price to income ratio

#merge income and house price data data

dt<-merge(rhpi.dt,rpdi.dt,by=c("date","country","cycle"))
dt[,ratio:=(rhpi/rhpi.90)/ (rpdi/rpdi.90)] # create ratio

ggplot(data=dt[year(date)>1989 & country !="Croatia"], #drop Croatia as it is extreme outlier
       aes(x=date,y=ratio,color=country,label=country))+
  geom_line(size=1.1,color=fivethirtyeight_pal()(2)[1])+
  scale_x_date(date_breaks="5 year", date_labels="%y")+
  facet_wrap(~country)+
  geom_hline(yintercept=1,linetype=2)+
  theme_fivethirtyeight()+
  theme(legend.position="none",plot.caption=element_text(hjust=0,size=7),plot.subtitle=element_text(face="italic"),
        axis.text=element_text(size=7.5))+
  labs(x="",y="",title="Ratio of real house prices to income",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
       subtitle="Ratio: 1990Q1 = 100")

# 10: Another strip chart


ggplot(data=dt[year(date)>1989 & 
                  country %in% c("US","Japan","UK","France","Germany","New Zealand","S. Korea","Spain","Ireland","Italy","Canada","Australia")],
       aes(x=date,y="",color=ratio,fill=ratio,label=country))+
  geom_col()+
  scale_fill_viridis(name="Ratio",discrete=F,option="A")+
  scale_color_viridis(name="Ratio",discrete=F,option="A")+
  theme_fivethirtyeight()+
  theme(legend.position="right",legend.direction="vertical")+
  facet_wrap(~country,ncol=2)+
  theme(axis.ticks.y=element_blank(),
        axis.text.y=element_blank(),
        panel.grid.major=element_blank(),
        panel.grid.minor=element_blank(),
        axis.text.x=element_text(size=6))+
  labs(x="",y="",title="Ratio of real house prices to real incomes",
             subtitle="ratio normalized so 1990Q1 = 100",
       caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/")+
  scale_x_date(date_breaks="5 years",date_labels="%Y")+
  theme(plot.title=element_text(size=14))+
  theme(axis.text.x  =element_text(size=9))+
  theme(plot.subtitle  =element_text(face="italic",size=11))+
  theme(plot.caption=element_text(hjust=0,size=7))


# 11: animated gif

dlist<-unique(hpi.dt$date) #get dates 2005 starts in 121

# make function to plot time series for sected countries:
hpi.plot<-function(i){
  ggplot(data=hpi.dt[year(date)>2004 & 
                       date<=dlist[i] &
                       country %in% c("US","Canada","Australia","New Zealand","UK")],
         aes(x=date,y=hpi,color=country,linetype=country,label=country))+
    geom_line(size=1.1)+
    scale_x_date(breaks=seq(dlist[121],dlist[166]+years(1),"1 year"),
                 date_labels="%Y",limits=c(dlist[121],dlist[166]+years(1)))+
    geom_text(data=hpi.dt[date==dlist[i] &
                            country %in% c("US","Canada","Australia","New Zealand","UK")],
              hjust=0,nudge_x=30)+
    theme_minimal()+ geom_hline(yintercept=100,linetype=2)+
    scale_y_log10(breaks=seq(75,200,25),limits=c(75,200))+ theme_fivethirtyeight()+
    theme(legend.position="none",plot.caption=element_text(hjust=0),plot.subtitle=element_text(face="italic"))+
    labs(x="",y="",title="Comparing house prices",
         caption="\n@lenkiefer Source: Dallas Federal Reserve International House Price Database: http://www.dallasfed.org/institute/houseprice/",
         subtitle="Seasonally adjusted index (2005=100, log scale)")
}
  
library(animation)
oopt = ani.options(interval = 0.15)
saveGIF({for (i in 121:166) {
  g<-hpi.plot(i)
   
  print(g)
 print(i)
  ani.pause()
}
  
  for (i2 in 1:20) {
    print(g)
    ani.pause()
  }
},movie.name="hpi compare international.gif",ani.width = 750, ani.height = 400)
{% endhighlight %}
