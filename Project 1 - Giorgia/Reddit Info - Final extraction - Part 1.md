

```python
#import dependencies
import praw
import pandas as pd
import numpy as np
import seaborn as sns
import time
import datetime
import requests
import requests.auth
import matplotlib.pyplot as plt
import matplotlib.dates as md
import matplotlib.units as units
import matplotlib.patches as patches
from matplotlib.finance import date2num
from scipy import stats

#importing my keys and secrets
from config import client_id, client_secret, user_agent

#importing vader
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

pd.set_option('display.max_colwidth', -1)
```

    C:\Users\Giorgia\Anaconda3\lib\site-packages\matplotlib\cbook\deprecation.py:106: MatplotlibDeprecationWarning: The finance module has been deprecated in mpl 2.0 and will be removed in mpl 2.2. Please use the module mpl_finance instead.
      warnings.warn(message, mplDeprecation, stacklevel=1)
    


```python
#set up credentials
reddit = praw.Reddit(client_id = client_id, client_secret = client_secret, user_agent = user_agent)
```


```python
print(reddit.read_only)
```

    True
    


```python
subreddits = ['all'] #'FoodAllergies', "ChicagoFood", 'GlutenFree', ]

searched_keyword = []
subreddit_name = []
subreddit_id = []
comment_id = []
created = []
title = []
score = []
comments_number = []
likes = []
permalink = []
domain = []

counts = 0

for name in subreddits:
    
    subreddit = reddit.subreddit(name)#.submissions(1420070400, 1514764839)
    print(subreddit)

    key_words = ['Chipotle Mexican Grill', 'McDonalds', 'Subway', 'Taco Bell', 'Wendy’s']
    
    for key_word in key_words:
        
        reddits = subreddit.search(str(key_word), limit=50000)

        for submission in reddits:

            searched_keyword.append(key_word)
            subreddit_name.append(name)
            subreddit_id.append(submission.subreddit_id)
            comment_id.append(submission.id)
            #print(counts, submission.id)
            created.append(submission.created)
            title.append(submission.title)
            score.append(submission.score)
            comments_number.append(submission.num_comments)
            likes.append(submission.likes)
            permalink.append(submission.permalink)
            domain.append(submission.domain)
            
            counts += 1

        
    print(subreddit.search)
    print(submission.created_utc)
    
print(counts)
```

    all
    <bound method Subreddit.search of Subreddit(display_name='all')>
    1408706013.0
    1826
    


```python
subreddit_info = {"Keyword" : searched_keyword, "Subreddit Name" : subreddit_name, "Subreddit ID" : subreddit_id, 
                  "Submission ID" : comment_id, 'Submission Created Date': created, "Submission Title" : title, 
                  "Submission Score" : score, "Submission Comments #" : comments_number, "Submission Likes" : likes, 
                  "Submission Permalink" : permalink, "Domain" : domain}

subreddit_info_df = pd.DataFrame(subreddit_info, columns = ["Keyword", "Subreddit ID", "Subreddit Name", "Submission ID", 
                                                            "Submission Created Date", "Submission Title", "Submission Score", 
                                                            "Submission Comments #","Submission Likes", "Submission Permalink",
                                                            "Domain"])

#subreddit_info_df['Submission Created Date'] = pd.to_datetime(subreddit_info_df['Submission Created Date'],unit='s').dt.date

print(subreddit_info_df.shape)
print(subreddit_info_df['Subreddit Name'].value_counts())
```

    (1826, 11)
    all    1826
    Name: Subreddit Name, dtype: int64
    


