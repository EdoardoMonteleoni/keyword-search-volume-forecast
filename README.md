# Forecasting the volume of keyword searches for efficient PPC budget allocation in the travel/tourism industry

### A Search Volume Forecasting Project in the Travel Industry Using Official Air Traffic Data as an Alternative Approach to Improve Projections and Budget Allocation.

![LinkedinBackGround2](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/b0658c0b-a92c-4ef6-8e81-0c74cb9e272e)

Search advertising budgets should be based on seasonality and demand fluctuations [^1], and a good **search volume** forecast at the keyword level can allow organisations in the tourism and travel industry to run efficient and effective search advertising campaigns.

A common approach to the question is using the built-in keyword planner tool in Google Ads or Microsoft Advertising. These platforms consider the account's historical data, plus other factors, such as seasonal values, organic volumes and competitor bidding for the intended group of keywords.[^2]

Since I don't want to rely solely on these "black-box" tools to gauge a short-term search volume for a keyword set, I will show a forecasting method based on a time series regression analysis and a data set derived from an official Airport's website.

# The business problem

Setting a proper budget for search engine advertising campaigns is paramount, and doing it beforehand is even more critical, particularly for travel and tourism businesses that have to deal with seasonal demand fluctuations. To efficiently and effectively set up proper search budgets, apart from historical Cost per Click, one could consider the volume of future searches for the targeted keywords.

