---
title       : Potentially Preventable Deaths in US
subtitle    : 
author      : Vivek Patil
job         : 
framework   : bootstrap        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---

## Introduction
The most recent *Morbidity and Mortality Weekly Report*, dated May 2, 2014, from the Centers for Disease Control and Prevention had [a report by Yoon et al. (2014) on potentially preventable deaths from 5 leading causes of death for people under the age of 80.](http://www.cdc.gov/mmwr/preview/mmwrhtml/mm6317a1.htm?s_cid=mm6317a1_w) In this post, I use interactive bar charts and choropleths to help visualize state-wise statistics. For these charts, I use googleVis and RStudio's shiny server platform. This post was generated using slidify and the [code necessary to recreate it can be found on github.](https://github.com/patilv/preventabledeaths) The [code for the accompanying shiny app can also be found on github.](https://github.com/patilv/preventableshiny)      

## Data
The report mentions that in 2010, the top 5 causes of death - diseases of the heart, cancer, chronic lower respiratory disease, cerebrovascular diseases (stroke), and unintentional injuries accounted for approximately 63% of all deaths. For the purposes of their report, they used mortality data from the National Vital Statistics System for 2008-2010. Please [read their report]((http://www.cdc.gov/mmwr/preview/mmwrhtml/mm6317a1.htm?s_cid=mm6317a1_w) for caveats associated with the data as well as the assumptions underlying the procedures used. Implications are also discussed in the report and the discussion section of the report is really worth a read. 

### Retrieve Data

This section of the R code retrieves data from CDC's report. 





```r
library(XML)
URL = "http://www.cdc.gov/mmwr/preview/mmwrhtml/mm6317a1.htm?s_cid=mm6317a1_w"
table = readHTMLTable(URL)
statewise = table[[1]]  # first of two tables on that page
```


### Data Cleaning and Manipulations

Let's clean the dataset by doing the following.

1. Changing column names
2. Removing the top 3 rows and the bottom two rows

Let's also check the structure of the data.

```r
colnames(statewise) = c("State", "HeartDiseasesObserved", "HeartDiseasesExpected", 
    "HeartDiseasesPreventable", "CancerDiseasesObserved", "CancerDiseasesExpected", 
    "CancerDiseasesPreventable", "ChroniclowerrespiratoryDiseasesObserved", 
    "ChroniclowerrespiratoryDiseasesExpected", "ChroniclowerrespiratoryDiseasesPreventable", 
    "CerebrovascularDiseasesObserved", "CerebrovascularDiseasesExpected", "CerebrovascularDiseasesPreventable", 
    "UnintentionalinjuriesObserved", "UnintentionalinjuriesExpected", "UnintentionalinjuriesPreventable")
statewise = statewise[-(1:3), ]
statewise = statewise[-(52:53), ]
str(statewise)
```

```
## 'data.frame':	51 obs. of  16 variables:
##  $ State                                     : Factor w/ 56 levels "Abbreviation: DC = District of Columbia.\r\n\t\t\t\t\t\t\t\t*\tExpected deaths are the lowest three-state average age-specific "| __truncated__,..: 2 3 4 5 6 7 8 11 9 12 ...
##  $ HeartDiseasesObserved                     : Factor w/ 54 levels "1,007","1,080",..: 43 31 29 24 22 20 17 49 46 12 ...
##  $ HeartDiseasesExpected                     : Factor w/ 54 levels "1,063","1,194",..: 22 33 29 8 14 20 15 42 32 12 ...
##  $ HeartDiseasesPreventable                  : Factor w/ 54 levels "0","1,089","1,092",..: 29 10 49 6 39 9 32 26 40 35 ...
##  $ CancerDiseasesObserved                    : Factor w/ 54 levels "1,054","1,304",..: 44 45 41 30 27 32 29 3 46 23 ...
##  $ CancerDiseasesExpected                    : Factor w/ 54 levels "1,006","1,112",..: 33 39 43 22 27 31 24 1 37 20 ...
##  $ CancerDiseasesPreventable                 : Factor w/ 51 levels "0","1,059","1,126",..: 21 14 44 7 30 16 41 35 18 40 ...
##  $ ChroniclowerrespiratoryDiseasesObserved   : Factor w/ 53 levels "1,016","1,035",..: 15 17 12 3 45 7 42 29 47 41 ...
##  $ ChroniclowerrespiratoryDiseasesExpected   : Factor w/ 53 levels "1,004","1,148",..: 42 43 1 32 28 38 34 12 44 24 ...
##  $ ChroniclowerrespiratoryDiseasesPreventable: Factor w/ 51 levels "0","1,013","1,117",..: 2 30 39 42 4 35 1 47 1 12 ...
##  $ CerebrovascularDiseasesObserved           : Factor w/ 52 levels "1,003","1,119",..: 5 49 45 41 36 38 31 18 12 28 ...
##  $ CerebrovascularDiseasesExpected           : Factor w/ 52 levels "1,015","1,108",..: 34 37 46 26 21 32 28 7 36 16 ...
##  $ CerebrovascularDiseasesPreventable        : Factor w/ 49 levels "1,527","1,783",..: 36 12 39 16 1 44 27 31 21 43 ...
##  $ UnintentionalinjuriesObserved             : Factor w/ 52 levels "1,010","1,013",..: 21 34 25 6 47 10 49 29 18 44 ...
##  $ UnintentionalinjuriesExpected             : Factor w/ 52 levels "1,074","1,093",..: 49 17 4 38 42 50 43 19 14 29 ...
##  $ UnintentionalinjuriesPreventable          : Factor w/ 51 levels "0","1,027","1,054",..: 4 23 5 46 12 39 25 16 38 30 ...
```


Let's change columns for numbers from factor variables to numeric variables and view the data using googleVis's table. **Entries can be sorted in this table by clicking on the header for a column.**

```r
for (i in 2:16){statewise[, i] = as.character(statewise[,i])}
for (i in 2:16){statewise[, i] = gsub(",","",statewise[,i])}
for (i in 2:16){statewise[, i] = as.numeric(statewise[,i])}

library(googleVis)
plot(gvisTable(statewise,options=list(height=400, width=800)))
```

<!-- Table generated in R 3.0.3 by googleVis 0.5.1 package -->
<!-- Sat May 03 14:14:48 2014 -->


<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataTableID1dc0551279ba () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "Alabama",
6604,
2993,
3611,
7595,
5227,
2368,
1778,
765,
1013,
1277,
588,
689,
2036,
910,
1126 
],
[
 "Alaska",
463,
331,
132,
703,
588,
115,
112,
77,
35,
91,
62,
29,
331,
131,
200 
],
[
 "Arizona",
4735,
3885,
850,
7460,
6775,
685,
1558,
1004,
554,
848,
771,
77,
2341,
1191,
1150 
],
[
 "Arkansas",
3808,
1845,
1963,
4720,
3219,
1501,
1101,
476,
625,
718,
365,
353,
1221,
551,
670 
],
[
 "California",
24707,
19742,
4965,
38226,
34454,
3772,
6047,
4904,
1143,
5366,
3839,
1527,
8627,
6886,
1741 
],
[
 "Colorado",
2815,
2707,
108,
4944,
4752,
192,
1141,
665,
476,
604,
520,
84,
1525,
940,
585 
],
[
 "Connecticut",
2569,
2176,
393,
4367,
3805,
562,
509,
544,
0,
425,
420,
5,
905,
679,
226 
],
[
 "Delaware",
857,
575,
282,
1352,
1006,
346,
224,
147,
77,
170,
113,
57,
296,
172,
124 
],
[
 "DC",
729,
310,
419,
742,
543,
199,
73,
78,
0,
107,
61,
46,
169,
117,
52 
],
[
 "Florida",
17586,
13352,
4234,
28249,
23195,
5054,
5327,
3501,
1826,
3481,
2655,
826,
6927,
3675,
3252 
],
[
 "Georgia",
9103,
5120,
3983,
11820,
8967,
2853,
2413,
1263,
1150,
1965,
989,
976,
3133,
1791,
1342 
],
[
 "Hawaii",
1007,
836,
171,
1555,
1467,
88,
141,
212,
0,
244,
163,
81,
344,
259,
85 
],
[
 "Idaho",
1080,
883,
197,
1753,
1546,
207,
409,
224,
185,
234,
174,
60,
516,
285,
231 
],
[
 "Illinois",
11424,
7249,
4175,
16558,
12654,
3904,
2740,
1815,
925,
2047,
1412,
635,
3093,
2395,
698 
],
[
 "Indiana",
6421,
3783,
2638,
9385,
6612,
2773,
2154,
954,
1200,
1240,
739,
501,
2064,
1209,
855 
],
[
 "Iowa",
2716,
1892,
824,
4127,
3295,
832,
859,
485,
374,
462,
373,
89,
892,
571,
321 
],
[
 "Kansas",
2248,
1636,
612,
3624,
2854,
770,
826,
414,
412,
485,
321,
164,
1010,
525,
485 
],
[
 "Kentucky",
5332,
2662,
2670,
7499,
4655,
2844,
1792,
675,
1117,
934,
520,
414,
2240,
826,
1414 
],
[
 "Louisiana",
5784,
2609,
3175,
6909,
4562,
2347,
1106,
658,
448,
1003,
510,
493,
1771,
850,
921 
],
[
 "Maine",
1083,
928,
155,
2259,
1627,
632,
443,
237,
206,
229,
180,
49,
390,
262,
128 
],
[
 "Maryland",
5321,
3303,
2018,
7218,
5788,
1430,
1035,
818,
217,
935,
636,
299,
1065,
1093,
0 
],
[
 "Massachusetts",
4416,
3926,
490,
8319,
6865,
1454,
1115,
984,
131,
807,
761,
46,
1507,
1252,
255 
],
[
 "Michigan",
10327,
6056,
4271,
14394,
10600,
3794,
2721,
1527,
1194,
1743,
1178,
565,
2923,
1869,
1054 
],
[
 "Minnesota",
2720,
3050,
0,
6273,
5328,
945,
960,
762,
198,
662,
592,
70,
1342,
993,
349 
],
[
 "Mississippi",
4183,
1750,
2433,
4731,
3055,
1676,
1016,
446,
570,
827,
344,
483,
1395,
553,
842 
],
[
 "Missouri",
6553,
3691,
2862,
9023,
6442,
2581,
2090,
941,
1149,
1164,
724,
440,
2328,
1133,
1195 
],
[
 "Montana",
826,
650,
176,
1304,
1143,
161,
341,
166,
175,
162,
127,
35,
416,
190,
226 
],
[
 "Nebraska",
1252,
1063,
189,
2254,
1852,
402,
543,
270,
273,
294,
209,
85,
490,
337,
153 
],
[
 "Nevada",
2903,
1566,
1337,
3370,
2743,
627,
701,
395,
306,
446,
305,
141,
952,
510,
442 
],
[
 "New Hampshire",
916,
828,
88,
1772,
1455,
317,
315,
206,
109,
163,
158,
5,
381,
255,
126 
],
[
 "New Jersey",
7106,
5243,
1863,
10948,
9147,
1801,
1436,
1312,
124,
1319,
1015,
304,
1888,
1665,
223 
],
[
 "New Mexico",
1510,
1253,
257,
2393,
2194,
199,
535,
320,
215,
310,
246,
64,
1013,
386,
627 
],
[
 "New York",
17371,
11522,
5849,
23787,
20112,
3675,
3358,
2906,
452,
2423,
2246,
177,
3804,
3692,
112 
],
[
 "North Carolina",
9021,
5679,
3342,
13297,
9931,
3366,
2698,
1436,
1262,
1894,
1108,
786,
3268,
1802,
1466 
],
[
 "North Dakota",
512,
406,
106,
780,
708,
72,
170,
104,
66,
127,
80,
47,
193,
127,
66 
],
[
 "Ohio",
11875,
7164,
4711,
17413,
12514,
4899,
3729,
1818,
1911,
2271,
1400,
871,
4016,
2184,
1832 
],
[
 "Oklahoma",
4857,
2267,
2590,
5787,
3957,
1830,
1736,
581,
1155,
889,
448,
441,
1870,
703,
1167 
],
[
 "Oregon",
2421,
2364,
57,
5212,
4153,
1059,
1110,
599,
511,
635,
461,
174,
1068,
730,
338 
],
[
 "Pennsylvania",
12668,
8221,
4447,
19114,
14340,
4774,
3051,
2101,
950,
2194,
1611,
583,
4319,
2435,
1884 
],
[
 "Rhode Island",
820,
636,
184,
1423,
1112,
311,
225,
160,
65,
148,
123,
25,
339,
200,
139 
],
[
 "South Carolina",
5413,
2896,
2517,
7063,
5079,
1984,
1391,
740,
651,
1119,
567,
552,
1910,
883,
1027 
],
[
 "South Dakota",
590,
491,
99,
1054,
856,
198,
226,
126,
100,
126,
97,
29,
284,
151,
133 
],
[
 "Tennessee",
7956,
3916,
4040,
10185,
6853,
3332,
2197,
995,
1202,
1463,
765,
698,
2895,
1209,
1686 
],
[
 "Texas",
19939,
12683,
7256,
27141,
22143,
4998,
5061,
3139,
1922,
4254,
2471,
1783,
7612,
4551,
3061 
],
[
 "Utah",
1229,
1194,
35,
1931,
2080,
0,
383,
298,
85,
282,
238,
44,
765,
470,
295 
],
[
 "Vermont",
482,
411,
71,
921,
723,
198,
167,
103,
64,
91,
79,
12,
181,
122,
59 
],
[
 "Virginia",
6588,
4609,
1979,
10162,
8073,
2089,
1647,
1148,
499,
1369,
891,
478,
1889,
1521,
368 
],
[
 "Washington",
4437,
3844,
593,
8193,
6754,
1439,
1451,
956,
495,
907,
743,
164,
1925,
1269,
656 
],
[
 "West Virginia",
2400,
1308,
1092,
3415,
2289,
1126,
921,
338,
583,
464,
257,
207,
1031,
364,
667 
],
[
 "Wisconsin",
4513,
3424,
1089,
7530,
5978,
1552,
1190,
862,
328,
869,
667,
202,
1666,
1074,
592 
],
[
 "Wyoming",
492,
333,
159,
695,
585,
110,
186,
83,
103,
73,
65,
8,
296,
106,
190 
] 
];
data.addColumn('string','State');
data.addColumn('number','HeartDiseasesObserved');
data.addColumn('number','HeartDiseasesExpected');
data.addColumn('number','HeartDiseasesPreventable');
data.addColumn('number','CancerDiseasesObserved');
data.addColumn('number','CancerDiseasesExpected');
data.addColumn('number','CancerDiseasesPreventable');
data.addColumn('number','ChroniclowerrespiratoryDiseasesObserved');
data.addColumn('number','ChroniclowerrespiratoryDiseasesExpected');
data.addColumn('number','ChroniclowerrespiratoryDiseasesPreventable');
data.addColumn('number','CerebrovascularDiseasesObserved');
data.addColumn('number','CerebrovascularDiseasesExpected');
data.addColumn('number','CerebrovascularDiseasesPreventable');
data.addColumn('number','UnintentionalinjuriesObserved');
data.addColumn('number','UnintentionalinjuriesExpected');
data.addColumn('number','UnintentionalinjuriesPreventable');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartTableID1dc0551279ba() {
var data = gvisDataTableID1dc0551279ba();
var options = {};
options["allowHtml"] = true;
options["height"] =    400;
options["width"] =    800;

    var chart = new google.visualization.Table(
    document.getElementById('TableID1dc0551279ba')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "table";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartTableID1dc0551279ba);
})();
function displayChartTableID1dc0551279ba() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartTableID1dc0551279ba"></script>
 
