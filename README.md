Lab 05 - Data Wrangling
================

# Learning goals

- Use the `merge()` function to join two datasets.
- Deal with missings and impute data.
- Identify relevant observations using `quantile()`.
- Practice your GitHub skills.

# Lab description

For this lab we will be dealing with the meteorological dataset `met`.
In this case, we will use `data.table` to answer some questions
regarding the `met` dataset, while at the same time practice your
Git+GitHub skills for this project.

This markdown document should be rendered using `github_document`
document.

# Part 1: Setup a Git project and the GitHub repository

1.  Go to wherever you are planning to store the data on your computer,
    and create a folder for this project

2.  In that folder, save [this
    template](https://github.com/JSC370/JSC370-2024/blob/main/labs/lab05/lab05-wrangling-gam.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository of the same
    name that your local folder has, e.g., “JSC370-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

Most of the steps can be done using command line:

``` sh
# Step 1
cd ~/Documents
mkdir JSC370-labs
cd JSC370-labs

# Step 2
wget https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd
mv lab05-wrangling-gam.Rmd README.Rmd
# if wget is not available,
curl https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd --output README.Rmd

# Step 3
# Happens on github

# Step 4
git init
git add README.Rmd
git commit -m "First commit"

# Step 5
git remote add origin git@github.com:[username]/JSC370-labs
git push -u origin master
```

You can also complete the steps in R (replace with your paths/username
when needed)

``` r
# Step 1
setwd("~/Documents")
dir.create("JSC370-labs")
setwd("JSC370-labs")

# Step 2
download.file(
  "https://raw.githubusercontent.com/JSC370/JSC370-2024/main/labs/lab05/lab05-wrangling-gam.Rmd",
  destfile = "README.Rmd"
  )

# Step 3: Happens on Github

# Step 4
system("git init && git add README.Rmd")
system('git commit -m "First commit"')

# Step 5
system("git remote add origin git@github.com:[username]/JSC370-labs")
system("git push -u origin master")
```

Once you are done setting up the project, you can now start working with
the MET data.

## Setup in R

1.  Load the `data.table` (and the `dtplyr` and `dplyr` packages),
    `mgcv`, `ggplot2`, `leaflet`, `kableExtra`.

``` r
library(data.table)
library(dtplyr)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(mgcv)
```

    ## Loading required package: nlme

    ## 
    ## Attaching package: 'nlme'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

    ## This is mgcv 1.9-0. For overview type 'help("mgcv-package")'.

``` r
library(ggplot2)
library(leaflet)
library(kableExtra)
```

    ## 
    ## Attaching package: 'kableExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     group_rows

``` r
fn <- "https://raw.githubusercontent.com/JSC370/JSC370-2024/main/data/met_all_2023.gz"
if (!file.exists("met_all_2023.gz"))
  download.file(fn, destfile = "met_all_2023.gz")
met <- data.table::fread("met_all_2023.gz")
```

2.  Load the met data from
    <https://github.com/JSC370/JSC370-2024/main/data/met_all_2023.gz> or
    (Use
    <https://raw.githubusercontent.com/JSC370/JSC370-2024/main/data/met_all_2023.gz>
    to download programmatically), and also the station data. For the
    latter, you can use the code we used during lecture to pre-process
    the stations data:

``` r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE, LAT, LON)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]

# Read in the met data and fix lat, lon, temp
met$lat <- met$lat/1000
met$lon <- met$lon/1000
met$wind.sp <- met$wind.sp/10
met$temp <- met$temp/10
met$dew.point <- met$dew.point/10
met$atm.press <- met$atm.press/10
met$rh <- 100*((112-0.1*met$temp+met$dew.point)/(112+0.9*met$temp))^8
met_cleaned <- met[temp >= 0 & temp <= 50]
```

3.  Merge the data as we did during the lecture. Use the `merge()` code
    and you can also try the tidy way with `left_join()`

``` r
met_dt <- merge(
  # Data
  x     = met_cleaned,      
  y     = stations, 
  # List of variables to match
  by.x  = "USAFID",
  by.y  = "USAF", 
  all.x = TRUE,      
  all.y = FALSE
)
nrow(met_dt)
```

    ## [1] 2499849

## Question 1: Identifying Representative Stations

Across all weather stations, which stations have the median values of
temperature, wind speed, and atmospheric pressure? Using the
`quantile()` function, identify these three stations. Do they coincide?

``` r
# Compute the median values of temperature, wind speed, and atmospheric pressure
# globally
national_medians <- met_dt[,
                         list(
                           temp_50=quantile(temp, prob=.5, na.rm=TRUE),
                           wind.sp_50=quantile(wind.sp, prob=.5, na.rm=TRUE),
                           atm.press_50=quantile(atm.press, prob=.5, na.rm=TRUE)
                         )]
national_medians
```

    ##    temp_50 wind.sp_50 atm.press_50
    ## 1:    21.7        3.1       1011.7

Next identify the stations have these median values.

``` r
# Compute the median values of temperature, wind speed, and atmospheric pressure
# for each station
station_median <- met_dt[,
                         list(
                           temp_50=quantile(temp, prob=.5, na.rm=TRUE),
                           wind.sp_50=quantile(wind.sp, prob=.5, na.rm=TRUE),
                           atm.press_50=quantile(atm.press, prob=.5, na.rm=TRUE)
                         ),
                         by=.(USAFID, STATE)]

# Find which stations have the median values of temperature
station_median[, temp_dist:=abs(temp_50 - national_medians$temp_50)]
median_temp_station <- station_median[temp_dist==0]
median_temp_station[, list(USAFID, temp_50)]
```

    ##     USAFID temp_50
    ##  1: 720263    21.7
    ##  2: 720312    21.7
    ##  3: 720327    21.7
    ##  4: 722076    21.7
    ##  5: 722180    21.7
    ##  6: 722196    21.7
    ##  7: 722197    21.7
    ##  8: 723075    21.7
    ##  9: 723086    21.7
    ## 10: 723110    21.7
    ## 11: 723119    21.7
    ## 12: 723190    21.7
    ## 13: 723194    21.7
    ## 14: 723200    21.7
    ## 15: 723658    21.7
    ## 16: 723895    21.7
    ## 17: 724010    21.7
    ## 18: 724345    21.7
    ## 19: 724356    21.7
    ## 20: 724365    21.7
    ## 21: 724373    21.7
    ## 22: 724380    21.7
    ## 23: 724397    21.7
    ## 24: 724454    21.7
    ## 25: 724517    21.7
    ## 26: 724585    21.7
    ## 27: 724815    21.7
    ## 28: 724838    21.7
    ## 29: 725116    21.7
    ## 30: 725317    21.7
    ## 31: 725326    21.7
    ## 32: 725340    21.7
    ## 33: 725450    21.7
    ## 34: 725472    21.7
    ## 35: 725473    21.7
    ## 36: 725480    21.7
    ## 37: 725499    21.7
    ## 38: 725513    21.7
    ## 39: 725720    21.7
    ## 40: 726515    21.7
    ## 41: 726525    21.7
    ## 42: 726546    21.7
    ## 43: 726556    21.7
    ## 44: 726560    21.7
    ## 45: 727570    21.7
    ## 46: 727845    21.7
    ## 47: 727900    21.7
    ## 48: 745046    21.7
    ## 49: 746410    21.7
    ## 50: 747808    21.7
    ##     USAFID temp_50

``` r
# Find which stations have the median values of wind speed
station_median[, wind.sp_dist:=abs(wind.sp_50 - national_medians$wind.sp_50)]
median_wind.sp_station <- station_median[wind.sp_dist==0]
median_wind.sp_station[, list(USAFID, wind.sp_50)]
```

    ##      USAFID wind.sp_50
    ##   1: 720110        3.1
    ##   2: 720113        3.1
    ##   3: 720258        3.1
    ##   4: 720261        3.1
    ##   5: 720266        3.1
    ##  ---                  
    ## 576: 747760        3.1
    ## 577: 747804        3.1
    ## 578: 747809        3.1
    ## 579: 747900        3.1
    ## 580: 747918        3.1

``` r
# Find which stations have the median values of atmospheric pressure
station_median[, atm.press_dist:=abs(atm.press_50 - national_medians$atm.press_50)]
median_atm.press_station <- station_median[atm.press_dist==0]
median_atm.press_station[, list(USAFID, atm.press_50)]
```

    ##     USAFID atm.press_50
    ##  1: 720394       1011.7
    ##  2: 722085       1011.7
    ##  3: 722348       1011.7
    ##  4: 723020       1011.7
    ##  5: 723119       1011.7
    ##  6: 723124       1011.7
    ##  7: 723270       1011.7
    ##  8: 723658       1011.7
    ##  9: 724010       1011.7
    ## 10: 724035       1011.7
    ## 11: 724100       1011.7
    ## 12: 724235       1011.7
    ## 13: 724280       1011.7
    ## 14: 724336       1011.7
    ## 15: 724926       1011.7
    ## 16: 725126       1011.7
    ## 17: 725266       1011.7
    ## 18: 725510       1011.7
    ## 19: 725570       1011.7
    ## 20: 725620       1011.7
    ## 21: 725845       1011.7
    ## 22: 726690       1011.7
    ## 23: 726810       1011.7
    ##     USAFID atm.press_50

``` r
# Find the coincide
coincide <- station_median[temp_dist == 0 & wind.sp_dist == 0 & atm.press_dist == 0]
coincide
```

    ##    USAFID STATE temp_50 wind.sp_50 atm.press_50 temp_dist wind.sp_dist
    ## 1: 723119    SC    21.7        3.1       1011.7         0            0
    ##    atm.press_dist
    ## 1:              0

**Answer:** The stations are listed above. They coincide in 1 station -
USAFID 723119.

Knit the document, commit your changes, and save it on GitHub. Don’t
forget to add `README.md` to the tree, the first time you render it.

## Question 2: Identifying Representative Stations per State

Now let’s find the weather stations by state with closest temperature
and wind speed based on the euclidean distance from these medians.

``` r
# Compute the median values of temperature, wind speed, and atmospheric pressure
# for each state
state_medians <- met_dt[,
                         list(
                           state_temp_50=quantile(temp, prob=.5, na.rm=TRUE),
                           state_wind.sp_50=quantile(wind.sp, prob=.5, na.rm=TRUE)
                         ),
                         by=.(STATE)]

# Compute the euclidean distance from each station median to the median of the state
station_euc_state <- left_join(station_median, state_medians, by = "STATE")
station_euc_state <- station_euc_state[, 
                                 state_euc_dist := 
                                   sqrt((temp_50 - state_temp_50)**2 + 
                                          (wind.sp_50 - state_wind.sp_50)**2)]

# Find the stations with minimum euclidean distance in each state
station_euc_state <- station_euc_state[,
                                      .SD[state_euc_dist == min(state_euc_dist)],
                                      by=STATE]
station_euc_state[, list(USAFID, STATE, temp_50, wind.sp_50, state_temp_50, 
                           state_wind.sp_50, state_euc_dist)]
```

    ##     USAFID STATE temp_50 wind.sp_50 state_temp_50 state_wind.sp_50
    ##  1: 722950    CA    16.7       3.60          17.1              3.6
    ##  2: 722448    TX    27.8       3.60          27.8              3.6
    ##  3: 722523    TX    27.8       3.60          27.8              3.6
    ##  4: 722533    TX    27.8       3.60          27.8              3.6
    ##  5: 720602    SC    23.0       3.10          23.0              3.1
    ##  6: 720611    SC    23.0       3.10          23.0              3.1
    ##  7: 720613    SC    23.0       3.10          23.0              3.1
    ##  8: 720633    SC    23.0       3.10          23.0              3.1
    ##  9: 747918    SC    23.0       3.10          23.0              3.1
    ## 10: 744666    IL    21.9       3.10          21.8              3.1
    ## 11: 723300    MO    23.9       3.10          23.8              3.1
    ## 12: 724450    MO    23.9       3.10          23.8              3.1
    ## 13: 722188    AR    23.9       2.60          24.0              2.6
    ## 14: 723405    AR    24.1       2.60          24.0              2.6
    ## 15: 743312    AR    23.9       2.60          24.0              2.6
    ## 16: 725976    OR    14.4       3.10          15.0              3.1
    ## 17: 726830    OR    15.6       3.10          15.0              3.1
    ## 18: 727924    WA    15.0       3.10          15.0              3.1
    ## 19: 720257    GA    23.0       2.60          23.0              2.6
    ## 20: 720962    GA    23.0       2.60          23.0              2.6
    ## 21: 722255    GA    23.0       2.60          23.0              2.6
    ## 22: 747805    GA    23.0       2.60          23.0              2.6
    ## 23: 720283    MN    21.0       3.10          21.0              3.1
    ## 24: 722006    MN    21.0       3.10          21.0              3.1
    ## 25: 722032    MN    21.0       3.10          21.0              3.1
    ## 26: 726561    MN    21.0       3.10          21.0              3.1
    ## 27: 726577    MN    21.0       3.10          21.0              3.1
    ## 28: 726596    MN    21.0       3.10          21.0              3.1
    ## 29: 720265    AL    23.1       2.60          23.3              2.6
    ## 30: 744660    IN    20.0       3.60          20.0              3.6
    ## 31: 720282    NC    22.0       2.60          21.8              2.6
    ## 32: 722131    NC    22.0       2.60          21.8              2.6
    ## 33: 722148    NC    21.6       2.60          21.8              2.6
    ## 34: 723067    NC    22.0       2.60          21.8              2.6
    ## 35: 720309    IA    22.0       3.10          22.0              3.1
    ## 36: 720412    IA    22.0       3.10          22.0              3.1
    ## 37: 722097    IA    22.0       3.10          22.0              3.1
    ## 38: 725453    IA    22.0       3.10          22.0              3.1
    ## 39: 725454    IA    22.0       3.10          22.0              3.1
    ## 40: 725464    IA    22.0       3.10          22.0              3.1
    ## 41: 725466    IA    22.0       3.10          22.0              3.1
    ## 42: 725469    IA    22.0       3.10          22.0              3.1
    ## 43: 725479    IA    22.0       3.10          22.0              3.1
    ## 44: 725487    IA    22.0       3.10          22.0              3.1
    ## 45: 725493    IA    22.0       3.10          22.0              3.1
    ## 46: 720324    PA    19.0       2.85          18.9              3.1
    ## 47: 725525    NE    21.1       3.60          21.5              3.6
    ## 48: 725624    NE    21.1       3.60          21.5              3.6
    ## 49: 725867    ID    16.1       3.10          16.1              3.1
    ## 50: 726415    WI    19.0       3.10          19.0              3.1
    ## 51: 726457    WI    19.0       3.10          19.0              3.1
    ## 52: 726509    WI    19.0       3.10          19.0              3.1
    ## 53: 724177    WV    18.3       3.10          18.3              2.6
    ## 54: 722728    AZ    25.6       3.10          25.6              3.6
    ## 55: 722164    OK    24.7       3.10          24.5              3.1
    ## 56: 720587    LA    27.6       2.60          27.7              2.6
    ## 57: 722334    LA    27.6       2.60          27.7              2.6
    ## 58: 724235    KY    21.1       3.10          21.0              3.1
    ## 59: 722024    FL    26.7       3.60          26.7              3.6
    ## 60: 722034    FL    26.7       3.60          26.7              3.6
    ## 61: 722037    FL    26.7       3.60          26.7              3.6
    ## 62: 747830    FL    26.7       3.60          26.7              3.6
    ## 63: 720414    OH    19.0       3.10          19.1              3.1
    ## 64: 720651    OH    19.0       3.10          19.1              3.1
    ## 65: 720713    OH    19.0       3.10          19.1              3.1
    ## 66: 720581    NJ    20.0       3.10          20.0              3.1
    ## 67: 724075    NJ    20.0       3.10          20.0              3.1
    ## 68: 725025    NJ    20.0       3.10          20.0              3.1
    ## 69: 722683    NM    22.0       4.60          22.4              4.1
    ## 70: 724655    KS    22.2       3.60          22.2              3.6
    ## 71: 720853    ND    20.0       3.60          20.0              3.6
    ## 72: 720861    ND    20.0       3.60          20.0              3.6
    ## 73: 720866    ND    20.0       3.60          20.0              3.6
    ## 74: 720867    ND    20.0       3.60          20.0              3.6
    ## 75: 720868    ND    20.0       3.60          20.0              3.6
    ## 76: 720871    ND    20.0       3.60          20.0              3.6
    ## 77: 727640    ND    20.0       3.60          20.0              3.6
    ## 78: 727677    ND    20.0       3.60          20.0              3.6
    ## 79: 722354    MS    25.6       3.10          25.6              3.1
    ## 80: 725040    CT    19.4       3.60          19.4              3.1
    ## 81: 724885    NV    19.4       3.10          21.0              3.6
    ## 82: 725724    UT    19.4       3.10          20.0              3.6
    ## 83: 725750    UT    20.6       3.10          20.0              3.6
    ## 84: 726518    SD    21.1       3.60          20.6              3.6
    ## 85: 720974    TN    22.0       2.60          22.0              2.6
    ## 86: 723249    TN    22.0       2.60          22.0              2.6
    ## 87: 725190    NY    18.3       3.10          18.3              3.1
    ## 88: 725054    RI    18.0       2.60          18.3              3.1
    ## 89: 725079    RI    18.0       3.60          18.3              3.1
    ## 90: 725059    MA    17.8       3.10          17.8              3.1
    ## 91: 724093    DE    21.1       3.60          21.3              3.6
    ## 92: 726165    NH    16.1       2.10          16.1              2.6
    ## 93: 726060    ME    15.0       3.10          15.0              3.1
    ## 94: 726190    ME    15.0       3.10          15.0              3.1
    ## 95: 727120    ME    15.0       3.10          15.0              3.1
    ## 96: 726770    MT    16.1       3.10          16.1              3.1
    ##     USAFID STATE temp_50 wind.sp_50 state_temp_50 state_wind.sp_50
    ##     state_euc_dist
    ##  1:      0.4000000
    ##  2:      0.0000000
    ##  3:      0.0000000
    ##  4:      0.0000000
    ##  5:      0.0000000
    ##  6:      0.0000000
    ##  7:      0.0000000
    ##  8:      0.0000000
    ##  9:      0.0000000
    ## 10:      0.1000000
    ## 11:      0.1000000
    ## 12:      0.1000000
    ## 13:      0.1000000
    ## 14:      0.1000000
    ## 15:      0.1000000
    ## 16:      0.6000000
    ## 17:      0.6000000
    ## 18:      0.0000000
    ## 19:      0.0000000
    ## 20:      0.0000000
    ## 21:      0.0000000
    ## 22:      0.0000000
    ## 23:      0.0000000
    ## 24:      0.0000000
    ## 25:      0.0000000
    ## 26:      0.0000000
    ## 27:      0.0000000
    ## 28:      0.0000000
    ## 29:      0.2000000
    ## 30:      0.0000000
    ## 31:      0.2000000
    ## 32:      0.2000000
    ## 33:      0.2000000
    ## 34:      0.2000000
    ## 35:      0.0000000
    ## 36:      0.0000000
    ## 37:      0.0000000
    ## 38:      0.0000000
    ## 39:      0.0000000
    ## 40:      0.0000000
    ## 41:      0.0000000
    ## 42:      0.0000000
    ## 43:      0.0000000
    ## 44:      0.0000000
    ## 45:      0.0000000
    ## 46:      0.2692582
    ## 47:      0.4000000
    ## 48:      0.4000000
    ## 49:      0.0000000
    ## 50:      0.0000000
    ## 51:      0.0000000
    ## 52:      0.0000000
    ## 53:      0.5000000
    ## 54:      0.5000000
    ## 55:      0.2000000
    ## 56:      0.1000000
    ## 57:      0.1000000
    ## 58:      0.1000000
    ## 59:      0.0000000
    ## 60:      0.0000000
    ## 61:      0.0000000
    ## 62:      0.0000000
    ## 63:      0.1000000
    ## 64:      0.1000000
    ## 65:      0.1000000
    ## 66:      0.0000000
    ## 67:      0.0000000
    ## 68:      0.0000000
    ## 69:      0.6403124
    ## 70:      0.0000000
    ## 71:      0.0000000
    ## 72:      0.0000000
    ## 73:      0.0000000
    ## 74:      0.0000000
    ## 75:      0.0000000
    ## 76:      0.0000000
    ## 77:      0.0000000
    ## 78:      0.0000000
    ## 79:      0.0000000
    ## 80:      0.5000000
    ## 81:      1.6763055
    ## 82:      0.7810250
    ## 83:      0.7810250
    ## 84:      0.5000000
    ## 85:      0.0000000
    ## 86:      0.0000000
    ## 87:      0.0000000
    ## 88:      0.5830952
    ## 89:      0.5830952
    ## 90:      0.0000000
    ## 91:      0.2000000
    ## 92:      0.5000000
    ## 93:      0.0000000
    ## 94:      0.0000000
    ## 95:      0.0000000
    ## 96:      0.0000000
    ##     state_euc_dist

Knit the doc and save it on GitHub.

## Question 3: In the Geographic Center?

For each state, identify which station is closest to the geographic
mid-point (median) of the state. Combining these with the stations you
identified in the previous question, use `leaflet()` to visualize all
~100 points in the same figure, applying different colors for the
geographic median and the temperature and wind speed median.

Knit the doc and save it on GitHub.

## Question 4: Summary Table with `kableExtra`

Generate a summary table using `kable` where the rows are each state and
the columns represent average temperature broken down by low, median,
and high elevation stations.

Use the following breakdown for elevation:

- Low: elev \< 93
- Mid: elev \>= 93 and elev \< 401
- High: elev \>= 401

Knit the document, commit your changes, and push them to GitHub.

## Question 5: Advanced Regression

Let’s practice running regression models with smooth functions on X. We
need the `mgcv` package and `gam()` function to do this.

- using your data with the median values per station, first create a
  lazy table. Filter out values of atmospheric pressure outside of the
  range 1000 to 1020. Examine the association between temperature (y)
  and atmospheric pressure (x). Create a scatterplot of the two
  variables using ggplot2. Add both a linear regression line and a
  smooth line.

- fit both a linear model and a spline model (use `gam()` with a cubic
  regression spline on wind speed). Summarize and plot the results from
  the models and interpret which model is the best fit and why.

## Deliverables

- .Rmd file (this file)

- link to the .md file (with all outputs) in your GitHub repository
