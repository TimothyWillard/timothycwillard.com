+++
title = "Why Shy Voters Probably Aren't A Thing To Worry About"
date = 2020-11-01
tags = ["Polotical Polls", "Shy Voters", "2020 Election"]
categories = ["Elections"]
+++

I know it's been a while since I've written anything, but 2020 is an election year and I can't help myself from diving headfirst into election mania. Anybody who knows me knows that I can talk for hours about election science, political polling, and US political history. Today, I plan on taking a deep dive into the "shy Trump voter" hypothesis. For those not familiar with the idea of "shy Trump voters" the basic idea is that there is a group of voters out there that who say they are are undecided or voting for Biden in political polls, but actually plan to vote for Trump and therefore the political polls show that Biden beating Trump by more than he actually is. Before I dive into the analysis I'll just say that most polling experts out there have disputed this theory ([ex: Ariel Edwards-Levy, senior reporter and polling editor at HuffPost Politics, wrote a good Twitter thread on this](https://twitter.com/aedwardslevy/status/1287212700049833984)). Furthermore, if you believe that "shy Trump voters" really had an effect on the 2016 presidential election I recommend that you take a look at [FiveThirtyEight's "The Real Story of 2016" series](https://fivethirtyeight.com/features/the-real-story-of-2016/), which explains that Trump's victory in 2016 is largely attributable to last-minute movement and education becoming a strong predictor of how white voters would vote.

It might be easy for pollsters/modelers/handicappers to cast doubt on a "shy Trump voter" theory in 2020 because the data from previous election cycles don't demonstrate a shy voter effect of any kind, so why should 2020? However, that write off from election experts is probably unsatisfactory for people who buy into the "shy Trump voters" theory. Today I'm going to demonstrate quantitatively why those election experts who don't really consider a "shy Trump voter" are probably right.

Before diving in I want to introduce the idea of bias in political polling. Bias is when a particular polling firm tends to publish polls that either favor Democrats or Republicans. For example, [Monmouth University](https://www.monmouth.edu/polling-institute/) has a [D+1.3 bias according to FiveThirtyEight](https://projects.fivethirtyeight.com/pollster-ratings/), so we would expect Monmouth University to publish polls that are, on average, 1.3 points better for Democrats than the actual state of the race. Bias is not inherently bad, bias can arise for a variety of reasons like a pollster's likely voter screens or a pollster's weighting methodology. Yet, on the whole, we would expect high-quality pollsters to not be biased because polls taken close to the election day should closely predict the outcome of the election.

Keeping that in mind, I'll introduce my big idea for estimating the "shy Trump voter" effect before the election. Pollsters have a historical bias that they've accumulated throughout multiple election cycles and can be compared with the bias that these same polling firms have this election cycle. This gives us the change in bias,

$$ \Delta\mathrm{Bias} = \mathrm{Bias}\_{2020} - \mathrm{Bias}\_{\mathrm{Historical}}, $$

where \\(\mathrm{Bias}\_{2020}\\) refers to the bias this year and $\mathrm{Bias}\_{\mathrm{Historical}}$ refers to the historical bias of the pollster. The way I've set it up is if $\mathrm{Bias}\_{2020}$ or $\mathrm{Bias}\_{\mathrm{Historical}}$ is positive that corresponds to a Democratic-leaning bias and negative corresponds to a Republican-leaning bias. 

If $\Delta\mathrm{Bias}$ is positive that means polls are more favorable to Democrats this year than what historical data would suggest, because that means $\mathrm{Bias}\_{2020} > \mathrm{Bias}\_{\mathrm{Historical}}$ so pollsters are more Democratic-leaning than expected, indicating shy Trump voters. Likewise, if $\Delta\mathrm{Bias}$ is negative means polls are more favorable to Republicans this year than what historical data would suggest, indicating shy Biden voters.

So let's dive into this.


```python
# The usual suspects
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
import time
```

First, I have to do a bit of data wrangling before actually getting to the calculation that we care about. I'll be using FiveThirtyEight's pollster ratings and their database of political polls.


```python
presidential_polls = pd.read_csv('https://projects.fivethirtyeight.com/polls-page/president_polls.csv')
pollster_ratings = pd.read_excel('https://github.com/fivethirtyeight/data/raw/master/pollster-ratings/pollster-stats-full.xlsx', sheet_name='pollster-stats-full-may-2020')
pollster_ratings.rename(columns={'Unnamed: 1':'pollster'}, inplace=True)
data = presidential_polls.merge(pollster_ratings, on='pollster')
```


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>question_id</th>
      <th>poll_id</th>
      <th>cycle</th>
      <th>state</th>
      <th>pollster_id</th>
      <th>pollster</th>
      <th>sponsor_ids</th>
      <th>sponsors</th>
      <th>display_name</th>
      <th>pollster_rating_id</th>
      <th>...</th>
      <th>Advanced Plus-Minus</th>
      <th>Mean-Reverted Advanced Plus Minus</th>
      <th>Unnamed: 21</th>
      <th># of Polls for Bias Analysis</th>
      <th>Bias</th>
      <th>House Effect</th>
      <th>Unnamed: 25</th>
      <th>Average Distance from Polling Average (ADPA)</th>
      <th>Herding Penalty</th>
      <th>Latest Poll</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.415035</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
    </tr>
    <tr>
      <th>1</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.415035</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
    </tr>
    <tr>
      <th>2</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.415035</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.415035</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
    </tr>
    <tr>
      <th>4</th>
      <td>135692</td>
      <td>72327</td>
      <td>2020</td>
      <td>Florida</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>1405</td>
      <td>Climate Power 2020</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.415035</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 66 columns</p>
</div>



Now, I'm going utilize FiveThirtyEight's polling averages for this presidential election cycle.


```python
states = ["National", "Alabama","Alaska","Arizona","Arkansas","California","Colorado",
  "Connecticut", 'District of Columbia',"Delaware","Florida","Georgia","Hawaii","Idaho","Illinois",
  "Indiana","Iowa","Kansas","Kentucky","Louisiana","Maine","Maryland",
  "Massachusetts","Michigan","Minnesota","Mississippi","Missouri","Montana",
  "Nebraska","Nevada","New Hampshire","New Jersey","New Mexico","New York",
  "North Carolina","North Dakota","Ohio","Oklahoma","Oregon","Pennsylvania",
  "Rhode Island","South Carolina","South Dakota","Tennessee","Texas","Utah",
  "Vermont","Virginia","Washington","West Virginia","Wisconsin","Wyoming"]

polling_avgs = []
for i, state in enumerate(states):
    try:
        url_state = state.lower().replace(' ', '-')
        polling_avgs.append(pd.read_json('https://projects.fivethirtyeight.com/polls/president-general/%s/polling-average.json'%state.lower()))
        time.sleep(0.5)
    except:
        continue
```

And just a bit more data wrangling.


```python
polling_avgs = pd.concat(polling_avgs)
data['state'] = data['state'].fillna('National')
data.rename(columns={'candidate_name':'candidate', 'end_date':'date'}, inplace=True)
data['date'] = pd.to_datetime(data['date'])
data = pd.merge(data, polling_avgs, on=['state', 'candidate', 'date'])
```


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>question_id</th>
      <th>poll_id</th>
      <th>cycle</th>
      <th>state</th>
      <th>pollster_id</th>
      <th>pollster</th>
      <th>sponsor_ids</th>
      <th>sponsors</th>
      <th>display_name</th>
      <th>pollster_rating_id</th>
      <th>...</th>
      <th>Mean-Reverted Advanced Plus Minus</th>
      <th>Unnamed: 21</th>
      <th># of Polls for Bias Analysis</th>
      <th>Bias</th>
      <th>House Effect</th>
      <th>Unnamed: 25</th>
      <th>Average Distance from Polling Average (ADPA)</th>
      <th>Herding Penalty</th>
      <th>Latest Poll</th>
      <th>pct_trend_adjusted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>51.11459</td>
    </tr>
    <tr>
      <th>1</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>42.45115</td>
    </tr>
    <tr>
      <th>2</th>
      <td>135692</td>
      <td>72327</td>
      <td>2020</td>
      <td>Florida</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>1405</td>
      <td>Climate Power 2020</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>48.68329</td>
    </tr>
    <tr>
      <th>3</th>
      <td>135521</td>
      <td>72235</td>
      <td>2020</td>
      <td>Florida</td>
      <td>1508</td>
      <td>Harris Insights &amp; Analytics</td>
      <td>960</td>
      <td>Hill.TV</td>
      <td>Harris Poll</td>
      <td>133.0</td>
      <td>...</td>
      <td>0.763159</td>
      <td>NaN</td>
      <td>169</td>
      <td>-1.521620</td>
      <td>0.676057</td>
      <td>NaN</td>
      <td>4.106260</td>
      <td>0.0</td>
      <td>2018-11-05</td>
      <td>48.68329</td>
    </tr>
    <tr>
      <th>4</th>
      <td>135692</td>
      <td>72327</td>
      <td>2020</td>
      <td>Florida</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>1405</td>
      <td>Climate Power 2020</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>-0.387086</td>
      <td>NaN</td>
      <td>294</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>46.50365</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 67 columns</p>
</div>



Now that the data is collected and munged we finally get to dive into the analysis. FiveThirtyEight calculates the historical bias of pollsters in two ways. They calculate the normal historical bias of pollsters by comparing the polls of a pollster to the actual election outcome if the poll was taken within two weeks of the election and they call this "Bias". However, FiveThirtyEight also calculates a version of the historical bias of pollsters by considering that pollsters often change their methodology right after an election to compensate for their mistakes that election and they call this "Mean-Reverted Bias".


```python
bias_2020 = np.zeros(len(data))

for i, x in data.iterrows():
    if x['candidate'] == 'Joseph R. Biden Jr.':
        try:
            y = data[(data['poll_id'] == x['poll_id']) & (data['candidate'] == 'Donald Trump') & (data['question_id'] == x['question_id'])]
            bias = (x['pct'] - y['pct']) - (x['pct_trend_adjusted'] - y['pct_trend_adjusted'])
            bias_2020[i] = bias.values[0]
        except:
            bias_2020[i] = np.nan
    else:
        bias_2020[i] = np.nan
```

Finally, we get to calculate $\Delta\mathrm{Bias}$. I'll compute it twice, using both "Bias" and "Mean-Reverted Bias", so I can compare the results.


```python
data['bias_2020'] = bias_2020
data['bias_diff'] = data['bias_2020'] - data['Bias']
data['mean_reverted_bias_diff'] = data['bias_2020'] - data['Mean-Reverted Bias']
```


```python
data = data[data["candidate"].isin(['Joseph R. Biden Jr.', 'Donald Trump'])]
```


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>question_id</th>
      <th>poll_id</th>
      <th>cycle</th>
      <th>state</th>
      <th>pollster_id</th>
      <th>pollster</th>
      <th>sponsor_ids</th>
      <th>sponsors</th>
      <th>display_name</th>
      <th>pollster_rating_id</th>
      <th>...</th>
      <th>Bias</th>
      <th>House Effect</th>
      <th>Unnamed: 25</th>
      <th>Average Distance from Polling Average (ADPA)</th>
      <th>Herding Penalty</th>
      <th>Latest Poll</th>
      <th>pct_trend_adjusted</th>
      <th>bias_2020</th>
      <th>bias_diff</th>
      <th>mean_reverted_bias_diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>51.11459</td>
      <td>1.33656</td>
      <td>0.966485</td>
      <td>1.002316</td>
    </tr>
    <tr>
      <th>1</th>
      <td>135712</td>
      <td>72333</td>
      <td>2020</td>
      <td>Michigan</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>338</td>
      <td>Progress Michigan</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>42.45115</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>135692</td>
      <td>72327</td>
      <td>2020</td>
      <td>Florida</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>1405</td>
      <td>Climate Power 2020</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>48.68329</td>
      <td>4.82036</td>
      <td>4.450285</td>
      <td>4.486116</td>
    </tr>
    <tr>
      <th>3</th>
      <td>135521</td>
      <td>72235</td>
      <td>2020</td>
      <td>Florida</td>
      <td>1508</td>
      <td>Harris Insights &amp; Analytics</td>
      <td>960</td>
      <td>Hill.TV</td>
      <td>Harris Poll</td>
      <td>133.0</td>
      <td>...</td>
      <td>-1.521620</td>
      <td>0.676057</td>
      <td>NaN</td>
      <td>4.106260</td>
      <td>0.0</td>
      <td>2018-11-05</td>
      <td>48.68329</td>
      <td>0.82036</td>
      <td>2.341980</td>
      <td>2.087401</td>
    </tr>
    <tr>
      <th>4</th>
      <td>135692</td>
      <td>72327</td>
      <td>2020</td>
      <td>Florida</td>
      <td>383</td>
      <td>Public Policy Polling</td>
      <td>1405</td>
      <td>Climate Power 2020</td>
      <td>Public Policy Polling</td>
      <td>263.0</td>
      <td>...</td>
      <td>0.370075</td>
      <td>0.917555</td>
      <td>NaN</td>
      <td>4.747617</td>
      <td>0.0</td>
      <td>2020-02-25</td>
      <td>46.50365</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 70 columns</p>
</div>



Now we're actually going to get to see the $\Delta\mathrm{Bias}$ data once and for all. I'm going to do the calculation $\Delta\mathrm{Bias}$ per a polling firm, by taking the mean of their bias across all 2020 polls for $\mathrm{Bias}_{2020}$, because different polling firms publish with different frequencies. For example, live-caller polling firms take more time to publish polls than online polling firms because it's much more difficult to get enough people to answer the phone and respond to a poll than it is to ask a large number of panelists to respond to an online survey.


```python
data_subset = data.groupby('pollster_id')[['bias_diff', 'mean_reverted_bias_diff']].mean()
bias_diff = data_subset[data_subset['bias_diff'].notna()]['bias_diff']
mean_reverted_bias_diff = data_subset[data_subset['bias_diff'].notna()]['mean_reverted_bias_diff']
```


```python
sns.distplot(bias_diff)
sns.distplot(mean_reverted_bias_diff)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f8c79e5b890>




{{< figure src="/images/2020-10-20-shy-voters_18_1.png" class="mid" >}}


Just inspecting the data graphically, it is important to note how the data naturally centers around zero and is fairly symmetric. This is already a good first sign that there isn't really a shy Trump/Biden voter effect worth talking about. Yet, it's worth actually trying to put a number on it.


```python
print(stats.bayes_mvs(bias_diff, alpha=0.8))
print(stats.bayes_mvs(mean_reverted_bias_diff, alpha=0.8))
```

    (Mean(statistic=-0.5755987877125385, minmax=(-1.1603023674907456, 0.009104792065668499)), Variance(statistic=25.541245417862307, minmax=(21.5079870326055, 29.93285373232226)), Std_dev(statistic=5.043228693701096, minmax=(4.637670431650518, 5.4710925538069874)))
    (Mean(statistic=-0.3778942872751871, minmax=(-0.792448790186189, 0.03666021563581473)), Variance(statistic=12.839078028063225, minmax=(10.811638947921985, 15.04665251770069)), Std_dev(statistic=3.575646253058365, minmax=(3.2881056777302615, 3.8790014846221297)))


For this, I'll be using `scipy.stats.bayes_mvs` which computes the mean, variance, and standard deviation of a sample and gives me a confidence interval for each of those estimates. It turns out that the mean is $\Delta\mathrm{Bias} = -0.58\%$ when compared with regular historical bias and $\Delta\mathrm{Bias}=-0.38\%$ when compared with the mean reverted bias. Furthermore, 0 is contained in the 80% confidence interval of the mean for both estimates of $\Delta\mathrm{Bias}$, meaning that both distributions could very reasonably have a mean of 0 and that there are no shy Trump/Biden voters.


```python
print(stats.ttest_1samp(bias_diff, 0.0))
print(stats.ttest_1samp(mean_reverted_bias_diff, 0.0))
```

    Ttest_1sampResult(statistic=-1.268521859021642, pvalue=0.20704672088192566)
    Ttest_1sampResult(statistic=-1.1746338914885417, pvalue=0.24244771567313306)


To directly figure out if the means of these distributions we can use a one-sample T-test with a null hypothesis that the mean is 0. Doing this for both estimates of $\Delta\mathrm{Bias}$ we find that the p-values are well above a reasonable threshold of $\alpha=0.05$, so I think it is safe to say that the data could have come from a distribution with a mean of 0. However, you should take the T-test with a grain of salt because it seems like the data is not normally distributed and that could be affecting the test.

I think it's interesting how the distributions have a wide spread. From before, we saw the standard deviation of $\Delta\mathrm{Bias}$ when compared with the regular historical bias was about 5.04%, and when compared with the mean reverted bias the standard deviation was 3.58%. These are really large numbers in terms of political polling, comparable or even larger to the margin of error on your standard poll. Well, one of the interesting outcomes of the 2016 election was how education ended up being a strong indicator of how white voters would vote. [Exit polling found that Donald Trump won white voters without a college degree by 39% points, one of the largest margins among this large and electorally decisive demographics in recent history.](https://www.nytimes.com/interactive/2016/11/08/us/politics/election-exit-polls.html) I'll take the data that I have and munge it with some education data to see how strong the relationship is with $\Delta\mathrm{Bias}$.


```python
# Data from 2013-2017 American Community Survey, thanks wikipedia!
education_data = pd.read_html('https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_educational_attainment')[1]
education_data.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State, federal district, or territory</th>
      <th>% High school graduate or higher</th>
      <th>High School rank</th>
      <th>% Bachelor's degree or higher</th>
      <th>Bachelor's rank</th>
      <th>% Advanced degree</th>
      <th>Advanced rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Montana</td>
      <td>93.0%</td>
      <td>1.0</td>
      <td>30.7%</td>
      <td>21.0</td>
      <td>10.1%</td>
      <td>33.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>New Hampshire</td>
      <td>92.8%</td>
      <td>2.0</td>
      <td>36.0%</td>
      <td>9.0</td>
      <td>13.8%</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Minnesota</td>
      <td>92.8%</td>
      <td>3.0</td>
      <td>34.8%</td>
      <td>11.0</td>
      <td>11.8%</td>
      <td>18.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Wyoming</td>
      <td>92.8%</td>
      <td>4.0</td>
      <td>26.7%</td>
      <td>41.0</td>
      <td>9.3%</td>
      <td>39.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alaska</td>
      <td>92.4%</td>
      <td>5.0</td>
      <td>29.0%</td>
      <td>28.0</td>
      <td>10.4%</td>
      <td>29.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>North Dakota</td>
      <td>92.3%</td>
      <td>6.0</td>
      <td>28.9%</td>
      <td>29.0</td>
      <td>7.8%</td>
      <td>51.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Vermont</td>
      <td>92.3%</td>
      <td>7.0</td>
      <td>36.8%</td>
      <td>8.0</td>
      <td>15.0%</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Maine</td>
      <td>92.1%</td>
      <td>8.0</td>
      <td>30.3%</td>
      <td>23.0</td>
      <td>10.9%</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Iowa</td>
      <td>91.8%</td>
      <td>9.0</td>
      <td>27.7%</td>
      <td>36.0</td>
      <td>9.0%</td>
      <td>42.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Utah</td>
      <td>91.8%</td>
      <td>10.0</td>
      <td>32.5%</td>
      <td>16.0</td>
      <td>11.0%</td>
      <td>23.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
education_data.replace({'United States': 'National'}, inplace=True)
by_state_pollster = data.groupby(by=['state', 'pollster'])[['bias_diff', 'mean_reverted_bias_diff']].mean()
```


```python
fig, ax = plt.subplots()

x_array1 = []
x_array2 = []
y_array1 = []
y_array2 = []

for i, state in enumerate(data['state'].unique()):
    y1 = by_state_pollster['bias_diff'][state].values
    y1 = y1[~np.isnan(y1)]
    y2 = by_state_pollster['mean_reverted_bias_diff'][state].values
    y2 = y2[~np.isnan(y2)]
    x = float(education_data[education_data['State, federal district, or territory'] == state]['% Bachelor\'s degree or higher'].values[0][:-1])
    x_array1.append(x*np.ones(len(y1)))
    y_array1.append(np.array(y1))
    x_array2.append(x*np.ones(len(y2)))
    y_array2.append(np.array(y2))
    ax.plot(x*np.ones(len(y1)), y1, 'x', color='tab:blue')
    ax.plot(x*np.ones(len(y2)), y2, 'x', color='tab:orange')

x_array1 = np.concatenate(x_array1)
x_array2 = np.concatenate(x_array2)
y_array1 = np.concatenate(y_array1)
y_array2 = np.concatenate(y_array2)

xlin = np.linspace(0.9*np.min(x_array1), 1.1*np.max(x_array1))

m1, b1, m_lo1, m_hi1 = stats.theilslopes(y_array1, x_array1, alpha=0.8)
ax.plot(xlin, m1*xlin + b1, '-', color='tab:blue', label=r'$y = %.3f_{%.3f}^{%.3f} x + %.3f$'%(m1, m_lo1, m_hi1, b1))
ax.fill_between(xlin, m_lo1*xlin + b1, m_hi1*xlin + b1, color='tab:blue', alpha=0.1)

m2, b2, m_lo2, m_hi2 = stats.theilslopes(y_array2, x_array2, alpha=0.8)
ax.plot(xlin, m2*xlin + b2, '-', color='tab:orange', label=r'$y = %.3f_{%.3f}^{%.3f} x + %.3f$'%(m2, m_lo2, m_hi2, b2))
ax.fill_between(xlin, m_lo2*xlin + b2, m_hi2*xlin + b2, color='tab:orange', alpha=0.1)

ax.set_xlabel(r"% Bachelor's degree or higher", fontsize=14)
ax.set_ylabel(r'$\Delta\mathrm{Bias}$', fontsize=14)
ax.tick_params(axis='both', which='major', labelsize=12)
ax.legend()

plt.show()
```


{{< figure src="/images/2020-10-20-shy-voters_26_0.png" class="mid" >}}


In the graph above the blue represents $\Delta\mathrm{Bias}$ when compared to the regular historical bias and the orange represents when compared to the mean reverted bias. It seems from the line fits that there is a positive trend between the percent of a state's population with a bachelor's degree or higher and $\Delta\mathrm{Bias}$. But it seems like that relationship is fairly weak, so I don't think education alone is enough to explain the variation in $\Delta\mathrm{Bias}$. If I had to make an educated guess, I would think that the major methodology change that lots of pollsters made to weight by education in their polls is the main reason for the large variation. However, the choices that polling firms make in how to weight by education are unique to that polling firm so perhaps that's why the relationship is fairly weak in the graph above. Also, accounting for education in polling is a much newer practice that only came to be seen as important after 2016 so I imagine the polling industry has not come to a consensus on what the standard way to do that is, especially when trying to account for likely-voters.

I don't want this graph to take away from my main point in this article though, shy voters don't exist. I showed it earlier that $\Delta\mathrm{Bias}$ could very reasonably be said to be 0, and is definitely very close to 0. I want everyone to say it with me __shy voters don't exist__. Pollsters/modelers/handicappers have been saying for a while now, there hasn't been historical evidence to suggest that shy voters have ever really been present in past elections, we can largely attribute Trump's victory in 2016 to a late shift in polls and a lack of education weighting, and now there's this blog post that also agrees with everyone else that shy voters don't exist. If you were previously worried about the existence of shy voters and are now unsure of what causes errors in political polling I suggest that you look into [sampling error](https://en.wikipedia.org/wiki/Sampling_error), [coverage error](https://en.wikipedia.org/wiki/Coverage_error), and [regular ole mistakes](https://www.nbcnews.com/politics/2020-election/des-moines-register-pulls-gold-standard-iowa-poll-after-potential-n1128366).

<script>
    window.MathJax = {
            tex: {
                inlineMath: [['$', '$'], ['\\(', '\\)']]
            }
        };
</script>
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>