<!-- divChart -->
  
<div id="TableID1dc0551279ba"
  style="width: 800px; height: 400px;">
</div>


For each type of disease, we do the following. Instead of dealing with raw numbers of potential deaths preventable, we compute the percentage of potential deaths preventable among the number of deaths observed. We then also compute the average percentage of potential deaths preventable among the 5 categories of diseases.   

```r
statewise$PercentageHeartDiseasesPreventable = round(statewise$HeartDiseasesPreventable * 
    100/statewise$HeartDiseasesObserved, 2)
statewise$PercentageCancerDiseasesPreventable = round(statewise$CancerDiseasesPreventable * 
    100/statewise$CancerDiseasesObserved, 2)
statewise$PercentageChroniclowerrespiratoryDiseasesPreventable = round(statewise$ChroniclowerrespiratoryDiseasesPreventable * 
    100/statewise$ChroniclowerrespiratoryDiseasesObserved, 2)
statewise$PercentageCerebrovascularDiseasesPreventable = round(statewise$CerebrovascularDiseasesPreventable * 
    100/statewise$CerebrovascularDiseasesObserved, 2)
statewise$PercentageUnintentionalinjuriesPreventable = round(statewise$UnintentionalinjuriesPreventable * 
    100/statewise$UnintentionalinjuriesObserved, 2)

statewise$PercentageAveragePreventableDeaths = round((statewise$PercentageHeartDiseasesPreventable + 
    statewise$PercentageCancerDiseasesPreventable + statewise$PercentageChroniclowerrespiratoryDiseasesPreventable + 
    statewise$PercentageCerebrovascularDiseasesPreventable + statewise$PercentageUnintentionalinjuriesPreventable)/5, 
    2)

save(statewise, file = "statewise.Rda")
```


## Visualizations

Let's now start plotting bar charts and choropleths using googleVis within the shiny server environment. This application is hosted by RStudio in their shinyapps.io server.  Before we do that, we make the following modifications to the dataset.

1. Convert it into a long form such that all columns, besides *State* are collapsed into a single column with a new column for the corresponding value.
2. Reorder levels of the column of different diseases so that the average percentages and disease percentages are among the first few levels.
 

```r
library(reshape2)
statewisemelt = melt(statewise, id = "State")
statewisemelt$variable = factor(statewisemelt$variable, levels(statewisemelt$variable)[c(21, 
    16:20, 1:15)])
save(statewisemelt, file = "statewisemelt.Rda")
```


As mentioned previously, this application is hosted on [R-Studio's shinyapps.io](shinyapps.io) platform. As mentioned at the beginning of the post, the [code necessary to recreate the post can be found on github.](https://github.com/patilv/preventabledeaths) The [code for the shiny app below can also be found on github.](https://github.com/patilv/preventableshiny) **You can hover over either the bars of the barchart or over the map to get the corresponding values.**  
<iframe height="800" src="https://patilv.shinyapps.io/preventableshiny/" width="1200"></iframe>
