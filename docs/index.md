---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

# Estimating Australia's COVID-19 instantaneous reproduction number

_This is an “old” write-up from April 2020._

**NB not a epidemiologist.**

[Link to R shiny dashboard](https://benqiu.shinyapps.io/Aus_COVID19_R/)

As of late April 2020, Australia is well past its peak for COVID-19 cases, at least for the first wave anyway. We have moved beyond the discussion of how to minimise the spread of disease, what policies to put in place and how strict they should be, and what emergency first aid is needed for the economy.

Numerous dashboards have been set up to track the number of new cases and tests, the deaths over time, and statistics on the demographics of the cases.

In early to mid April there was a somewhat lack of information on what R for Australia was. Cases were lowering for sure and the peak had truly been passed, but what about R?

For those who haven’t been glued to the minute by minute COVID-19 / SARS-Cov-2 updates, R is the _effective_ (or _instantaneous_) reproductive number, a dimensionless number (not a rate) that represents the average number of secondary cases that will arise from a single infected individual. This is also often denoted as R_e (or R_t). R can and generally will change over time depending on the characteristics of the population and the mitigation strategies put in place.

R_0 on the other hand is the _basic_ reproduction number, a constant that describes the average number of secondary cases that will arise from a single case, where an entire population remains susceptible and no intervening strategies are put in place. This number does not change over time and is used to describe the general characteristic of an infectious disease. What is interesting is that R_0 is not meant to be constant across different populations of countries, it is context specific. This is why you see a range of quoted R_0’s for different diseases, and that a range is presented for SARS-Cov-2 for reasons beyond the fact that it’s a novel disease. With that in mind, it should follow that R_0 may not be a super useful number to assess an ongoing outbreak, especially when comparing across regions. 

I decided to try my hand at helping myself figure out where Australia was at with its R. 

I was also curious to see what I could get out of building a R shiny app. In my day job my focus is usually on building gigantic, complex but elegantly presented excel monsters, and producing code that bake in simulations and sensitivities -- rarely do I need to produce a dedicated dashboard like interface to explicitly explore different scenarios (I did build an excel dashboard for one particular case with a client that was in the process of going through a contract pricing review for a eight-figure contract).

To cut to the chase, the engine behind this entire R estimation exercise is the [EpiEstim package](https://sites.google.com/site/therepiproject/r-pac/epiestim). Written by Dr. Anne Cori at Imperial College London, EpiEstim estimates R via two inputs: incidence data (infections over time) and some distribution of the case serial interval (time between the appearance of symptoms from the primary case to secondary cases).

This leads us nicely into part 1 of the build -- finding inputs for incidence data, and the serial interval. On incidence data, many outlets were producing cases of infection at the time, although often with visible differences among them. Three sources were chosen on no particular criteria apart from the fact that various outlets have quoted these sources on different occasions - John Hopkins University, Our world in data, and Worldometer. John Hopkins University and Our world in data have kindly provided their data in easy to access form on Github. For Worldometer I decided to web scrape in a small exercise.

If the slightly different input data sets caused large changes in R estimates it would be cause for alarm, and which might initiate further research on why these inputs are different, but so far that has not been the case.

For the serial interval I relied on this [one paper](https://wwwnc.cdc.gov/eid/article/26/6/20-0357_article). The EpiEstim package allows the serial interval to be provided to it, via a mean and a standard distribution, or it can estimate the serial interval based on raw interval information (i.e. the range of estimated symptom onset of the infector, and the range of estimated symptom onset of the infectee. Effectively this is the underlying data that lets you arrive at a mean and standard deviation of time to transmission).

For part 2 of the build, I added a tab to the dashboard which allowed you to generate a wave of new cases some point in the future. This served two purposes: to observe (i) what different sized and shaped outbreaks do to outcomes of R, and (ii) what parameters in this particular curve generation function most closely matched the shape of the existing Australian outbreak.

For the simulated case incidence curve I have chosen arbitrarily to use a piecewise function made up of two logistic functions. The most popular epidemiology models are generally built up from a series of [first order differentials](https://timchurches.github.io/blog/posts/2020-03-10-modelling-the-effects-of-public-health-interventions-on-covid-19-transmission-part-1/). Here I have simply decided on a global growth rate and decline rate, each for the two logistic functions that represent the ramping up stage and ramping down stage of cases.

I broke down the simulated case curve into two parts simply because I wanted the curve to be asymmetrical: right skew is generally observed in mechanistic models and in actual case counts. The decay in the number of  newly infected cases is generally stretched out and more prolonged compared to the growth phase of newly infected cases. By tweaking the slope term in the logistic function you’re able to tighten up or spread out the shape of either curve.

Once you provide the simulator with a day 1 outbreak size, the case count on day 2 is equal to the growth rate times the day 1 outbreak, hence the weirdly shaped case count curve: day 2 looks to be a precipitous drop from day 1, but then the case count grows smoothly from then on as the new cases on day three is a function of both day 1 and day 2 and so on and so forth.

I won’t go through the entire code here, as other resources serve a much better starting point to building a shiny app, but there are examples of some issues I found to be interesting (sizing and stacked placement of the ggplot outputs, setting the y axis to be dynamic or fixed, figuring out how to stitch together the logistic curves etc.)

This exercise was a quick and fast one (took about two afternoons) and it was an enjoyable build.

Here are a few quick takeaways: Your R is highly sensitive to the number of absolute cases, the error bars are wide and kind of useless when your level of incidence is low. A small outbreak that grows from 10 to 50 in 3 days will give you a R that far exceeds 1 -- R is agnostic to relative size but sensitive to shape. This means R isn’t a great number to track at the beginning of outbreaks, it’s utility is greatest with large outbreaks that are in the decay phase.