An important metric used in Search Engine Advertising to assess the number of times a user searches for a particular (paid) keyword is [Search Volume](https://adalysis.com/blog/ppc-kpi-monitoring-how-to-diagnose-changes-to-your-impression-search-volume), which is the number of Ad impressions divided by the search network [Impression Share](https://adalysis.com/blog/ppc-kpi-monitoring-how-to-diagnose-changes-to-your-impression-search-volume).
One of its advantages is that it is a keyword-based index, which considers the number of relevant queries from interested users about the advertised product.

Considering historical data available at the account keyword level, forecasting Search Volume is relatively easy. The hard part is building and presenting forecasts when we have no other metric to rely on. 
The cons I found in using this variable are two:

1 -  This metric depends on "Total Eligible Impressions", an index based on factors such as targeting settings and quality. It is a hidden-computed numerical estimate made by the ad platform and based only on contextual elements. Modifying negative keyword lists or adding a particular target audience might increase or decrease this metric considerably.

2 - In Search Engine Marketing, "Impressions" measures the times an Ad appears on a search result page after typing a search term. If the advertiser specifies a set of keywords that are relevant to that search query, and also the [Ad Rank](https://support.google.com/google-ads/answer/1752122?hl=en&ref_topic=24937&sjid=13026874370645627094-EU) is above a certain threshold, then the Ads could be shown. The problem is that those are intent-based keywords, and a random number of users who seek the product will miss the bottom of the sales funnel. This gap can make "search volume" forecasts more or less uncertain.

More reliable measures to rely on, such as the ones from official statistics, could reduce this gap and allow marketing analysts to set more reliable budget forecasts. For example, for this work, I have referred to Rome Fiumicino's Air Traffic Data [web page](https://www.adr.it/web/aeroporti-di-roma-en/bsn-traffic-data?p_p_id=it_adr_trafficdata_web_portlet_TrafficDataWebPortlet&p_p_lifecycle=0&p_p_state=normal&p_p_mode=view&_it_adr_trafficdata_web_portlet_TrafficDataWebPortlet_dataRif=202312&_it_adr_trafficdata_web_portlet_TrafficDataWebPortlet_tabs1=FCO).

The general idea is to weigh the keyword search volume with the projected monthly number of air passengers gathered from the above data source. Here, the dependent variable "keyword search volume" is derived as a linear relationship with its future trend and seasonal components plus the official air passenger volume. The forecasting period is about the first three months of 2023.

# Data understanding

The time frame considered is from February 2018 to December 2022, and the final data set is made up of a date column filled with year-month values, a second column consisting of monthly search volume derived from the company's Google Ads account, and a final field with a historical number of passengers for the intended Airport. 

Since the data frame is made up of only 59 records,  I did not use any particular structured tool to build it. As for the forecasts, I used the programming language R, particularly [feasts](https://feasts.tidyverts.org/) for time series analyses and [fable](https://fable.tidyverts.org/) for the forecast part.

Comparing two [time series](Search_vs_passengers_variation.pdf) with significant differences in scale is quite complex, so instead of computing transformations, for this work, I will consider month-over-month per cent variations, both for "search volume" and for "number of passengers". Also, treating relative measures instead of absolute ones prevents further masking sensible data.

![Searc_vs_passenger_variation](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/d3568b3c-f53e-4d44-b88f-bea844342a83)

The time series have similar trends, suggesting a [positive linear relationship](feature_linear_relationship.pdf) between them. To a certain extent, this behaviour is assumed since the product research by a prospective customer comes before the actual flight booking. Rather, one can effectively deploy this aspect to enforce the explicability of a linear model to produce more accurate and reliable predictions.

![feature_linear_relationship](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/2d8f4817-db85-43e9-a066-9c0981081679)

A trough in March 2020 and a significant peak in June are due to the COVID-19 restrictions, followed by gradual Airport reopenings, 
with digital users searching for information for that upcoming summer holiday. Hence, these values are not considered outliers.

The seasonal plot confirms the gap between when users search for a parking service and when the actual holiday season starts; for example, peaks in November are related to the Christmas Holiday season. The same is true for July's search volume peaks, which refer mostly to the users who booked a flight in August.

The subseries plot, other than emphasizing the seasonal patterns of the time series, shows a spike in January 2020 a month before the COVID-19 virus started spreading worldwide. After three months of negative peaks due to the restrictions, at the end of May 2020, the Italian Government allowed the reopening of many activities. So, users started searching for booking flights, hence the peak in June 2020.  

![seasons](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/52f34ee1-2b8d-4e9a-8033-a12cdbfe153b)

![subseries](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/8f819028-b3b7-40af-8a25-4a9c7906c69d)

# Data preparation

I split the data frame into a [training](training_df.csv) and a [test](test_df.csv) data set. Considering the highly seasonal demand for the service, the "testing" part comprises the entire solar year, from January to December 2022.

# Modelling

Two variations of the Linear Regression and Exponential Smoothing models are used.[^3] 
As for the Time Series Linear Model, a first version including air passengers with trend and seasonal components is implemented. The other version does not include air passengers as an exogenous feature.
The last is an exponential smoothing model with the Holt-Winters method, with additive trend and seasonal components.

# Model Evaluation

Looking at the residual plots, neither model sufficiently fits the data. Nevertheless, the Linear model with the "passenger" feature accurately forecasted the metric on the Test data set. Moreover, compared to the built-in search volume projection of the Ad platform, the estimates give a more comprehensive and helpful frame of the short-term search volume expectations.

![forecast_accuracy](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/fd842d02-2b83-4cf5-b83e-132f68d0522c)

Looking instead at the point forecast accuracy such as Mean Absolute Percent Error (MAPE) or Root Mean Squared Error (RMSE), "tslm1" with its two scenarios, is the model with the smallest values among the entire group.

# Conclusions

Although the selected model did not fit the data training correctly, the three-month [forecasts](three_month_forecast.pdf) seem to better adhere to the market situation after the COVID-19 restrictions. 

![three_month_forecast](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/5ce0c4fd-5efa-45b4-8d0a-70086e4510fe)

_A plot of search volume variation forecast for the Q1-2023_

While the Ad platform estimated no variation in search volume for the selected keyword in the first quarter (0%), the model selected performed more optimistic [values](forecast.csv): a 21% (first scenario) and a 15% (second scenario) of search volume increasing (on average). The real search volume delta variation that actually occurred was 16%. So, the regression model with the estimated Airport passengers using the seasonal naive model (second scenario), produced a more accurate forecast.

![Gads Forecast](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/c969bb64-68f6-4ad9-84c3-462f8338e4d8)

_The three-month Google Ads forecast for the target keyword_

![forecast](https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/5b749eb2-e72a-42ce-90a8-8633071cf629)

_The three-month forecast computed using the time series linear model and two different scenarios for the near future number of passengers_

Considering the highly seasonal behaviour of the metric being forecasted, the Seasonal Naive method (scenario n.2) used in the linear model produced a search volume prediction that fitted the actual trend considerably well. 


<img width="500" alt="forecast_accuarcy_chart" src="https://github.com/EdoardoMonteleoni/keyword-search-volume-forecast/assets/105068746/e84c54ef-8fac-4de7-890b-3d192e472408">


Google Ads keyword planner is a great and reliable forecasting tool that relies on AI and other fundamental account-based factors for its projections. On the other hand, its "black-box" framework and a certain lack of flexibility in choosing customised timeframes for volume forecasts, should prompt the analytics team to seek and implement other methods to produce month-over-month predictions. For example, the one shown in this project could help effectively and efficiently allocate search ad budget, particularly where monthly adjustments are needed.

[^1]: Think with Google: _"3 budgeting tips to help marketers plan for 2021 amid uncertainty"_ (https://www.thinkwithgoogle.com/intl/en-emea/future-of-marketing/management-and-culture/3-budgeting-tips-help-marketers-plan-2021-amid-uncertainty/). Last access Feb 2024

[^2]: Google Ads Help - _"About Keyword Planner forecasts"_ (https://support.google.com/google-ads/answer/3022575?sjid=13323525779093924143-EU&visit_id=638430680710415783-1787451205&rd=1). Last access Feb 2024

[^3]: Hyndman, R.J., & Athanasopoulos, G. (2021) _Forecasting: principles and practice_, 3rd edition, OTexts: Melbourne, Australia. OTexts.com/fpp3. Accessed on Feb 2024
