

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

sns.set()

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

    key_words = ['Domino’s Pizza', 'Burger King', 'Costco', 'Popeyes', 'Panda Express']
    
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
    1376320597.0
    1901
    


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

    (1901, 11)
    all    1901
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
      <td>Domino’s Pizza</td>
      <td>t5_2qqjc</td>
      <td>all</td>
      <td>3712y2</td>
      <td>1.432446e+09</td>
      <td>TIL in the 1980s, Dominos Pizza had a campaign centered around "The Noid". It was discontinued in 1989 after a mentally ill man named Kenneth Noid took Dominos Pizza workers hostage after he thought the ads were a personal attack on him.</td>
      <td>3549</td>
      <td>239</td>
      <td>None</td>
      <td>/r/todayilearned/comments/3712y2/til_in_the_1980s_dominos_pizza_had_a_campaign/</td>
      <td>en.wikipedia.org</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Domino’s Pizza</td>
      <td>t5_2r0z9</td>
      <td>all</td>
      <td>29xpib</td>
      <td>1.404637e+09</td>
      <td>Dominos Pizza Box from the 60s</td>
      <td>6977</td>
      <td>140</td>
      <td>None</td>
      <td>/r/minimalism/comments/29xpib/dominos_pizza_box_from_the_60s/</td>
      <td>i.imgur.com</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Domino’s Pizza</td>
      <td>t5_2x93b</td>
      <td>all</td>
      <td>2fb1jo</td>
      <td>1.409729e+09</td>
      <td>This Domino's Pizza Box from the 60s (Xpost /r/Minimalism)</td>
      <td>1647</td>
      <td>40</td>
      <td>None</td>
      <td>/r/oddlysatisfying/comments/2fb1jo/this_dominos_pizza_box_from_the_60s_xpost/</td>
      <td>i.imgur.com</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Domino’s Pizza</td>
      <td>t5_2qh0u</td>
      <td>all</td>
      <td>4h5ewj</td>
      <td>1.462067e+09</td>
      <td>Domino's pizza box in the 60s</td>
      <td>542</td>
      <td>20</td>
      <td>None</td>
      <td>/r/pics/comments/4h5ewj/dominos_pizza_box_in_the_60s/</td>
      <td>i.imgur.com</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Domino’s Pizza</td>
      <td>t5_2rgdp</td>
      <td>all</td>
      <td>739tdv</td>
      <td>1.506739e+09</td>
      <td>(1980)s Domino's Pizza's Avoid The Noid commercials featuring a claymation, rabbit-man villain type character who hates pizza. Nine commercials on the playlist, varying quality.</td>
      <td>183</td>
      <td>52</td>
      <td>None</td>
      <td>/r/ObscureMedia/comments/739tdv/1980s_dominos_pizzas_avoid_the_noid_commercials/</td>
      <td>youtube.com</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1901 entries, 0 to 1900
    Data columns (total 11 columns):
    Keyword                    1901 non-null object
    Subreddit ID               1901 non-null object
    Subreddit Name             1901 non-null object
    Submission ID              1901 non-null object
    Submission Created Date    1901 non-null float64
    Submission Title           1901 non-null object
    Submission Score           1901 non-null int64
    Submission Comments #      1901 non-null int64
    Submission Likes           0 non-null object
    Submission Permalink       1901 non-null object
    Domain                     1901 non-null object
    dtypes: float64(1), int64(2), object(8)
    memory usage: 163.4+ KB
    


