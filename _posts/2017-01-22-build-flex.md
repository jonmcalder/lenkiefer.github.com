---
layout: post
title: "A guide to building an interactive flexdashboard"
author: "Len Kiefer"
date: "2017-01-22"
summary: "R statistics dataviz remix flexdashboard"
group: navigation
theme :
  name : lentheme
---




INTERACTIVE DASHBOARDS CAN BE AN EFFECTIVE WAY to explore and present data.  Recently, I have been using [flexdashboards](http://rmarkdown.rstudio.com/flexdashboard/index.html) created with [R](https://www.r-project.org/). Over January 2017 I've posted the following examples:

* [Mortgage rates viewer]({% post_url 2017-01-08-mortgage-rate-viewer %}) 

* [Year in review remix]({% post_url 2017-01-14-year-in-review-remix %}) 

* [Cross talk dashboard]({% post_url 2017-01-16-cross-talk-dashboard %}) 

* [Flexin Friday]({% post_url 2017-01-20-flexin-friday %}) 

For each of these you can get the code by clicking on the source link in the upper right corner of the visualizations at the respective links. While I tried to include helpful comments in the code it might be hard to build your own from scratch. While the [documentation](http://rmarkdown.rstudio.com/flexdashboard/using.html) for flexdashboards is good and there are several examples in the [gallery](http://rmarkdown.rstudio.com/flexdashboard/examples.html) you can learn from, I thought I'd take some time to walk through the construction of a new flexdashboard.

## The Plan

We'll build an interactive flexdashboard to explore trends in house prices across several areas. In this example I'm going to try to show you the following:

* How to set up a multipage dashboard 
    -use a storyboard on one page
* How to use [plotly](https://plot.ly/r/) to create an interactive chart
* How to combine plotly with [crosstalk](https://github.com/rstudio/crosstalk) to add more interactions
* How to animate a plotly chart

### The data

For this project I'm going to revisit the house price data [we used in our house price meditations]({% post_url 2016-05-08-visual-meditations-on-house-prices %}). These house price data allow us to explore data that vary over both space and time, and that have interesting hierarchies we will explore.

While data wrangling is an important subject (see for example, [this post on wrangling house price data](({% post_url 2016-05-08-visual-meditations-on-house-prices-part1 %}))), I don't want to distract from the dashboard.  For this post, we'll begin with our data compiled.  

The data structure is fairly simple.  We have columns corresponding to date, metro name, primary state for the metro area (the state of the metro's principal city),  Census region of the primary state [(based on Census definitions)](https://www2.census.gov/geo/pdfs/maps-data/maps/reference/us_regdiv.pdf) the house price index, and the latitude and longitude of the principal city for the metro area. We've also computed the 12-month percent change in the house price index, named *hpa12*.  

We're using the [Freddie Mac House Price Index](http://www.freddiemac.com/finance/house_price_index.html).

I've arranged these data and saved them as a simple csv files. 

* Metro hpi files [hpimetro.csv]({{ site.url}}/chartbooks/jan2017/data/hpimetro.csv)
* Stae house price file [hpistate.csv]({{ site.url}}/chartbooks/jan2017/data/hpistate.csv)
* National house price file [hpiusa.csv]({{ site.url}}/chartbooks/jan2017/data/hpiusa.csv)

Here's how these data look (examining the metros):

<table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr><td colspan='9' style='text-align: left;'>
House Price Data</td></tr>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>date</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>geo</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>statename</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>region</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpi</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>lat</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>long</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>hpa12</th>
</tr>
</thead>
<tbody>
<tr>
<td style='text-align: left;'>1</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Phoenix-Mesa-Scottsdale, AZ</td>
<td style='text-align: center;'>Arizona</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>177.31</td>
<td style='text-align: center;'>33.54</td>
<td style='text-align: center;'>-112.07</td>
<td style='text-align: center;'>0.06</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>2</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Los Angeles-Long Beach-Anaheim, CA</td>
<td style='background-color: #f7f7f7; text-align: center;'>California</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>242.37</td>
<td style='background-color: #f7f7f7; text-align: center;'>34.11</td>
<td style='background-color: #f7f7f7; text-align: center;'>-118.41</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.07</td>
</tr>
<tr>
<td style='text-align: left;'>3</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Riverside-San Bernardino-Ontario, CA</td>
<td style='text-align: center;'>California</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>203.1</td>
<td style='text-align: center;'>33.94</td>
<td style='text-align: center;'>-117.4</td>
<td style='text-align: center;'>0.06</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>4</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sacramento--Roseville--Arden-Arcade, CA</td>
<td style='background-color: #f7f7f7; text-align: center;'>California</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>184.01</td>
<td style='background-color: #f7f7f7; text-align: center;'>38.57</td>
<td style='background-color: #f7f7f7; text-align: center;'>-121.47</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.09</td>
</tr>
<tr>
<td style='text-align: left;'>5</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>San Diego-Carlsbad, CA</td>
<td style='text-align: center;'>California</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>207.68</td>
<td style='text-align: center;'>32.81</td>
<td style='text-align: center;'>-117.14</td>
<td style='text-align: center;'>0.08</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>6</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>San Francisco-Oakland-Hayward, CA</td>
<td style='background-color: #f7f7f7; text-align: center;'>California</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>202.34</td>
<td style='background-color: #f7f7f7; text-align: center;'>37.77</td>
<td style='background-color: #f7f7f7; text-align: center;'>-122.45</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.09</td>
</tr>
<tr>
<td style='text-align: left;'>7</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Denver-Aurora-Lakewood, CO</td>
<td style='text-align: center;'>Colorado</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>181.02</td>
<td style='text-align: center;'>39.77</td>
<td style='text-align: center;'>-104.87</td>
<td style='text-align: center;'>0.11</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>8</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Washington-Arlington-Alexandria, DC-VA-MD-WV</td>
<td style='background-color: #f7f7f7; text-align: center;'>District of Columbia</td>
<td style='background-color: #f7f7f7; text-align: center;'>South Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>205.34</td>
<td style='background-color: #f7f7f7; text-align: center;'>38.91</td>
<td style='background-color: #f7f7f7; text-align: center;'>-77.02</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.03</td>
</tr>
<tr>
<td style='text-align: left;'>9</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Miami-Fort Lauderdale-West Palm Beach, FL</td>
<td style='text-align: center;'>Florida</td>
<td style='text-align: center;'>South Region</td>
<td style='text-align: center;'>215.97</td>
<td style='text-align: center;'>25.78</td>
<td style='text-align: center;'>-80.21</td>
<td style='text-align: center;'>0.1</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>10</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Orlando-Kissimmee-Sanford, FL</td>
<td style='background-color: #f7f7f7; text-align: center;'>Florida</td>
<td style='background-color: #f7f7f7; text-align: center;'>South Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>170.62</td>
<td style='background-color: #f7f7f7; text-align: center;'>28.5</td>
<td style='background-color: #f7f7f7; text-align: center;'>-81.37</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.1</td>
</tr>
<tr>
<td style='text-align: left;'>11</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Tampa-St. Petersburg-Clearwater, FL</td>
<td style='text-align: center;'>Florida</td>
<td style='text-align: center;'>South Region</td>
<td style='text-align: center;'>183.43</td>
<td style='text-align: center;'>27.96</td>
<td style='text-align: center;'>-82.48</td>
<td style='text-align: center;'>0.11</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>12</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Atlanta-Sandy Springs-Roswell, GA</td>
<td style='background-color: #f7f7f7; text-align: center;'>Georgia</td>
<td style='background-color: #f7f7f7; text-align: center;'>South Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>134.06</td>
<td style='background-color: #f7f7f7; text-align: center;'>33.76</td>
<td style='background-color: #f7f7f7; text-align: center;'>-84.42</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.08</td>
</tr>
<tr>
<td style='text-align: left;'>13</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Chicago-Naperville-Elgin, IL-IN-WI</td>
<td style='text-align: center;'>Illinois</td>
<td style='text-align: center;'>Midwest Region</td>
<td style='text-align: center;'>128.3</td>
<td style='text-align: center;'>41.84</td>
<td style='text-align: center;'>-87.68</td>
<td style='text-align: center;'>0.05</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>14</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Boston-Cambridge-Newton, MA-NH</td>
<td style='background-color: #f7f7f7; text-align: center;'>Massachusetts</td>
<td style='background-color: #f7f7f7; text-align: center;'>Northeast Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>162.05</td>
<td style='background-color: #f7f7f7; text-align: center;'>42.34</td>
<td style='background-color: #f7f7f7; text-align: center;'>-71.02</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.06</td>
</tr>
<tr>
<td style='text-align: left;'>15</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Baltimore-Columbia-Towson, MD</td>
<td style='text-align: center;'>Maryland</td>
<td style='text-align: center;'>South Region</td>
<td style='text-align: center;'>178.69</td>
<td style='text-align: center;'>39.3</td>
<td style='text-align: center;'>-76.61</td>
<td style='text-align: center;'>0.03</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>16</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Detroit-Warren-Dearborn, MI</td>
<td style='background-color: #f7f7f7; text-align: center;'>Michigan</td>
<td style='background-color: #f7f7f7; text-align: center;'>Midwest Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>103.23</td>
<td style='background-color: #f7f7f7; text-align: center;'>42.38</td>
<td style='background-color: #f7f7f7; text-align: center;'>-83.1</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.07</td>
</tr>
<tr>
<td style='text-align: left;'>17</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Minneapolis-St. Paul-Bloomington, MN-WI</td>
<td style='text-align: center;'>Minnesota</td>
<td style='text-align: center;'>Midwest Region</td>
<td style='text-align: center;'>143.04</td>
<td style='text-align: center;'>44.96</td>
<td style='text-align: center;'>-93.27</td>
<td style='text-align: center;'>0.05</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>18</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Kansas City, MO-KS</td>
<td style='background-color: #f7f7f7; text-align: center;'>Missouri</td>
<td style='background-color: #f7f7f7; text-align: center;'>Midwest Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>137.99</td>
<td style='background-color: #f7f7f7; text-align: center;'>39.12</td>
<td style='background-color: #f7f7f7; text-align: center;'>-94.55</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.08</td>
</tr>
<tr>
<td style='text-align: left;'>19</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>St. Louis, MO-IL</td>
<td style='text-align: center;'>Missouri</td>
<td style='text-align: center;'>Midwest Region</td>
<td style='text-align: center;'>135.58</td>
<td style='text-align: center;'>38.64</td>
<td style='text-align: center;'>-90.24</td>
<td style='text-align: center;'>0.05</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>20</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Charlotte-Concord-Gastonia, NC-SC</td>
<td style='background-color: #f7f7f7; text-align: center;'>North Carolina</td>
<td style='background-color: #f7f7f7; text-align: center;'>South Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>146.26</td>
<td style='background-color: #f7f7f7; text-align: center;'>35.2</td>
<td style='background-color: #f7f7f7; text-align: center;'>-80.83</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.08</td>
</tr>
<tr>
<td style='text-align: left;'>21</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Las Vegas-Henderson-Paradise, NV</td>
<td style='text-align: center;'>Nevada</td>
<td style='text-align: center;'>West Region</td>
<td style='text-align: center;'>150.66</td>
<td style='text-align: center;'>36.21</td>
<td style='text-align: center;'>-115.22</td>
<td style='text-align: center;'>0.09</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>22</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>New York-Newark-Jersey City, NY-NJ-PA</td>
<td style='background-color: #f7f7f7; text-align: center;'>New York</td>
<td style='background-color: #f7f7f7; text-align: center;'>Northeast Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>168.58</td>
<td style='background-color: #f7f7f7; text-align: center;'>40.67</td>
<td style='background-color: #f7f7f7; text-align: center;'>-73.94</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.04</td>
</tr>
<tr>
<td style='text-align: left;'>23</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Cincinnati, OH-KY-IN</td>
<td style='text-align: center;'>Ohio</td>
<td style='text-align: center;'>Midwest Region</td>
<td style='text-align: center;'>122.06</td>
<td style='text-align: center;'>39.14</td>
<td style='text-align: center;'>-84.51</td>
<td style='text-align: center;'>0.06</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>24</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Portland-Vancouver-Hillsboro, OR-WA</td>
<td style='background-color: #f7f7f7; text-align: center;'>Oregon</td>
<td style='background-color: #f7f7f7; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>212.83</td>
<td style='background-color: #f7f7f7; text-align: center;'>45.54</td>
<td style='background-color: #f7f7f7; text-align: center;'>-122.66</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.13</td>
</tr>
<tr>
<td style='text-align: left;'>25</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Philadelphia-Camden-Wilmington, PA-NJ-DE-MD</td>
<td style='text-align: center;'>Pennsylvania</td>
<td style='text-align: center;'>Northeast Region</td>
<td style='text-align: center;'>166.12</td>
<td style='text-align: center;'>40.01</td>
<td style='text-align: center;'>-75.13</td>
<td style='text-align: center;'>0.03</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>26</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pittsburgh, PA</td>
<td style='background-color: #f7f7f7; text-align: center;'>Pennsylvania</td>
<td style='background-color: #f7f7f7; text-align: center;'>Northeast Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>157.07</td>
<td style='background-color: #f7f7f7; text-align: center;'>40.44</td>
<td style='background-color: #f7f7f7; text-align: center;'>-79.98</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.04</td>
</tr>
<tr>
<td style='text-align: left;'>27</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>Dallas-Fort Worth-Arlington, TX</td>
<td style='text-align: center;'>Texas</td>
<td style='text-align: center;'>South Region</td>
<td style='text-align: center;'>176.13</td>
<td style='text-align: center;'>32.79</td>
<td style='text-align: center;'>-96.77</td>
<td style='text-align: center;'>0.11</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; text-align: left;'>28</td>
<td style='background-color: #f7f7f7; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; text-align: center;'>Houston-The Woodlands-Sugar Land, TX</td>
<td style='background-color: #f7f7f7; text-align: center;'>Texas</td>
<td style='background-color: #f7f7f7; text-align: center;'>South Region</td>
<td style='background-color: #f7f7f7; text-align: center;'>186.68</td>
<td style='background-color: #f7f7f7; text-align: center;'>29.77</td>
<td style='background-color: #f7f7f7; text-align: center;'>-95.39</td>
<td style='background-color: #f7f7f7; text-align: center;'>0.04</td>
</tr>
<tr>
<td style='text-align: left;'>29</td>
<td style='text-align: center;'>Sep 01,2016</td>
<td style='text-align: center;'>San Antonio-New Braunfels, TX</td>
<td style='text-align: center;'>Texas</td>
<td style='text-align: center;'>South Region</td>
<td style='text-align: center;'>190.4</td>
<td style='text-align: center;'>29.46</td>
<td style='text-align: center;'>-98.51</td>
<td style='text-align: center;'>0.07</td>
</tr>
<tr style='background-color: #f7f7f7;'>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: left;'>30</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Sep 01,2016</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Seattle-Tacoma-Bellevue, WA</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>Washington</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>West Region</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>203.79</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>47.62</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>-122.35</td>
<td style='background-color: #f7f7f7; border-bottom: 2px solid grey; text-align: center;'>0.13</td>
</tr>
</tbody>
<tfoot><tr><td colspan='9'>
Source: Freddie Mac House Price Index</td></tr></tfoot>
</table>

For tractability I restricted the number of metro areas, roughly corresponding to the top 20 metro areas based on population.

Here's a map view of the metros in our data:

![plot of chunk jan222017-map1](/img/Rfig/jan222017-map1-1.svg)


# Building a dashboard

The idea behind this dashboard is to compare housing market conditions across areas and across time.  These data automatically lend themselves to these comparisons.  Indeed, the very nature of a house price index is to compare trends in average quality-adjusted house prices over time. 



[Flexdashboards](http://rmarkdown.rstudio.com/flexdashboard/) are a powerful tool for visualizing data. We will combine multiple interactive plots together into a single self-contained webpage.


## Getting started - Data

First we need to load our data.  As we're going to use crosstalk to enable our widgets to talk to each other, we'll also need to do set up some [Shared Data](http://rstudio.github.io/crosstalk/using.html). The shared data can act like data frame in compatible HTML widgets but respond to selections and filters.

In the code below we load the data with metro house prices and create the Shared Data:


{% highlight r %}
df<-fread("data/hpimetro.csv")
df$date<-as.Date(df$date, format="%m/%d/%Y")
# Set up metro data for cross talk:
df.metro<-group_by(df[year(date)>1999,],geo)
sd.metro <- SharedData$new(df.metro, ~geo)
{% endhighlight %}

## Layout

Our dashboard is going to have several pages.  To explore the different layout options we'll create four pages:

1. A general information/about page
2. A [storyboard](http://rmarkdown.rstudio.com/flexdashboard/layouts.html#storyboard) page
3. A page with an interactive widget we can filter
4. A page with an animated chart

# Getting stated

Let's walk through the construction of each of these individual pages, starting with the landing page.

## About

The about page is quite important as it is where our new visitors will land. We want to include a brief description along with some hints at what else is in the dashboard. But we want to do it without overwhelming visitors. For this page we'll include:

1. Short introductory text
2. A map (same as above) showing the metros in our data.


{% highlight r %}
About {data-navmenu="Explore"}
===================================== 

Column {data-width=200}
-------------------------------------

### About this flexdashboard

This dashboard allows you to explore trends in house prices across 30 large metro areas. The metro areas covered are depicted in the nearby map.  The map is colored  according to Census regions.  We picked the 30 largest metro areas based on population. Explore the different data visualizations above.

Column {data-width=800}
-------------------------------------

### Areas covered

# Run this to create map
#```{r jan222017-ex1-map,echo=F}
g.map<-
  ggplot(df[date=="2016-09-01" ], aes(x = long, y = lat,label=geo)) +
  geom_map(data=df.state[date=="2016-09-01",],aes(fill = region,map_id=tolower(statename)), map = states_map,alpha=0.25)+
  borders("state",  colour = "grey70",alpha=0.4)+
  theme_void()+
  scale_fill_viridis(name="Census Region",discrete=T,option="C")+
  theme(legend.position="top",
        plot.title=element_text(face="bold",size=18))+
  geom_point(alpha=0.85,color="black",size=2)+
  geom_text(hjust=0,size=1.75,nudge_y=-0.7)+
  labs(title="Metro areas in our data",
       subtitle="30 large metro areas",
       caption="@lenkiefer Metro population based on U.S. Census: http://www.census.gov/programs-surveys/popest.html")+
  theme(plot.caption=element_text(hjust=0,size=7))
# ```
{% endhighlight %}

Some things to note about this page.  We want the navigation to be collapsed. The default is for each page to get its own link on the top navigation, but by selecting `About {data-navmenu="Explore"}` we force this page to fall under the "Explore" link at the top. We also want the map to take up most of the space, so we set `{data-width=200}` for the first column and `{data-width=800}` for the second. This ensures the map gets 80% of the available width.

## Storyboard

Now we can move on to the second page, which uses a storyboard.  Consider the code below:


{% highlight r %}
Storyboard {.storyboard data-navmenu="Explore"}
=========================================

### Map of areas we plot

#```{r}
g.map
#```

### Small multiple, House Price Index

#```{r sm-1-jan22-2017,fig.width=10}
g1<-ggplot(data=df,aes(x=date,y=hpi))+geom_line()+facet_wrap(~geo)+
  theme_minimal()+labs(x="",y="",title="House price index by metro",
                       caption="@lenkiefer Source: Freddie Mac House Price Index")+
  theme(plot.caption=element_text(hjust=0,size=7),        plot.title=element_text(size=10),
        strip.text=element_text(size=4),
        axis.text.x=element_text(size=4)  ,
              axis.text.y=element_text(size=5)   )
g1

### Small multiple, Annual house price appreciation

#```{r sm-2-jan22-2017,fig.width=10}
g2<-ggplot(data=df,aes(x=date,y=hpa12))+geom_line()+facet_wrap(~geo)+
    theme_minimal()+labs(x="",y="",title="Annual percent change in house price index by metro",
                       caption="@lenkiefer Source: Freddie Mac House Price Index")+
  scale_y_continuous(label=percent)+
  theme(plot.caption=element_text(hjust=0,size=7),        plot.title=element_text(size=10),
        strip.text=element_text(size=4),
        axis.text.x=element_text(size=4)  ,
              axis.text.y=element_text(size=5)   )
g2
#```
{% endhighlight %}

We start the storyboard page by declaring that this page has a storyboard structure with `Storyboard {.storyboard data-navmenu="Explore"}`.  Note we also force this page to belong under the "Explore" navigation.  By adding `.storyboard` this tells the flexdashboard to arrange subsections on different storyboard panes.  

In the code above I included the first three panes (corresponding to the map g.map and graphs g1 & g2).  In the full dashboard I actually include 7 panes.  The text we include under the headers (denoted with `###`) will be included in the story pane navigation filmstrip.

## Interactive chart

So far, the elements we have included are standard, and well described in the [flexdashboard](http://rmarkdown.rstudio.com/flexdashboard/index.html) documentation. These next two pages are more complex.  The first, an interactive chart uses [crosstalk](https://github.com/rstudio/crosstalk) and [plotly](https://plot.ly/r/) to create a dynamic interactive chart in a static webpage.

[Crosstalk](https://github.com/rstudio/crosstalk) allows [htmlwidgets](http://www.htmlwidgets.org/) to talk to one another on a static webpage. What we are going to do is create three plotly graphs and have them linked via crosstalk and include a filter box.

First we need to create the widgets, which are individual plotly charts:


{% highlight r %}
g.map<-
  ggplot(sd.metro, aes(x = long, y = lat)) +
  borders("state",  colour = "grey70",fill="lightgray",alpha=0.5)+
  theme_void()+
  theme(legend.position="none",
        plot.title=element_text(face="bold",size=18))+
  geom_point(alpha=0.82,color="black",size=3)+
  labs(title="Selected Metro(s)",
       subtitle=head(df,1)$geo,
       caption="@lenkiefer Source: Freddie Mac House Price Index through September 2016")+
  theme(plot.caption=element_text(hjust=0))

p0<-
   plot_ly(data=sd.metro,x = ~date, y = ~hpi, height=750) %>% 
    add_lines(name="Index",colors="gray",alpha=0.7) %>% 
    add_lines(name="All metros",data=df,x=~date,y=~hpi,
              colors="black",color=~geo,alpha=0.1,showlegend=F,hoverinfo="none") %>%
     layout(title = "House Price Trends by Metro",xaxis = list(title="Date"), yaxis = list(title="House Price Index"))

p1<-
   plot_ly(data=sd.metro,x = ~date, y = ~hpa12, height=750) %>% 
    add_lines(name="Annual % change",colors="gray",alpha=0.7) %>% 
    add_lines(name="All metros",data=df,x=~date,y=~hpa12,
              colors="black",color=~geo,alpha=0.1,showlegend=F,hoverinfo="none") %>%
     layout(title = "House Price Trends by Metro",xaxis = list(title="Date"), yaxis = list(title="Annual % Change in House Price Index"))
{% endhighlight %}

*g.map* is a ggplot2 graph while *p0* and *p1* are plotly graphs. We will apply 
[ggplotly](https://plot.ly/ggplot2/) to convert our ggplot map into a plotly thing.

Once we have the graphs, we can combine them using the crosstalk function *bscols* and include a *filter_select* to filter the charts.  The code is not very long:


{% highlight r %}
bscols(widths=c(2,6,4),
  list(filter_select("metro", "Select metro to highlight for plot", sd.metro, ~geo,multiple=FALSE)),
  subplot(p0,p1,nrows=2,titleY=T),
  ggplotly(g.map)
  )
{% endhighlight %}

The *bscols* function first allocates our graphs over the page with *widths*.  Next, we include a *filter_select* that uses the SharedData sd.metro (discussed above). We set *multiple* equal to FALSE so that only one metro can be selected at a time.


### Animation

Our final page is an animated chart.  Animations require the development version of plotly for R. Install via:

`devtools::install_github("ropensci/plotly")`

The animation is pretty straightforward.  Once again, we link the data through SharedData. In our plots, we income a *frame* argument inside of *aes*.  Then we instruct plotly to animate the graphs:


{% highlight r %}
g.map2<-
  ggplot(sd.metro, aes(x = long, y = lat,frame=geo,label=paste("\n\n  ",geo),color=geo)) +
  borders("state",  colour = "grey70",fill="lightgray",alpha=0.5)+
  theme_void()+
  theme(legend.position="none",
        plot.title=element_text(face="bold",size=18))+
  geom_point(alpha=0.5,size=1)+
  geom_text(hjust=0)+
  labs(title="House price trends around the U.S.")+
  theme(plot.caption=element_text(hjust=0))

p2<-
  ggplot(data=sd.metro,aes(x=date,y=hpa12))+
  #geom_point()+geom_segment(aes(xend=date,yend=0))+
  geom_line(aes(frame=geo,ids=date,label=geo,color=geo))+
  scale_y_continuous(labels=scales::percent)+
  geom_line(data=df.us,color="gray",linetype=2)+
  #scale_fill_viridis()+  scale_color_viridis()+
  theme_minimal()+labs(x="",y="Annual House Price Appreciation y/y %",title="Annual Price Growth")
#+    geom_text(data=d3.m[date==median(d3.m$date)],fontface="bold",y=16,size=8)

p3<-
  ggplot(data=sd.metro,aes(x=date,y=hpi))+
  geom_line(aes(frame=geo,ids=date,label=geo,color=geo))+
  theme_minimal()+labs(x="",y="House Price Index",title="House Price Index")+
  geom_line(data=df.us,color="gray",linetype=2)


subplot(subplot(ggplotly(p3),ggplotly(p2),nrows=2,titleY=T), ggplotly(g.map2),  nrows = 1, widths = c(0.35, 0.65), titleX = TRUE,titleY=T) %>%
  hide_legend() %>%
  animation_opts(2000,transition=500) %>% layout(title="House price tour, solid line metro, dotted line U.S.")
{% endhighlight %}

I want to arrange the graphs, so I use a nested call of plotly's [subplot](https://cpsievert.github.io/plotly_book/arranging-multiple-views.html) function.

# Putting it all together

Combining all these steps we can create the following dashboard:

You can see a fullscreen version [here]({{ site.url}}/chartbooks/jan2017/hpi-tour.html). 

<iframe src="{{ site.url}}/chartbooks/jan2017/hpi-tour.html" height="800" width="1200"></iframe>


# Next Steps

I've found flexdashboards to be a fun way to interact with data. Hopefully more htmlwidgets will be made compatible with crosstalk. But because plotly allows you to translate most ggplot graphs into widgets, there's already a huge potential with what's available.

Perhaps you will find flexdashboards to be something you would like to explore. Hopefully this guide can be helpful, by giving you a working example of several features.