```python
subreddit_info_df.head()
#subreddit_info_df.tail()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Keyword</th>
      <th>Subreddit ID</th>
      <th>Subreddit Name</th>
      <th>Submission ID</th>
      <th>Submission Created Date</th>
      <th>Submission Title</th>
      <th>Submission Score</th>
      <th>Submission Comments #</th>
      <th>Submission Likes</th>
      <th>Submission Permalink</th>
      <th>Domain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Chipotle Mexican Grill</td>
      <td>t5_2qgzg</td>
      <td>all</td>
      <td>38wus9</td>
      <td>1.433717e+09</td>
      <td>"Chipotle Mexican Grill Inc. will expand benefits formerly reserved for salaried workers, including full tuition reimbursements, sick pay and paid vacations, to all employees on July 1, executives said Thursday [4 June 2015]."</td>
      <td>1352</td>
      <td>105</td>
      <td>None</td>
      <td>/r/business/comments/38wus9/chipotle_mexican_grill_inc_will_expand_benefits/</td>
      <td>nrn.com</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chipotle Mexican Grill</td>
      <td>t5_2qhg1</td>
      <td>all</td>
      <td>12mgrv</td>
      <td>1.352085e+09</td>
      <td>Chipotle Mexican Grill to Test Craft Beer in Chicago</td>
      <td>574</td>
      <td>74</td>
      <td>None</td>
      <td>/r/beer/comments/12mgrv/chipotle_mexican_grill_to_test_craft_beer_in/</td>
      <td>thedailymeal.com</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chipotle Mexican Grill</td>
      <td>t5_2th52</td>
      <td>all</td>
      <td>6dtzc2</td>
      <td>1.496010e+09</td>
      <td>Almost ALL Chipotle Mexican Grill were infected with malware! #OldNews but $CMG hasn't reacted to the fact, Its time to short to $300 vm/</td>
      <td>335</td>
      <td>55</td>
      <td>None</td>
      <td>/r/wallstreetbets/comments/6dtzc2/almost_all_chipotle_mexican_grill_were_infected/</td>
      <td>theverge.com</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chipotle Mexican Grill</td>
      <td>t5_37cer</td>
      <td>all</td>
      <td>5w4a3q</td>
      <td>1.488062e+09</td>
      <td>Chipotle Mexican Grill - Chicken Burritos (recipe in comments)</td>
      <td>212</td>
      <td>26</td>
      <td>None</td>
      <td>/r/MealPrepSunday/comments/5w4a3q/chipotle_mexican_grill_chicken_burritos_recipe_in/</td>
      <td>youtube.com</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chipotle Mexican Grill</td>
      <td>t5_2ti4h</td>
      <td>all</td>
      <td>2354qb</td>
      <td>1.397638e+09</td>
      <td>I work as a cashier at Chipotle: Mexican Grill. Today a customer came in and bought his burrito with this. My manger compensated his meal and let me keep the coin. It'll all metal, quite heavy and about the size of a half dollar.</td>
      <td>143</td>
      <td>15</td>
      <td>None</td>
      <td>/r/mildlyinteresting/comments/2354qb/i_work_as_a_cashier_at_chipotle_mexican_grill/</td>
      <td>imgur.com</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1826 entries, 0 to 1825
    Data columns (total 11 columns):
    Keyword                    1826 non-null object
    Subreddit ID               1826 non-null object
    Subreddit Name             1826 non-null object
    Submission ID              1826 non-null object
    Submission Created Date    1826 non-null float64
    Submission Title           1826 non-null object
    Submission Score           1826 non-null int64
    Submission Comments #      1826 non-null int64
    Submission Likes           0 non-null object
    Submission Permalink       1826 non-null object
    Domain                     1826 non-null object
    dtypes: float64(1), int64(2), object(8)
    memory usage: 157.0+ KB
    


```python
subreddit_info_df['Submission Comments #'].sum()
```




    961839




```python
submissionId = []
comment_body = []

for submission_id in subreddit_info_df['Submission ID']:
    
    submission = reddit.submission(id=submission_id)
    
    for comment in submission.comments:
        
        try:
            words = comment.body
            
        except(AttributeError):
            
            continue
        
        submissionId.append(submission_id)
        comment_body.append(comment.body)  
```


```python
print(len(submissionId))
print(len(comment_body))

comments = {"Submission ID" : submissionId, "Comment" : comment_body}

comments_df = pd.DataFrame(comments, columns = ['Submission ID', 'Comment'])

