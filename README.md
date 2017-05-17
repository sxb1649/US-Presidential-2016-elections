# US-Presidential-2016-elections
import pandas as pd
import numpy as np
from pandas import Series,DataFrame
from numpy.random import randn
from scipy import stats
import matplotlib as mlp
import matplotlib.pyplot as plt
import seaborn as sns
from pandas_datareader import data
import pandas_datareader.data as web
# For time stamps
from datetime import datetime
# For division
from __future__ import division
import requests
#important
from io import StringIO


# Use to grab data from the web(HTTP capabilities)
import requests

# We'll also use StringIO to work with the csv file, the DataFrame will require a .read() method
from io import StringIO

# This is the url link for the poll data in csv form
url = "http://elections.huffingtonpost.com/pollster/2012-general-election-romney-vs-obama.csv"

# Use requests to get the information in text form
source = requests.get(url).text

# Use StringIO to avoid an IO error with pandas
poll_data = StringIO(source) 

#import CSV

# Set poll data as pandas DataFrame
poll_df=pd.read_csv(poll_data)

# Let's get a glimpse at the data
poll_df.info()

# Preview DataFrame
poll_df.head()

#Factorplot the Affiliation
sns.factorplot('Affiliation',data=poll_df,kind='count',order=['Dem','Rep','None'],color='indianred')
plt.show()

Looks like we are overall relatively neutral, but still leaning towards Democratic Affiliation, it will be good to keep this in mind. 
Let's see if sorting by the Population hue gives us any further insight into the data.

# Factorplot the affiliation by Population
sns.factorplot('Affiliation',data=poll_df,kind='count',hue='Population',order=['Dem','Rep','None'],color='indianred')
plt.show()

Looks like we have a strong showing of likely voters and Registered Voters,
so the poll data should hopefully be a good reflection on the populations polled. 
Let's take another quick overview of the DataFrame.

# Preview DataFrame
poll_df.head()

Let's go ahead and take a look at the averages for Obama, Romney , and the polled people who remained undecided.

#get poll avgs and drop Number of Observations    
avg = pd.DataFrame(poll_df.mean())
avg.drop('Number of Observations', axis=0, inplace=True)
avg

#get poll erros and drop row of Number of Observations
std = pd.DataFrame(poll_df.std())
std.drop('Number of Observations', axis=0, inplace=True)
std

#ExtraInformation
#axis : {0 or ‘index’, 1 or ‘columns’}, default 0
#0 or ‘index’: apply function to each column
#1 or ‘columns’: apply function to each row

#Theory
{dataFrame.set_index(keys, drop=True, append=False, inplace=False, verify_integrity=False)
Set the DataFrame index (row labels) using one or more existing columns. By default yields a new object.

Parameters:	
keys : column label or list of column labels / arrays
drop : boolean, default True
Delete columns to be used as the new index
append : boolean, default False
Whether to append columns to existing index
inplace : boolean, default False
Modify the DataFrame in place (do not create a new object)
verify_integrity : boolean, default False
Check the new index for duplicates. Otherwise defer the check until necessary. Setting to False will improve the performance of this method
Returns:	
dataframe : DataFrame
}

# now plot using pandas built-in plot, with kind='bar' and yerr='std'
avg.plot(yerr=std,kind='bar',legend=False)
#Yerr=Variable error on Y-axis

avg.plot(xerr=std,kind='bar',legend=False)
#Yerr=Variable error on X-axis

(y -vertical line on the bar graph, x- horizontal line on the bar graph)

