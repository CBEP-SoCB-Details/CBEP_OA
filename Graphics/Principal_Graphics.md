Analysis of Casco Bay OA data through 2018 – Principal Graphics
================
Curtis C. Bohlen

  - [Introduction](#introduction)
  - [Load Libraries](#load-libraries)
      - [Generate color palette](#generate-color-palette)
  - [Load Data](#load-data)
      - [Establish Folder References](#establish-folder-references)
      - [Read Data](#read-data)
      - [Rearrange the data to facilitate
        plotting](#rearrange-the-data-to-facilitate-plotting)
      - [Constants for Axis Labels](#constants-for-axis-labels)
  - [Seasonal Profiles](#seasonal-profiles)
      - [Raw pCO2](#raw-pco2)
      - [Temperature Corrected pCO2](#temperature-corrected-pco2)
      - [Both Raw and Corrected pco<sub>2</sub> on One
        Graph.](#both-raw-and-corrected-pco2-on-one-graph.)
      - [pH](#ph)
      - [Aragonite Saturation State](#aragonite-saturation-state)
  - [Time Series Charts](#time-series-charts)
      - [Temperature, pH, and pCO2](#temperature-ph-and-pco2)
      - [Temperature, Dissolved Oxygen, and
        pCO2](#temperature-dissolved-oxygen-and-pco2)
      - [Temperature, Dissolved Oxygen, and pCO2 2017
        only](#temperature-dissolved-oxygen-and-pco2-2017-only)
  - [Correlation Plots](#correlation-plots)
      - [PH by Temperature](#ph-by-temperature)
          - [Recolored with a Sequential
            Palette](#recolored-with-a-sequential-palette)
      - [PCO2 by Temperature](#pco2-by-temperature)
      - [pCO<sub>2</sub> by Oxygen](#pco2-by-oxygen)
      - [pH by pCO<sub>2</sub>](#ph-by-pco2)
      - [Omega by pCO<sub>2</sub>](#omega-by-pco2)
      - [Alkalinity vs. Salinity](#alkalinity-vs.-salinity)
      - [Alkalinity by Salinity,
        seasonal](#alkalinity-by-salinity-seasonal)

<img
    src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
    style="position:absolute;top:10px;right:50px;" />

# Introduction

This notebook and related notebooks document exploratory data analysis
of data derived from a multi-year deployment of ocean acidification
monitoring equipment at the Southern Maine Community College pier, in
South Portland.

The monitoring set up was designed and operated by Joe Kelly, of UNH and
his colleagues, on behalf of the Casco Bay Estuary Partnership. This was
one of the first long-term OA monitoring facilities in the northeast,
and was intended to test available technologies as well as gain
operational experience working with acidification monitoring.

In this Notebook, we develop the primary graphics used to examine
acidification in Casco Bay Estuary partnership’s 2020 State of the Bay
report.

# Load Libraries

``` r
library(tidyverse)
```

    ## -- Attaching packages ---------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.1     v dplyr   1.0.0
    ## v tidyr   1.1.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ------------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)
library(GGally)
```

    ## Registered S3 method overwritten by 'GGally':
    ##   method from   
    ##   +.gg   ggplot2

``` r
library(CBEPgraphics)
load_cbep_fonts()

library(RColorBrewer)
display.brewer.all(n=9, type="seq", exact.n=TRUE, colorblindFriendly = TRUE)
```

![](Principal_Graphics_files/figure-gfm/load_libraries-1.png)<!-- -->

## Generate color palette

To display data by months in complex graphics, we want a 12 item
sequential color palette that’s color-blind / reproduction friendly.

The rColorBrewer colorRampPalette() function creates a FUNCTION that
takes the number of colors, and returns a suitable color ramp, based on
another (usually shorter) color ramp. Note getPalette is a FUNCTION.
We’ll use this function later to generate the colors we want on the
fly.

``` r
getPalette = colorRampPalette(brewer.pal(9, "YlGnBu")[2:9])  # generates a palette function from an existing color ramp
```

# Load Data

## Establish Folder References

``` r
sibfldnm <- 'Derived_Data'
parent   <- dirname(getwd())
sibling  <- file.path(parent,sibfldnm)

fn    <- 'CascoBayOAData.csv'
fpath <- file.path(sibling,fn)
```

The following loads existing data, including a “temperature corrected”
pCO2 value based on Takehashi et al. 2002. It then collapses that data
to daily summaries.

> Takahashi, Taro & Sutherland, Stewart & Sweeney, Colm & Poisson, Alain
> & Metzl, Nicolas & Tilbrook, Bronte & Bates, Nicholas & Wanninkhof,
> Rik & Feely, Richard & Chris, Sabine & Olafsson, Jon & Nojiri,
> Yukihiro. (2002). Global sea-air CO2 flux based on climatological
> surface ocean pCO2, and seasonal biological and temperature effects.
> Deep Sea Research Part II: Topical Studies in Oceanography. 49.
> 1601-1622. 10.1016/S0967-0645(02)00003-6.

(See the “Data\_Review\_And\_Filtering” R Notebook for details on why
and how we calculated temperature-corrected pCO2 values.)

## Read Data

``` r
all_data <- read_csv(fpath,
                     col_types = cols(dd = col_integer(), 
                                      doy = col_integer(),
                                      hh = col_integer(),
                                      mm = col_integer(),
                                      yyyy = col_integer())) %>%
    select(c(datetime, yyyy, mm, dd, hh, doy, 
           temp, sal, do, ph, co2, co2_corr,
           omega_a, omega_c, TA_calc))
```

## Rearrange the data to facilitate plotting

ggplot often likes data in “long” form, so that the data from multiple
categories can be graphed the same way. Here, we need to move all the
different parameters into one column, so that we can use ggplot’s
faceting capabilities to plot selected parameters in neat and tidy
coordinated plots.

``` r
long_data <- all_data %>%
  pivot_longer(cols= temp:TA_calc, names_to='Parameter', values_to = 'Value') %>%
  mutate(Parameter = factor(Parameter,
                            levels = c('temp', 'sal', 'do', 'co2', 'co2_corr', 'ph',
                                       'omega-a', 'omega-c', 'TA_calc')))
```

## Constants for Axis Labels

``` r
monthlengths <-  c(31,28,31, 30,31,30,31,31,30,31,30,31)
cutpoints    <- c(0, cumsum(monthlengths)[1:12])
monthlabs    <- c(month.abb,'')
```

# Seasonal Profiles

These graphs combine data from multiple years to generate a picture of
seasonal conditions across multiple years. Since data coverage is
inconsistent year to year, data for some times of year are derived from
just one or two years, which could bias the results.

## Raw pCO2

It’s not technically OK to show a reference line on a figure with
temperature-corrected pCO2. The equilibrium between \[co<sub>2</sub>\]
and fugacity is temperature dependent. It would be appropriate on a
graphs showing uncorrected pCO<sub>2</sub>.

``` r
plt <- ggplot(all_data, aes(doy, co2_corr)) + geom_point(aes(color = factor(yyyy)), alpha = 0.1) +
  
  # geom_hline(aes(yintercept = 400), lty = 'dotted', color = 'gray') +
  # annotate('text', x=365, y=370, label= expression(pCO[2*(cor)]~'='~ 400), hjust=1, size=3) +
  
  xlab('Day of Year') +
  ylab(expression (pCO[2]~(mu*Atm))) +
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = monthlabs) +
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
plt
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/pc02_Raw_by_doy-1.png)<!-- -->

``` r
ggsave('pco2RawSeasonal.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

``` r
ggsave('pco2RawSeasonal.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

## Temperature Corrected pCO2

It’s not technically OK to show a reference line on a figure with
temperature-corrected pCO2. The equilibrium between \[co<sub>2</sub>\]
and fugacity is temperature dependent. It would be appropriate on a
graphs showing uncorrected pCO<sub>2</sub>.

``` r
plt <- ggplot(all_data, aes(doy, co2_corr)) + geom_point(aes(color = factor(yyyy)), alpha = 0.1) +
  
  # geom_hline(aes(yintercept = 400), lty = 'dotted', color = 'gray') +
  # annotate('text', x=365, y=370, label= expression(pCO[2*(cor)]~'='~ 400), hjust=1, size=3) +
  
  xlab('Day of Year') +
  ylab(expression (pCO[2*(cor)]~(mu*Atm))) +
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = monthlabs) +
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
plt
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/pc02_by_doy-1.png)<!-- -->

``` r
ggsave('pco2Seasonal.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

``` r
ggsave('pco2Seasonal.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 6158 rows containing missing values (geom_point).

## Both Raw and Corrected pco<sub>2</sub> on One Graph.

``` r
plt  <- long_data %>% filter(Parameter %in% c('co2', 'co2_corr')) %>%
  
  mutate(Parameter = factor(Parameter, levels = c('co2', 'co2_corr'),
                            labels = c('Observed',
                                       'Temp. Corrected'))) %>% 
  
  ggplot(aes(x=doy, y=Value, alpha=Parameter)) + geom_line(aes(color = Parameter)) +
 
  scale_alpha_discrete(range = c(0.25, 1), name = '') +
  scale_x_continuous(breaks = cutpoints, labels = monthlabs) + 
  scale_color_manual(values=cbep_colors2(), name='') +

  xlab('') +
  ylab(expression (pCO[2]~(mu*Atm))) +

  #guides(colour = guide_legend(override.aes = list(alpha = c(0.25, 1)))) +
  
  theme_cbep(base_size = 10) +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5)) +
  theme(legend.position= c(.3, .8))
```

    ## Warning: Using alpha for a discrete variable is not advised.

``` r
plt
```

    ## Warning: Removed 50 row(s) containing missing values (geom_path).

![](Principal_Graphics_files/figure-gfm/pco2_comparison-1.png)<!-- -->

``` r
ggsave('pco2compare.png', type = 'cairo', width = 3, height = 2)
```

    ## Warning: Removed 50 row(s) containing missing values (geom_path).

``` r
ggsave('pco2compare.pdf', device=cairo_pdf, width = 3, height = 2)
```

    ## Warning: Removed 50 row(s) containing missing values (geom_path).

## pH

``` r
plt <- ggplot(all_data, aes(doy, ph)) + geom_point(aes(color = factor(yyyy)),alpha = 0.05) +
  xlab('Day of Year') +
  ylab('pH') +
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = monthlabs) +

  scale_y_continuous(limits = c(7.5, 8.5), breaks = c(7.5, 7.75, 8.0, 8.25, 8.5)) +
  
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
plt
```

    ## Warning: Removed 11858 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/ph_by_doy-1.png)<!-- -->

``` r
ggsave('phSeasonal.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 11858 rows containing missing values (geom_point).

``` r
ggsave('phSeasonal.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 11858 rows containing missing values (geom_point).

## Aragonite Saturation State

``` r
plt <- ggplot(all_data, aes(doy, omega_a)) + geom_point(aes(color = factor(yyyy)), alpha = 0.1) +
  
  # geom_hline(aes(yintercept = 1.5), lty = 'dotted', color = 'gray') +
  # geom_text(aes(x=0, y=1.4, label= 'Omega = 1.5', hjust = 0), size=3) +
  
  geom_hline(aes(yintercept = 1), lty = 'dotted', color = 'gray') +
  geom_text(aes(x=0, y=0.9, label= 'Omega = 1.0', hjust = 0), size=3) +
  
  
  xlab('Day of Year') +
  ylab(expression(Omega[a])) +
  
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = monthlabs) +
  
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
  
plt
```

    ## Warning: Removed 17575 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/omega_by_doy-1.png)<!-- -->

``` r
ggsave('omegaSeasonal.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 17575 rows containing missing values (geom_point).

``` r
ggsave('omegaSeasonal.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 17575 rows containing missing values (geom_point).

# Time Series Charts

## Temperature, pH, and pCO2

The only complexity in this code is telling ggplot not to link lines
across missing values. That is accomplished here by creating a dummy
variable for each day. Assigning that to the group aesthetic forces
ggplot to only draw lines between points within the same day. At this
scale, this works well, but it would NOT work zoomed in enough to see
the lack of lines connecting between 23:00 one day and 00:00 the next
day.

``` r
plt  <- long_data %>% filter(Parameter %in% c('temp', 'ph', 'co2_corr')) %>%
  mutate(Parameter = factor(Parameter, levels = c('temp', 'ph', 'co2_corr'),
                            labels = c(expression(Temp.~degree*C),
                                       'pH',
                                       expression(pCO[2*(cor)]~(mu*Atm))))) %>%
  mutate(daygroup = paste0(sprintf("%02d",dd),
                           sprintf("%02d",mm),
                           sprintf("%04d",yyyy))) %>%
  
  ggplot(aes(x=datetime, y=Value, group = daygroup)) + geom_line(aes(color = Parameter)) +
  #theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank()) +
  xlab('Date')  +
  ylab(NULL) +
  
  theme_cbep() +
  theme(strip.background = element_blank(),
        strip.placement = "outside",
        strip.text = element_text(size = 10),
        legend.position = "none") + 
  
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  
  facet_wrap(~Parameter, nrow = 3, scales = 'free_y',
             strip.position = "left",
             labeller = label_parsed) 
plt
```

    ## Warning: Removed 6824 row(s) containing missing values (geom_path).

![](Principal_Graphics_files/figure-gfm/t_ph_pco2-1.png)<!-- --> Note
that there’s one wonky temperature reading in mid 2015. There may even
be a couple of other odd measurements in 2019. I have not filtered them
out because I have no clear data QA/QC standards for doing so, and I
received data from UNH that had already gone through some QA/QC
processes. (For more on this problems, see the Data Review and Filtering
notebook).

## Temperature, Dissolved Oxygen, and pCO2

``` r
plt  <- long_data %>% 
  filter(Parameter %in% c('temp', 'do', 'co2_corr')) %>%

  mutate(Parameter = factor(Parameter, levels = c('temp', 'do', 'co2_corr'),
                            labels = c(expression(Temp.~degree*C),
                                       expression( DO~(mu*Mol %.% kg^{-1}) ),
                                       expression(pCO[2*(cor)]~(mu*Atm  ))  ) )  ) %>%
  mutate(daygroup = paste0(sprintf("%02d",dd),
                           sprintf("%02d",mm),
                           sprintf("%04d",yyyy))) %>%

  ggplot(aes(x=datetime, y=Value, group=daygroup)) + geom_line(aes(color = Parameter)) +
  
  theme_cbep() +
  theme(strip.background = element_blank(),
        strip.placement = "outside",
        strip.text = element_text(size = 9),
        legend.position = "none") +
  
  xlab('Date')  +
  ylab(NULL) +
  
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  
  facet_wrap(~Parameter, nrow = 3, scales = 'free_y',
             strip.position = "left",
             labeller = label_parsed)


plt
```

    ## Warning: Removed 9003 row(s) containing missing values (geom_path).

![](Principal_Graphics_files/figure-gfm/t_do_pco2-1.png)<!-- -->

``` r
ggsave('t.02.co2.png', type = 'cairo', height = 5, width = 8)
```

    ## Warning: Removed 9003 row(s) containing missing values (geom_path).

``` r
ggsave('t.02.co2.pdf', device = cairo_pdf, height = 5, width = 8)
```

    ## Warning: Removed 9003 row(s) containing missing values (geom_path).

## Temperature, Dissolved Oxygen, and pCO2 2017 only

``` r
plt  <- long_data %>% 
  filter(Parameter %in% c('temp', 'do', 'co2', 'co2_corr')) %>%
  filter(yyyy==2017) %>%
  
  mutate(Parameter = factor(Parameter, levels = c('temp', 'do', 'co2', 'co2_corr'),
                            labels = c(expression(Temp.~degree*C),
                                       expression( DO~(mu*Mol %.% kg^{-1}) ),
                                       expression(pCO[2]~(mu*Atm  )),                                        
                                       expression(pCO[2*(cor)]~(mu*Atm  ))  ) )  ) %>%
  mutate(daygroup = paste0(sprintf("%02d",dd),
                           sprintf("%02d",mm),
                           sprintf("%04d",yyyy))) %>%

  ggplot(aes(x=datetime, y=Value, group=daygroup)) + geom_line(aes(color = Parameter)) +
  
  theme_cbep() +
  theme(strip.background = element_blank(),
        strip.placement = "outside",
        strip.text = element_text(size = 9),
        legend.position = "none") +
  
  xlab('Date (2017)')  +
  ylab(NULL) +
  
  scale_color_manual(values=cbep_colors()[c(3,2,5,4)], name='Year') +
  
  facet_wrap(~Parameter, nrow = 4, scales = 'free_y',
             strip.position = "left",
             labeller = label_parsed)

plt
```

    ## Warning: Removed 685 row(s) containing missing values (geom_path).

![](Principal_Graphics_files/figure-gfm/t_do_pco2_2017-1.png)<!-- -->

``` r
#ggsave('t.02.co2_2017.png', type = 'cairo', height = 5, width = 8)
#ggsave('t.02.co2_2017.pdf', device = cairo_pdf, height = 5, width = 8)
```

Notice the effect of correcting pCO2 for temperature. In the absence of
temperature correction, DO and pCO2 look like inverses. When one goes
up, the other goes down. But much of that pattern is driven by
temperature, so it becomes much less striking following temperature
correction. pCO<sub>2</sub> does climb for most of the year, however,
the DO and PCO2 correlation really only breaks down in the early winter
months.

# Correlation Plots

I find it is often very interesting to plot out the relationships
between two related time series. This effectively allows you to look at
the relationships between variables in a phase space, rather than in the
time domain. I find this often helps clarify interdependencies among
variables (although it can tell you nothing about causality, and can be
misleading, because time series observations are generally highly
autocorrelated).

## PH by Temperature

### Recolored with a Sequential Palette

``` r
plt <- ggplot(all_data, aes(temp, ph)) + 
  geom_point(aes(color = factor(mm)), alpha = 0.4, size = 0.5) +
  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +

  xlab(expression(Temperature~'('*degree*C*')')) +
  ylab('pH') +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 3))) +

  theme_cbep()
plt
```

    ## Warning: Removed 11921 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/ph_by_temp_gradient-1.png)<!-- -->

``` r
ggsave('phbytemp.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 11921 rows containing missing values (geom_point).

``` r
ggsave('phbytemp.pdf', device = cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 11921 rows containing missing values (geom_point).

## PCO2 by Temperature

``` r
plt <- ggplot(all_data, aes(temp, co2_corr)) +
  geom_point(aes(color = factor(mm)), alpha = 0.4, size=0.5) +
  
  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +

  xlab(expression(Temperature~'('*degree*C*')')) +
  ylab(expression(paste("pCO"[2*(cor)],'(', mu,"Atm)"))) +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 3))) +

  theme_cbep()
  
plt
```

    ## Warning: Removed 6157 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/pco2_by_temp-1.png)<!-- -->

``` r
ggsave('pco2bytemp.png', type = 'cairo', width = 7, height = 5)
```

    ## Warning: Removed 6157 rows containing missing values (geom_point).

``` r
ggsave('pco2bytemp.pdf', device = cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 6157 rows containing missing values (geom_point).

## pCO<sub>2</sub> by Oxygen

``` r
plt <- ggplot(all_data, aes(do, co2_corr)) +
  geom_point(aes(color = factor(mm)), alpha = 0.4, size = 0.5) +
  #geom_smooth(method = 'gam', formula = y~s(x, k=5)) +
 
  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +

  xlab(expression('Dissolved Oxygen ('~mu~'Mol kg'^{-1}~')')) +
  ylab(expression(paste("pCO"[2*(cor)],'(', mu,"Atm)"))) +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 3))) +
 
  theme_cbep()
plt
```

    ## Warning: Removed 9040 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/pco2_by_o2-1.png)<!-- -->

``` r
#ggsave('pco2bydo.png', type = 'cairo', width = 7, height = 5)
#ggsave('pco2bydo.pdf', device = cairo_pdf, width = 7, height = 5)
```

## pH by pCO<sub>2</sub>

``` r
plt <- ggplot(all_data, aes(co2_corr, ph)) +
  geom_point(aes(color = factor(mm)), alpha = 0.4, size=0.5) +
  #geom_smooth(method = 'gam', formula = y~s(x, k=5)) +
  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +
  
  ylab('pH') +
  xlab(expression(paste("pCO"[2*(cor)],'(', mu,"Atm)"))) +
  #ylim(c(7.0, 8.5)) +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 3))) +
  
  theme_cbep()
plt
```

    ## Warning: Removed 16817 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/pco2_by_ph-1.png)<!-- -->

``` r
#ggsave('pco2byph.png', type = 'cairo', width = 7, height = 5)
#ggsave('pco2byph.pdf', device = cairo_pdf, width = 7, height = 5)
```

Note that the lowest pCO2 and the highest pH are cold weather phenomena.
Once the weather warms up, pCO<sub>2</sub> gets much more variable, and
pH tends to be lower.

## Omega by pCO<sub>2</sub>

``` r
plt <- ggplot(all_data, aes(co2_corr, omega_a)) + 
  geom_point(aes(color = factor(mm)), alpha = 0.4, size = 0.5) +
  #geom_smooth(method = 'gam', formula = y~s(x, k=5)) +
  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +

  ylab(expression(Omega[a])) +
  xlab(expression(paste("pCO"[2*(cor)],'(', mu,"Atm)"))) +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 3))) +

  theme_cbep()
plt
```

    ## Warning: Removed 17575 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
#ggsave('omeganypco2.png', type = 'cairo', width = 7, height = 5)
#ggsave('omegabypco2.pdf', device = cairo_pdf, width = 7, height = 5)
```

Omega is remarkably independent of pCO<sub>2</sub>, with only weak
seasonality. Some of that may reflect limited seasonal data.

## Alkalinity vs. Salinity

``` r
plt <- ggplot(all_data, aes(sal, TA_calc)) +
  geom_point(aes(color = factor(mm)), alpha = 0.4, size = 0.5)  +
  xlab('Salinity (PSU)') +
  ylab(expression('Total Alkalinity ('~mu~'Mol' *.* 'kg'^{-1}~')')) +

  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 4))) +
  scale_x_continuous(limits = c(22,32)) +

  scale_color_manual(values = getPalette(12), name = 'Month',
                     labels=month.abb) +

  theme_cbep()
plt
```

    ## Warning: Removed 17577 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
#ggsave('alkbysalinity.png', type = 'cairo', width = 7, height = 5)
#ggsave('alkbysalinity.pdf', device = cairo_pdf, width = 7, height = 5)
```

## Alkalinity by Salinity, seasonal

``` r
tmp <- all_data %>%
  mutate(season = cut(mm, 4, c('Winter', 'Spring', 'Summer','Fall')))
```

``` r
plt <- ggplot(tmp, aes(sal, TA_calc)) +
  geom_point(aes(color = season), alpha = 0.4 , size = .5)  +
  xlab('Salinity (PSU)') +
  ylab(expression('Total Alkalinity ('~mu~'Mol kg'^{-1}~')')) +
  scale_color_discrete(name = 'Season') +
  guides(colour = guide_legend(override.aes = list(alpha = 1, size = 4))) +
  scale_x_continuous(limits = c(22,32)) +
  #theme_minimal() +
  
  theme_cbep()
plt
```

    ## Warning: Removed 17577 rows containing missing values (geom_point).

![](Principal_Graphics_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
#ggsave('alkbysalinity_seasons.png', type = 'cairo', width = 7, height = 5)
#ggsave('alkbysalinity_Seasons.pdf', device = cairo_pdf, width = 7, height = 5)
```