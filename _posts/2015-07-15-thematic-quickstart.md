---
layout: post
title: Thematic Mapping in R without the Tears
tags: journalism
--- 

By: Richard Careaga <img src='https://careaga.s3.amazonaws.com/technocrat.png' alt='logo' style='height: 25px;'/> 2015-07-15

# Quick Start Edition

## Prerequisites

* A current, working installation of [R] 
* Ability to install required packages
* Basic familiarity with [R]
* Understanding of directory paths

## What you'll see

* The default basemap of the contiguous 48 states and the District of Columbia
 ![](https://careaga.s3.amazonaws.com/2015-07-15-default.png)
* The same map with an uncolored background  ![](https://careaga.s3.amazonaws.com/2015-07-15-white.png)
* The basemap with states shown by population with the default continuous scale and color scheme
 ![](https://careaga.s3.amazonaws.com/2015-07-15-pop.default.png)
* The basemap in four alternative discrete color scales with a neutral palette and polyconic projection
  ![](https://careaga.s3.amazonaws.com/2015-07-15-scaling.png)

## Next up

I will be preparing a detailed explanation of how this works and recommendations for deciding when this type of presentation works well and what data transforms to consider.

## Disclaimer
I am unable to answer questions relating to the use of this code under Windows.

## Credits

### Algorithms

[Interval scales]

### Data

* [Population U.S. Census]
* [Cartographic Boundary File U.S. Census] 
* [U.S. Census FIPS]

### R Packages
* Baptiste Auguie (2012). gridExtra: functions in Grid graphics. R package version 0.9.1. [gridExtra]
* Roger Bivand (2015). classInt: Choose Univariate Class Intervals. R package version 0.1-22.[classInt]
* Roger Bivand, Tim Keitt and Barry Rowlingson (2015). rgdal: Bindings for the Geospatial Data Abstraction Library. R package version 1.0-4 [rgdal]
* D. Kahle and H. Wickham. ggmap: Spatial Visualization with ggplot2. The R Journal, 5(1), 144-161.  [ggmap]
* Duncan Temple Lang and the CRAN team (2015). RCurl: General Network (HTTP/FTP/...) Client Interface for R. R package version 1.95-4.7. [RCurl] 
Hadley Wickham and Francois Romain. [dplyr]
* Erich Neuwirth (2014). RColorBrewer: ColorBrewer Palettes. R package version 1.1-2. [RColorBrewer]
* R Core Team (2015). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria. [grid]
* H. Wickham. ggplot2: elegant graphics for data analysis. Springer New York, 2009.  [ggplot2]

## Data
* Cartographic Boundary File: Download latest [cartographic boundary file for states] from the U.S Census, currently [cb_2014_us_state_5m.zip]
* [State postal codes], copy to csv, as follows:
<pre>
	Alabama,01,AL
	Alaska,02,AK
	Arizona,04,AZ
	Arkansas,05,AR
	California,06,CA
	Colorado,08,CO
	Connecticut,09,CT
	Delaware,10,DE
	District of Columbia,11,DC
	Florida,12,FL
	Georgia,13,GA
	Hawaii,15,HI
	Idaho,16,ID
	Illinois,17,IL
	Indiana,18,IN
	Iowa,19,IA
	Kansas,20,KS
	Kentucky,21,KY
	Louisiana,22,LA
	Maine,23,ME
	Maryland,24,MD
	Massachusetts,25,MA
	Michigan,26,MI
	Minnesota,27,MN
	Mississippi,28,MS
	Missouri,29,MO
	Montana,30,MT
	Nebraska,31,NE
	Nevada,32,NV
	New Hampshire,33,NH
	New Jersey,34,NJ
	New Mexico,35,NM
	New York,36,NY
	North Carolina,37,NC
	North Dakota,38,ND
	Ohio,39,OH
	Oklahoma,40,OK
	Oregon,41,OR
	Pennsylvania,42,PA
	Rhode Island,44,RI
	South Carolina,45,SC
	South Dakota,46,SD
	Tennessee,47,TN
	Texas,48,TX
	Utah,49,UT
	Vermont,50,VT
	Virginia,51,VA
	Washington,53,WA
	West Virginia,54,WV
	Wisconsin,55,WI
	Wyoming,56,WY
</pre>

## License

Copyright (c) 2015, Richard Careaga

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## R Code

```R
# 
x <- c("dplyr", "ggmap", "ggplot2", "RColorBrewer", "rgdal", "classInt", "RCurl",  "grid","gridExtra")
lapply(x, library, character.only = TRUE)

no_ylab = ylab("") 
no_xlab = xlab("")

plain_theme = theme(axis.text=element_blank()) + theme(panel.background = element_blank(), panel.grid = element_blank(), axis.ticks = element_blank())

remove.farflung = function(.df) {
	subset(.df, 
		.df$id != "AS" &
		.df$id != "MP" &
		.df$id != "GU" & 
		.df$id != "PR" &
		.df$id != "VI" &
		.df$id != "AK" &
		.df$id != "HI"
	)
}

intervals = function(.df, ...){
	argList = match.call(expand.dots=FALSE)$... 
	for(i in 1:length(argList)){
		colName <- argList[[i]]
		series_colName = eval(substitute(colName), envir=.df, enclos=parent.frame())
		min <- min(series_colName)
		max <- max(series_colName)
		diff <- max - min
		std <- sd(series_colName)
		equal.interval <- seq(min, max, by = diff/6)
		quantile.interval <- quantile(series_colName, probs = seq(0, 1, by = 1/6))
		std.interval <- c(seq(min, max, by = std), max)
		natural.interval <- classIntervals(series_colName, n = 6, style = 'jenks')$brks
		.df$equal <- cut(series_colName, breaks = equal.interval, include.lowest = TRUE)
		names(.df)[names(.df)=="equal"] <- paste(colName,".","equal", sep = '')
		.df$quantile <- cut(series_colName, breaks = quantile.interval, include.lowest = TRUE)
		names(.df)[names(.df)=="quantile"] <- paste(colName,".","quantile", sep = '')
		.df$std <- cut(series_colName, breaks = std.interval, include.lowest =  TRUE)
		names(.df)[names(.df)=="std"] <- paste(colName,".","std", sep = '')
		.df$natural <- cut(series_colName, breaks = natural.interval, include.lowest = TRUE)
		names(.df)[names(.df)=="natural"] <- paste(colName,".","natural", sep = '')
	}
	return(.df)
}
setwd("/Users/rc/Desktop/gitmo/playpen/r/")  # Your directory here
dsn <- "data/cb_2014_us_state_5m"
layer <- "cb_2014_us_state_5m"

cb5 = readOGR(dsn, layer)
us = fortify(cb5, region = "STUSPS")

population<- read.csv(textConnection(getURL('https://www.census.gov/popest/data/national/totals/2014/files/NST_EST2014_ALLDATA.csv')))[-(1:5),]


pop <- data.frame(population$NAME,population$POPESTIMATE2014)
names(pop) = c("state", "pop")

converter = read.csv("csv/state_fips_postal.csv", header = FALSE)
converter = rename(converter, state = V1, fips = V2, id = V3)

pop = merge(pop, converter)
pop$state = NULL
pop$fips = NULL

pop48 = remove.farflung(pop)

pop48 = intervals(pop, pop)

us48 = remove.farflung(us)

map48 = merge(us48,pop48) # has to be geo first

my_title = "Map of 48 Contiguous States and District of Columbia, default settings"
sources <- "Source: U.S. Census Bureau 2014 Cartographic Boundary File (States, cb_2014_us_state_5m) http://1.usa.gov/1IZgFLV\nPrepared by: @technocrat 2015-07-15"

b = ggplot(data = map48, aes(x=long, y=lat, group = group))

d = b + geom_polygon(aes(x=long, y=lat, group=group), color = "dark grey", size = 0.3) + ggtitle(my_title) + annotate("text", x = -110, y = 18, label = sources, size = 3, family = "Times", colour = "black")

d

my_title = "Map of 48 Contiguous States and District of Columbia, white background"

w = d + geom_polygon(aes(x=long, y=lat, group=group), color = "dark grey", size = 0.3, fill="white") + ggtitle(my_title) + annotate("text", x = -110, y = 18, label = sources, size = 3, family = "Times", colour = "black")

w

my_title = "Population Map of 48 Contiguous States and District of Columbia, default settings"
sources <- "Source: U.S. Census Bureau 2014 Population Estimates http://1.usa.gov/1LPM7hc\n and Cartographic Boundary File (States, cb_2014_us_state_5m) http://1.usa.gov/1IZgFLV\nPrepared by: @technocrat 2015-07-15"
p = b + geom_polygon(color = "dark grey", size = 0.3, aes(fill = pop)) + ggtitle(my_title) + annotate("text", x = -110, y = 18, label = sources, size = 3, family = "Times", colour = "black") # blues, reversed

p

my_title = "Population Map of 48 Contiguous States and District of Columbia,\n no background or axis legends, polyconic projection"

p + plain_theme + no_ylab + no_xlab + coord_map("polyconic")  + ggtitle(my_title) + annotate("text", x = -110, y = 18, label = sources, size = 3, family = "Times", colour = "black") # blues, reversed

my_title = "Equal Interval Scale"

e = b + geom_polygon(aes(x=long, y=lat, group=group, fill=pop.equal), color = "dark grey", size = 0.3)

e = e +
plain_theme + no_ylab + no_xlab + coord_map("polyconic") + scale_fill_brewer(palette = "YlOrBr") + ggtitle(my_title)

my_title = "Quantile Interval Scale"

q = b + geom_polygon(aes(x=long, y=lat, group=group, fill=pop.quantile), color = "dark grey", size = 0.3)

q = q +
plain_theme + no_ylab + no_xlab + coord_map("polyconic") + scale_fill_brewer(palette = "YlOrBr") + ggtitle(my_title)

my_title = "Standard Deviation Interval Scale"

s = b + geom_polygon(aes(x=long, y=lat, group=group, fill=pop.std), color = "dark grey", size = 0.3)

s = s + plain_theme + no_ylab + no_xlab + coord_map("polyconic") + scale_fill_brewer(palette = "YlOrBr") + ggtitle(my_title)

my_title = "Natural Interval Scale"

n = b + geom_polygon(aes(x=long, y=lat, group=group, fill=pop.natural), color = "dark grey", size = 0.3)

n = n + plain_theme + no_ylab + no_xlab + coord_map("polyconic") + scale_fill_brewer(palette = "YlOrBr") + ggtitle(my_title)

grid.arrange(e, q, s, n, ncol = 2, main = "Population Maps of 48 Contiguous States and District of Columbia\nEffect of Scaling (with neutral palette)")
```

<!---
rbitlyApi("YOUR_KEY_HERE")
pop_source <- links_Shorten("https://www.census.gov/popest/data/national/totals/2014/files/NST_EST2014_ALLDATA.csv")$url
-->



[cartographic boundary file for states]: http://www.census.gov/geo/maps-data/data/cbf/cbf_state.html
[Cartographic Boundary File U.S. Census]: http://1.usa.gov/1IZgFLV
[cb_2014_us_state_5m.zip]: http://www2.census.gov/geo/tiger/GENZ2014/shp/cb_2014_us_state_5m.zip
[classInt]: http://CRAN.R-project.org/package=classInt
[dplyr]: https://github.com/hadley/dplyr
[U.S. Census FIPS Codes]: https://www.census.gov/geo/reference/ansi_statetables.html
[ggmap]: http://journal.r-project.org/archive/2013-1/kahle-wickham.pdf
[ggplot2]: http://had.co.nz/ggplot2/book
[grid]: http://r-project.org
[gridExtra]: http://CRAN.R-project.org/package=gridExtra
[Interval scales]: https://rpubs.com/danielkirsch/styling-choropleth-maps
[Population U.S. Census]: http://1.usa.gov/1LPM7hc
[R]: http://rproject.org
[RColorBrewer]: http://CRAN.R-project.org/package=RColorBrewer
[RCurl]: http://CRAN.R-project.org/package=RCurl
[rgdal]: http://CRAN.R-project.org/package=rgdal
[State postal codes]:  https://www.census.gov/geo/reference/ansi_statetables.html
