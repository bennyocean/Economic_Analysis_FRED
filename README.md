# Introduction
Dedicated project on Economic Data Analysis with Fred & Python to discover insights from unemployment and participation rates in the US job market.

Check the detailled analysis out here: [Economic_Analysis_FRED folder](/)

Based on the Economic Data Analysis project created by [Rob Mulla](https://www.youtube.com/watch?v=R67XuYc9NQ4). See also the [Kaggle project](https://www.kaggle.com/code/robikscube/economic-analysis-with-pandas-youtube-tutorial/notebook)

# Background
Driven by my quest to become a professional data scientist, this project helped me to reinforce my Python skills for data analysis, particularly, scraping, cleaning, exploring, and visualizing economic data.

Data is retrieved from Federal Reserve Economic Data, [FRED](https://fred.stlouisfed.org/), an online database consisting of hundreds of thousands of economic data time series from scores of national, international, public, and private sources.

# Tools I Used
For my deep dive into the econimic data analysis of the US job market, I harnessed the power of several key tools:

- **Python** 
- **Jupyty Notebook**
- **Pandas**
- **APIs**
- **Github**

# The Analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market.

### 1. Pull & Clean Data
To clean and manipulate the unemployment rate data from the US job market, I filtered the monthly, seasonally adjusted unemployment rate for all states. Then, I concatonated the different data by looping overy every state and removed non-relevant columns and NANs. To adjust the title column, I replaced some text and applied list comprehension to rename column names.

```python
# pull monthly, seasonally adjusted unemployment rate for all states in percent
unemp_df = fred.search('unemployment rate state', filter=('frequency','Monthly'))
unemp_df = unemp_df.query('seasonal_adjustment == "Seasonally Adjusted" and units == "Percent"')
unemp_df = unemp_df.loc[unemp_df['title'].str.contains('Unemployment Rate')]

# get unemployment rates for all ids and concatonate them in one dataframe
all_results = []

for myid in unemp_df.index:
    results = fred.get_series(myid)
    results = results.to_frame(name=myid)
    all_results.append(results)
    time.sleep(2.2) # Don't request to fast and get blocked
uemp_results = pd.concat(all_results, axis=1)

# drop non relevant columns
cols_to_drop = []
for i in uemp_results:
    if len(i) > 4:
        cols_to_drop.append(i)
uemp_results = uemp_results.drop(columns = cols_to_drop, axis=1)

# create new dataframe (uemp_states) and drop NANs. Also replace 
uemp_states = uemp_results.copy()  #.drop('UNRATE', axis=1)
uemp_states = uemp_states.dropna()

# Mapping id to state name by removing verbose text "Unemployment Rate in " in the title's column
id_to_state = unemp_df['title'].str.replace('Unemployment Rate in ','').to_dict()

# list comprehension to rename column names using id_to_state dictionary with mapped state names for each id 
uemp_states.columns = [id_to_state[c] for c in uemp_states.columns]
```
Here's the breakdown of the unemployment rate in the US job market from 1976 to 2024:
- **Wide Unemployment Range:** Unemployment rate ranges from 1.7% to 24.7% during that period, with an average of 5.9% over all states.
- **Corona Peak:** The Covid19 pandemic hit the job market badly, demonstrated by a sharpe unemployment rate increase over all states, slightly moving back to pre-pandemic times in early 2022.
- **Outlier State:** Puerto Rico shows the highest unemployment ratem however, the state's rate seemed to converge to peer states during since the 2020's.

Average US Unemployment Rate (1976 - 2024)
| State               | Unemployment Rate (%) |
|---------------------|-----------------------|
| Puerto Rico         | 14.19375              |
| West Virginia       | 7.863715              |
| Alaska              | 7.639236              |
| Michigan            | 7.62934               |
| District Of Columbia| 7.463542              |
| ...                 | ...                   |
| Vermont             | 4.427257              |
| New Hampshire       | 4.156771              |
| North Dakota        | 3.682292              |
| South Dakota        | 3.544965              |
| Nebraska            | 3.399306              |

*This table starts with the 5 states having the highest unemployment rates and concludes with the 5 states having the lowest unemployment rates, based to the FRED.*

### 2. February 2024 Unemployment Rate 
To understand the current (February 2024) unemployment rate in the US, I ploted a bar chart for all states and sorted the values, providing insights into which state exhibits the highest/lowest unemployment rate in the US.

```python
ax = uemp_states.loc[uemp_states.index == '2024-02-01'].T \
    .sort_values('2024-02-01') \
    .plot(kind='barh', figsize=(8, 12), width=0.7, edgecolor='black',
          title='Unemployment Rate by State, February 2024')
ax.legend().remove()
ax.set_xlabel('% Unemployed')
plt.savefig('uemp_rates_states_feb_2024.png', dpi=300)
plt.show()
```

Here's the breakdown of the February's 2024 unemployment rate per state:
- **Puerto Rico Again** Although converging to peer state's unemployment rate, Puerto Rico still shows the highest rate (5.7%) among all states.
- **Lowest Rate** North (2.0%) and South Dakota (2.1%) have the lowest unemployment rates.

![Unemp_Rates_States](assets/uemp_rates_states_feb_2024.png)
*Horizontal bar graph visualizing the unemployment rate by state, as of February 2024*

### 3. Comparison w/ Participation Rate

This comparison helped understand the participation rate against the unemployment rate per State since 2020, indicating effective job integration measures.

```python
fig, axs = plt.subplots(10, 5, figsize=(30, 30), sharex=True)
axs = axs.flatten()

i = 0
for state in uemp_states.columns:
    if state in ["District Of Columbia","Puerto Rico"]:
        continue
    ax2 = axs[i].twinx()
    uemp_states.query('index >= 2020 and index < 2023')[state] \
        .plot(ax=axs[i], label='Unemployment')
    part_states.query('index >= 2020 and index < 2023')[state] \
        .plot(ax=ax2, label='Participation', color=color_pal[1])
    ax2.grid(False)
    axs[i].set_title(state)
    i += 1
plt.tight_layout()
plt.savefig('unemp_vs_part.png', dpi=300)
plt.show()
```
Here's the breakdown of the unemployment and participation rate in the US since 2020:
- **Heterogeneity:** Some states (e.g., Wisconsin) show converging (e.g., New York) rates, while some other states have diverging rates.
- **Homogeneity:** However, we observe also some states that have congruent unemployment and participation rates, such as New Hampshire.

![Unemp_vs_Part](assets/unemp_vs_part.png)
*Multiple line graph visualizing the unemployment vs. participation rate by state since 2020*

# What I Learned

Throughout this adventure, I've turbocharged my Data Analytic toolkit with some serious firepower:
- 📈 Economic Analysis Proficiency: I refined my skills in economic data analysis using Python and Pandas, adeptly manipulating and interpreting datasets to extract meaningful insights.
- 🎨 Data Visualization Expertise: I honed my ability to convey complex information clearly through advanced visualizations with Matplotlib and Plotly, enhancing the interpretability of economic trends.
- 🔍 API Integration Skill: I mastered the use of the FRED API for data retrieval, efficiently incorporating real-time economic data into my analysis for up-to-date and accurate insights.

# Conclusions

### Insights
The comprehensive analysis of the US job market from 1976 to 2024 highlights significant trends and outliers in unemployment rates across various states, offering a nuanced understanding of the labor market's dynamics over nearly five decades. Here are the main findings summarized:
- **Unemployment Range:** US unemployment rates varied widely from 1.7% to 24.7%, averaging 5.9%, reflecting economic fluctuations and the impact of various events on job markets.
- **Covid-19 Impact:** The pandemic caused a notable spike in unemployment, termed the **"Corona Peak,"** with a gradual return to pre-pandemic levels by early 2022.
- **Puerto Rico's Outlier Status:** Historically high unemployment rates in Puerto Rico began to align more with other states in the 2020s.
- **February 2024 Update:** Puerto Rico's unemployment rate was still high at 5.7%, while North and South Dakota reported the lowest rates (2.0% and 2.1%).
- **Unemployment vs. Participation Dynamics:** Since 2020, variations in unemployment and participation rates across states highlighted diverse economic recovery patterns.
- **Regional Economic Insights:** Analysis of state-specific data revealed varying patterns in unemployment and participation rates, suggesting different local economic and policy impacts.


This analysis offers critical insights into the resilience and challenges of the US job market, highlighting the importance of tailored policy interventions to address regional disparities and support a sustainable economic recovery across all states.

### Closing Thoughts

This project honed my data analysis and visualization skills, offering insights into US unemployment trends and the impact of major events like the Covid-19 pandemic. It emphasized the importance of adaptability, continuous learning, and using data to inform economic and policy decisions.
