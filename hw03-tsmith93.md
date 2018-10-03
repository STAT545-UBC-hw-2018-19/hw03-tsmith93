hw03-tsmith93
================
Thomas Smith
2018-09-27

Loading packages
----------------

If you haven't already done so, download both gapminder and tidyverse using `install.packages()`

Next load gapminder, tidyverse and knitr:

``` r
#suppressPackageStartupMessages stops unecessary messages from popping up
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(gapminder))
suppressPackageStartupMessages(library(knitr))
```

Great! Now we're ready to practice manipulating our data with `diplyr`, and using `ggplot2` for visualization. Think of this file as a cheat sheet.

Problem solving uing dplyr and ggplot2
--------------------------------------

### Maximum and minimum of GDP per capita for all continents

For this situation, we will creatw a table with three collumns, titled "Continent", "Minimum GDP per capita", and "Maximum GDP per capita".

``` r
#call upton gapminder
gapminder %>% 
#Next we wantto group the table by continent  
  group_by(continent) %>% 
#Make the table look clean by rounding to 2 decimal places  
  mutate_each(funs(round(.,2)), gdpPercap) %>% 
#Now summarize the data, so only minimum and maximum gdpPercap are shown  
  summarize(minimum = min(gdpPercap),
  maximum =max(gdpPercap)) %>% 
#And finally, oresent data in a clean looking table with clear collumn names  
  kable(col.names = c("Continent", " Minimum GDP per Capita ($)", "Maximum GDP per Capita ($)"))  
```

    ## `mutate_each()` is deprecated.
    ## Use `mutate_all()`, `mutate_at()` or `mutate_if()` instead.
    ## To map `funs` over a selection of variables, use `mutate_at()`

| Continent |  Minimum GDP per Capita ($)|  Maximum GDP per Capita ($)|
|:----------|---------------------------:|---------------------------:|
| Africa    |                      241.17|                    21951.21|
| Americas  |                     1201.64|                    42951.65|
| Asia      |                      331.00|                   113523.13|
| Europe    |                      973.53|                    49357.19|
| Oceania   |                    10039.60|                    34435.37|

There we have it! It's to see that Asia had the highest gdpPercap at 113523.13 and Africa had the lowest gdpPercap at 241.17.

Let's double check to see if these values are what we are looking for:

``` r
gapminder %>% 
#Let's just look at Asia's data using filter  
  filter(continent == "Asia") %>% 
#Arrange the data from lowest gdpPercap to highest  
  arrange(gdpPercap)
```

    ## # A tibble: 396 x 6
    ##    country  continent  year lifeExp       pop gdpPercap
    ##    <fct>    <fct>     <int>   <dbl>     <int>     <dbl>
    ##  1 Myanmar  Asia       1952    36.3  20092996      331 
    ##  2 Myanmar  Asia       1992    59.3  40546538      347 
    ##  3 Myanmar  Asia       1967    49.4  25870271      349 
    ##  4 Myanmar  Asia       1957    41.9  21731844      350 
    ##  5 Myanmar  Asia       1972    53.1  28466390      357 
    ##  6 Cambodia Asia       1952    39.4   4693836      368.
    ##  7 Myanmar  Asia       1977    56.1  31528087      371 
    ##  8 Myanmar  Asia       1987    58.3  38028578      385 
    ##  9 Myanmar  Asia       1962    45.1  23634436      388 
    ## 10 China    Asia       1952    44   556263527      400.
    ## # ... with 386 more rows

Voila! The lowest gdpPercap was $331. Our diplyr manipulations worked!

Now, lets present this data via a boxplot. This type of figure will show the minimum and maximum data points, along with many other insigthful data spread information. First, lets fill out a grammar component table:

| Grammar Component     | Specification    |
|-----------------------|------------------|
| **data**              | `gapminder`      |
| **aesthetic mapping** | `x` and `y`      |
| **geometric object**  | boxplot          |
| scale                 | none             |
| statistical transform | 5-number summary |

``` r
#Lets assign this ggplot to "gdp", making continent the x variable, and gdpPercap the y variable
gdp <-ggplot(gapminder, aes(continent, gdpPercap)) 
#let's make continent/box have a different colour
gdp + geom_boxplot(aes(fill=continent)) +
#Capitalise the axis names for a clean look  
  xlab("Continent") +
  ylab("GDP per capita") +
#Captialise the legend title too  
  guides(fill=guide_legend(title="Continent")) +
#Now some code that will remove the gray grid background, and make the axes lines a sharo black  
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black")) 
```

![](hw03-tsmith93_files/figure-markdown_github/GDP%20per%20cap%20figure-1.png)

There you go. A more visual way of showing that Asia has the highest gdp per capita, and Africa has the lowest.

Life Expectancy over year
-------------------------

For this problem, a scatterplot could work nicely. We will start with a grammar component table, as per usual:

| Grammar Component     | Specification |
|-----------------------|---------------|
| **data**              | `gapminder`   |
| **aesthetic mapping** | `x`and `y`    |
| **geometric object**  | point         |
| scale                 | linear        |
| statistical transform | none          |
| coordinate system     | rectangular   |
| facetting             | none          |