print(comments_df.shape)
```

    120830
    120830
    (120830, 2)
    


```python
comments_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Submission ID</th>
      <th>Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>38wus9</td>
      <td>I'm reading this on my way to Chipotle. I guess they really need me to buy that Guac now, huh?</td>
    </tr>
    <tr>
      <th>1</th>
      <td>38wus9</td>
      <td>[deleted]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>38wus9</td>
      <td>This is the best tl;dr I could make, [original](http://nrn.com/hr-training/chipotle-expand-benefits-hourly-employes) reduced by 81%. (I'm a bot)\n*****\n&gt; JD Cummings, recruitment strategy manager at Denver-based Chipotle, and William Espey, brand voice lead in Chipotle&amp;#039;s marketing division, made the announcement during a presentation at the annual Summer Brand Camp marketing and human resources conference, sponsored by Dallas-based analytics firm TDn2K.\n\n&gt; &amp;quot;We just made an announcement internally that we are now going to be offering sick pay and paid vacation time for all employees at all levels of the company, including all entry-level employees&amp;quot; Cummings said.\n\n&gt; Cummings said the move reflects Chipotle&amp;#039;s commitment to provide career paths for its workers.\n\n\n*****\n[**Extended Summary**](http://np.reddit.com/r/autotldr/comments/38wvu6/chipotle_mexican_grill_inc_will_expand_benefits/) | [FAQ](http://np.reddit.com/r/autotldr/comments/31b9fm/faq_autotldr_bot/ "Version 1.5, ~4285 tl;drs so far.") | [Theory](http://np.reddit.com/r/autotldr/comments/31bfht/theory_autotldr_concept/) | [Feedback](http://np.reddit.com/message/compose?to=%23autotldr "PMs and comment replies are read by the bot admin, constructive feedback is welcome.") | *Top* *five* *keywords*: **employees**^#1 **Cummings**^#2 **Chipotle**^#3 **year**^#4 **going**^#5\n\nPost found in [/r/business](/r/business/comments/38wus9/chipotle_mexican_grill_inc_will_expand_benefits/), [/r/Chipotle](/r/Chipotle/comments/38t8kw/chipotle_expanding_benefits_college_reimbursement/) and [/r/Stuff](/r/Stuff/comments/38t8z8/chipotle_expanding_benefits_college_reimbursement/).</td>
    </tr>
    <tr>
      <th>3</th>
      <td>38wus9</td>
      <td>So if I get a part time job making burritos they'll cover 100% tuition?</td>
    </tr>
    <tr>
      <th>4</th>
      <td>38wus9</td>
      <td>This is GREAT. I am often one of the oldest customers, but I'm always treated well by the many young workers - And I don't know how they can look at that much beans all day and still be cheerful. They deserve the best benefits. Honestly. There's no tip jar, so if this claim is true, I like Chipotle even MORE than I did before.</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_comments_df = pd.merge(subreddit_info_df, comments_df, on="Submission ID", how="outer")
```


```python
subreddit_info_comments_df = subreddit_info_comments_df.sort_values(by=['Keyword','Submission Created Date'], ascending = False)

subreddit_info_comments_df.to_csv("Reddits Submissions with Comments - Part 1.csv", encoding="utf-8", index=False)
```


```python
compound_list = []
positive_list = []
negative_list = []
neutral_list = []

for comment in subreddit_info_comments_df["Comment"]:

    # Run Vader Analysis on each tweet
    sentiment_analysis = analyzer.polarity_scores(str(comment))

    #print(sentiment_analysis)
    compound_list.append(sentiment_analysis["compound"])
    positive_list.append(sentiment_analysis["pos"])
    negative_list.append(sentiment_analysis["neg"])
    neutral_list.append(sentiment_analysis["neu"])

#add the sentiment analysis columns into the dataframe
subreddit_info_comments_df["Compound Score"] = compound_list
subreddit_info_comments_df["Positive Score"] = positive_list
subreddit_info_comments_df["Negative Score"] = negative_list
subreddit_info_comments_df["Neutral Score"] = neutral_list
```


```python
subreddit_info_comments_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Keyword</th>
      <th>Subreddit ID</th>
      <th>Subreddit Name</th>
      <th>Submission ID</th>
      <th>Submission Created Date</th>
      <th>Submission Title</th>
      <th>Submission Score</th>
      <th>Submission Comments #</th>
      <th>Submission Likes</th>
      <th>Submission Permalink</th>
      <th>Domain</th>
      <th>Comment</th>
      <th>Compound Score</th>
      <th>Positive Score</th>
      <th>Negative Score</th>
      <th>Neutral Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>121251</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>holy fuck it's fire\n</td>
      <td>-0.7096</td>
      <td>0.000</td>
      <td>0.747</td>
      <td>0.253</td>
    </tr>
    <tr>
      <th>121252</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>The linked tweet was tweeted by [@Wendys](https://twitter.com/Wendys) on Mar 23, 2018 16:34:18 UTC\n\n-------------------------------------------------\n\n[@vlonemanuel](https://twitter.com/vlonemanuel) [https://open.spotify.com/album/5bBpYxgZ1zm89a6GlnuQHu](https://open.spotify.com/album/5bBpYxgZ1zm89a6GlnuQHu)\n\n-------------------------------------------------\n\n^• Beep boop I'm a bot • Find out more about me at /r/tweettranscriberbot/ •</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>1.000</td>
    </tr>
    <tr>
      <th>121253</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>Still not as good as the hamburger helper mixtape</td>
      <td>-0.0015</td>
      <td>0.203</td>
      <td>0.204</td>
      <td>0.593</td>
    </tr>
    <tr>
      <th>121254</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>Idontwanttoliveonthisplanetanymore.jpg</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>1.000</td>
    </tr>
    <tr>
      <th>121536</th>
      <td>Wendy’s</td>
      <td>t5_2y5dx</td>
      <td>all</td>
      <td>84rz2p</td>
      <td>1.521194e+09</td>
      <td>J.S. Bach - Two Part Invention #1 (played in the style of Wendy Carlos on the Oberheim OB-8)</td>
      <td>2</td>
      <td>1</td>
      <td>None</td>
      <td>/r/Musicthemetime/comments/84rz2p/js_bach_two_part_invention_1_played_in_the_style/</td>
      <td>youtube.com</td>
      <td>It looks like Wendy Carlos has gotten her music off of YouTube. So, instead of linking to a performance of hers, I am linking to something done in her style. At least this fits with the theme by showing how her legacy has already begun. In 1968, Carlos put out *Switched-On Bach*, an LP of Bach's music played on the Moog synthesizer. This was the first great album of all electronic music, and Carlos will be remembered for the pivotal role she played in helping electronic music catch on.</td>
      <td>0.9118</td>
      <td>0.142</td>
      <td>0.000</td>
      <td>0.858</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_comments_df['Created Date'] = subreddit_info_comments_df['Submission Created Date'].apply(lambda x: 
                                                                                                         datetime.datetime.
                                                                                                         fromtimestamp(x).
                                                                                                         strftime('%Y%m%d'))
subreddit_info_comments_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Keyword</th>
      <th>Subreddit ID</th>
      <th>Subreddit Name</th>
      <th>Submission ID</th>
      <th>Submission Created Date</th>
      <th>Submission Title</th>
      <th>Submission Score</th>
      <th>Submission Comments #</th>
      <th>Submission Likes</th>
      <th>Submission Permalink</th>
      <th>Domain</th>
      <th>Comment</th>
      <th>Compound Score</th>
      <th>Positive Score</th>
      <th>Negative Score</th>
      <th>Neutral Score</th>
      <th>Created Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>121251</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>holy fuck it's fire\n</td>
      <td>-0.7096</td>
      <td>0.000</td>
      <td>0.747</td>
      <td>0.253</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>121252</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>The linked tweet was tweeted by [@Wendys](https://twitter.com/Wendys) on Mar 23, 2018 16:34:18 UTC\n\n-------------------------------------------------\n\n[@vlonemanuel](https://twitter.com/vlonemanuel) [https://open.spotify.com/album/5bBpYxgZ1zm89a6GlnuQHu](https://open.spotify.com/album/5bBpYxgZ1zm89a6GlnuQHu)\n\n-------------------------------------------------\n\n^• Beep boop I'm a bot • Find out more about me at /r/tweettranscriberbot/ •</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>1.000</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>121253</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>Still not as good as the hamburger helper mixtape</td>
      <td>-0.0015</td>
      <td>0.203</td>
      <td>0.204</td>
      <td>0.593</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>121254</th>
      <td>Wendy’s</td>
      <td>t5_2xshw</td>
      <td>all</td>
      <td>86v843</td>
      <td>1.521947e+09</td>
      <td>mfw fuckin Wendy's drops their diss track(s) before Ryab</td>
      <td>58</td>
      <td>5</td>
      <td>None</td>
      <td>/r/NLSSCircleJerk/comments/86v843/mfw_fuckin_wendys_drops_their_diss_tracks_before/</td>
      <td>twitter.com</td>
      <td>Idontwanttoliveonthisplanetanymore.jpg</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>1.000</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>121536</th>
      <td>Wendy’s</td>
      <td>t5_2y5dx</td>
      <td>all</td>
      <td>84rz2p</td>
      <td>1.521194e+09</td>
      <td>J.S. Bach - Two Part Invention #1 (played in the style of Wendy Carlos on the Oberheim OB-8)</td>
      <td>2</td>
      <td>1</td>
      <td>None</td>
      <td>/r/Musicthemetime/comments/84rz2p/js_bach_two_part_invention_1_played_in_the_style/</td>
      <td>youtube.com</td>
      <td>It looks like Wendy Carlos has gotten her music off of YouTube. So, instead of linking to a performance of hers, I am linking to something done in her style. At least this fits with the theme by showing how her legacy has already begun. In 1968, Carlos put out *Switched-On Bach*, an LP of Bach's music played on the Moog synthesizer. This was the first great album of all electronic music, and Carlos will be remembered for the pivotal role she played in helping electronic music catch on.</td>
      <td>0.9118</td>
      <td>0.142</td>
      <td>0.000</td>
      <td>0.858</td>
      <td>20180316</td>
    </tr>
  </tbody>
</table>
</div>




```python
groupedKeyword = subreddit_info_comments_df.groupby(['Keyword'], as_index=False)

avgCompound = groupedKeyword["Compound Score"].mean()
avgCompound
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Keyword</th>
      <th>Compound Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Chipotle Mexican Grill</td>
      <td>0.130086</td>
    </tr>
    <tr>
      <th>1</th>
      <td>McDonalds</td>
      <td>0.057964</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Subway</td>
      <td>0.054209</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Taco Bell</td>
      <td>0.063429</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wendy’s</td>
      <td>0.067379</td>
    </tr>
  </tbody>
</table>
</div>




```python
#determining the arrays for the plot
x_axis = np.arange(len(avgCompound["Compound Score"]))
compound_score = avgCompound["Compound Score"]

#creating the ticks for our bar chart's x axis
tick_locations = [value for value in x_axis]

ax = plt.xticks(tick_locations, ['Chipotle Mexican Grill', 'McDonalds', 'Subway', 'Taco Bell', 'Wendy’s'])
plt.bar(x_axis, compound_score, color = ['cyan','g','r','b','y'], alpha=0.5, align="edge")
plt.xlabel("Restaurant Chains")
#ax.set_xticks(compound_score)
plt.ylabel("Comments Polarity")
plt.ylim(-0.3,0.3)
plt.title("Overall Comments Sentiment Based on Reddit (%s)" % (time.strftime("%m/%d/%Y")))

plt.xticks(rotation=45)

sns.set()
plt.grid()
plt.savefig("Overall Comments Sentiment based on Reddit - p1.png", bbox_inches='tight')
plt.show()
```


![png](output_17_0.png)



```python
keyword_data = {}

for key_word in subreddit_info_comments_df['Keyword']:
   
   keyword_data[key_word] = subreddit_info_comments_df[subreddit_info_comments_df['Keyword']== key_word]
```


```python
for kw in keyword_data.keys():
    
    dataframe = keyword_data[kw]
    
    groupedSubmissions = dataframe.groupby(["Keyword", 'Submission ID', 'Created Date'], as_index=False)
    #.mean()#.apply(stats.mode)
    
    avgSubComp = groupedSubmissions["Compound Score"].mean()
    
    avgSubComp['Year'] = pd.to_datetime(avgSubComp['Created Date'], format='%Y%m%d').dt.year
    
    #print(avgSubComp)

   #days = pd.to_datetime(dataframe['Submission Created Date'].astype(str), unit='s') #format='%Y%m%d')
    
    #plt.xticks(rotation=45)
    
    #try:
        
    #ax=plt.gca()
    #xfmt = md.DateFormatter('%Y')
    #ax.xaxis.set_major_formatter(xfmt)
    #ax.xaxis_date()

    #plt.scatter(dataframe['Created Date'], dataframe["Compound Score"], c='c', edgecolor='k', label= kw)
    
    colors = ['cyan','g','r','b','y']
    
    color_count = 0
    
    for color in colors:

        plt.scatter(avgSubComp['Created Date'], avgSubComp["Compound Score"], c= color, edgecolor='k', label= kw)

        #adding legend to the plot
        plt.title("Sentiment Analysis of %s (%s)" % (kw, time.strftime("%m/%d/%Y")))
        plt.xlabel("Comments Year")
        plt.ylabel("Comments Polarity")
        plt.ylim(-1.1,1.1)
        
        color_count +=1
        
    #except(AttributeError, ValueError) as e:
        
        #continue
    
    #plt.scatter(avgSubComp['Created Date'], avgSubComp["Compound Score"], c='c', edgecolor='k', label= kw)
    
    #plt.gca().autofmt_xdate()
    
    plt.savefig("Sentiment Analysis of " + kw + ".png")

    plt.show()
    
    
```


![png](output_19_0.png)



![png](output_19_1.png)



![png](output_19_2.png)



![png](output_19_3.png)



![png](output_19_4.png)



```python
avgSubComp.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Keyword</th>
      <th>Submission ID</th>
      <th>Created Date</th>
      <th>Compound Score</th>
      <th>Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Chipotle Mexican Grill</td>
      <td>10g2tj</td>
      <td>20120925</td>
      <td>0.119775</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chipotle Mexican Grill</td>
      <td>10xzgu</td>
      <td>20121004</td>
      <td>0.000000</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chipotle Mexican Grill</td>
      <td>12mgrv</td>
      <td>20121104</td>
      <td>0.217567</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chipotle Mexican Grill</td>
      <td>19vdl1</td>
      <td>20130307</td>
      <td>0.113458</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chipotle Mexican Grill</td>
      <td>1akwu1</td>
      <td>20130319</td>
      <td>0.354600</td>
      <td>2013</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_comments_df['Year'] = pd.to_datetime(subreddit_info_comments_df['Created Date'], format='%Y%m%d').dt.year

submissions_year_grouped = subreddit_info_comments_df.groupby(["Year",'Keyword'], as_index=False ).mean()
submissions_year_grouped.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Keyword</th>
      <th>Submission Created Date</th>
      <th>Submission Score</th>
      <th>Submission Comments #</th>
      <th>Compound Score</th>
      <th>Positive Score</th>
      <th>Negative Score</th>
      <th>Neutral Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2007</td>
      <td>Wendy’s</td>
      <td>1.178408e+09</td>
      <td>4.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2008</td>
      <td>Chipotle Mexican Grill</td>
      <td>1.224788e+09</td>
      <td>0.333333</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2008</td>
      <td>Subway</td>
      <td>1.227054e+09</td>
      <td>2076.000000</td>
      <td>398.0</td>
      <td>0.125757</td>
      <td>0.135764</td>
      <td>0.083875</td>
      <td>0.780403</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>Wendy’s</td>
      <td>1.215867e+09</td>
      <td>5.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009</td>
      <td>Chipotle Mexican Grill</td>
      <td>1.247322e+09</td>
      <td>13.000000</td>
      <td>6.0</td>
      <td>-0.083800</td>
      <td>0.090000</td>
      <td>0.104667</td>
      <td>0.805333</td>
    </tr>
  </tbody>
</table>
</div>




```python
#splitting the data frame in different sections 'Chipotle Mexican Grill', 'McDonalds', 'Subway', 'Taco Bell', 'Wendy’s'
CMG_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Chipotle Mexican Grill"]
MD_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="McDonalds"]
S_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Subway"]
TB_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Taco Bell"]
W_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Wendy’s"]

#creating the plot for the different mini dataframes
CMG, = plt.plot(CMG_df['Year'], CMG_df['Compound Score'], c='c', label="Chipotle Mexican Grill")
MD, = plt.plot(MD_df['Year'], MD_df['Compound Score'], c='g', label='McDonalds')
S, = plt.plot(S_df['Year'], S_df['Compound Score'], c='r', label='Subway')
TB, = plt.plot(TB_df['Year'], TB_df['Compound Score'], c='b', label='Taco Bell')
W, = plt.plot(W_df['Year'], W_df['Compound Score'], c='m', label='Wendy’s')
```


```python
ymax = submissions_year_grouped['Compound Score'].max()
#print(ymax)

ymin = submissions_year_grouped['Compound Score'].min()
#print(ymin)

#adding legend to the plot
plt.title("Sentiment Analysis of Reddit Comments (%s)" % (time.strftime("%m/%d/%Y")))
plt.xlabel("Year")
#plt.xlim(105,-5)
plt.ylabel("Comment Polarity")
plt.ylim(ymin-0.15,ymax+0.15)
plt.legend(handles=[CMG,MD,S,TB,W],title = 'Restaurants', labels=["Chipotle Mexican Grill", "McDonalds", "Subway", 
                                                                            "Taco Bell", "Wendy’s"], 
           loc="center left",bbox_to_anchor=(1,0.5))

#saving the plot
plt.savefig("Sentiment_Analysis_of_Reddit_Comments_p1.png", bbox_inches='tight')
plt.show()
```


![png](output_23_0.png)

