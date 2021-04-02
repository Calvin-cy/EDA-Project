# MTA Turnstile Data : Exploratory Data Analysis
Calvin Yu

## Abstract

The goal of this project was to explore the Nypd crimes report data set and the MTA turnstile data set and see if higher numbers of crime reports happened during the areas of heavy traffic. Since the MTA turnstile data set was not cleaned, the first task for me was to clean them up by using Pandas. I was trying to compare the daily reported crimes with the daily entries for each borough. However,since this exploratory analysis is based on the boroughs and the MTA turnstile data set doesn't have the borough column, I created a dictionary to map the top twenty busiest stations along with the busiest station for each of the borough to give me an estimate of the daily entries for each borough. After mapping the dictionary with the MTA turnstile data, I generated the graph to see the daily entries of each Borough and compare it with the graph of daily reported crimes to see if my higher numbers of crime reports really happen during the areas of heavy traffic.

## Design
### Backstory

Calvin - 

It was nice to meet you at the networking event last week. As I mentioned in our previous conversation, the New York Police Department is trying to hire more polices. We believe that higher levels of crime reports happen during areas of heavy traffic. However, it is only our assumption that areas with high density will lead to more crime reports. Therefore, We would kindly ask you to evaluate our crime reports data and the MTA Turnstile data to give us an insight into which borough should need more polices. Do you think this is something that you would be interested in? If you are interested, please contact us, and we will discuss what kind of engagements would make sense for all of us.

  -NYPD

In order to answer the question from the NYPD, we will need to dig into the data sets that they provided. After studying the data sets, I find out that the daily entries of each borough from the MTA turnstile data, and the daily reported crimes in each borough from the NYPD date set can be used to answer this question. By comparing both graphs, the NYPD can see if the crimes were reported in the Borough with heavier traffic, and can also give them an idea whether their assumption is right or not.

## Data
### MTA turnstile data set:
- Period: 2015-01-01 to 2015-03-31
- contains 445594 rows and 11 columns
- few features highlights: Station, Entries, Date
### Nypd data set:
- Period: 2015-01-01 to 2015-03-31 
- contains 1048575 rows and 24 columns
- few features highlights: date of occurrence for the reported event, Borough 

## Algorithms
### MTA turnstile data set:
 1. Using Urllib to download the MTA data sets 
 2. Using SQLalchemy to enable reading the data set from Pandas and import all the modules that I may need: Seaborn, Pandas, and Matplotlib

 3. Dropping the first row of the data which was duplicated from the index
 4. Using pd.to_datetime to convert the DATE and TIME column which dtype were objects 
 5. Using groupby to find duplicated entries
```
python all_data.groupby(["CA", "UNIT", "SCP", "STATION", "DateTime"]).ENTRIES.count().reset_index().sort_values("ENTRIES", ascending=False).head(5)
```

6. Dropping the duplicated entries 

``` 
all_data.drop_duplicates(subset=["CA", "UNIT", "SCP", "STATION", "DATE"], inplace=True)
```
7. group all the useful features together and assign it with a new name: all_data_daily

```
all_data_daily = (all_data.groupby(["CA", "UNIT", "SCP", "STATION", "DATE"],as_index=False).ENTRIES.first())
```

8. Create a column of previous date and a column of previous entries
```
all_data_daily[["PREV_DATE", "PREV_ENTRIES"]] = all_data_daily.groupby(["CA", "UNIT", "SCP", "STATION"])["DATE", "ENTRIES"].apply(lambda grp: grp.shift(1))
```

9. Drop the null value and check to see if any previous entries is greater than the current entries

``` 
all_data_daily.dropna(subset=["PREV_DATE"], axis=0, inplace=True)
```
```
all_data_daily[all_data_daily["ENTRIES"] < all_data_daily["PREV_ENTRIES"]].head()
```

10. Create a function to deal with the problem of reversed entries and perform it 

```
def get_daily_counts(row, max_counter):
    counter = row["ENTRIES"] - row["PREV_ENTRIES"]
    if counter < 0:
        
        counter = -counter
    if counter > max_counter:
        
    if counter > max_counter:
        
    return counter
```
11. Create a column named Daily Entries by applying the get_daily_counts function

```
all_data_daily['DAILY ENTRIES'] = all_data_daily.apply(get_daily_counts,axis = 1 , max_counter = 1000000)
```
12. Grouping by the columns that we need and rename it as station_daily

