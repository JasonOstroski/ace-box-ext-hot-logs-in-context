## Lab 3: Business Use Case
 
### What will this lab focus on? 
In this lab we’ll use the App “*Easytrade*”, which is a trading platform App where people can buy, sell, transfer and withdraw funds. We will demonstrate for you how *Easytrade* works and the importance of observing the App to see how much money is being traded (stock market never closes here). We’ll also show you how to find logs via *Notebooks* of trades and complete an exercise in data analysis to build a business dashboard.  
 
### Lab Objective
Learn how *log data*, *notebooks*, and *dashboards* in Dynatrace can answer important business performance questions.  
 
### Lab Scenario 
Our business App, *EasyTrade* has log data that we need to analyze to determine if our business is making money, or not. Our App executives have asked you to answer the following questions:  
  
1. How many **deposits** were made in the last two hours?  
2. How many **withdrawals** were made in the last two hours?  
3. How much money is the platform **earning**?  
4. Which account had the **biggest increase/decrease**? 
5. Can we **predict the future earnings** of the platform?  
6. What are the **top 5 accounts** with the most money going out?  
7. How much money is traded **long compared to market trades**?  
8. How much money is **deposited vs withdrawn**? 
 
We’ll go through all the necessary steps to pull actionable log data together, in a Dynatrace *notebook*, and how to go about analyzing the data so that you can answer each question above. Once we’ve answered each question, summarized into one notebook and then dashboard - you’ll be able to confidently answer all your leaderships business questions, accurately, and visually! 


### Why is knowing/doing this important? 
Ultimately, we want to use this data, not solely just for observability purposes, but to specifically report on how our business is doing. Excercises in this lab will demonstrate how we can turn actionable data in to answers for critical performance questions like, "*Can we predict the future earnings of the platform?*", "*How much money is the platform earning?*", and, ultimately, "*is my business profitable?*"  

### Step 1: Selecting and filtering required log data
1. As we did in our first lab, we will be using the Logs app for the intial investigation of the data. Open the **logs app**
2. Add a filter to view the data particular to EasyTrade. To achieve this simply add a filter for the **k8.namespace.name** with the value of “**easytrade**”. This will filter the data just for the *EasyTrade* application.

```DQL
Filter: k8.namespace.name
Value: easytrade
```

3. Let us filter this content further to view data related to transactions and changes in balance. To achieve this add a filter on the **content** with the value of “**balance**”  

![Content Filter by Balance](../../assets/images/ContentBalanceFilter.png)

4. Click on **any log line** and click on “**view details**”. Explore the data that is available to understand how different metadata and context to this log line

5. Click on “**xx columns hidden**” and add the following columns,
    - **accountId**
    - **actionType**
    - **oldValue**
    - **valueChange**

You should now be able to see the data for those columns and understand the different transaction types and the money related to it.  

![Collumns Added](../../assets/images/CollumnsAdded.png)

6. Now that we have the right data set, it is time to dig in further to analyse our data. To achieve this click “**open with**” and choose **Notebooks**. You should be familiar with this step as we performed it in previous labs
7. In the notebook we can see the query that is being used to generate our data. For the analysis we do not need all the columns and we can drop the unwanted columns by modifying the query. This will be more efficient. We can achieve this by defining the exact fields required using the fields command. You can simply copy and paste **the below query** into your notebook and click **Run**.

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| sort timestamp desc
```

Now that we have our data selected and prepared for analysis, it’s time to review the data to answer our prescribed business-level questions.

Let's answer the first question: **How many deposits were made in the last two hours?**

To achieve this we need to filter the data for the deposit action and count the number of entries for the last two hours

8. Click on any “**deposit**” field in the “**actionType**” column and click on **filter**. This will filter the data for all deposits.
9. Next click on the column “**actionType**” and click on “**summarize**”. This will generate the number of deposits for the selected timeframe.

![Summarize by Action Type](../../assets/images/SummarizeByActionType.png)

With that, we have answered the first question, and know how many deposits were made in the last 2 hours.

Let's move to the next question: **How many withdrawals were made in the last two hours?**

10.	Let's proceed by duplicating **the current section** where we have the data for deposits.  

![Duplicate Depostis Section](../../assets/images/DuplicateDepositsSection.png)

11. In the duplicated section, simply replace the **actionType** in the filter with the value of “**withdraw**”. The rest of the query stays the same. Click **Run** and you should have the same data for withdrawals. Below is the final query;

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| sort timestamp desc
| filter actionType == "withdraw"
| summarize count = count(), by:{actionType}
```
![Withdrawal Version](../../assets/images/WithdrawalsSummarization.png)

Let’s answer the third question: **How much money is our platform earning?**

We know that the business makes money through the transaction fee and the *actionType* related to it is “**collectfee**”. Therefore we need to filter and summarize data related to this action.

12. To start with this, you can again duplicate **the previous section**. There two changes required for this query. First, we need to change the filter value for **actionType** to “**collectfee**” and then summarize the total of the **valueChange** instead of counting it.

Below is the final query;

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| filter actionType == "collectfee"
| summarize TransactionFeesCollected = sum(valueChange)
```

![Transaction Fees Collected](../../assets/images/TransactionFeesCollected.png)
 
Now we know that the business is making profit and how much money was earned in the last 2 hours

Let’s answer the fourth question: **Which account had the biggest increase / decrease?**

To achieve this, we will track the *valueChange* corresponding to the *accountId* and order the resulting data in descending order.

13. To start with this, you can again duplicate **the previous section**. For this query we will add an additional field “**absoluteValueChange**” to track the absolute change in value as we want to know biggest change regardless of **increase/decrease**. You can copy and paste the below DQL query and hit “**Run**”.

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| fieldsAdd absoluteValueChange = abs(valueChange)
| sort absoluteValueChange desc
| fields accountId, absoluteValueChange
| limit 10
```

