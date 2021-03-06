README
================

``` r
library(readr)
library(ggplot2)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(blscrapeR)
```

1.  Modify code to do 10+ states and as many years as
possible

<!-- end list -->

``` r
for (state in c("AZ", "CA",  "CO", "DE", "MD", "FL", "DC", "UT", "NC", "WI")) {
   for (year in c(12:18)){
      bridges_URL <- paste("https://www.fhwa.dot.gov/bridge/nbi/20",
      year,
      "/delimited/",
      state,
      year,
      ".txt", sep = "")
   data <- read_delim(bridges_URL, ",", col_types = cols_only(
                                                      'STATE_CODE_001'= col_integer(),
                                                      'COUNTY_CODE_003' = col_integer(),
                                                      'STRUCTURE_NUMBER_008' = 'c',
                                                      'SERVICE_LEVEL_005C' = 'c',
                                                      'HIGHWAY_DISTRICT_002' = 'c', 
                                                      'COUNTY_CODE_003' = col_integer(), 
                                                      'MAINTENANCE_021' = col_integer(),
                                                      'YEAR_BUILT_027' = col_integer(),
                                                      'TRAFFIC_LANES_ON_028A' = col_integer(),
                                                      'DECK_COND_058' = 'c',
                                                      'SUPERSTRUCTURE_COND_059' = 'c',
                                                      'ADT_029' = col_integer()))
   data$DATA_YEAR <- paste("20", year, sep = "")
   nam <- paste("Bridges", state, year, sep = "_")
   assign(nam, data)
   }
}
```

``` r
Bridges_AZ <- rbind.data.frame(Bridges_AZ_12, Bridges_AZ_13, Bridges_AZ_14, Bridges_AZ_15, Bridges_AZ_16, Bridges_AZ_17, Bridges_AZ_18)
Bridges_CA <- rbind.data.frame(Bridges_CA_12, Bridges_CA_13, Bridges_CA_14, Bridges_CA_15, Bridges_CA_16, Bridges_CA_17, Bridges_CA_18)
Bridges_CO <- rbind.data.frame(Bridges_CO_12, Bridges_CO_13, Bridges_CO_14, Bridges_CO_15, Bridges_CO_16, Bridges_CO_17, Bridges_CO_18)
Bridges_DE <- rbind.data.frame(Bridges_DE_12, Bridges_DE_13, Bridges_DE_14, Bridges_DE_15, Bridges_DE_16, Bridges_DE_17, Bridges_DE_18)
Bridges_MD <- rbind.data.frame(Bridges_MD_12, Bridges_MD_13, Bridges_MD_14, Bridges_MD_15, Bridges_MD_16, Bridges_MD_17, Bridges_MD_18)
Bridges_FL <- rbind.data.frame(Bridges_FL_12, Bridges_FL_13, Bridges_FL_14, Bridges_FL_15, Bridges_FL_16, Bridges_FL_17, Bridges_FL_18)
Bridges_DC <- rbind.data.frame(Bridges_DC_12, Bridges_DC_13, Bridges_DC_14, Bridges_DC_15, Bridges_DC_16, Bridges_DC_17, Bridges_DC_18)
Bridges_UT <- rbind.data.frame(Bridges_UT_12, Bridges_UT_13, Bridges_UT_14, Bridges_UT_15, Bridges_UT_16, Bridges_UT_17, Bridges_UT_18)
Bridges_NC <- rbind.data.frame(Bridges_NC_12, Bridges_NC_13, Bridges_NC_14, Bridges_NC_15, Bridges_NC_16, Bridges_NC_17, Bridges_NC_18)
Bridges_WI <- rbind.data.frame(Bridges_WI_12, Bridges_WI_13, Bridges_WI_14, Bridges_WI_15, Bridges_WI_16, Bridges_WI_17, Bridges_WI_18)

Bridges_Full <- rbind.data.frame(Bridges_AZ, Bridges_CA, Bridges_CO, Bridges_DE, Bridges_MD, Bridges_FL,
                                 Bridges_DC, Bridges_UT, Bridges_NC, Bridges_WI)
Bridges_Full$SUPERSTRUCTURE_COND_059 <- ifelse(Bridges_Full$SUPERSTRUCTURE_COND_059 == "N", NA, Bridges_Full$SUPERSTRUCTURE_COND_059)
Bridges_Full$STATE_CODE_001 <- as.factor(as.character(Bridges_Full$STATE_CODE_001))
Bridges_Full$SUPERSTRUCTURE_COND_059 <- as.numeric(as.character(Bridges_Full$SUPERSTRUCTURE_COND_059))
```

2.  Explore more with ggplot

<!-- end list -->

