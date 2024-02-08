# Forecasting the volume of keyword searches for efficient budget allocation in Search Engine Advertising for the travel/tourism industry

### Keywords Search Volume Forecasting in the Travel Industry: Another Approach Using Official Air Traffic Data to Improve the Projections

Search advertising budgets in the tourism and travel industry are strictly based on seasons, and a good search volume forecast at the keyword level allows the organisation to run efficient search ad campaigns.

A common approach to the question is using the built-in keyword planner tool in Google Ads or Microsoft Advertising. These platforms consider the account's historical data, plus other factors, such as seasonal values, organic volumes and competitor bidding for the intended group of keywords.

Since I don't want to rely solely on these "black-box" tools to gauge a short-term search volume for a keyword set, I will show a forecasting method based on a time series regression analysis and a data set derived from an official Airport's website.

# The business problem

Setting a proper budget for search engine advertising campaigns is paramount for many reasons, and doing it in advance is even more critical, particularly for travel businesses that have to deal with seasonal demand fluctuations. To efficiently and effectively set up proper search budgets for future marketing campaigns, apart from historical Cost per Click, one should consider the volume of future searches for the targeted keywords.

An important metric used in Search Engine Advertising to assess the number of times a user searches for a particular (paid) keyword is [Search Volume](https://adalysis.com/blog/ppc-kpi-monitoring-how-to-diagnose-changes-to-your-impression-search-volume), which is the number of Ad impressions divided by the search network [Impression Share](https://adalysis.com/blog/ppc-kpi-monitoring-how-to-diagnose-changes-to-your-impression-search-volume).
One of its advantages is that it is a keyword-based index, which considers the number of relevant queries from interested users about the advertised product.

Considering historical data available at the account keyword level, forecasting Search Volume is relatively easy. The hard part is building and presenting forecasts when entirely dependent on this metric. The cons I found in using this variable are two:

1 -  This metric depends on "Total Eligible Impressions", an 
Index based on factors such as targeting settings and quality. It is a hidden numerical estimate made by the ad platform considering only contextual elements.

2 - Impressions are based on intent-based keywords, and looking for a product doesn't necessarily mean a future sale. There is always the likelihood that a random number of users will miss the bottom of the sales funnel.

More reliable measures to rely on, such as the ones from official statistics, could reduce this "intent-do" gap and allow marketing analysts to build more reliable budget forecasts, a source such as the Rome Fiumicino's Air Traffic Data [web page](https://www.adr.it/web/aeroporti-di-roma-en/bsn-traffic-data?p_p_id=it_adr_trafficdata_web_portlet_TrafficDataWebPortlet&p_p_lifecycle=0&p_p_state=normal&p_p_mode=view&_it_adr_trafficdata_web_portlet_TrafficDataWebPortlet_dataRif=202312&_it_adr_trafficdata_web_portlet_TrafficDataWebPortlet_tabs1=FCO) I used in this very project.

The general idea is to weigh the keyword search volume with the projected monthly number of air passengers gathered from the above data source. Here, the dependent variable "keyword search volume" is derived as a linear relationship with its future trend and seasonal components plus the official air passenger volume. The forecasting period is about the first three months of 2023.

# Data understanding

The time frame considered is from February 2018 to December 2022, and the final data set is made up of a date column filled with year-month values, a second column consisting of monthly search volume derived from the company's Google Ads account, and a final field with historical number of passengers for the intended Airport. 

Since the data frame is made up of only 59 records,  I did not use any particular structured tool to build it. As for the forecasts, I used the programming language R, particularly [feasts](https://feasts.tidyverts.org/) for time series analyses and [fable](https://fable.tidyverts.org/) for the forecast part.

Comparing two [time series](Search_vs_passengers_variation.pdf) with significant differences in scale is quite complex, so instead of computing transformations, for this work, I will consider month-over-month per cent variations, both for "search volume" and for "number of passengers". Also, treating relative measures instead of absolute ones prevents further masking sensible data.

![feature_linear_relationship](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/96f46664-9737-40ae-990b-93fc139ec998)

The time series have similar trends, suggesting a [positive linear relationship](feature_linear_relationship.pdf) between them. To a certain extent, this behaviour is assumed since the user intent tends to precede the actual flight booking. 

A trough in March 2020 and a significant peak in June are due to the COVID-19 restrictions and the following Airport reopenings, 
with digital users searching for information for that upcoming summer holiday. Hence, these values are not considered outliers.

# Data preparation

I split the data frame into a [training](training_df.csv) and a [test](test_df.csv) data set. Considering the highly seasonal demand for the service, the "testing" part comprises the entire solar year, from January to December 2022.

# Modelling

Two variations of the Linear Regression and Exponential Smoothing models are used. 
As for the Time Series Linear Model, a first version including air passengers with trend and seasonal components is implemented. The other version does not include air passengers as an exogenous feature.
The last is an exponential smoothing model with the Holt-Winters method, with additive trend and seasonal components.

# Model Evaluation

Looking at the residual plots, neither model sufficiently fits the data. Nevertheless, the Linear model with the "passenger" feature accurately forecasted the metric on the Test data set. Moreover, compared to the built-in search volume projection of the Ad platform, the estimates give a more comprehensive and helpful frame of the short-term search volume expectations.

# Conclusions

Although the selected model did not fit the data training correctly, the three-month [forecasts](three_month_forecast.pdf) seem to adhere better to the market situation after COVID-19 restrictions. 

![three_month_forecast](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/5ce0c4fd-5efa-45b4-8d0a-70086e4510fe)
Plot of search volume variation forecast for the first quarter 2023: 
While the Ad platform estimated for the first quarter no average search volume variation for the selected keyword (0%), the model selected performed more optimistic [values](forecast.csv): a 21% (first scenario) and a 15% (second scenario) search volume increase (on average). The actual delta variation was 9.05%, as the average search volume occurred in 2023-Q1. So, the regression model, where the Airport passengers were estimated using the seasonal naive model (second scenario), produced a more accurate forecast.

![Gads Forecast](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/f1ba85a8-cf72-4756-92ee-343645043eba)
The three-month Google Ads forecast for the target keyword
