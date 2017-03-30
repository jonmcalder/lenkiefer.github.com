---
layout: post
title: "Resampling"
author: "Len Kiefer"
date: "2017-03-29"
summary: "R statistics dataviz housing mortgage data"
group: navigation
theme :
  name : lentheme
---
  
THIS PAST MONTH HAS BEEN BUSY.  People have been traveling, I've been traveling, kids have been sick, and we've had the March Madness basketball keeping me occupied.  Today I wanted to just explore a little analysis I've put together on resampling.

Because *reasons* I've recently been interested in sample sizes and how quickly certain estimates might converge.  

There is of course, a vast literature on this topic. But armed with powerful computers maybe we can avoid too much mathy work and try to simulate our way through some problems. 

## Setup

For this exercise I want to keep things simple.  Let's imagine that we have a sample drawn from an **independent and identically distributed** (i.i.d.) Normal distribution. We'll assume that our original sample is 100 observations and we're interested in the properties of the mean.

Per usual we'll use R to do our analysis.  And because we'll be making up our data we won't need to worry about importing data.  Usually I use the [data.table](https://cran.r-project.org/web/packages/data.table/) package, but today I'm going to try to use the [tidyverse](http://tidyverse.org/).

To keep sanity, we'll need to start out after loading our libraries by setting the seed and drawing 100 observations:


{% highlight r %}
library(tidyverse,quietly=T,warn.conflicts=F)
set.seed(03292017)
x<-rnorm(100)
{% endhighlight %}

And let's look at the data:


{% highlight r %}
summary(x)
{% endhighlight %}



{% highlight text %}
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## -2.40900 -0.70900 -0.04204 -0.03664  0.76350  1.94400
{% endhighlight %}



{% highlight r %}
ggplot(data=data.frame(x),aes(x))+
  geom_histogram(aes(x,..density..),binwidth=.25,alpha=0.75,color="black")+
  stat_function(fun=dnorm,color="red",size=1.1)+
  theme_minimal()+
  labs(title="Histogram of 100 draws of i.i.d. Normal(0,1)",
       subtitle="Red line normal density")
{% endhighlight %}

![plot of chunk mar-29-2017-data-setup-2](/img/Rfig/mar-29-2017-data-setup-2-1.svg)

Well okay.  Now what are we going to do with it?

## Reducing the sample size

Let's imagine that collecting these data was expensive so we were interested in knowing how well we could approximate them with some `n < N` where `N` was our original sample size (100) and `n` is some smaller number.

Of course, if we knew the distribution of the data we could derive it analytically.  For an i.i.d. standard normal distribution the standard deviation of our sample mean should be $\frac{1}{\sqrt{n}}$ .

But if we didn't know the true underlying distribution we might try to estimate it through resampling.  The idea, would be to draw random samples (or subsamples) of our draw and see how estimates of the mean varied across draws.

What we will do is draw 5,000 random samples from our original sample of size 1,2,...100.  We'll end up with `5000*100=500,000` samples.


{% highlight r %}
################################################
# Create a function to draw samples
################################################

mysamp=function(n=100,  # subsample size
                r=F,    # replace after drawing
                in.x=x  # input data
                )
  {return(sample(in.x,size=n,replace=r))}

################################################
# Initialize a big vector to hold all our stuff
# Use tbl() so we can use piping
################################################

id<-seq(1,100)   # draws from size n=1 to n=100
idn<-seq(1,5000) # 5,000 draws

df<-as.tbl(expand.grid(n=id,idn=idn))

## Now use map to add draws:

df<- df %>%
  mutate(samp.nr = map(.x=n,.f=mysamp,r=F),       # sample without replacement
         samp.wr = map(.x=n,.f=mysamp,r=T) ) %>%  # sample with replacement
  mutate(mean.nr = map(samp.nr,mean),
         mean.wr = map(samp.wr,mean)) %>%
  unnest(mean.nr,mean.wr)                        # unnest so we don't have lists
{% endhighlight %}

Okay, that was fun.  We've got a giant set of resamples.  Now we can use some [dplyr](https://cran.r-project.org/web/packages/dplyr/) to summarize the data.


{% highlight r %}
df <- group_by(df, n)  # group by n, the size of each subsample

df2 <- df %>%
  summarize(count=n(),
            sd.nr=sd(mean.nr),
            sd.wr=sd(mean.wr)) %>%
  mutate(
    sd.dg= 1/sqrt(n),
    e.nr = abs(sd.nr - 1/sqrt(n)),
    e.wr = abs(sd.wr - 1/sqrt(n)))
{% endhighlight %}

Now let's compare the theoretical standard deviation to the estimates from our resamples.




{% highlight r %}
ggplot(data=df2,aes(x=n,y=sd.dg))+geom_line(aes(color="Theoretical"),linetype=2)+
  geom_line(aes(y=sd.nr,color="Subsample without replacement"),size=1.1)+
  geom_line(aes(y=sd.wr,color="Subsample with replacement"),size=1.1,linetype=3)+
  scale_color_discrete(name="Draw based on sample")+
  theme_minimal()+
  theme(legend.position="top",plot.caption=element_text(hjust=0))+
  labs(x="Sample size n (resampled from sample of size N=100)",
       y=expression(paste(hat(sigma)[bar(x)[n]])),
       subtitle=expression(paste("Sample mean: ",hat(sigma)[bar(x)[n]]," and theoretical standard deviation: ",sigma[n]," =", frac(1,sqrt(n)))),
       title="Approximating the standard deviation of sample mean")
{% endhighlight %}

![plot of chunk mar-29-2017-data-plot-1](/img/Rfig/mar-29-2017-data-plot-1-1.svg)

This shows that for smaller `n` the resampling approaches approximate the theoretical standard deviation pretty well, but as n approaches `N` the dependence created by resampling without replacement causes that approximation to perform worse.

We might be able to see that better by plotting the distributions.


{% highlight r %}
# Function for multiple plots via
# http://www.cookbook-r.com/Graphs/Multiple_graphs_on_one_page_(ggplot2)/

# Multiple plot function
#
# ggplot objects can be passed in ..., or to plotlist (as a list of ggplot objects)
# - cols:   Number of columns in layout
# - layout: A matrix specifying the layout. If present, 'cols' is ignored.
#
# If the layout is something like matrix(c(1,2,3,3), nrow=2, byrow=TRUE),
# then plot 1 will go in the upper left, 2 will go in the upper right, and
# 3 will go all the way across the bottom.
#
multiplot <- function(..., plotlist=NULL, file, cols=1, layout=NULL) {
  library(grid)
  
  # Make a list from the ... arguments and plotlist
  plots <- c(list(...), plotlist)
  
  numPlots = length(plots)
  
  # If layout is NULL, then use 'cols' to determine layout
  if (is.null(layout)) {
    # Make the panel
    # ncol: Number of columns of plots
    # nrow: Number of rows needed, calculated from # of cols
    layout <- matrix(seq(1, cols * ceiling(numPlots/cols)),
                     ncol = cols, nrow = ceiling(numPlots/cols))
  }
  
  if (numPlots==1) {
    print(plots[[1]])
    
  } else {
    # Set up the page
    grid.newpage()
    pushViewport(viewport(layout = grid.layout(nrow(layout), ncol(layout))))
    
    # Make each plot, in the correct location
    for (i in 1:numPlots) {
      # Get the i,j matrix positions of the regions that contain this subplot
      matchidx <- as.data.frame(which(layout == i, arr.ind = TRUE))
      
      print(plots[[i]], vp = viewport(layout.pos.row = matchidx$row,
                                      layout.pos.col = matchidx$col))
    }
  }
}

# Make a plot to make our plots

myplot=function(N=30)
{
  return(
    ggplot(data=filter(df,n==N), aes(mean.nr-mean(x)))+
      geom_density(aes(fill="Without replacement"),linetype=2,alpha=0.25,color="black",size=1.1)+
      geom_density(aes(mean.wr-mean(x),fill="With replacement"),linetype=1,size=1.1,alpha=0.25)+
      geom_vline(xintercept=0,linetype=2)+
      stat_function(fun=dnorm,size=1.1,color="red",aes(fill="Normal Density"),
                    args=list(mean=0,sd=1/sqrt(N)))+
      theme_minimal()+
      theme(legend.position="top")+
      labs(x="Mean of Resample - Sample Mean",
            title=paste(N,"Draws"))
  )
}
{% endhighlight %}

Create density plots over draws of varying sample sizes:


{% highlight r %}
multiplot(myplot(25),myplot(50)+theme(legend.position="none"),
          myplot(75)+theme(legend.position="none"),myplot(99)+theme(legend.position="none"),
          layout=matrix(c(1,2,3,4), nrow=2,byrow=T))
{% endhighlight %}

![plot of chunk mar-29-2017-data-plot-3](/img/Rfig/mar-29-2017-data-plot-3-1.svg)

# Okay so what?

This post let us simulate some data and draw some plots.  We also used *dplyr* to manipulate data and the *map* function to store data inside a data frame.

We might be able to use the techniques for more sophisticated analysis in future.