``` r
#Make facets over states.
ggplot(data = subset(Bridges_Full, !is.na(SUPERSTRUCTURE_COND_059)), aes(x=YEAR_BUILT_027, y=SUPERSTRUCTURE_COND_059)) + geom_point() + geom_jitter() + facet_wrap(~STATE_CODE_001) + ggtitle("Superstructure Condition vs. Year Built")
```

    ## Warning: Removed 32 rows containing missing values (geom_point).
    
    ## Warning: Removed 32 rows containing missing values (geom_point).

![](README_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
#Fit linear model with bridges and unemployment data
#Need to import November/December data to predict future periods
```

``` r
#Make one plot with x-axis as time and y-axis as something about bridges. make one line for each state.
#Going to make summary statistics to track change over time for the data
Bridges_Summarized <- Bridges_Full %>% 
   group_by(STATE_CODE_001, DATA_YEAR) %>% 
   summarize(Mean_Superstructure_Cond = mean(SUPERSTRUCTURE_COND_059, na.rm = T))
   
ggplot(Bridges_Summarized, aes(x=DATA_YEAR, y=Mean_Superstructure_Cond, group = STATE_CODE_001, color = STATE_CODE_001)) + geom_line() + ggtitle("Mean Superstructure Condition Per State")
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
#This graph shows Florida consistently has the highest mean superstructure condition, while Maryland has one of the lowest
```

Use the bridges data to find features that you think would be predictive
of unemployment. Fit a linear model to predict the number of unemployed.
Fit another model to predict the unemployment rate. Then, use the
unemployed number and rate from the previous month as additional
predictors. How do things change? Make a nice github repo with your
work\!

``` r
# Grab BLS Unemployment Data To Start
BLS_data <- get_bls_county(c("September 2019", "December 2019"),
                           stateName = c("Arizona", "California", "Colorado", "Delaware", "Maryland", 
                                         "Florida", "District of Columbia", "Utah", "North Carolina",  "Wisconsin"))
```

``` r
#Prepare Bridges data to be merged
Bridges_Full$STATE_CODE_001 <- as.numeric(as.character(Bridges_Full$STATE_CODE_001))
Bridges_Full$fips <- paste(Bridges_Full$STATE_CODE_001, Bridges_Full$COUNTY_CODE_003, sep ="")
Bridges_Full$fips <- as.numeric(as.character(Bridges_Full$fips))
```

    ## Warning: NAs introduced by coercion

``` r
BLS_data$fips <- as.numeric(as.character(BLS_data$fips))
Bridges_Full$DECK_COND_058 <- as.numeric(as.character(Bridges_Full$DECK_COND_058))
```

    ## Warning: NAs introduced by coercion

``` r
Bridges_of_interest <- Bridges_Full %>% 
   filter(DATA_YEAR == "2018") %>% 
   group_by(fips) %>% 
   summarize(mean_traffic_lanes = mean(TRAFFIC_LANES_ON_028A),
             mean_traffic = mean(ADT_029), 
             mean_deck_cond = mean(DECK_COND_058, na.rm = T),
             mean_superstructure = mean(SUPERSTRUCTURE_COND_059, na.rm = T))
```

``` r
#Join Data
Bridges_and_BLS <- BLS_data %>% 
   inner_join(Bridges_of_interest, by = "fips")
```

``` r
lm_unemployed <- lm(unemployed ~ mean_traffic + mean_deck_cond + mean_superstructure, data = Bridges_and_BLS)

summary(lm_unemployed)
```

    ## 
    ## Call:
    ## lm(formula = unemployed ~ mean_traffic + mean_deck_cond + mean_superstructure, 
    ##     data = Bridges_and_BLS)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -6020.9  -734.3   -63.9   321.2 16869.3 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)          8.667e+03  4.199e+03   2.064   0.0402 *  
    ## mean_traffic         5.315e-01  2.808e-02  18.929   <2e-16 ***
    ## mean_deck_cond      -1.618e+03  8.324e+02  -1.944   0.0532 .  
    ## mean_superstructure  2.128e+02  6.395e+02   0.333   0.7396    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 2266 on 214 degrees of freedom
    ##   (2 observations deleted due to missingness)
    ## Multiple R-squared:  0.6478, Adjusted R-squared:  0.6429 
    ## F-statistic: 131.2 on 3 and 214 DF,  p-value: < 2.2e-16

``` r
#We see superstructure condition is not statistically significant
#Adj R-Sq of 0.6429
```

``` r
lm_unemployment_rate <- lm(unemployed_rate ~ mean_traffic + mean_deck_cond + mean_superstructure, data = Bridges_and_BLS)

summary(lm_unemployment_rate)
```

    ## 
    ## Call:
    ## lm(formula = unemployed_rate ~ mean_traffic + mean_deck_cond + 
    ##     mean_superstructure, data = Bridges_and_BLS)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.4675 -0.5512 -0.1625  0.4550  5.7917 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error t value Pr(>|t|)   
    ## (Intercept)          6.220e+00  2.010e+00   3.095  0.00223 **
    ## mean_traffic        -1.174e-05  1.344e-05  -0.874  0.38323   
    ## mean_deck_cond      -4.717e-01  3.984e-01  -1.184  0.23770   
    ## mean_superstructure  6.662e-02  3.060e-01   0.218  0.82786   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.084 on 214 degrees of freedom
    ##   (2 observations deleted due to missingness)
    ## Multiple R-squared:  0.01721,    Adjusted R-squared:  0.003431 
    ## F-statistic: 1.249 on 3 and 214 DF,  p-value: 0.2929

``` r
#none of the predictors are statistically significant
#Adj R-Sq of 0.003431
```

``` r
#Create lagged variable column to capture previous months BLS data
BLS_data_September2019 <- get_bls_county("September 2019",
                           stateName = c("Arizona", "California", "Colorado", "Delaware", "Maryland", 
                                         "Florida", "District of Columbia", "Utah", "North Carolina",  "Wisconsin"))
                           
BLS_data_December2019 <- get_bls_county("December 2019",
                           stateName = c("Arizona", "California", "Colorado", "Delaware", "Maryland", 
                                         "Florida", "District of Columbia", "Utah", "North Carolina",  "Wisconsin"))
BLS_data_September2019 <- BLS_data_September2019[, c(7,8), ]
colnames(BLS_data_September2019)[1] <- "unemployed_lag"
colnames(BLS_data_September2019)[2] <- "unemployment_rate_lag"

BLS_data <- cbind.data.frame(BLS_data_December2019, BLS_data_September2019)

BLS_data$fips <- as.numeric(as.character(BLS_data$fips))
#Join Data
Bridges_and_BLS <- BLS_data %>% 
   inner_join(Bridges_of_interest, by = "fips")
```

``` r
#lagged regressor model
lm_unemployed <- lm(unemployed ~ mean_traffic + mean_deck_cond + mean_superstructure + unemployed_lag, data = Bridges_and_BLS)

summary(lm_unemployed)
```

    ## 
    ## Call:
    ## lm(formula = unemployed ~ mean_traffic + mean_deck_cond + mean_superstructure + 
    ##     unemployed_lag, data = Bridges_and_BLS)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2946.4  -288.6  -121.4    77.5 12841.7 
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)          3.756e+03  3.760e+03   0.999    0.320    
    ## mean_traffic        -6.783e-03  4.616e-02  -0.147    0.883    
    ## mean_deck_cond      -6.744e+02  7.461e+02  -0.904    0.368    
    ## mean_superstructure  1.328e+02  5.697e+02   0.233    0.816    
    ## unemployed_lag       3.169e-02  2.368e-03  13.382   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1427 on 104 degrees of freedom
    ##   (1 observation deleted due to missingness)
    ## Multiple R-squared:  0.8605, Adjusted R-squared:  0.8551 
    ## F-statistic: 160.4 on 4 and 104 DF,  p-value: < 2.2e-16

``` r
#Unemployed lag is highliy significant
#Adj R-Sq of 0.8551, which is significantly higher than without lagged regressor from previous period
```

``` r
#lagged regressor model
lm_unemployment_rate <- lm(unemployed_rate ~ mean_traffic + mean_deck_cond + mean_superstructure + unemployment_rate_lag, data = Bridges_and_BLS)

summary(lm_unemployment_rate)
```

    ## 
    ## Call:
    ## lm(formula = unemployed_rate ~ mean_traffic + mean_deck_cond + 
    ##     mean_superstructure + unemployment_rate_lag, data = Bridges_and_BLS)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.6696 -0.5802 -0.1702  0.5737  3.6150 
    ## 
    ## Coefficients:
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            4.443e+00  2.917e+00   1.523 0.130792    
    ## mean_traffic          -1.167e-04  3.290e-05  -3.546 0.000588 ***
    ## mean_deck_cond        -6.373e-01  5.768e-01  -1.105 0.271769    
    ## mean_superstructure    5.471e-01  4.399e-01   1.244 0.216394    
    ## unemployment_rate_lag  1.513e-04  4.836e-05   3.129 0.002279 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.102 on 104 degrees of freedom
    ##   (1 observation deleted due to missingness)
    ## Multiple R-squared:  0.1314, Adjusted R-squared:  0.098 
    ## F-statistic: 3.933 on 4 and 104 DF,  p-value: 0.005158

``` r
#Unemployment_rate lag is highliy significant
#Adj R-Sq of 0.005158, which is higher, but still not good
```

Based on these models, it’s clear that it’s much easier to predict the
number of unemployed persons in each fips county than the unemployment
rate. The R-squared value for all of the unemployed models was
drastically higher than the models that used the rate instead.
Additionally, the lagged variables were highly significant, which shows
us that the previous periods unemployed and unemployed\_rate are both
good predictors for the next period. Overall, it seems like bridges data
gives us some insight into the number of unemployed persons in a fip
code, but it is not perfect and could be improved.