```
station_daily = all_data_daily.groupby(["STATION", "DATE"])[['DAILY ENTRIES']].sum().reset_index()
```

13. Since the original data set doesn't have a Borough column, we will create a dictionary to map the station with the Borough. To save time, we will only map the top 20 busiest stations, and the busiest station of each borough if its busiest station isn't in the top 20 list.

```
STATION_TO_LOCATION = {'34 ST-PENN STA' : 'MANHATTAN' , '42 ST-GRD CNTRL' : 'MANHATTAN' , '34 ST-HERALD SQ': 'MANHATTAN', '14 ST-UNION SQ': 'MANHATTAN', '86 ST':'MANHATTAN','42 ST-PA BUS TE':'MANHATTAN','42 ST-TIMES SQ':'MANHATTAN','125 ST':'MANHATTAN', 'FULTON ST':'MANHATTAN','96 ST':'MANHATTAN','CHAMBERS ST':'MANHATTAN','CHAMBERS ST':'MANHATTAN','CANAL ST	':'MANHATTAN','23 ST':'MANHATTAN','59 ST':'MANHATTAN','47-50 ST-ROCK':'MANHATTAN','MAIN ST': 'QUEENS','BARCLAYS CENTER':'BROOKLYN','ROOSEVELT AVE':'QUEENS','72 ST':'MANHATTAN','161 ST-YANKEE' : 'BRONX','ST. GEORGE':'STATEN ISLAND'}
```

14. Create a column of Borough and drop the rows that weren't in the top 20 busiest stations or the busiest of the borough. 
```
station_daily['BORO_NM']  = station_daily['STATION'].map(STATION_TO_LOCATION)
```
```
station_daily.dropna(inplace = True)
```
15. So now when we group by the borough, date ,and station,  we can visualize the daily entries by borough 

16. I find some outliers in the daily entries column, since the daily entries of one station should not exceed 300,000, I will replace the outliers with the median of each borough

17. This is the graph of daily entries for each borough 

![MTAdaily](https://user-images.githubusercontent.com/63031028/113382486-68298280-9336-11eb-8bf9-f298c84a69ce.png)

### Nypd Data set
1. Download the dataframe, and read it using Pandas
2. Change the column name of CMPLNT_FR_DT to DATE for convenience 
3. Filter out the months and years that we need on this dataset and rename it as Nypd
4. Group by the Date and Borough to get the count for the total reported crimes per borough and per date
```
Nypd = Nypd_df_Jan_to_Mar.groupby(['DATE', 'BORO_NM'])[['BORO_NM']].count()
```
5. Rename the last column as Daily Entries 
```
Nypd.rename({'BORO_NM' : 'Daily Entries'}, inplace = True,axis = 'columns')
```
6. sort the date for Nypd, and graph it out 

![Nypd](https://user-images.githubusercontent.com/63031028/113383195-3dd8c480-9338-11eb-8eb2-b1d369bcbd00.png)

## Tools
- Numpy and Pandas for data cleaning, data aggregation, data manipulation
- Seaborn for plotting out the graphs
- Urllib to download the files for the Mta data sets
- SQLAlchemy to read the files in the python environment
- SQL for data storage , and data scanning 

## Communication
To summarized what I found, the MTA traffic in Manhattan is way higher than in any other boroughs. However, the reported crimes are more likely to occur in Brooklyn. This evidence indicates that higher number of crime reports doesn't happen during the heavy areas of higher traffic. Therefore, they should not hire more polices in Manhattan. Moreover, I realized that the peaks and the troughs from the both the graphs share a really similar pattern which both peak on the weekends, and trough on the weekdays. 

Please go to https://github.com/Calvin-cy/EDA-Project to see a more detailed explanations with the slides. 

## Further steps 
Here are the following that we could explore in the future:

1. Explore further on the TIME columns for both data sets to see if the crime reports happen during specific time and compare the numbers of crimes in specific time to the number of traffic.

2. Incorporate a wealth distribution data to determine if the areas of high number of crime reported are generally poor.

3. Explore further on the Description of internal classification , and the victim race column along with the TIME columns to see what kind of crime would happen during what time and target which sex.

## Takeaways
1. The importance of data cleaning
2. How to deal with outliers,and null values
3. How to use different modules to explore and manipulate the data sets