---
layout: post
title: "An analysis of the WeRateDogs Twitter account"
date: 2019-03-05
---

For my latest project with the Udacity Data Analyst Nanodegree we were tasked to use our Python wrangling skills to gather and analyze data associated with the [WeRateDogs Twitter page](https://twitter.com/dog_rates?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor).
![image](https://pbs.twimg.com/media/D0gmjyHUYAAf1wV.jpg:large)

This was a fun project for me because

1. I'm in the 99th percentile of general dog enthusiasm. I LOVE dogs, and
2. I learned how to make a twitter api to extra the JSON data associated with each tweet (based on the tweet id).

You can see the Jupyter Notebook that steps through data procurement, wrangling, and visualization [here](https://github.com/eherdter/FunData/tree/master/WeRateDogs_TwitterAnalysis). Unfortunately, you won't be able to actually get the twitter data yourself because that relies on a python function that I've kept to myself because it contains API access tokens etc.

Here's a description of the general wrangling steps.

## Wrangling Methods
I used general assessment techniques to identify quality and tidiness issues. These were `.head()` , `.value_counts()`, `.sample()` etc. I identified eight issues quality issues and two tidiness issues within this master dataset. These were:

### Dirty data issues:

1. Some retweets present (RT @ ...)
2. Some replies to other users present
3. 55 names 'a', 7 names 'an' , 7 names 'the'
4. Some none recordings when a name is available: Index 240: Name is recorded as None when name should be Atlas, Index 1414: Name is recorded as None when should be Thea, Index 122, Index 129
5. Several rows with rating denominators wrong (962, 996, 1400, 2046, 871), 2 rows with rating denominator below 10 (likely mistake)
6. Text and http url are joined together in the text column
7. timestamp and retweeted_status_timestamp datatypes are strs (object)
8. Many unnecessary columns

### Tidiness issues:

1. Datasets in separate dataframes
2. Dogger, floofer, pupper, and puppo seperate columns when should be in one column (stage).
3. Rating is in separate columns (numerator and denominator)

## Quality Issues

To remove the retweets I selected only the rows where the retweet id status was null as such:
`df1 = df1[df1.retweeted_status_id.isnull()]. `  
Then to remove the replies I selected only the rows where the reply to user id was null as such: `[df1.in_reply_to_user_id.isnull()].`  
To inspect issues with the naming I first used the `df1.name.value_counts()`. I found 55 items when name was 'a' which I thought was a mistake. I inspected these rows as such: `df1[df1['name'] == 'a']`.  
I found that these rows had the format of 'This is a floofer named [NAME]' as opposed to 'This is [NAME]'. I selected these rows and looped through each as such:

  new_names = []  
  for index, row in a_new.iterrows():

      string = str(row['text'])
      befor_keyowrd, keyword, after_keyword = string.partition('named')
      name = after_keyword.split('.')[0]
      new_names.append({'index': index, 'name': name})


  new_names = pd.DataFrame(new_names)  
  new_names = new_names.set_index('index')

I created a new dataframe and used `.update()` to update the new names in the master df. I manually replaced the names where the names were first assigned 'an' and 'the' and also manually replaced some names at mentioned index locations. Next, I split the tweet text and http url into two separate columns using `df1.text.str.split('https', expand=True)[1]`. I then added this to an 'https' string to make a url column. To get a column with just text I did nearly the same thing but selected the first item as such [0]. Finally, I changed the time variables which were set to objects to datetimes using `pd.to_datetime().`

## Tidiness Issues

Before I did any cleaning for quality I joined the three dataframes together using the common column of tweet_id.

Next, I collapsed the stage columns and essentially reversed a one-hot encoding-like issue using pandas.series.map to replace the contents to a binary 0/1 format. I then added a column for multi_stage which indicated where more than 1 stage was noted in the tweet. I also added temporary column which I called 'nostage' to make fully expanded dummies set. Then I assigned a new column to the dataframe which I called 'stage' and used .idmax() to assign the stage based on where the max value was present within the dummies for doggo, floofer, pupper, puppo, multi_stage.

Finally, I collapsed the rating numerator and rating denominator into a single column by first converting them to strings and then just pasting them together, separated by a '/' to making a rating column.

## Final Cleaning for Quality
Once I completed the three tidying steps I removed several unnecessary columns. These were:  '`doggo', 'floofer', 'pupper','puppo', 'nostage', 'multi_stage', 'rating_numerator', 'rating_denominator', 'in_reply_to_status_id', 'in_reply_to_user_id','retweeted_status_id','retweeted_status_user_id', 'retweeted_status_timestamp'`

## Results

First off, this a very funny twitter account. If you've never read any of the tweets I highly recommend you do.

After doing some wrangling and reading through many of their hilarious tweets and the submissions I have made a few insights regarding trends in dog naming and general breed type popularity. To note, I wasn't able to fix all of the problems in this dataset and some of the breed predictions might not be 100% accurate so take these results at face value...

First, Golden retrievers, Labrador retrievers, Pembroke corgis, Chihuahuas and Pugs were the top five tweeted about dog breeds. Not surprisingly. These dogs are both extremely cute and loved by many people (Golden retrievers and labs). Also, who can resist the cuteness of a Pembroke corgi!? A dog fit for a queen! [image](https://i.pinimg.com/originals/8d/77/74/8d77740c02d607813c8248906802e57e.jpg)  

Also, chihuahas and pugs. So cute, so weird looking. So much to say about them!

Here's the ranking in terms of tweets about them...

| Breed| Number of Times They Appeared in a Tweet |
| :---: | :---: |
| Golden retriever |137|
| Labrador retrivers | 94|
|Pembroke corgis | 88|
|Chihuahua| 78|
|Pugs| 54|


Also, people have some very creative names for dogs... Take for example, **[Shnuggles](https://t.co/GwvpQiQ7oQ)**
![alt text](https://pbs.twimg.com/media/CVlkid8WoAAqDlB.jpg)

or **[Cilantro](https://twitter.com/dog_rates/status/727685679342333952/photo/1)**
![Image](https://pbs.twimg.com/media/ChlCQg-VIAQ_8g4.jpg)

Some posts are not accompanied with a name but using those with a name I was able to determine the top ten most popular names for dogs. Here they are...

| Name| Number of Times They Appeared in a Tweet |
| :---: | :---: |
| Cooper |10|
| Oliver |10|
|Charlie |10|
|Lucy| 10|
|Penny| 9|
|Tucker|9|
|Sadie|8|
|Winston|8|
|Toby|7|
|Lola|7|

Finally, the most favorited tweet (163518 total!) was in response to this [tweet](https://t.co/7wE9LTEXC4). An unnamed golden retriever doggo that just loves the water!
