# LA Collisions Project

## Introduction
Residents of Los Angeles City and County live with a day to day concern of being involved in car accidents. Similarly, collisions are a concern for governmental agencies and first responders, who must make policy decisions and allocate resources to preventing and responding to motor accidents, particularly to those of the highest severity. Congestion in Los Angeles further complicates first response efforts. Consequently, an analysis of the existing data on traffic collisions in Los Angeles is warranted, to help improve the precision and efficiency of these efforts. 

## Dataset
+ **Sources**: Data acquired from Kaggle, titled "Los Angeles Traffic Collision Data" (https://www.kaggle.com/datasets/cityofLA/los-angeles-traffic-collision-data/data?select=traffic-collision-data-from-2010-to-present.csv). I used this source from LA City to translate the MO Codes from the police report into relevant information about the collision: https://data.lacity.org/api/views/d5tf-ez2w/files/8957b3b1-771a-4686-8f19-281d23a11f1b?download=true&filename=MO_CODES_Numerical_20180627.pdf

+ **Structure**: This data shows information like the police report number, date reported, date occurred, time occurred, area information, victim demographic information, MO codes from the police report, zip codes, and geospacial coordinates. The MO Codes data lists the codes that appear on the police report and a description that explains if the collision involved a certain situation (such as a pedestrial being involved).

## Tools Used ##
+ All analysis for this project was done in Python with the help of various mathematical, statistical, and visualization libraries.

## Data Collection and Preprocessing ##

The data from the source was not in very useful format, so required quite a bit of preprocessing before analysis could begin. 

First, the MO Codes were formatting as a string, like "3036 3004 3026 3101 4003". Each of these codes means that a certain circumstance was present in the accident. For example "3036" means that the accident occurred at an intersection. The first step data cleaning step was to translate these codes into useful binary variables, defined in columns in the data frame. For example, if a pedestrian was involved in the collision it would be coded as "3036" or "3501", based on the MO Codes information. So if either of those values were present in the MO Codes string, the new column "Pedestrian_involved" would take a value of 1, otherwise it would be coded as a 0. I coded as many independent variables as seemed relevent to be useful in predicting the severity of the outome of the collision. The outcomes "No_injury", "Severe_injury", "Fatality", and "Other_injury" were also coded as new columns using the MO Codes data, and those variables will be considered the dependent variables in this analysis. 

Next steps in the data cleaning were to drop unnecessary columns from the dataset, and format data and time correctly. Times were also given as strings in the original data. For example, "1341" would indicate a time of 1:41 PM. I created a function to convert all these strings into correctly formatted times. Later, I also coded the times into a categorical variable, with categories of 'Late Night', 'Early Morning', 'Morning', 'Afternoon', 'Evening', 'Night' corresponding to 4 hour blocks of time starting from midnight to 4 AM ('Late Night') and so on. 

Finally, Latitude and Longitude were listed as strings in a tuple-esque format, as such: "(34.0255, -118.3002)". For the purposes of creating a folium map of fatal accidents later in the analysis, I extracted the Latitude and Longitude values into separate columns in the data frame. 

At this point the data was now in a useful format for conducting analysis. 

## Exploratory Data Analysis
### Time Series Trends

The first thing I wanted to visualize were the time series trends of each category of collision. Starting with fatal accidents, there's a clear uptrend over time: 

![download](https://github.com/user-attachments/assets/e51b83be-fbfe-40da-ad68-db16a497121b)

Similarly, the trend for collisions resulting in severe injuries is also upsloping: 

![download](https://github.com/user-attachments/assets/cbeb5c6b-4a39-4021-8cb9-5472849af2f4)

Conversely, the trends for total collisions and collisions categorized as "other injuries" peak around 2017-2019, and sharply decline after that: 

![download](https://github.com/user-attachments/assets/0a2140d3-62e0-4d41-8a42-8038fd36815a)
![download](https://github.com/user-attachments/assets/7724f8b1-cad5-43b9-9d9f-63d15eaf4052)

This discrepancy begs the question why collisions resulting in severe and fatal injuries have been rising, while overall collisions have been falling. Explaining that definitively is beyond the scope of this analysis, but some possible explanations could be changes in reporting practices, or could be due to technological improvements resulting in fewer minor accidents, while perhaps people paying less attention or using phones while driving could in part explain the increase in severe accidents. The divergence in these trends warrants additional investigation. 

I also looked at time series of collisions involving pedestrians and cyclists. 
![download](https://github.com/user-attachments/assets/4e8545b1-cbbd-453e-b1a4-753e02997cf9)
![download](https://github.com/user-attachments/assets/6190f05d-69fb-458d-bf6e-be24a84d4fd5)

Collisions involving cyclists peaked in 2013 and has been on the decline since then, while collisions involving pedestrians peaked in 2019 and has declines since that time. The time series involving collisions with pedestrian involvement looks quite similar to the overall collisions time series, while for collisions involving cyclists, the declining trend started earlier. This could be due to greater awareness on the part of drivers to cyclists or to policies that encouraged cycling by creating new bike lanes or making bike lanes more segregated from normal traffic. According to LAIST, the city had 245 miles of bicycle infrastructure in 2005, which grew to over 562 miles in 2015. This association needs more supporting evidence, but it appears to suggest a return on investment to the city in bicycle infrastructure. Collisions involving bicycles decreased from about 2500 in 2013 to around 500 in 2023. 
+ Source: https://laist.com/news/kpcc-archive/watch-a-decade-of-growth-in-la-s-bike-infrastructu

### Sex and Ethnicity Statistics
Bar graphs of number of collisions by sex and ethnicity show some interesting correlations. Clearly, more collisions involved men than women - a finding that aligns with prior data suggesting men cause about 60% of accidents in LA county while women cause approximately 40%. Men are also consistently killed at 2-3 times the rate that women are in motor vehicle accidents in the US as a whole. 
![download](https://github.com/user-attachments/assets/ced9d284-30b8-4835-bfc0-7a59c8dc43fb)

+ Source: https://www.enjuris.com/blog/ca/men-versus-women/#:~:text=According%20to%20traffic%20collision%20data,40%20percent%20of%20car%20accidents.

In terms of ethnicity, Hispanics are most involved in vehicular collisions of all ethnic groups in LA city, followed by whites and then Blacks/Other.

![download](https://github.com/user-attachments/assets/26045441-f123-4989-966a-01a6672f31e8)

### Analysis of Accidents by Time of Day

Among all accidents, the most frequent time of day is the afternoon (from 12 PM to 4 PM) follow closely by evening (4PM to 8PM). 

![download](https://github.com/user-attachments/assets/7caca640-6217-4476-9719-3485033e9526)

However, collisions involving severe injuries are more skewed to occuring in the evening, while fatal collisions occur most often in the evening, night, and late night (altogether, from 4PM to 4AM). 
![download](https://github.com/user-attachments/assets/8e2a2056-7cf7-4149-acd2-d2454a419ecc)

![download](https://github.com/user-attachments/assets/6f5d0aee-42c3-4fb5-af9c-411833b6baaa)

Interestingly, although the afternoon ranks first as the time of day in which the most accidents occur, it ranks second to last in terms of fatal accidents. A cursory look at these bar plots suggests that total collisions is most correlated with when the most people tend to be driving (from 8am to 8pm), but that the severity of accidents seems more correlated to daylight/nighttime. 


