# AB-Testing-for-a-Local-SPA-using-Alteryx
<br>
We are part of a team, looking to increase the gross margin for a chain of spas. We have decided to discount the price of facials, to see its effect on the gross margin. Currently the store offers facials at $98.99, and we decide to run 2 promotional campaigns, one of them would lead to a group of stores offering facials at $76.99 and another group offering facials at $87.99.<br><br>

<b>The project compromises of 3 parts</b><br>
1. Select treatment and control unit
2. Prepare & run the test
3. Analyze the result


<b>Selection of treatment and control units -</b><br>
For our experiment, our we have selected the treatment units, and all the other stores(control units) will continue to offer facials at the same old price of $98.99.
We will match these treatment units to control units based on both continuous variables and discreet variables, but the process of matching them will differ.
For the selection of variables to used for matching, we start by looking at the list of all the variables present. They are as follows - <br>
a. Invoice Number<br>
b. Invoice Date <br>
c. SKU<br>
d. SKU Category<br>
e. Gross margin<br>
f. Sales<br>
g. Store id<br>
h. Region<br>

We start by matching the discreet variables. You cannot match discreet variables directly like continuous variables. You need to match stores which have the same common value of the discreet variable. In our case, we can match stores which belong to the same region. This is backed up the assumption that customer behavior varies by region.<br>

Now lets move on to matching stores based on continuous trends. To form a fair matching between stores, we will look at 2 measures.<br>
a. Number of invoices per week for a store (This gives us the foot traffic per week in a store)<br>
b. Total sales per store per week (This talks about how much revenue is the store generating)<br>
<b>Note:</b> These measures are not avalable directly in our data, and will have to calcualted using transformation and summarization<br><br>

<b>Prepare and run the test</b><br>
It is important to run the test for atleast cycle.An average customer visits the store once every 10 weeks. Therefore in our case the cycle duration is 10 weeks.We will be running the test from March 20th to May 28th, which is exactly 70 days or 10 weeks.<br>

<b>The below workflow can be found in Data_preprocessing_1</b><br>
Lets start by preparing the data. The sales data files has transaction from January 1st 2013 to  June 29th 2014.
Similarly we have another file, listing the treatment units(stores which modify the price of their facials), and the discounted price these stores would sell their facials for either $76.99 or $87.99.

We need to prepare the data so the AB Trend tool, can compare control units to the pre-selected treatment units we have.<br>
From the sales data file, we start by filtering data from February 7th 2013 to May 28th 2014. The time duration over here is 476 days or 68 weeks. Out of these 68 weeks 52 weeks(1 year) + 6 weeks(6 periods) will be used by the trend tool to form measures of trends and seasonality for matching control units to treatment units. The remaining 10 weeks is the duration for which the experiment was run.<br>

Next, we enrich the data, by calculating the following measures for which record - week number, week start date, week end date, facial flag(if the transaction was for a facial).
After the we aggregate the number of distinct weeks for each store. We then filter out stores with less than 68 weeks of data. We then join these stores which have 68 weeks of data to our earlier stream to filter out the transactions of the stores with less than 68 weeks of data<br>

next, we aggregate by invoice number and get the sum of sales, sum of gross margin,number of facials for each invoice. This data is further aggregated by store and week to get the total number of invoices per week. Hence giving us the weekly store traffic for all stores. This data is stored into a seperate file called Weekly_store_traffic.


Next, we draw the earlier sales data grouped by index and filter out the invoices without any facials.After this we aggregate the data by store and region to get the number of facials per store. We further join this data with the treatment units list that we have, to get the number of facials for these treatment units. At this stage we also enrich the data with the price level used at the treatment units in the experiment duration. The data that was not joined, i.e. stores part of the control group are marked as 'Control unit' and then further unioned to get a combined store list. This data is stored in a seperate file called store_list.<br><br>

<b>The below pipeline can be found in pipeline_for_unit_matching file</b><br>
Now that our data is controlled and organized, we would move on to make control treatment store pairs. The measures used for matching were the foot traffic in stores and the sales trends for these stores.

We start by pushing the data from weekly_store_traffic to the AB trend tool, the output of the AB Trend tool is score measures for each store for trend and seasonality.
This data is joined with the store_list file from before to give us a table with store, trend score, seasonality score, region, and test group.<br>

The next part is to divide this data into streams bifurcated by region(East, West and MidWest). Each of these streams is then fed into a AB Control module, which gives us a list of table matching treatment unit and control unit, and the statistical distance between them. This data is further enriched with the test_group field, signiying the group the treatment unit belongs to ($76.99 or $87.99) along with the region both the control and treatment unit belong to.<br><br>

<b>Analyze the results</b><br>
<b>The pipeline for the analysis can be found in the final analysis file</b><br>
To analyze the results we use the AB analysis module. The control treatment pairs file is divided into 2 parts, one with the treatment units belonging to the $87.99 group, and the other belonging to $76.99 group, these 2 groups are fed into the AB analysis module, which further takes input from the Store_sales_analysis file from before.<br>
Based on the results, we observed that there was a positive percentage change of 12.2% in gross margin for the treatment group($87.99) for the test period as compared to the comparison period. Where as there was a 24.6% decline for the control group in the test period as compared to the comparison period.<br>
Coming to $79.99 treatment group, there was a 20.4% decline for the treatment units between the test period and the comparison period, where as the decline for control units was 18.7%.<br>
Based on this we can say that it would be advisable & profitable to shift the prices to $87.99, but the same can't be inferred for shifting the price to &79.99.   
  
