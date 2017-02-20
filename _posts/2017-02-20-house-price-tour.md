---
layout: post
title: "A grand tour of house price trends"
author: "Len Kiefer"
date: "2017-02-20"
summary: "R statistics dataviz housing mortgage data"
group: navigation
theme :
  name : lentheme
---
  
LET US BUILD ON YESTERDAY'S POST ([LINK]({% post_url 2017-02-19-house-price-update%})) and construct more VISUALIZATIONS of house prices.  In this post, I'll include some [R](https://www.r-project.org/) code so you can play along.   
  
We are going to construct our own GRAND TOUR ([Wikipedia](https://en.wikipedia.org/wiki/Grand_Tour)) except instead of touring European antiquity, we will explore recent trends in house prices around the continental United States. But we will perhaps still pick up some culture, or at least a new [ggplot2](http://ggplot2.org/) theme.

## A new theme

For this post, we'll most ignore the [ticks out]({% post_url 2017-02-06-ticks-out %}) business from an earlier post and instead try a different theme.  We will use the newly released [hrbrthemes](https://github.com/hrbrmstr/hrbrthemes) package.

# Data carpentry

This morning, [via twitter](https://twitter.com/robinlovelace/status/833622374037721089) I came across the following essay ([LINK](https://csgillespie.github.io/efficientR/data-carpentry.html)) arguing that instead of data wrangling, munging and the like, we should describe data manipulation attempts as *Data Carpentry*.  A good idea, and it inspired me to [tweet](https://twitter.com/lenkiefer/status/833666399650328576) the following poem:




> **Data carpentry**  
*Some are expert artisans crafting custom design,*  
*Others cogs on efficient assembly line,*  
*Me: Chair wobbling. Seems just fine!*

So let us construct our wobbly chair, full of loving care.

## Get data

We are going to combine data from two sources.  First, we will get house price data from the publicly available [Freddie Mac House Price Index](http://www.freddiemac.com/finance/fmhpi/about.html) and second we'll use employment data from the [Bureau of Labor Statistics (BLS)](https://www.bls.gov/sae/data.htm).

For more details on these data see respectively:

* [Visual Meditations on House Prices Part 1: data wrangling]({% post_url 2016-05-08-visual-meditations-on-house-prices-part1 %})

* [Wrangling employment data, plotting trends]({% post_url 2017-02-01-emp-trends %}) 

Following the house price post, we'll begin with a text file with metro area house prices, and following the second we'll get employment data directly from BLS.  To make things easier, I've post a link to the updated house price text file below, and copied the employment data code below that.

### House price data

In order to make our map, we'll want to merge on the latitude and longitude of the principal city for each metro.

We are going to use the *us.cities* data that comes with the maps library.  To get the cbsa locations, we need to merge on the principal city of each metro area to the *us.cities* data.  The *us.cities* file has the latitude and longitude of many U.S. cities. 

The U.S. Census Bureau has convenient files [here](https://www.census.gov/data/tables/2015/demo/popest/total-metro-and-micro-statistical-areas.html) that will allows us to map U.S. cities to metro areas. We can grab a mapping of principal cities to CBSA and merge to the *us.cities* data. I've also added metro population (in 2015), which will be useful for sorting later.

I've also created a utility file that has the principal state and Census Region and Census Division for that principal state for each metro area.  I've called that *region.txt*.

In summary, we'll need these three files:

* File with house price data [fmhpi2016q4metro.txt]({{ site.url}}/img/charts_feb_20_2017/fmhpi2016q4metro.txt)

* File with principal cities of CBSA along with metro pop [cbsa.city.txt]({{ site.url}}/img/charts_feb_20_2017/cbsa.city.txt)

* File with regions [region.txt]({{ site.url}}/img/charts_feb_20_2017/region.txt)

Armed with these files we can run the following data preparation code:


{% highlight r %}
################################################
## Load libraries
################################################
library(tidyverse)
library(data.table)
library(viridis)  #we want pretty colors later
library(maps)
library(scales)

#### HRBRTHEMES ################################

### Run to install #############################
#devtools::install_github("hrbrmstr/hrbrthemes")
library(hrbrthemes)
library(gcookbook)
library(extrafont)
################################################

data(us.cities) # from the package maps
cbsa.data <-fread("data/cbsa.city.txt") #our first utility file
cbsa.metro<-cbsa.data[metro.micro=="Metropolitan Statistical Area"]

#create lowercase names
cbsa.metro[,nameL:=tolower(name)]
us.cities<-data.table(us.cities)[,nameL:=tolower(name)]

d<-merge(cbsa.metro,us.cities,by="nameL")
#get rid of duplicates
# see: http://stackoverflow.com/questions/15776064/r-first-observation-by-group-using-data-table-self-join
d<-d[order(-pop)]
d<-d[d[,list(row1 = .I[1]), by = list(cbsa)][,row1]]

# load house price data
dm<-fread("data/fmhpi2016q4metro.txt")
dm$date<-as.Date(dm$date, format="%m/%d/%Y")
#compute year-over-year house price growth
dm[,hpa12:=hpi/shift(hpi,12,fill=NA)-1,by=metro]

# for merging:
setkey(d,cbsa.name)
setkey(dm,metro)
dm2<-merge(dm,d,by.y="cbsa.name",by.x="metro",all.x=T)

#load regions
regions<-fread("data/region.txt")
dm2<-merge(dm2,regions,by.x="state",by.y="statecode")

# Only keep necessary columns
dm2<-dm2[,c("state","statename","division","region","metro","cbsa","lat","long","date","year","month","hpi","hpa12","metro.pop"), with=F]
{% endhighlight %}


{% highlight r %}
# Check data
library("htmlTable")
library(lubridate)
# make tables for viewing formatting dates with purr %>% operations
htmlTable(head(dm2 %>% map_if(is.Date, as.character,format="%b %d,%Y") %>% map_if(is.numeric, round,0) %>%as.data.frame() ,10), col.rgroup = c("none", "#F7F7F7"),caption="Merged house price data",
          tfoot="Source: Freddie Mac House Price Index, Census")
{% endhighlight %}

<!--html_preserve--><table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr><td colspan='15' style='text-align: left;'>
Merged house price data</td></tr>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>state</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>statename</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>division</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>region</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>metro</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>cbsa</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>lat</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>long</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>date</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>year</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>month</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpi</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpa12</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>metro.pop</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>Alaska</td>
<td style='text-align: center;'>Pacific Division</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>Anchorage, AK</td>
<td style='text-align: center;'>11260</td>
<td style='text-align: center;'>61</td>
<td style='text-align: center;'>-149</td>
<td style='text-align: center;'>Jan 01,1975</td>
<td style='text-align: center;'>1975</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>36</td>
<td style='text-align: center;'></td>
<td style='text-align: center;'>399790</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>Alaska</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pacific Division</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>Anchorage, AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>11260</td>
<td style='background-color: #f7f7f7; text-align: center;'>61</td>
<td style='background-color: #f7f7f7; text-align: center;'>-149</td>
<td style='background-color: #f7f7f7; text-align: center;'>Feb 01,1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>36</td>
<td style='background-color: #f7f7f7; text-align: center;'></td>
<td style='background-color: #f7f7f7; text-align: center;'>399790</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>Alaska</td>
<td style='text-align: center;'>Pacific Division</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>Anchorage, AK</td>
<td style='text-align: center;'>11260</td>
<td style='text-align: center;'>61</td>
<td style='text-align: center;'>-149</td>
<td style='text-align: center;'>Mar 01,1975</td>
<td style='text-align: center;'>1975</td>
<td style='text-align: center;'>3</td>
<td style='text-align: center;'>37</td>
<td style='text-align: center;'></td>
<td style='text-align: center;'>399790</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>Alaska</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pacific Division</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>Anchorage, AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>11260</td>
<td style='background-color: #f7f7f7; text-align: center;'>61</td>
<td style='background-color: #f7f7f7; text-align: center;'>-149</td>
<td style='background-color: #f7f7f7; text-align: center;'>Apr 01,1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>37</td>
<td style='background-color: #f7f7f7; text-align: center;'></td>
<td style='background-color: #f7f7f7; text-align: center;'>399790</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>Alaska</td>
<td style='text-align: center;'>Pacific Division</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>Anchorage, AK</td>
<td style='text-align: center;'>11260</td>
<td style='text-align: center;'>61</td>
<td style='text-align: center;'>-149</td>
<td style='text-align: center;'>May 01,1975</td>
<td style='text-align: center;'>1975</td>
<td style='text-align: center;'>5</td>
<td style='text-align: center;'>38</td>
<td style='text-align: center;'></td>
<td style='text-align: center;'>399790</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>6</td>
<td style='background-color: #f7f7f7; text-align: center;'>AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>Alaska</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pacific Division</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>Anchorage, AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>11260</td>
<td style='background-color: #f7f7f7; text-align: center;'>61</td>
<td style='background-color: #f7f7f7; text-align: center;'>-149</td>
<td style='background-color: #f7f7f7; text-align: center;'>Jun 01,1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>6</td>
<td style='background-color: #f7f7f7; text-align: center;'>39</td>
<td style='background-color: #f7f7f7; text-align: center;'></td>
<td style='background-color: #f7f7f7; text-align: center;'>399790</td>
</tr>
<tr>
<td style='text-align: left;'>7</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>Alaska</td>
<td style='text-align: center;'>Pacific Division</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>Anchorage, AK</td>
<td style='text-align: center;'>11260</td>
<td style='text-align: center;'>61</td>
<td style='text-align: center;'>-149</td>
<td style='text-align: center;'>Jul 01,1975</td>
<td style='text-align: center;'>1975</td>
<td style='text-align: center;'>7</td>
<td style='text-align: center;'>39</td>
<td style='text-align: center;'></td>
<td style='text-align: center;'>399790</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>8</td>
<td style='background-color: #f7f7f7; text-align: center;'>AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>Alaska</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pacific Division</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>Anchorage, AK</td>
<td style='background-color: #f7f7f7; text-align: center;'>11260</td>
<td style='background-color: #f7f7f7; text-align: center;'>61</td>
<td style='background-color: #f7f7f7; text-align: center;'>-149</td>
<td style='background-color: #f7f7f7; text-align: center;'>Aug 01,1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>1975</td>
<td style='background-color: #f7f7f7; text-align: center;'>8</td>
<td style='background-color: #f7f7f7; text-align: center;'>40</td>
<td style='background-color: #f7f7f7; text-align: center;'></td>
<td style='background-color: #f7f7f7; text-align: center;'>399790</td>
</tr>
<tr>
<td style='text-align: left;'>9</td>
<td style='text-align: center;'>AK</td>
<td style='text-align: center;'>Alaska</td>
<td style='text-align: center;'>Pacific Division</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>Anchorage, AK</td>
<td style='text-align: center;'>11260</td>
<td style='text-align: center;'>61</td>
<td style='text-align: center;'>-149</td>
<td style='text-align: center;'>Sep 01,1975</td>
<td style='text-align: center;'>1975</td>
<td style='text-align: center;'>9</td>
<td style='text-align: center;'>40</td>
<td style='text-align: center;'></td>
<td style='text-align: center;'>399790</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: left;'>10</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>AK</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Alaska</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Pacific Division</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Anchorage, AK</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>11260</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>61</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>-149</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Oct 01,1975</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>1975</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>10</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>41</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'></td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>399790</td>
</tr>
</tbody>
<tfoot><tr><td colspan='15'>
Source: Freddie Mac House Price Index, Census</td></tr></tfoot>
</table><!--/html_preserve-->

These data have geographic identifies (state=Principal state, metro=name,cbsa=cbsa code, statename=State long name, division=Census division,region=Census region, lat=latitude,long=longtitutde), and data variables (date, year, month), house price information (hpi=house price index, hpa12=year-over-year percent change in index,metro.pop=metro population in 2015).

### Testing...

We could just let that data chill for a second, but let's give that new theme a spin. Let's make a simple line chart showing the house price index for Columbus, Ohio.


{% highlight r %}
ggplot(data=dm2[metro=="Columbus, OH"],aes(x=date,y=hpi))+
  geom_line()+
  theme_ipsum()+  # try new theme
  labs(x="",y="House Price Index, (NSA, Dec 2000=100)",
       subtitle="Testing a line plot",
       title="House prices in Columbus, Ohio",
       caption="@lenkiefer Source: Freddie Mac House Price Index" )
{% endhighlight %}

![plot of chunk feb-20-2017-graph-1](/img/Rfig/feb-20-2017-graph-1-1.svg)

For whatever reason, I think the gridlines are a little too dark, so I'd rather they be *lightgray*. Let's tweak the theme and redraw with annual house price appreciation for Miami, Florida.


{% highlight r %}
ggplot(data=dm2[metro=="Miami-Fort Lauderdale-West Palm Beach, FL"],aes(x=date,y=hpa12))+
  geom_line()+
  theme_ipsum()+  # try new theme
  scale_y_percent()+  #use scale_y_percent function from hrbrtheme
  labs(x="",y="House Price Index, (NSA, Dec 2000=100)",
       subtitle="Testing a line plot",
       title="House price appreciation in Miami-Fort Lauderdale-West Palm Beach, FL",
       caption="@lenkiefer Source: Freddie Mac House Price Index" )+
  theme(panel.grid.major=element_line(color="lightgray"),
        panel.grid.minor=element_line(color="lightgray"))+
  geom_hline(yintercept=0) #darken the x axis at 0
{% endhighlight %}

![plot of chunk feb-20-2017-graph-2](/img/Rfig/feb-20-2017-graph-2-1.svg)



## Employment data

Getting the employment data is exactly as in my [wrangling employment data post](({% post_url 2017-02-01-emp-trends %})), but we'll recreate it here for completeness.


{% highlight r %}
emp.data<-fread("https://download.bls.gov/pub/time.series/sm/sm.data.54.TotalNonFarm.All")
emp.series<-fread("https://download.bls.gov/pub/time.series/sm/sm.series")

emp.list<-emp.series[industry_code==0 # get all employment
                     & data_type_code==1 # get employment in thousands
                     & seasonal=="U",]  # get seasonally adjusted data]

emp.area<-fread("https://download.bls.gov/pub/time.series/sm/sm.area",
                col.names=c("area_code","area_name","drop"))[,c("area_code","area_name"),with=F]

emp.st<-fread("https://download.bls.gov/pub/time.series/sm/sm.state",
              col.names=c("state_code","state_name","drop"))[,c("state_code","state_name"),with=F]

# merge data
emp.dt<-merge(emp.data,emp.list,by="series_id",all.y=T)

#create month variable
emp.dt=emp.dt[,month:=as.numeric(substr(emp.dt$period,2,3))]
# (this assignment is to get around knitr/data table printing error)
# see e.g. http://stackoverflow.com/questions/15267018/knitr-gets-tricked-by-data-table-assignment

# M13 = Annual average, drop it:
emp.dt<-emp.dt[month<13,]

#create date variable
emp.dt$date<- as.Date(ISOdate(emp.dt$year,emp.dt$month,1) ) 

# merge on area and state codes
emp.dt<-merge(emp.dt,emp.area,by="area_code")
emp.dt<-merge(emp.dt,emp.st,by="state_code")
emp.dt=emp.dt[,c("state_name","area_name","date","year","month","value"),with=F]

emp.dt=emp.dt[,emp:=as.numeric(value)] #convert value to numeric
# Compute year-over-year change in employment and year-over-year percent change
emp.dt=emp.dt[,emp.yoy:=emp-shift(emp,12,fill=NA),by=c("area_name","state_name")]
emp.dt=emp.dt[,emp.pc:=(emp-shift(emp,12,fill=NA))/shift(emp,12,fill=NA),by=c("area_name","state_name")]
emp.dt=emp.dt[,max.emp.st:=max(emp),by=c("state_name")]
emp.dt=emp.dt[,type:=ifelse(area_name=="Statewide","State","Metro")]

# drop states in c("Puerto Rico","Virgin Islands")
emp.dt=emp.dt[year>1989  &!(state_name %in% c("Puerto Rico","Virgin Islands")),]
{% endhighlight %}

Now that we have those data, let's make a quick plot, replicating our employment plot for Ohio metros, but using the new theme:


{% highlight r %}
#set up recessions:
recessions.df = read.table(textConnection(
  "Peak, Trough
  1973-11-01, 1975-03-01
  1980-01-01, 1980-07-01
  1981-07-01, 1982-11-01
  1990-07-01, 1991-03-01
  2001-03-01, 2001-11-01
  2007-12-01, 2009-06-01"), sep=',',
colClasses=c('Date', 'Date'), header=TRUE)
# trim down
recessions.trim = subset(recessions.df, Peak >= "1990-01-01" )
# Plot employment series for Ohio:
g1<-
  ggplot(data=emp.dt[state_name=="Ohio" & type=="Metro"& year>1989])+
  geom_rect(data=recessions.trim, aes(xmin=Peak, xmax=Trough, ymin=-Inf, ymax=+Inf), fill='gray', alpha=0.5)+
  geom_line(aes(x=date,y=emp.pc,group=area_name))+
  theme_ipsum()+
  facet_wrap(~area_name,ncol=3)+scale_y_continuous(labels=percent)+
  geom_hline(color="black",yintercept=0)+
  labs(x="",y="",title="Annual percent change in total nonfarm employment",
       subtitle="Ohio Metro Areas (NSA)",
       caption="@lenkiefer Source: U.S. Bureau of Labor Statistics")+
  theme(panel.grid.major=element_line(color="lightgray"),
        panel.grid.minor=element_line(color="lightgray"))
g1
{% endhighlight %}

![plot of chunk feb-20-2017-ohio-plot-1](/img/Rfig/feb-20-2017-ohio-plot-1-1.svg)

Okay, we've got the data we want. Now let's put it together. Things are about to get slightly interesting.

## Combining data

We want to merge our house price data in *dm2* with the employment data in *emp.dt*. Fortunately, we have a common key in metro names and for the most part names line up. However, it turns out that the metros in New England are recorded as NECTAs rather than MSAs.

There are some options, but we'll choose the hacksaw rather than the scalpel.  We're really only going to be interested in the largest metro areas, and the only large metro area in New England is Boston, so we'll take care of that and ignore the smaller metro areas for today. For our purposes the Boston NECTA and Boston MSA are close enough. For details see: [https://www.census.gov/population/metro/data/def.html](https://www.census.gov/population/metro/data/def.html).



{% highlight r %}
# replace Boston-Cambridge-Nashua, MA-NH NECTA with Boston-Cambridge-Newton, MA-NH
# FIX for Boston (approximate)
emp.dt[,area_name:=ifelse(area_name=="Boston-Cambridge-Newton, MA NECTA Division",
                "Boston-Cambridge-Newton, MA-NH",
                area_name)]

# Create common key mergind date and metro
dm2[,md:=paste(as.character(date),metro)]
emp.dt[,md:=paste(as.character(date),area_name)]
dt<-merge(dm2,emp.dt2[,c("md","emp","emp.yoy","emp.pc","type"),with=F],by="md")
{% endhighlight %}

Now that w ehave our merged data, let's create a plot showing the relationship of year-over-year house price percentage changes to employment percentage changes by metro area in December of 2016.


{% highlight r %}
ggplot(data=dt[date=="2016-12-01",],
       aes(x=emp.pc,y=hpa12,color=region,size=metro.pop))+
  geom_point(alpha=0.82)+
  theme_ipsum()+
  scale_color_ipsum() + # special theme colors
  scale_y_percent()+
  scale_x_percent()+
      geom_hline(yintercept=0,color="black")+
    guides(size=F)+
    geom_vline(xintercept=0,color="black")+
  labs(x="Annual percentage change\nin metro employment",
       y="Annual percentage change\nin metro house prices",
       title="Metros with stronger labor markets \nexperience stronger house price growth",
       subtitle="House prices and employment in December 2016",
       caption="@lenkiefer Source: Freddie Mac House Price Index, U.S. Bureau of Labor Statistics\nEach dot represents one metro area color coded by region and size based on metro area population (2015).")+
    theme(panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="bottom")
{% endhighlight %}

![plot of chunk feb-20-2017-graph-3](/img/Rfig/feb-20-2017-graph-3-1.svg)

This chart tells a host of stories. In general house prices rise with stronger employment growth (we'll soon see that's true across most time periods).  Also, there are regional differences.  While I've color coded the regions, it might not jump out, so let's use *facet_wrap* to create a small multiple by region and add a regression line.



{% highlight r %}
ggplot(data=dt[date=="2016-12-01",],
       aes(x=emp.pc,y=hpa12,color=region,size=metro.pop))+
  geom_point(alpha=0.82)+
  theme_ipsum()+
  scale_color_ipsum() + # special theme colors
  scale_y_percent()+
  scale_x_percent()+
      geom_hline(yintercept=0,color="black")+
    guides(size=F)+
    geom_vline(xintercept=0,color="black")+
  labs(x="Annual percentage change\nin metro employment",
       y="Annual percentage change\nin metro house prices",
       title="Metros with stronger labor markets \nexperience stronger house price growth",
       subtitle="House prices and employment in December 2016",
       caption="@lenkiefer Source: Freddie Mac House Price Index, U.S. Bureau of Labor Statistics\nEach dot represents one metro area color coded by region and size based on metro area population (2015).")+
    theme(panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="bottom")+
  facet_wrap(~region)+stat_smooth(method="lm",size=0.75,formula=y~x,se=F)
{% endhighlight %}

![plot of chunk feb-20-2017-graph-4](/img/Rfig/feb-20-2017-graph-4-1.svg)

Here, we can see that the positive relationship between house price growth and employment appears to be strongest in the South and West Regions.

### Animate it

Of course, this scatterplot is practically dying to be animated, so let's do it!

If you've been about, you know the drill.  For the newcomers (welcome!, consider [following me](https://twitter.com/lenkiefer)), [see this simple example]({% post_url 2016-12-19-more-tweenr-housing %}).

For smoother animations we'll use tweenr. See my earlier [post about tweenr]({% post_url 2016-05-29-improving-R-animated-gifs-with-tweenr %}) for an introduction to tweenr, and more examples [here]({% post_url 2016-05-30-more-tweenr-animations %}) and [here]({% post_url 2016-06-26-week-in-review %}). 


{% highlight r %}
library(tweenr)
library(animation)

# Function for data prepartion
# Subset on December
myf<-function(in.y){
  dt2<-subset(dt, year==in.y & month==12)
  dt2 %>% map_if(is.character, as.factor) %>% as.data.frame -> dt.out
  dt.out$year<-factor(dt.out$year)
  return(dt.out)
}

# cycle through years
my.list<-lapply(c(2016,seq(1991,2016,1)),myf)

# Tweenr functions
tf <- tween_states(my.list, tweenlength= 2, statelength=3, ease=rep('cubic-in-out',3),
                   nframes=300)

tf<-data.table(tf)

# Loop to animate:
oopt = ani.options(interval = 0.15)
saveGIF({for (i in 1:max(tf$.frame)) {
  g<-
    ggplot(data=tf[.frame==i],
           aes(x=emp.pc,y=hpa12,group=metro,color=region,size=metro.pop))+
    geom_hline(yintercept=0,color="black")+
    guides(size=F)+
    geom_vline(xintercept=0,color="black")+
    geom_point(alpha=0.82)+theme_ipsum()+  scale_color_ipsum(name="Region") +
    scale_y_continuous(label=percent,limits=c(-.41,.4),breaks=seq(-.4,.4,.1))+
    scale_x_continuous(label=percent,limits=c(-.3,.2),breaks=seq(-.3,.2,.1))+
    theme(panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="bottom")+
    labs(x="Annual percentage change in metro employment",
         y="Annual percentage change in metro house prices",
         title="Metros with stronger labor markets \nexperience stronger house price growth",
         subtitle=paste("House prices and employment in December",tf[.frame==i]$year),
         caption="@lenkiefer Source: Freddie Mac House Price Index, U.S. Bureau of Labor Statistics\nEach dot represents one metro area color coded by region and size based on metro area population (2015).")
  print(g)
  print(paste(i,"out of",max(tf$.frame)))
  ani.pause()}
},movie.name="emp hpi tween 02 20 2017.gif",ani.width = 650, ani.height = 550)
{% endhighlight %}

Running this generates:

<img src="{{ site.url }}/img/charts_feb_20_2017/emp hpi tween 02 20 2017.gif" alt="scatter gif weekly"/>

And if we simply add `facet_wrap(~region)` to the `g` function we get:

<img src="{{ site.url }}/img/charts_feb_20_2017/emp hpi tween region 02 20 2017.gif" alt="scatter gif weekly"/>

# Let's take a tour

Now that we've got our wobbly data tables let's go ahead and build up a tour.

The idea is to generate a composite plot combining a map (to tell us where we are), a scatterplot and two line plots.  Then we'll animate it.

## The composite static plot

Let's first build our static plot for a particular metro area, Washington D.C.

We'll call the [multiplot](http://www.cookbook-r.com/Graphs/Multiple_graphs_on_one_page_(ggplot2)/) function to combine ggplot graphs and wrap it in a function.


{% highlight r %}
myplot<-function(df){

#make a map (sorry no AlbersUSA today)
  g.map<-
    ggplot(df, aes(x = long, y = lat)) +
    borders("state",  colour = "grey70",fill="lightgray",alpha=0.5)+
    theme_void()+
    theme(legend.position="none",
          plot.title=element_text(face="bold",size=18))+
    geom_point(alpha=0.82,color="black",size=3)+
    labs(title="House price & employment trends",
         subtitle=head(df,1)$metro,
         caption="@lenkiefer Source: Freddie Mac House Price Index, U.S. Bureau of Labor Statistics")+
    theme(plot.caption=element_text(hjust=0))

  #house price bar  
  g.bar<-
    ggplot(data=df,aes(x=date,y=hpa12,fill=hpa12))+geom_col()+
    theme_ipsum()+
    scale_y_continuous(label=percent,limits=c(-.45,.45),breaks=seq(-.45,.45,.15))+
    scale_fill_viridis(option="B",limits=c(-.45,.45))+
    labs(x="",y="",
         title="House Price Appreciation",
         subtitle="year-over-year percent change in index")+
    theme(plot.caption=element_text(hjust=0),
          panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="none")
  
  #employment bar
  g.bar2<-
    ggplot(data=df,aes(x=date,y=emp.pc,fill=emp.pc))+geom_col()+
    theme_ipsum()+
    scale_y_continuous(label=percent,limits=c(-.11,.11),breaks=seq(-.1,.1,.05))+
    scale_fill_viridis(option="B",limits=c(-.11,.11))+
    labs(x="",y="",
         title="Employment growth",
         subtitle="year-over-year percent change metro employment")+
    theme(plot.caption=element_text(hjust=0),
          panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="none")
  # scatter
  g.scatter<-
    ggplot(data=df,aes(x=emp.pc,y=hpa12))+
    geom_point()+theme_ipsum()+
    geom_hline(yintercept=0,color="black")+
    guides(size=F)+
    geom_vline(xintercept=0,color="black")+
    geom_point(alpha=0.82)+theme_ipsum()+  scale_color_ipsum(name="Region") +
    scale_y_continuous(label=percent,limits=c(-.41,.4),breaks=seq(-.4,.4,.1))+
    scale_x_continuous(label=percent,limits=c(-.11,.11),breaks=seq(-.1,.1,.05))+
    theme(panel.grid.major=element_line(color="lightgray"),
          panel.grid.minor=element_line(color="lightgray"),
          legend.position="bottom")+
    labs(x="Annual percentage change\n in metro employment",
         y="Annual percentage change\n in metro house prices",
         title="",
         subtitle="",
         caption="")
  
    #combine
  g<-multiplot(g.map,g.scatter,g.bar,g.bar2,
               layout=matrix(c(1,2,3,4), nrow=2, byrow=TRUE))
  
  return(g)
}

# test for Washington DC
df<-subset(dt,metro=="Washington-Arlington-Alexandria, DC-VA-MD-WV")
myplot(df)
{% endhighlight %}

![plot of chunk feb-20-2017-graph-5](/img/Rfig/feb-20-2017-graph-5-1.svg)

{% highlight text %}
## NULL
{% endhighlight %}

This plot shows what you would expect, that periods of strong employment growth are generally periods of strong house price growth and vice versa.

Now, using our tweenr tricks, we can creat an animated tour:


{% highlight r %}
# sort metros by metro.pop
cbsa.list3<-d[order(-metro.pop)]$cbsa.name


myf<-function(i){
  dt2<-subset(dt,metro==d[order(-metro.pop)]$cbsa.name[i] & year>1989)
  dt2 %>% map_if(is.character, as.factor) %>% as.data.frame -> dt.out
  return(dt.out)
}

# take top 20 metros
my.list<-lapply(c(seq(1,20),1),myf)

tf <- tween_states(my.list, tweenlength= 2, statelength=3, ease=rep('cubic-in-out',3),
                   nframes=200)

tf<-data.table(tf)

oopt = ani.options(interval = 0.15)
saveGIF({for (i in 1:max(tf$.frame)) {
  g<-myplot(tf[.frame==i])
  print(g)
  print(paste(i,"out of",max(tf$.frame)))
  ani.pause()}
},movie.name="geo tour emp hpi tween 02 20 2017.gif",ani.width = 650, ani.height = 600)
{% endhighlight %}

Running this generates:

<img src="{{ site.url }}/img/charts_feb_20_2017/geo tour emp hpi tween 02 20 2017.gif" alt="tour gif"/>

### Other options

We could use [plotly](https://plot.ly/r/) and [crosstalk](https://github.com/rstudio/crosstalk) to make an interactive version of this. Indeed, that's exactly what I did in [this post *A guide to building an interactive flexdashboard*]({% post_url 2017-01-22-build-flex %}) where you'll find detailed instructions.

Perhaps you'll find this post useful, and be inspired to construct your own wobbly table with *data carpentry*.
