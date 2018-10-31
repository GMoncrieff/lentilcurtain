---
title: Near-Real time detection of land cover change
author: Glenn Moncrieff
date: '2018-10-31'
slug: near-real-time-detection-of-land-cover-change
categories: []
tags:
  - Sentinel 2
  - Planet
  - Thicket
---

If you were to judge by media coverage, climate change and illegal hunting would probably rank as the biggest threats to biodiversity globally. Yet when [systematicaly assessed](https://www.worldwildlife.org/pages/living-planet-report-2018), the most important factor causing the massive declines in species across all ecosystem both terretrial and aquatic is [habitat transformation](https://www.ipbes.net/assessment-reports/ldr). In South Africa's terrestial ecosytems the [same is true](https://www.sanbi.org/biodiversity/building-knowledge/biodiversity-monitoring-assessment/national-biodiversity-assessment/). High rates of irreversible change to natural ecosystems is ongoing and loss of natural habitat continues to be the leading threat to terrestrial biodiversity.

A range of tools are available to aid in mapping and monitoring land cover change. [Global Forest Watch](https://www.globalforestwatch.org/) is an amazing tool developed by the World Resources Institute for monitoring deforestation worldwide. They continuously upate maps of where deforestation is occuring based upon data collected by the USGS/NASA satellite [Landsat](https://landsat.usgs.gov/). While this is a great tool, it is not designed with South Africa's ecosystems in mind, and thus has limited applicability. South Africa has a [national land cover map](http://bgis.sanbi.org/DEA_Landcover/project.asp) tracking the distribution of a range of natural and non-natrual land cover types based upon the same Landsat data as Global Forest Watch. The problem with the national land cover map is that is is only updated every 5-10 years on a somewhat ad hoc basis. It allows us to report on the rate at which habitats are begin destroyed, but neither it nor Global Forest Watch (which provdes updates every year) produce information fast enough to inform interventions that could halt habitat destruction while it is happening. Enforcement authorities are in particular need of information that might help them to catch land owner's sin the act of illegally transforming natural vegetation, as curently only [5%] (https://www.environment.gov.za/sites/default/files/reports/nationalenvironmentalcomplianceandenforcementreport2016_17.pdf) of investigated cases end in successful prosecution.

Newly launched satellite platforms have reduced the time between image acquisitions and increased image resolution to the point that it is now possible to detect land transformation within days of its occurence. [Sentinel 2](https://sentinel.esa.int/web/sentinel/missions/sentinel-2) - launched by the European Space Agency (ESA) on the 23rd of June 2015 consists of 2 satellites capable of revisitng any place on earth every 5 days and producing images at 10 meter spatial resolution. Even more detail is availabe from the constellation of almost 200 microsatellites owned by [Planet Inc](https://www.planet.com). Planet's satellites reimage the Earth every day at 3 meter resolution. ESA's open data policy makes it possible to monitor large areas using their data as it comes at no cost, while Planet provide a limited amount of data for free to academic users allowing focussed monitoirng over key areas. 

I set out to test how rapidly land transformation could be detected using these two platforms in an area of South Africa that is undergoing rapid irreversible land cover change. The Albany Thicket biome is a part of a global [biodiversity hotspot](https://www.conservation.org/global/ci_south_africa/where-we-work/maputaland-pondoland-albany/Pages/maputaland-pondoland-albany-hotspot.aspx) and home to an amazing array of indigenous plants and animals - many of which are of immense [economic](https://www.sciencedirect.com/science/article/pii/S2212041617303960), [cultural and spiritual](http://www.scielo.org.za/scielo.php?pid=S0038-23532012000300016&script=sci_abstract&tlng=en) value to local people. 

![Albany Thicket](/images/thicket.jpg "The Albany thicket biome")

Enforcement authorities have reported high rates of illgeal vegetation clearing across the biome, but an area of particular concern surrounds the town of Alexandria near Port Elizabeth, where patches of thicket and forest undistubred for thousands of years are being cleared for diary pasture at an alarming rate.  I identified a small area where a patch of vegetation had recently been cleared and set my self the task of attempting to automate the detection of this change within days of it's occurence. Here you can see a timelapse covwering the period from the 23rd of January 2018 to the 2nd of Februay. From this imagey is is apparetnt that the vegetation was cleared around the 1st of February. (Disclaimer: I make no allegation about legality of this vegetation clearing. It is possible that what this farmer did is entirely legal) 

![thicket clearing](/images/pl_gif.gif "thicket clearing")

All the code to reproduce my analysis is available at this github repo. But her eis a quick brekadown fo how I went about this:
First I downloaded all the data availalbe for this area for Planet and Sentinel 2 from when their records began until the 31st of March - about 2 months after the clearing event). This amounts to about 2 years of data for each. In the end I had around 120 images to analyse from Planet and 50 from Sentinel 2. THen I need to correct the raw data from level 1 data, or what the sensor sees at the top of the atmospehre, to surfacne reflectacen - what the earths surface looks like without the interference of the atmospehre. this is done by correcting the data for the effects of things like clouds, haze, the varyin agnle and intensity of the sun's illuminations, and topography. Fortuantly PLanet have done this for me already and I can simply download their surface reflectance data. Sentinel provide a tool SEn2Cor which you can run on level 1 imagery to calculate surface refelcetacne. 
Then using the processed surface reflectance data I calucate a measure of vegetation greeness caleed NDVI or Normalized Vegetation difference Index. NDVI is a measure of hw much photosynthietic activty there is. A high NDVI value (near 1) indivcate lush vegetation like a forest, a low value (around 0.2) indicated bare soil. A sudden drop in NDVI indicate a reduction in photosynthesis, and may be related to vegetation loss.
Nown what I have is a stack of images fof NDVI for a seires of dates across the region I wish to monitor

![ndvi](/images/stack.jpg "ndvi stack")

These data are the basis upon which we detect land transformation using an algoritmth called BFAST (Breaks For Additive Season and Trend). BFast works by defining in aperiod in which we know vegetation to be stable and not subject to transformation. The trends and behavour of NDVI through time within this period is used to build of model of what the expectated pattern ought to look like if this vegetation where to continue to function similarly. 

image

we then project expected NDVI patterns into the future and compare these to new data as they are acquired. 

image

If the new data excceds our expectation by some threshold we flag this as an anomlay and a potenital case of land cover change.

image

THis is the procdudre that was followed using the Sentinel 2 and Planet data. We build models usig data up to the 31st of december 2018 and monitored from chagen from the 1st of Jaunray 2018 onwards. Using the Planet data I was able to detect the land transfomation we were interested in within 1 to 2 days of it's occuerence! This is what the actual data look like. The breakpoint is detected on day 33 of the year - or the 2nd of Feb. Remember the raw images showed the change ocrrruing onthe the 1st.

image

Running the over the whole area of intered shows how we can accuratly outline the clearing and automate the detecint of aras that are cleared. This porcess is not perfect. You can see how we outline some araeas that have not been transofmred, and simialry it is likely that we will fail to detect some areas that have been cleared.

![thicket clearing2](/images/pl_gif2.gif "thicket clearing")

It takes around a month after clearing to detect the event using Sentinel data, but it also provides a good outline of areas that have chagne during the monitoreing period. By the 13th of March clearing had been accurately detected in the areas we are concered with, and an addiotnal areas to the north that I ha not been previously aware of.

<iframe frameborder="0" class="juxtapose" width="100%" height="540" src="https://cdn.knightlab.com/libs/juxtapose/latest/embed/index.html?uid=eb4db728-dcf7-11e8-9dba-0edaf8f81e27"></iframe>

I think this is really promising. Satellite imagery has advacned enrouemly in the last few year. We are now at a point where we have the imageyry avaialbe to identify land cover change within days of it's occurence. Of cousre it is not possible to manulayy search thourgh images and comapre each day, thus we need algorithms like those I have demonstared to help with this process. I dont think we can fully automate this, there will always be the need for humans inth e loop to confirm detectjon ands theow away flase positives. We have some grants that are currently being reveiwed in whih we prosed to further develop this technology and roll it out over large areas. Ifnthings comes together you will see this approach in a thciket near you soon.