{‘bar’ or ‘barh’ for bar plots
‘hist’ for histogram
‘box’ for boxplot
‘kde’ or 'density' for density plots
‘area’ for area plots
‘scatter’ for scatter plots
‘hexbin’ for hexagonal bin plots
‘pie’ for pie plots
#http://pandas.pydata.org/pandas-docs/version/0.18.0/visualization.html}

Interesting to see how close these polls seem to be, 
especially considering the undecided factor. Let's take a look at the numbers.

# Concatenate our Average and Std DataFrames
poll_avg = pd.concat([avg,std],axis=1)

#Rename columns
poll_avg.columns = ['Average','STD']

#Show
poll_avg.head()


Looks like the polls indicate it as a fairly close race, but what about the undecided voters?
Most of them will likely vote for one of the candidates once the election occurs. 
If we assume we split the undecided evenly between the two candidates the observed difference 
should be an unbiased estimate of the final difference.

# Take a look at the DataFrame again
poll_df.head()

If we wanted to, we could also do a quick (and messy) time series
analysis of the voter sentiment by plotting Obama/Romney favor versus the Poll End Dates.
Let's take a look at how we could quickly do tht in pandas.

# Quick plot of sentiment in the polls versus time.(http://text-processing.com/demo/sentiment/)
poll_df.plot(x='End Date',y=['Obama','Romney','Undecided'],marker='o',linestyle='')

While this may give you a quick idea, go ahead and try creating a new DataFrame or editing poll_df to make a better visualization of the above idea!

To lead you along the right path for plotting, we'll go ahead and answer another question related to plotting the sentiment versus time. 
Let's go ahead and plot out the difference between Obama and Romney and how it changes as time moves along. 
Remember from the last data project we used the datetime module to create timestamps, let's go ahead and use it now.
# For timestamps
from datetime import datetime

Now we'll define a new column in our poll_df DataFrame to take into account the difference between Romney and Obama in the polls.


# Create a new column for the difference between the two candidates
poll_df['Difference'] = (poll_df.Obama - poll_df.Romney)/100
# Preview the new column
poll_df.head()

# Set as_index=Flase to keep the 0,1,2,... index. Then we'll take the mean of the polls on that day.
poll_df = poll_df.groupby(['Start Date'],as_index=False).mean()

# Let's go ahead and see what this looks like
poll_df.head() 
#{as_index= False, will put indexes,if true no index like 0,1,2,3..}

# Set as_index=Flase to keep the 0,1,2,... index. Then we'll take the mean of the polls on that day.
poll_df = poll_df.groupby(['Start Date'],as_index=False).mean()

# Let's go ahead and see what this looks like
poll_df.head()

# Plotting the difference in polls between Obama and Romney
fig = poll_df.plot('Start Date','Difference',figsize=(12,4),marker='o',linestyle='-',color='purple')
plt.show()

It would be very interesting to plot marker lines on the dates of the debates and see if there is any general insight to the poll results.
The debate dates were Oct 3rd, Oct 11, and Oct 22nd. Let's plot some lines as markers and then zoom in on the month of October.
In order to find where to set the x limits for the figure we need to find out where the index for the month of October in 2012 is. 
Here's a simple for loop to find that row. Note, the string format of the date makes this difficult to do without using a lambda expression or a map.

# Set row count and xlimit list
row_in = 0
xlimit = []

# Cycle through dates until 2012-10 is found, then print row index
for date in poll_df['Start Date']:
    if date[0:7] == '2012-10':
        xlimit.append(row_in)
        row_in +=1
    else:
        row_in += 1
        
print min(xlimit)
print max(xlimit)

#Great now we know where to set our x limits for the month of October in our figure.

# Start with original figure
fig = poll_df.plot('Start Date','Difference',figsize=(12,4),marker='o',linestyle='-',color='purple',xlim=(329,356))

# Now add the debate markers
plt.axvline(x=329+2, linewidth=4, color='grey')
plt.axvline(x=329+10, linewidth=4, color='grey')
plt.axvline(x=329+21, linewidth=4, color='grey')

Surprisingly, thse polls reflect a dip for Obama after the second debate against Romney, 
even though memory serves that he performed much worse against Romney during the first debate.

For all these polls it is important to remeber how geographical location can effect the value of a poll 
in predicting the outcomes of a national election.


Let's go ahead and switch gears and take a look at a data set consisting of information on donations to the federal campaign.

This is going to be the biggest data set we've looked at so far. You can download it here , 
then make sure to save it to the same folder your iPython Notebooks are in.

The questions we will be trying to answer while looking at this Data Set is:

1.) How much was donated and what was the average donation?
2.) How did the donations differ between candidates?
3.) How did the donations differ between Democrats and Republicans?
4.) What were the demographics of the donors?
5.) Is there a pattern to donation amounts?

