---
layout: post
title: "Mortgage rate flexdashboard"
author: "Len Kiefer"
date: "2017-01-08"
summary: "R statistics dataviz flexdashboard plotly"
group: navigation
theme :
  name : lentheme
---

IN THE PAST I HAVE USED MANY DIFFERENT programs to visualize data. I've [done quite a few visualizations](https://public.tableau.com/profile/leonard.kiefer#!/) using [Tableau](http://www.tableau.com/). I enjoy using that program, but it does have some drawbacks.

As an alternative, I have been exploring using  [flexdashboards](http://rmarkdown.rstudio.com/flexdashboard/) for [R](https://www.r-project.org/). One advantage of flexdashboards is that you can easily incorporate R code into the design of dashboards.

I figured I would give it a try. The example below, uses mortgage rates. I used [DT](https://rstudio.github.io/DT/) to make interactive data tables and [Plotly for R](https://plot.ly/r/) to make interactive times series charts. I also used [stargazer](https://cran.r-project.org/web/packages/stargazer/index.html) to make nicely formatted regression output.

All the source code is available in the tab at the right.  I used a simple excel spreadsheet called "rates.xlsx", which you can download from the [here]({{ site.url}}/chartbooks/jan2017/rates.xlsx).

You can see a fullscreen version [here]({{ site.url}}/chartbooks/jan2017/mortgage_rate_viewer_jan_2017.html). 

<iframe src="{{ site.url}}/chartbooks/jan2017/mortgage_rate_viewer_jan_2017.html" height="800" width="1200"></iframe>