![Value Change Section](../../assets/images/ValueChange.png)

Now we can the list of accounts that had the biggest change in their balance.

Let’s answer the fifth question: **Can we predict future earnings of our platform?**

To achieve this, we need to create a *timeseries* of our *current earnings* and use this data to *forecast* future earnings.

14. Start by duplicating **the third section**. Since we need a timeseries we use a *makeTimeseries* command instead of *summarize*. Just like summarize you can also click a column and click on “Convert to timeseries”. In this case click on the **“oldValue**” column and click on **convert to timeseries**.

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| filter actionType == "collectfee"
| makeTimeseries avg(oldValue), time:{timestamp}
```

![Make Timeseries on Earnings](../../assets/images/MakeTimeSeriesEarning.png)

We have the required result but in the wrong visualization format. We would like to see a graph instead of listing all of the data points.

15. To achieve this, click on **options** to select **visualizations** as we had seen previously. In the visualization panel select “**Line chart**”  

![Line Chart of Earnings](../../assets/images/LineChartEarnings.png)

16.	Now that we have a timeseries, we can ask *Davis AI* to create a *forecast*. To achieve this, click on “**Options**” again and in the visualization panel scroll to the bottom of the list to find **Davis AI**, click it.  

![Davis AI](../../assets/images/DavisAI.png)

17.	Click the **check mark** to activate the analyser, and select “**forecast**” to predict (instead of  baseline). Forecast will predict the future based on the original data set.  Hit **Run** to chart.  

![Forecast](../../assets/images/Forecast.png)
 
Change the time frame from 100 to “**120**” to reflect the same time period of two hours. You can increase your time prediction to max 600 min.  

![Time Period for Forecast](../../assets/images/2HourPrediction.png)

Now we have a forecast of our earnings

Now we can see the forecast of our earnings for the next hours.

Let’s answer the sixth question - *What are the top 5 accounts with the most money going out?*  

To achieve this we need to track *withdrawals* and calculate the sum of *valueChange*

18.	Duplicate the very **first section** to get started, and hit “**Run**”. 
19.	Now we need to edit the **actionType** filter value to “**withdraw**” and add the summarize command for **sum(valueChange)** and sort it in an **ascending** order. You can use **the below query** to achieve the result

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| filter actionType == "withdraw"
| summarize sum(valueChange), by:{accountId}
| sort `sum(valueChange)` asc
| limit 5
 ```

Now we can see these are our top five accounts who have the most money going out in our time frame

![Top 5 Accounts](../../assets/images/top5Accounts.png)

Let’s now answer the seventh question - *How much money is traded long compared to market trades?*
To answer this question, we’re going to look at actionTypes, “sell” or “buy” transactions. 
 
20.	Duplicate the very first section again, and hit “**Run**”. Now we need to edit the script by filtering with 

```DQL
contains(content, “sell”) or contains(content, “buy”). 
```

Hit **Run** to reveal the results. 

![Buys and Sells](../../assets/images/BuysAndSells.png)
 
Now we can see all transactions related to *buying* and *selling*, but we want to filter for a particular transaction types. To achieve this simply summarize your data by **value** change count and by “**actionType**”

Below is the final query

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| filter contains(content,"sell") or contains(content,"buy")
| summarize sum(valueChange), by:{actionType}
```
 
![Summarize by Action type 2](../../assets/images/SummarizebyActionType2.png)

21.	To improve visualisation simply go to options and change the visualization to a **pie chart**. Use the “**color**” feature to separate sells from buys (all types):  

![Pie Chart Color Change](../../assets/images/PieChartColors.png)

Our final question to answer: **How much money is deposited vs withdrawn?**

22.	Just as with each section, duplicate our very **first section** and hit “**Run**”. Now we need to edit the query and filter for actionType “**withdrawal**” or “**deposit**”. Once we have filtered the data, the next step is to create a **timesries**. Do this by click **valueChange** column and “**convert to timeseries**” 

You can use the below query

```DQL
fetch logs
| filter k8s.namespace.name == "easytrade" and k8s.container.name == "brokerservice"
| filter contains(content, "balance")
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange
| filter actionType == "deposit" or actionType == "withdraw"
| makeTimeseries sum(valueChange), time:{timestamp}, by:{actionType}
 ```

![Value Change of Withdrawal and Deposits](../../assets/images/ValueChangeWithdrawalDeposit.png)

23.	Change the visualization to bar chart get better insights. Click on “**Options**” and choose **bar chart under** visualization.   

![Bar Chart Visualization](../../assets/images/BarChartValueChange.png)

### Step 3: Adding data to a dashboard

Now that we have created a notebook with multiple business insights we can see how to add some of them to the dashboard.

24.	Select the **first section** of your notebook with data about deposits. To place it in a dashboard, click on the **elipses** (three dots) in the top menu and select the “**open with**” option. In the following pop-up, select **Dashboard**.
25.	You can rename the dashboard to whatever you’d like. Click the **pencil icon** to edit the dashboard tile data  

![Edit Icon](../../assets/images/EditIcon.png)

Click the **visualize** tab to showcase the same visualizations available as in notebooks and select **single value** for impactful numbers and have a more readable format. 

![Single Value Deposit Count](../../assets/images/SingleValueDepositCount.png)
 
Now you have learnt how to place the data from a notebook into a dashboard. Try to create more tiles for the other sections you have in your notebook.

**You have successfully completed the lab!**
