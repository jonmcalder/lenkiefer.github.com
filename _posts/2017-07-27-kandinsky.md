---
layout: post
title: "Mortgage Rate Kandinsky"
author: "Len Kiefer"
date: "2017-07-27"
summary: "rstats data visualizations of housing data"
group: navigation
theme :
  name : lentheme
---



THINGS ARE ABOUT TO GET A BIT MORE ABSTRACT IN THIS SPACE. Today we make some Kandinsky-style images with [R](https://www.r-project.org/).

This summer I was fortunate to spend some time at the [Pompidou Centre](https://www.centrepompidou.fr/en) in Paris. The Pompidou Centre houses the [largest collection](https://en.wikipedia.org/wiki/Centre_Georges_Pompidou) of modern art in Europe. I really enjoyed their collection of abstract and minimalist paintings.

Well, turns out we can make our own abstract-style art using a [Kandinsky R package](https://github.com/gsimchoni/kandinsky) from [Giora Simchoni](http://giorasimchoni.com/). Read about the package [here at Giora's blog](http://giorasimchoni.com/2017/07/30/2017-07-30-data-paintings-the-kandinsky-package/).

The package takes any data frame and turns it into Kandisky-style painting.

### Data and code

For these paintings I'm going to use the history of U.S. average weekly 30-year fixed rate mortgage rates from the [Freddie Mac Primary Mortgage Market Survey](http://www.freddiemac.com/pmms/).  See my earlier post for [10 amazing ways to visualize rates]({% post_url 2016-12-08-10-ways-to-visualize-rates %}) and [here for even more amazing visualizations]({% post_url 2016-12-15-more-amazing-mortgage-viz %}).

I've saved a simple text file with two columns, one with the date and one with the historical rate called *rate30yrfrm.txt* [download]({{ site.url }}/img/charts_jul_27_2017/rate30yrfrm.txt). 

Just save them in a directory (I called mine "data") and load the libraries.


{% highlight r %}
#####################################################################################
# load libraries
library(data.table) # i want to use fread and other data.table utilities later
library(kandinsky)  # available on Github at https://github.com/gsimchoni/kandinsky
#####################################################################################

#####################################################################################
# load data:
#####################################################################################

dt<-fread("data/rate30yrfrm.txt")
dt$date<-as.Date(dt$date, format="%m/%d/%Y")  # conver to date

#####################################################################################
#  draw plot
#####################################################################################
kandinsky(dt)
{% endhighlight %}

![plot of chunk 07-27-2017-kandinsky1](/img/Rfig/07-27-2017-kandinsky1-1.svg)

That's pretty fun. Let's try it for just the year 2016. Let's also add a label.


{% highlight r %}
kandinsky(dt[year(date)==2016])
  grid.text(label=paste("30-year fixed mortgage rates in 2016",
                        "\n@lenkiefer, Made using R package Kandinsky"),
            gp=gpar(fontsize=12),
            x=.95,y=0.05,just="right")
{% endhighlight %}

![plot of chunk 07-27-2017-kandinsky2](/img/Rfig/07-27-2017-kandinsky2-1.svg)

That's pretty fun. What if we want to animate it?  You know, using [tweenr](https://cran.r-project.org/web/packages/tweenr/index.html)?

See this [post about tweenr]({% post_url 2016-05-29-improving-R-animated-gifs-with-tweenr %}) for an introduction to tweenr, and more examples [here]({% post_url 2016-05-30-more-tweenr-animations %}) and [here]({% post_url 2016-06-26-week-in-review %}). 

In this post we'll do things slightly differently. Instead of using the animation package, I'm going to save the image files and then call a program, [imagemagick](https://www.imagemagick.org/script/index.php), outside of R to make the gif. 

We are going to take a subset of our data, just the weeks in 2017.  Then we'll take 4 week blocks and draw a Kandinsky plot. We'll tween between 4 week intervals and see how they evolve.


{% highlight r %}
#####################################################################################
library(tidyverse)  # we'll need some tidyverse functions
library(tweenr)
#####################################################################################

#####################################################################################
# Data stuff: 
df<-filter(dt,year(date)>2016)  #just get data for 2017
N<-nrow(df)  # get the number of weeks in our data

# Create a function to subset our data
myf4 <- function(i) {
  # take a four week interval starting at week i
  return( df[i:(i+3),] %>% select(date,rate))  
  }

# use lapply to generate the list of data sets:
# stagger from weeks 1 to N-3 by 4
my.list<-lapply(seq(1,N-3,4),myf4)  

tf <- tween_states(my.list, tweenlength= 2, statelength=3,
                   ease=rep('cubic-in-out',24),
                   nframes=120)
# Turn the tweened data fram into a data.table
tf<-data.table(tf)
#####################################################################################

#####################################################################################

pathtoyourfolder <- ""
# be sure to set pathtoyourfolder to some folder where you can store images

#####################################################################################

#####################################################################################
# Loop for animation
for (ii in 1:max(tf$.frame)) {
  file_path = paste0(pathtoyourfolder, "/plot-",5000+ii ,".png")
  # I add 5000 to the index so that image 10 comes after (not before) image 9
  # (need to pad with some leading numbers)
  png(file_path) 
  kandinsky(tf[.frame==ii])
  grid.text(label=paste("30-year rates from",
                        tf[.frame==ii,][1,]$date,"to",
                        tf[.frame==ii,][4,]$date,
                        "\n@lenkiefer, Made using R package Kandinsky"),
            gp=gpar(fontsize=12),
            x=.95,y=0.05,just="right")
  dev.off()
  print(ii)
}
#####################################################################################
{% endhighlight %}

For this to work, you'll need [imagemagick](https://www.imagemagick.org/script/index.php) on your machine. Navigate to the directory where you saved your images and run the following command in the console:

`magick convert -delay 10 loop -0 *.png kandinsky.gif`

That should compile all your images (*.png* files) into a single gif.

<img src="{{ site.url }}/img/charts_jul_27_2017/kandinsky.gif" alt="kandinsky"/>

## How could it work for you?

You can feed all different types of datasets into the `kandinsky` function. Choose some that are near and dear to you and try them out.
