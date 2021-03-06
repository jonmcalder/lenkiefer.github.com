---
layout: post
title: "Bivariate choropleth maps with R"
author: "Len Kiefer"
date: "2017-04-24"
summary: "R statistics rstats mortgage rates dataviz"
group: navigation
theme :
  name : lentheme
---

**NOTE: After I posted this (like within 5 minutes) I found [this post](http://rpubs.com/apsteinmetz/prek) which also constructs bivariate chropleths in R.**

IN THIS POST I WANT TO REVISIT SOME MAPS I MADE LAST YEAR.  At that time, I was using [Tableau](https://www.tableau.com) to create [choropleth](https://en.wikipedia.org/wiki/Choropleth_map) maps, but in this post I want to reimagine the maps and make them in [R](https://www.r-project.org/).

Last year [in this post]({% post_url 2016-05-22-population-growth-housing-supply-and-house-prices %}) we looked at the relationship between population growth and the growth in housing units from 2010 to 2015. While the Census [has released](https://www.census.gov/newsroom/press-releases/2017/cb17-tps38-population-estimates-single-year-age.html) national level estimates for population, the county level estimates are not updated yet. However, you [can get](https://www.census.gov/programs-surveys/popest/data/tables.html) estimates for 2010 through 2015, which is what I used last year and will use again this year.

# Bivariate choropleth

In last year's post I wanted to compare population growth to expansion of the housing supply. I constructed a ratio of the growth in population from 2010 to 2015 to the growth in housing units from 2010 to 2015 by county.  Then I plotted this ratio on a diverging color scheme.

Instead, we could use a bivariate color scale. If you are into reading, see an extended discussion [here](http://www.joshuastevens.net/cartography/make-a-bivariate-choropleth-map/). 

Or you could do [like me]({% post_url 2017-04-23-horizon %}). Just jump in we'll and figure it out as we go.  Here goes!



{% highlight r %}
library(tidyverse)
library(viridis)
d<-expand.grid(x=1:100,y=1:100)

ggplot(d, aes(x,y,fill=atan(y/x),alpha=x+y))+
  geom_tile()+
  scale_fill_viridis()+
  theme(legend.position="none",
        panel.background=element_blank())+
  labs(title="A bivariate color scheme (Viridis)")
{% endhighlight %}

![plot of chunk 04-24-2017-setup-1](/img/Rfig/04-24-2017-setup-1-1.svg)

We could allow this scheme to vary smoothly, or it might be easier to see what's going on, but using a discrete scale.  A 3x3 grid seems to work pretty well.


{% highlight r %}
# Add some labels
d<-expand.grid(x=1:3,y=1:3)
#dlabel<-data.frame(x=1:3,xlabel=c("X low", "X middle","X High"))
d<-merge(d,data.frame(x=1:3,xlabel=c("X low", "X middle","X high")),by="x")
d<-merge(d,data.frame(y=1:3,ylabel=c("Y low", "Y middle","Y high")),by="y")

g.legend<-
  ggplot(d, aes(x,y,fill=atan(y/x),alpha=x+y,label=paste0(xlabel,"\n",ylabel)))+
  geom_tile()+
  geom_text(alpha=1)+
  scale_fill_viridis()+
  theme_void()+
  theme(legend.position="none",
        panel.background=element_blank(),
        plot.margin=margin(t=10,b=10,l=10))+
  labs(title="A bivariate color scheme (Viridis)",x="X",y="Y")+
    theme(axis.title=element_text(color="black"))+
  # Draw some arrows:
  geom_segment(aes(x=1, xend = 3 , y=0, yend = 0), size=1.5,
               arrow = arrow(length = unit(0.6,"cm"))) +
  geom_segment(aes(x=0, xend = 0 , y=1, yend = 3), size=1.5,
               arrow = arrow(length = unit(0.6,"cm"))) 
g.legend
{% endhighlight %}

![plot of chunk 04-24-2017-setup-2](/img/Rfig/04-24-2017-setup-2-1.svg)

### Get some data

I happened to have the data in a spreadsheet called *pophous.xlsx*.  You can get estimates direct from the U.S. Cesnsus Bureau ([here for housing units](http://factfinder.census.gov/bkmk/table/1.0/en/PEP/2015/PEPANNHU/0100000US.05000.003) and [here for population](https://factfinder.census.gov/bkmk/table/1.0/en/PEP/2015/PEPANNRES/0400000US01.05000)).

I rearranged my data in a spreadsheet like this:

<img src="{{ site.url }}/img/charts_apr_24_2017/data.PNG" alt="data img"/>
  
This was of course, before I [wised up]({% post_url 2017-04-20-global-hpi-readxl %}) about [readxl](http://readxl.tidyverse.org/index.html).  But since I had these data around, we'll just start here. 


{% highlight r %}
# Load data and check

dfh<-read_excel("data/pophous.xlsx", sheet = "housing1")
dfp<-read_excel("data/pophous.xlsx", sheet = "pop1")
df<-left_join(dfh,dfp,by="fips")
df <- df %>% select(-h2010apr,-h2010base,-p2010apr,-p2010base)


htmlTable::htmlTable(rbind(head(df %>% 
                                  map_if(is_numeric,round,0) %>% 
                                  data.frame() %>% as.tbl())))
{% endhighlight %}

<!--html_preserve--><table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>fips</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2010</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2011</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2012</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2013</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2014</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2015</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2010</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2011</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2012</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2013</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2014</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2015</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>1001</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>22284</td>
<td style='text-align: center;'>22352</td>
<td style='text-align: center;'>22670</td>
<td style='text-align: center;'>22751</td>
<td style='text-align: center;'>22847</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>55253</td>
<td style='text-align: center;'>55175</td>
<td style='text-align: center;'>55038</td>
<td style='text-align: center;'>55290</td>
<td style='text-align: center;'>55347</td>
</tr>
<tr>
<td style='text-align: left;'>2</td>
<td style='text-align: center;'>1003</td>
<td style='text-align: center;'>104248</td>
<td style='text-align: center;'>104701</td>
<td style='text-align: center;'>105264</td>
<td style='text-align: center;'>106227</td>
<td style='text-align: center;'>107368</td>
<td style='text-align: center;'>108564</td>
<td style='text-align: center;'>183193</td>
<td style='text-align: center;'>186659</td>
<td style='text-align: center;'>190396</td>
<td style='text-align: center;'>195126</td>
<td style='text-align: center;'>199713</td>
<td style='text-align: center;'>203709</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>1005</td>
<td style='text-align: center;'>11827</td>
<td style='text-align: center;'>11808</td>
<td style='text-align: center;'>11841</td>
<td style='text-align: center;'>11816</td>
<td style='text-align: center;'>11799</td>
<td style='text-align: center;'>11789</td>
<td style='text-align: center;'>27341</td>
<td style='text-align: center;'>27226</td>
<td style='text-align: center;'>27159</td>
<td style='text-align: center;'>26973</td>
<td style='text-align: center;'>26815</td>
<td style='text-align: center;'>26489</td>
</tr>
<tr>
<td style='text-align: left;'>4</td>
<td style='text-align: center;'>1007</td>
<td style='text-align: center;'>8982</td>
<td style='text-align: center;'>8972</td>
<td style='text-align: center;'>8972</td>
<td style='text-align: center;'>8972</td>
<td style='text-align: center;'>8977</td>
<td style='text-align: center;'>8986</td>
<td style='text-align: center;'>22861</td>
<td style='text-align: center;'>22733</td>
<td style='text-align: center;'>22642</td>
<td style='text-align: center;'>22512</td>
<td style='text-align: center;'>22549</td>
<td style='text-align: center;'>22583</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>1009</td>
<td style='text-align: center;'>23887</td>
<td style='text-align: center;'>23875</td>
<td style='text-align: center;'>23862</td>
<td style='text-align: center;'>23841</td>
<td style='text-align: center;'>23826</td>
<td style='text-align: center;'>23817</td>
<td style='text-align: center;'>57373</td>
<td style='text-align: center;'>57711</td>
<td style='text-align: center;'>57776</td>
<td style='text-align: center;'>57734</td>
<td style='text-align: center;'>57658</td>
<td style='text-align: center;'>57673</td>
</tr>
<tr>
<td style='border-bottom: 2px solid grey; text-align: left;'>6</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>1011</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4492</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4483</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4477</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4468</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4461</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>4456</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10887</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10629</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10606</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10628</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10829</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>10696</td>
</tr>
</tbody>
</table><!--/html_preserve-->

After clearning our data are arranged with state/county [FIPS](https://www.census.gov/geo/reference/geoidentifiers.html) numbers and then variables *p2010-p2015* and *h2010-h2015* corresponding to population estimates for each year 2010-2015 and housing units for each year 2010-2015 respectively.

The FIPS numbers are stored as numbers (thanks Excel!) and we'd like to rearrange the data, so let's do some tidying.


{% highlight r %}
# gather up columns by fips
df2<-df %>% gather(var,value,-fips)

# Create a type varialbe (h for housing, p for population) & year variable
df2<-df2 %>% mutate( type=substr(var,1,1), year=as.numeric(substr(var,2,5)))

# spread back out housing units and population
df3 <- df2 %>% select(fips,type,year,value) %>% spread(type,value)

# turn fips into string by padding with a 0
df3<-mutate(df3, fips=str_pad(fips, 5, pad = "0"))

# comput housing units and population in 2010
df3.sums<-df3 %>% group_by(fips) %>%  summarize(h2010= h[sum(year==2010)],
                                           p2010= p[sum(year==2010)])
df3<-left_join(df3,df3.sums,by="fips")

# create variables for housing unites (hr) and population (pr) relative to 2010
# also create ratio 
df3 <- df3 %>% mutate(hr=h/h2010, pr=p/p2010, ratio=(h/h2010)/(p/p2010))

# Check it out
htmlTable::htmlTable(rbind(head(df3 %>% 
                                  map_if(is_numeric,round,0) %>% 
                                  data.frame() %>% as.tbl())))
{% endhighlight %}

<!--html_preserve--><table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>fips</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>year</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>h2010</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>p2010</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hr</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>pr</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>ratio</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>01001</td>
<td style='text-align: center;'>2010</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
</tr>
<tr>
<td style='text-align: left;'>2</td>
<td style='text-align: center;'>01001</td>
<td style='text-align: center;'>2011</td>
<td style='text-align: center;'>22284</td>
<td style='text-align: center;'>55253</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>01001</td>
<td style='text-align: center;'>2012</td>
<td style='text-align: center;'>22352</td>
<td style='text-align: center;'>55175</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
</tr>
<tr>
<td style='text-align: left;'>4</td>
<td style='text-align: center;'>01001</td>
<td style='text-align: center;'>2013</td>
<td style='text-align: center;'>22670</td>
<td style='text-align: center;'>55038</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>01001</td>
<td style='text-align: center;'>2014</td>
<td style='text-align: center;'>22751</td>
<td style='text-align: center;'>55290</td>
<td style='text-align: center;'>22152</td>
<td style='text-align: center;'>54660</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
<td style='text-align: center;'>1</td>
</tr>
<tr>
<td style='border-bottom: 2px solid grey; text-align: left;'>6</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>01001</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>2015</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>22847</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>55347</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>22152</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>54660</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>1</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>1</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>1</td>
</tr>
</tbody>
</table><!--/html_preserve-->

Now things are looking a lot better.

Let's filter to just 2015 and get ready to make our map.  

First, for housing units and population (both relative to 2010) we'll compute terciles (divide the range into 3 parts each with 1/3 of observations).  We'll then categorize each observation as falling within one of these terciles. Then, we'll let these terciles be ordered 1 through 3 and construct a scatterplot.



{% highlight r %}
d2015<-filter(df3,year==2015)

# compute quantiles (dividing into 3rds) 
h.v<-quantile(d2015$hr,c(0.33,0.66,1))
p.v<-quantile(d2015$pr,c(0.33,0.66,1))

d2015<- d2015 %>% mutate(y= ifelse(pr<p.v[1],1,ifelse(pr<p.v[2],2,3)) ,
                     x= ifelse(hr<h.v[1],1,ifelse(hr<h.v[2],2,3))  )  

ggplot(data=d2015,aes(x=hr,y=pr,color=atan(y/x),alpha=x+y))+
  geom_point(size=1)+  guides(alpha=F,color=F)+
  geom_hline(yintercept=p.v,color="gray20",linetype=2)+
  geom_vline(xintercept=h.v,color="gray20",linetype=2)+
  scale_color_viridis(name="Color scale")+theme_minimal()+
  theme(plot.caption=element_text(size = 9, hjust=0),
        panel.grid=element_blank()) +
  
  labs(x="Housing units in 2015 relative to 2010 (log scale)",
       y="Population in 2015 relative to 2010 (log scale)",
       caption="@lenkiefer Source: U.S. Census Bureau\nEach dot one county, lines divide (univariate) terciles")+
  # limit the rang e
  scale_x_log10(limits=c(0.95,1.05), breaks=c(h.v),
                labels=round(c(h.v),2)) +
  scale_y_log10(limits=c(0.95,1.05),breaks=c(p.v),
                labels=round(c(p.v),2)) 
{% endhighlight %}

![plot of chunk 04-24-2017-plot-1](/img/Rfig/04-24-2017-plot-1-1.svg)

Now that we've divided up the observations, let's map them!



{% highlight r %}
#################################################################################
### Map Libraries
library(ggthemes)
library(ggalt)
library(maps)
library(rgeos)
library(maptools)
library(albersusa)
library(grid)
#################################################################################

#################################################################################
# Let's load some maps:
states<-usa_composite()  #create a state map thing
smap<-fortify(states,region="fips_state")
counties <- counties_composite()   #create a county map thing
#################################################################################
#add on summary stats by county using FIPS code
counties@data <- left_join(counties@data, d2015, by = "fips")   
#################################################################################
cmap <- fortify(counties_composite(), region="fips")
#create state and county FIPS codes 
cmap$state<-substr(cmap$id,1,2)  
cmap$county<-substr(cmap$id,3,5)
cmap$fips<-paste0(cmap$state,cmap$county)

#### Make a legend
g.legend<-
  ggplot(d, aes(x,y,fill=atan(y/x),alpha=x+y,label=paste0(xlabel,"\n",ylabel)))+
  geom_tile()+
  scale_fill_viridis()+
  theme_void()+
  theme(legend.position="none",
        axis.title=element_text(size=5),
        panel.background=element_blank(),
        plot.margin=margin(t=10,b=10,l=10))+
  theme(axis.title=element_text(color="black"))+ 
  labs(x="Housing unit growth",
       y="Population growth")


gmap<-
ggplot() +
  geom_map(data =cmap, map = cmap,
           aes(x = long, y = lat, map_id = id),
           color = "#2b2b2b", size = 0.05, fill = NA) +
  geom_map(data = filter(counties@data,year==2015), map = cmap,
           aes(fill =atan(y/x),alpha=x+y, map_id = fips),
           color = "gray50") +
  #add black state borders (just to see if things are working)
  geom_map(data = smap, map = smap,
           aes(x = long, y = lat, map_id = id),
           color = "black", size = .5, fill = NA) +
  theme_map(base_size = 12) +
  theme(plot.title=element_text(size = 16, face="bold",margin=margin(b=10))) +
  theme(plot.subtitle=element_text(size = 14, margin=margin(b=-20))) +
  theme(plot.caption=element_text(size = 9, margin=margin(t=-15),hjust=0)) +
 # scale_fill_gradient(low="red",high="blue")
  scale_fill_viridis()+guides(alpha=F,fill=F)+
  labs(caption="@lenkiefer Source: U.S. Census Bureau",
       title="Housing units and Population growth 2010-2015",
       subtitle="Bivariate choropleth")
  
vp<-viewport(width=0.24,height=0.24,x=0.58,y=0.14)

print(gmap)
print(g.legend+labs(title=""),vp=vp)
{% endhighlight %}

![plot of chunk 04-24-2017-plot-2](/img/Rfig/04-24-2017-plot-2-1.svg)

### So what?

Well, that was pretty neat.  

I think [you probably should](http://www.stat.columbia.edu/~gelman/research/published/allmaps.pdf) be careful about interpreting these maps, particularly when you have some counties that might not be estimated as well as others.  Still, the ability to display two variables on one map might be handy sometime.

In the past I compared [employment and house prices]({% post_url 2017-02-01-emp-trends %}) and a bivariate choropleth might be a good option for comparing how these variables evolve.  I've also go some research going looking at some other aspects of the housing market where such a plot might at least be useful at the data exploration stage.  

How could it work for you?