drop_df=pd.read_csv('C:\\Users\\Sparsh\\Documents\\MS\\First_Sem\\P_Data\\Election_Donor_Data.csv')
#There will be a warning og loading a high amount of database into the sysytem, to avoid this warning we need to:
#(donor_df=pd.read_csv('C:\\Users\\Sparsh\\Documents\\MS\\First_Sem\\P_Data\\Election_Donor_Data.csv',low_memory=False)

# Set the DataFrame as the csv file
donor_df = pd.read_csv('Election_Donor_Data.csv')

# Get a quick overview
donor_df.info()

# let's also just take a glimpse
donor_df.head()

# Get a quick look at the various donation amounts
donor_df['contb_receipt_amt'].value_counts()
#8079 different amounts! Thats quite a variation. Let's look at the average and the std.

# Get the mean donation
don_mean = donor_df['contb_receipt_amt'].mean()

# Get the std of the donation
don_std = donor_df['contb_receipt_amt'].std()

print (('The average donation was %.2f with a std of %.2f')%(don_mean,don_std))
#The average donation was 298.24 with a std of 3749.67

#Wow! That's a huge standard deviation! Let's see if there are any large donations or 
other factors messing with the distribution of the donations.

# Let's make a Series from the DataFrame, use .copy() to avoid view errors
top_donor = donor_df['contb_receipt_amt'].copy()

# Now sort it
top_donor.sort()

__main__:1: FutureWarning: sort is deprecated, use sort_values(inplace=True) for INPLACE sorting
#To remove Warning
#top_donor.sort(inplace=True)
top_donor.sort(inplace=False)

Looks like we have some negative values, as well as some huge donation amounts! The negative values are due to the
FEC recording refunds as well as donations, let's go ahead and only look at the positive contribution amounts.

# Get rid of the negative values
top_donor = top_donor[top_donor >0]

# Sort the Series
top_donor.sort()

# Look at the top 10 most common donations value counts
top_donor.value_counts().head(10)

Here we can see that the top 10 most common donations ranged from 10 to 2500 dollars.

A quick question we could verify is if donations are usually made in round number amounts? (e.g. 10,20,50,100,500 etc.) 
We can quickly visualize this by making a histogram and checking for peaks at those values. 
Let's go ahead and do this for the most common amounts, up to 2500 dollars.


# Create a Series of the common donations limited to 2500
com_don = top_donor[top_donor < 2500]

# Set a high number of bins to account for the non-round donations and check histogram for spikes.
com_don.hist(bins=100)

Looks like our intuition was right, since we spikes at the round numbers.

Let's dive deeper into the data and see if we can seperate donations by Party, in order to do this
we'll have to figure out a way of creating a new 'Party' column.
We can do this by starting with the candidates and their affliliation. Now let's go ahead and get a list of candidates.

# Grab the unique object from the candidate column
candidates = donor_df.cand_nm.unique()
#Show
candidates

Let's go ahead and seperate Obama from the Republican Candidates by adding a Party Affiliation column. 
We can do this by using map along a dictionary of party affiliations. Lecture 36 has a review of this topic.

# Dictionary of party affiliation
party_map = {'Bachmann, Michelle': 'Republican',
           'Cain, Herman': 'Republican',
           'Gingrich, Newt': 'Republican',
           'Huntsman, Jon': 'Republican',
           'Johnson, Gary Earl': 'Republican',
           'McCotter, Thaddeus G': 'Republican',
           'Obama, Barack': 'Democrat',
           'Paul, Ron': 'Republican',
           'Pawlenty, Timothy': 'Republican',
           'Perry, Rick': 'Republican',
           "Roemer, Charles E. 'Buddy' III": 'Republican',
           'Romney, Mitt': 'Republican',
           'Santorum, Rick': 'Republican'}

# Now map the party with candidate
donor_df['Party'] = donor_df.cand_nm.map(party_map)

A quick note, we could have done this same operation manually using a for loop,
however this operation would be much slower than using the map method.

'''
for i in xrange(0,len(donor_df)):
    if donor_df['cand_nm'][i] == 'Obama,Barack':
        donor_df['Party'][i] = 'Democrat'
    else:
        donor_df['Party'][i] = 'Republican'
''' #To do this manually.
#But this would have slowed down the process for a long time.