``` r
gapminder %>% 
#select year as x axis, and lifeExp as y  
ggplot(aes(year, lifeExp)) +
#Rename axes  
  xlab("Continent") +
  ylab("Life expectancy (years)") +
#Add geometric object - point in this case, coloured based on continent  
  geom_point(aes(colour = continent)) +
#Add trendlines, removing standard error bars, coloured by continent again  
   geom_smooth(se = FALSE, aes(colour = continent)) +
#Again, clean up the look of the figure by making background white and axes lines black
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black"))
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](hw03-tsmith93_files/figure-markdown_github/Life%20expectancy%20per%20continent%20figure-1.png)

This figure shows that the life expectancy in Oceania is the highest at around 80, and seems to increasing the most rapidly. The lowest is in Africa, at around 50, which seems to have not changed too drastically in recent years.

Now, lets present this data with a table. This may take a few steps to get the table exactly how we want it!

``` r
#First, assign the initial dataset manipulations to variable "longdata"
longdata <- gapminder %>% 
#Filter the first (1952) and last years (2007) of data out 
  filter(year == "1952" | year == "2007") %>% 
#Select only the variables continent, year and lifeExp
  select(continent, year, lifeExp) %>% 
#Group by continent and year  
  group_by(continent, year) %>% 
#Create a collumn containign mean life expectancy  
  summarize(Life = mean(lifeExp))

#Now convert longata to widedata using `spread`, selecing only year and "Life""
widedata <- spread(longdata, year, Life)
#Name new collumn names
colnames(widedata) <- c("continent", "first","last")

widedata %>% 
#Add another collumn to highlight the change in life expectancy  
  mutate(difference = last - first) %>% 
#Round to wo decimal places  
  mutate_each(funs(round(.,2))) %>% 
#Rename collumn names using `kable` for sharp look  
  kable(col.names = c("Continent", "Life expectancy in 1952", "Life expectancy in 2007", "Change in life expectancy"))
```

    ## `mutate_each()` is deprecated.
    ## Use `mutate_all()`, `mutate_at()` or `mutate_if()` instead.
    ## To map `funs` over all variables, use `mutate_all()`

| Continent |  Life expectancy in 1952|  Life expectancy in 2007|  Change in life expectancy|
|:----------|------------------------:|------------------------:|--------------------------:|
| Africa    |                    39.14|                    54.81|                      15.67|
| Americas  |                    53.28|                    73.61|                      20.33|
| Asia      |                    46.31|                    70.73|                      24.41|
| Europe    |                    64.41|                    77.65|                      13.24|
| Oceania   |                    69.25|                    80.72|                      11.46|

There ya go! `diplyr` and `kable()` can make a good looking graph. As show in the figure, Africa had the lowest life expectancy most recently at just over 54, and Oceania had the highest at jsut over 80. Though thsi tabl does clarif that the life expectancy in Asia has increased the most since 1952, by over 24 years. This show that the use of tables may sometimes be best for assessing datasets, as opposed to figures.

Interesting history
-------------------

Finally, we will use `diplyr` and `ggplot2` to explore the histor of a country. For this, we will look at Cambodia, as between 1975 and 1979, a terrible genocide occured in the country. Specifically, we will look at the population over time.

First we will make a scatterplot. | Grammar Component | Specification | |-----------------------|---------------| | **data** | `gapminder` | | **aesthetic mapping** | `x`and `y` | | **geometric object** | point | | scale | linear | | statistical transform | none | | coordinate system | rectangular | | facetting | none |

``` r
#Filter data for only "Cambodia", asssigning it to camb
camb <- gapminder %>% 
  filter(country == "Cambodia")

#Now further manipulate camb, reassinging to camgeno
camgeno <- camb %>% 
#Lets highligt the only data point (population in 1977) during the genocide for emphasis  
  mutate(highlight_year = ifelse(year == 1977 & pop > 0, T, F)) %>% 
#Now add the plot  
  ggplot(aes(x = year, y = pop)) +
#Rename axes for sharp look  
  xlab("Year") +
  ylab("Population") +
#Remove the legend  
  geom_point(aes(color = highlight_year), show.legend=F) +
#Make higlighted point red  
  scale_color_manual(values = c('#595959', 'red')) +
#Make background white, and axes sharp looking  
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(), panel.background = element_blank(),
  axis.line = element_line(colour = "black")) +
#Now add vertical lines to show the extent of the genocide period for emphasis  
  geom_vline(aes(xintercept=1975), color="red", linetype="dashed") +
  geom_vline(aes(xintercept=1979), color="red", linetype="dashed")

#Print the figure!
camgeno
```

![](hw03-tsmith93_files/figure-markdown_github/interesting%20history%20figure-1.png)

Finally, we will look at the table! This will be pretty straight forward. For this, we want 3 collums: Year, population and change in population.

``` r
#Select "Cambodia" data
camb %>% 
#Group by year  
  group_by(year) %>% 
#Select year and population  
  select(year, pop) %>% 
#Ungroup so values actually show when you create the "Change"" collumn  
  ungroup() %>% 
#Use lag function to make the collumn showing the change in population from the year before. Lag refers to the prior year.  
  mutate(change = pop - lag(pop)) %>%
#Give the table nice collumn names!  
  kable(col.names = c("Year", "Population", "Change in population")) 
```

|  Year|  Population|  Change in population|
|-----:|-----------:|---------------------:|
|  1952|     4693836|                    NA|
|  1957|     5322536|                628700|
|  1962|     6083619|                761083|
|  1967|     6960067|                876448|
|  1972|     7450606|                490539|
|  1977|     6978607|               -471999|
|  1982|     7272485|                293878|
|  1987|     8371791|               1099306|
|  1992|    10150094|               1778303|
|  1997|    11782962|               1632868|
|  2002|    12926707|               1143745|
|  2007|    14131858|               1205151|

Again, a beautiful table. It is clear here that during the year 1977, something aweful was occurring as the change in population from the year before was -471,999.

##Reference:

For getting "clean" figure look go [here](http://felixfan.github.io/ggplot2-remove-grid-background-margin/)
