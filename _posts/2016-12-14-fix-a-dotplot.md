---
layout: post
title: "Let's fix a dot plot"
author: "Len Kiefer"
date: "2016-12-14"
summary: "Make an animated gif of the Fed dot plot"
group: navigation
theme :
  name : lentheme
---



IN THIS POST WE'RE GOING TO REVISE the dotplot code I [posted]({% post_url 2016-06-22-Make-a-dotplot %}) that lets you take the Federal Open Market Commitee (FOMC) [projections](https://www.federalreserve.gov/monetarypolicy/fomccalendars.htm) and turn them into an animated dotplot.

The problem is that the code I posted was for projections through June 2016 that only provided annual projections for 2016, 2017, and 2018, but the FOMC added 2019 in their September and December 2016 projections.

My original code was hard coded to only handle projections for 3 years and longer-run and now a 4th year was added.  Of course, everything broke.

### The fix

In the original code I relied on the fact that the input data, projections from a quarter, would have  the same number of rows. But adding in 2019 results in additional rows for the extra year.  For an individual plot this wouldn't be a problem, but it creates an issue for [tweenr](https://cran.r-project.org/web/packages/tweenr/index.html) because it requires the same number of rows in each data set we're interpolating.

The simple fix I came up with is to pad the datasets for March and June 2016 to account for the missing 2019 projections.  I set it up so the dots fly "down" (as opposed to up which I reserved for the [St Louis Fed president](http://www.bloomberg.com/news/articles/2016-06-17/st-louis-fed-s-bullard-claims-the-dot-missing-from-fed-estimate)).

First, as before we'll load the data which I've stored in individual text files described in my [earlier post]({% post_url 2016-06-22-Make-a-dotplot %}).


{% highlight r %}
# data are stored in text files see http://lenkiefer.com/2016/06/22/Make-a-dotplot for details
# one file for each projection (March, June, September, and December)
d3<-fread("mar2016.txt")
ylist<-unique(d3$y)
df<-data.frame(rate=numeric(),x=numeric())
for (yy in 1:length(ylist)){
  for (i in 1:length(d3[y==ylist[yy]]$x) ){
    for (j in 1:d3[y==ylist[yy] ]$count[i])
    {if (d3[y==ylist[yy]]$count[i]>0){
      myc<-j
      df1<-data.frame(rate=d3[y==ylist[yy]]$x[i],x=ifelse(d3[y==ylist[yy]]$count[i] %% 2 ==1, 
                                                          ifelse(myc %% 2 ==1,yy+(-1)^myc * (myc-1)*0.04,yy+(-1)^myc * (myc)*0.04),
                                                          yy-.02+(-1)^myc * (myc)*0.04)   )
      df<-rbind(df,df1)
    }}}}

df3<-df

d6<-fread("jun2016.txt")
xlist<-unique(d6$x)
df<-data.frame(rate=numeric(),x=numeric())
for (yy in 1:length(xlist)){
  for (i in 1:length(d6[x==xlist[yy]]$rate) ){
    for (j in 1:d6[x==xlist[yy] ]$count[i])
    {if (d6[x==xlist[yy]]$count[i]>0){
      myc<-j
      df1<-data.frame(rate=d6[x==xlist[yy]]$rate[i],x=ifelse(d6[x==xlist[yy]]$count[i] %% 2 ==1, 
                                                          ifelse(myc %% 2 ==1,yy+(-1)^myc * (myc-1)*0.04,yy+(-1)^myc * (myc)*0.04),
                                                          yy-.02+(-1)^myc * (myc)*0.04)   )
      df<-rbind(df,df1)
}}}}

df6<-df
df6<-rbind(df6,data.frame(rate=8,x=4))  #let missing dot fly away

d9<-fread("sep2016.txt")
ylist<-unique(d9$y)
df<-data.frame(rate=numeric(),x=numeric())
for (yy in 1:length(ylist)){
  for (i in 1:length(d9[y==ylist[yy]]$x) ){
    for (j in 1:d9[y==ylist[yy] ]$count[i])
    {if (d9[y==ylist[yy]]$count[i]>0){
      myc<-j
      df1<-data.frame(rate=d9[y==ylist[yy]]$x[i],x=ifelse(d9[y==ylist[yy]]$count[i] %% 2 ==1, 
                                                          ifelse(myc %% 2 ==1,yy+(-1)^myc * (myc-1)*0.04,yy+(-1)^myc * (myc)*0.04),
                                                          yy-.02+(-1)^myc * (myc)*0.04)   )
      df<-rbind(df,df1)
    }}}}

df9<-df
df9<-rbind(df9,data.frame(rate=8,x=4))


d12<-fread("dec2016.txt")
ylist<-unique(d12$y)
df<-data.frame(rate=numeric(),x=numeric())
for (yy in 1:length(ylist)){
  for (i in 1:length(d12[y==ylist[yy]]$x) ){
    for (j in 1:d12[y==ylist[yy] ]$count[i])
    {if (d12[y==ylist[yy]]$count[i]>0){
      myc<-j
      df1<-data.frame(rate=d12[y==ylist[yy]]$x[i],x=ifelse(d12[y==ylist[yy]]$count[i] %% 2 ==1, 
                                                          ifelse(myc %% 2 ==1,yy+(-1)^myc * (myc-1)*0.04,yy+(-1)^myc * (myc)*0.04),
                                                          yy-.02+(-1)^myc * (myc)*0.04)   )
      df<-rbind(df,df1)
    }}}}

df12<-df
df12<-rbind(df12,data.frame(rate=8,x=4))
{% endhighlight %}

Now that we have the data we can pad the extra rows we need for March (df3) and June (df6).


{% highlight r %}
# number of rows to pad
n.pad<-nrow(df9)-nrow(df3)

# pad the data 
df.pad<-data.frame(x=rep(3,n.pad),rate=rep(-5,n.pad))
df3<-rbind(df3,df.pad)
df6<-rbind(df6,df.pad)

# add date labels
df3$date<-factor("March 2016") 
df6$date<-factor("June 2016")  
df9$date<-factor("September 2016")
df12$date<-factor("December 2016")

# overwrite values to move the "longer run" dots from March and June
# over to where the longer run dots are in September and December

df3<-data.table(df3)[x>3.5,x:=x+1]  
df6<-data.table(df6)[x>3.5,x:=x+1]

#convert to data frames
df3<-data.frame(df3)
df6<-data.frame(df6)

#now we can tween data
tf <- tween_states(list(df12,df3,df6,df9,df12), tweenlength= 3, statelength=1, ease=rep('cubic-in-out',2),nframes=60)
tf<-data.table(tf)
{% endhighlight %}

Now, equipped with this solution we can construct our plot:


{% highlight r %}
saveGIF({for (i in 1:max(tf$.frame)) {
  g<- 
    ggplot(data=tf[.frame==i],aes(x=x,y=rate,color=date,fill=date))+
    theme_minimal()+scale_x_continuous(breaks=seq(1,5,1),labels=c("2016","2017","2018","2019","Longer Run"))+
    geom_point(shape=21,aes(color=date),alpha=0.82,size=3)+
    scale_y_continuous(limits=c(0,4.5))+
    scale_color_manual(limits=c("March 2016","June 2016","September 2016","December 2016"),values=c(viridis(10)[2],viridis(10)[4],viridis(10)[6],viridis(10)[8]))+
    scale_fill_manual(limits=c("March 2016","June 2016","September 2016","December 2016"),values=c(viridis(10)[2],viridis(10)[4],viridis(10)[6],viridis(10)[8]))+
    
     labs(y="Midpoint of target range or target level for the federal funds rate (%)",x="Horizon",
         subtitle=tf[.frame==i & rate>0]$date,
         title="FOMC participants' assessments of appropriate monetary policy:\nMidpoint of target range or target level for the federal funds rate",
         caption=label_wrap_gen(100)(caption))+
    theme(plot.title=element_text(size=14))+theme(plot.caption=element_text(hjust=0,vjust=1,margin=margin(t=10)))+
    theme(plot.margin=unit(c(0.25,0.25,0.25,0.25),"cm"))+  theme(legend.justification=c(0,0), legend.position="none")
  print(g)
  ani.pause()
  print(i)
}
},movie.name="fed_dots_2016 dec 14 2016 v2.gif",ani.width = 575, ani.height = 450)
{% endhighlight %}

Giving us:

<img src="{{ site.url }}/img/charts_dec_14_2016/fed_dots_2016 dec 14 2016 v2.gif" alt="dot v2"/>

Alternatively, we could have just dropped 2019 from the graph and things would have been easier:

<img src="{{ site.url }}/img/charts_dec_14_2016/fed_dots_2016 dec 14 2016.gif" alt="dot v2"/>