```python
subreddit_info_df['Submission Comments #'].sum()
```




    350863




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

    63404
    63404
    (63404, 2)
    


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
      <td>3712y2</td>
      <td>&gt;After forcing them to make him a pizza and making demands for $100,000, getaway transportation, and a copy of The Widow's Son, Noid surrendered to the police.[2] **After the incident ended, Police Chief Reed Miller offered a memorable assessment to reporters: "He's paranoid.**"  \n\nDamn.</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3712y2</td>
      <td>Huh. Now I know why the Noid went away. It was weird. The ads were everywhere, and suddenly stopped.</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3712y2</td>
      <td>wait...wait... don't tell me</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3712y2</td>
      <td>True Story:\n\nThey had a heavily-advertised promotion where if you ordered a certain amount of food, you'd get a free Noid action figure.\n\nOne day, my brothers and I were being particularly difficult... Mom had asked us to turn off the TV and clean our rooms a couple of times. The next time she came into the room, she said "That's it! I'm getting annoyed!"\n\nGiven all the Domino's TV commercials about the deal, I sort of misheard what she said, and immediately screamed "Can I have it??!"\n\n(This did not reduce Mom's annoyance.)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3712y2</td>
      <td>One of my childhood favorite memories involved the noid.The local Domino's Pizza sent a guy dressed up as the noid to the high school football game in my town. Most of the kids walk around the track during the game and the noid was walking around too.  At some point he was completely surrounded by about 30 kids picking at him.  He began to swing his arms at them and he had these heavy plastic hands and was nailing kids in the head.  Eventually he ran away.</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_comments_df = pd.merge(subreddit_info_df, comments_df, on="Submission ID", how="outer")
```


```python
subreddit_info_comments_df = subreddit_info_comments_df.sort_values(by=['Keyword','Submission Created Date'], ascending = False)

subreddit_info_comments_df.to_csv("Reddits Submissions with Comments - Part 2.csv", encoding="utf-8", index=False)
```


```python
#subreddit_info_comments_df['Submission Created Date'].value_counts()
```


```python
#subreddit_info_comments_df["Comment"]
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
      <th>56911</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>I think i’ve gotten free popeyes like 50+ times this year 😂</td>
      <td>0.7003</td>
      <td>0.420</td>
      <td>0.000</td>
      <td>0.580</td>
    </tr>
    <tr>
      <th>56912</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>As someone who has his own house with a wife... i envy the shit outa this bach pad.  You guys need another roomie?</td>
      <td>0.4404</td>
      <td>0.159</td>
      <td>0.084</td>
      <td>0.757</td>
    </tr>
    <tr>
      <th>56913</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>looks like a cool place to watch the game</td>
      <td>0.5859</td>
      <td>0.444</td>
      <td>0.000</td>
      <td>0.556</td>
    </tr>
    <tr>
      <th>56914</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>Oh boy does this pic make me nostalgic of my College days. Great set up.</td>
      <td>0.6249</td>
      <td>0.227</td>
      <td>0.000</td>
      <td>0.773</td>
    </tr>
    <tr>
      <th>56915</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>This is sick!</td>
      <td>-0.5562</td>
      <td>0.000</td>
      <td>0.642</td>
      <td>0.358</td>
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
      <th>56911</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>I think i’ve gotten free popeyes like 50+ times this year 😂</td>
      <td>0.7003</td>
      <td>0.420</td>
      <td>0.000</td>
      <td>0.580</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>56912</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>As someone who has his own house with a wife... i envy the shit outa this bach pad.  You guys need another roomie?</td>
      <td>0.4404</td>
      <td>0.159</td>
      <td>0.084</td>
      <td>0.757</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>56913</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>looks like a cool place to watch the game</td>
      <td>0.5859</td>
      <td>0.444</td>
      <td>0.000</td>
      <td>0.556</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>56914</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>Oh boy does this pic make me nostalgic of my College days. Great set up.</td>
      <td>0.6249</td>
      <td>0.227</td>
      <td>0.000</td>
      <td>0.773</td>
      <td>20180324</td>
    </tr>
    <tr>
      <th>56915</th>
      <td>Popeyes</td>
      <td>t5_2s5sb</td>
      <td>all</td>
      <td>86q35p</td>
      <td>1.521886e+09</td>
      <td>My roommate made this counter for free Popeyes!</td>
      <td>223</td>
      <td>26</td>
      <td>None</td>
      <td>/r/torontoraptors/comments/86q35p/my_roommate_made_this_counter_for_free_popeyes/</td>
      <td>i.redd.it</td>
      <td>This is sick!</td>
      <td>-0.5562</td>
      <td>0.000</td>
      <td>0.642</td>
      <td>0.358</td>
      <td>20180324</td>
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
      <td>Burger King</td>
      <td>0.051744</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Costco</td>
      <td>0.137831</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Domino’s Pizza</td>
      <td>0.083424</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Panda Express</td>
      <td>0.137742</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Popeyes</td>
      <td>0.081191</td>
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

ax = plt.xticks(tick_locations, ['Domino’s Pizza', 'Burger King', 'Costco', 'Popeyes', 'Panda Express'])
plt.bar(x_axis, compound_score, color = ['cyan','g','r','b','y'], alpha=0.5, align="edge")
plt.xlabel("Restaurant Chains")
#ax.set_xticks(compound_score)
plt.ylabel("Comments Polarity")
plt.ylim(-0.3,0.3)
plt.title("Overall Comments Sentiment Based on Reddit (%s)" % (time.strftime("%m/%d/%Y")))

plt.xticks(rotation=45)

sns.set()
plt.grid()
plt.savefig("Overall Comments Sentiment based on Reddit - p2.png", bbox_inches='tight')
plt.show()
```


![png](output_19_0.png)



```python
keyword_data = {}

for key_word in subreddit_info_comments_df['Keyword']:
   
   keyword_data[key_word] = subreddit_info_comments_df[subreddit_info_comments_df['Keyword']== key_word]
```


```python
#print(keyword_data)
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


![png](output_22_0.png)



![png](output_22_1.png)



![png](output_22_2.png)



![png](output_22_3.png)



![png](output_22_4.png)



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
      <td>Burger King</td>
      <td>10bcum</td>
      <td>20120923</td>
      <td>-0.143856</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Burger King</td>
      <td>10mfie</td>
      <td>20120928</td>
      <td>0.258211</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Burger King</td>
      <td>121spq</td>
      <td>20121025</td>
      <td>-0.129609</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Burger King</td>
      <td>12lwuq</td>
      <td>20121104</td>
      <td>0.105582</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Burger King</td>
      <td>143r41</td>
      <td>20121201</td>
      <td>0.055910</td>
      <td>2012</td>
    </tr>
  </tbody>
</table>
</div>




```python
subreddit_info_comments_df['Year'] = pd.to_datetime(subreddit_info_comments_df['Created Date'], format='%Y%m%d').dt.year

submissions_year_grouped = subreddit_info_comments_df.groupby(["Year",'Keyword'], as_index=False ).mean()
submissions_year_grouped.head()

#avg_compound = pd.DataFrame(submissions_year_grouped['Compound Score'].mean())

#avg_compound('Keyword')['Compound Score'].plot()

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
      <td>Costco</td>
      <td>1.179869e+09</td>
      <td>759.953488</td>
      <td>194.662791</td>
      <td>0.152319</td>
      <td>0.100314</td>
      <td>0.056640</td>
      <td>0.843116</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2007</td>
      <td>Domino’s Pizza</td>
      <td>1.196647e+09</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2008</td>
      <td>Burger King</td>
      <td>1.220786e+09</td>
      <td>1135.124000</td>
      <td>271.868000</td>
      <td>0.044987</td>
      <td>0.123152</td>
      <td>0.087888</td>
      <td>0.788976</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>Costco</td>
      <td>1.215218e+09</td>
      <td>295.000000</td>
      <td>317.000000</td>
      <td>0.168562</td>
      <td>0.128986</td>
      <td>0.040000</td>
      <td>0.830972</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2008</td>
      <td>Popeyes</td>
      <td>1.203041e+09</td>
      <td>185.000000</td>
      <td>97.000000</td>
      <td>-0.017498</td>
      <td>0.109273</td>
      <td>0.099886</td>
      <td>0.790795</td>
    </tr>
  </tbody>
</table>
</div>




```python
#splitting the data frame in different sections
DP_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Domino’s Pizza"]
BK_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Burger King"]
Costco_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Costco"]
Popeyes_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Popeyes"]
PE_df = submissions_year_grouped[submissions_year_grouped["Keyword"]=="Panda Express"]

#creating the plot for the different mini dataframes
DP, = plt.plot(DP_df['Year'], DP_df['Compound Score'], c='c', label="Domino's Pizza")
BK, = plt.plot(BK_df['Year'], BK_df['Compound Score'], c='g', label='Burger King')
Costco, = plt.plot(Costco_df['Year'], Costco_df['Compound Score'], c='r', label='Costco')
Popeyes, = plt.plot(Popeyes_df['Year'], Popeyes_df['Compound Score'], c='b', label='Popeyes')
PE, = plt.plot(PE_df['Year'], PE_df['Compound Score'], c='m', label='Panda Express')

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
plt.legend(handles=[DP,BK,Costco,Popeyes,PE],title = 'Restaurants', labels=["Domino’s Pizza", "Burger King", "Costco", 
                                                                            "Popeyes", "Panda Express"], 
           loc="center left",bbox_to_anchor=(1,0.5))

#saving the plot
plt.savefig("Sentiment_Analysis_of_Reddit_Comments_p2.png", bbox_inches='tight')
plt.show()
```


![png](output_26_0.png)

