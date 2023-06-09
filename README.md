# Baseball-Betting-Model


![mlbbetting](https://user-images.githubusercontent.com/73828790/225275823-ba4e5eb7-e423-4611-9aa4-59b345a9b747.jpeg)

### Overview
### Can we used advanced statistics and machine learning to predict winners of MLB games?

The repository contains two R markdown files: 
#### 1. Creating, Training and Validating Model (MLB Model)
The XG Boost model is trained and validated on over 50,000 and 5,000, respectively, MLB games. As a logit model, its output is the probability of the home team winning.

#### 2. Deploying the Model for Upcoming Games (Daily Run for Baseball Picks)
This file allows the user to deploy the model against the day’s games after entering the teams playing one another and the posted odds for each team winning. The output is a csv file which shows the “edge” of each pick as defined by the implied probability of the given odds to win minus the probability of winning provided by the model.  

### Data Collection
The data used for this comes from www.baseball-reference.com and www.retrosheet.com via the baseballr and retrosheet packages in R. 

### Data Cleaning 
Data cleaning was require to created consistency for both the dependent and indpendent variables. Specific efforts included:
1. Creating consistent team names 
2. Creating consistent results
3. Removing incomplete/unfinished games (NA values)

### Feature Engineering
The goal for this project was to create a single observation for each game. A significant feature engineering effort was required to create season-to-date and recent (defined as last 5 games based off performance testing) for two key team-level statistics with predictive power:

***Pythagorean record***:  
Pythagorean record is a well-understood metric for determining what a team's record "should be" based on the numbers of runs scored and allowed. This metric uses the following formula: $$Runs^{1.83}/(Runs^{1.83} - Runs Allowed^{1.83})$$ I created Pythagorean recrods based on season-to-date and last 5 games. All four inputs (home and visitor runs scored and allowed) as well as the recent and season to date Pythagorean records were independent variables in the model. 

***Log 5***:  
Log 5 is the probability a team A will defeat team B based on the odds ratio between winning percentages of each team. The following formula calculates this: 
```math
(p_A-p_A*p_B)/(p_A+p_B-2*p_A*p_B)
```
All six inputs (home and visitor wins, losses, and winning percentages) as well as the recent and season to date Log 5 for the home team were independent variables in the model. 

### Data Leakage
In training a model based on games already played, it was important to avoid any data leakage. Season to date and recent totals were calculated using *cumsum* and *roll away* formulas, respectively. In so doing, these formulas would include data from that day's games, data not available when making a prediction. As such, I subtracted the data from that day's games from each total.  This resulted in choosing a value of *k* for recent games one higher than the number of games you want to evaluate (e.g. if you want to look at the last 7 games, the value of *k* should be 8).

### Data Transformation
Heretofore, each observation in the data was for a team at each point in the season to enable calculations of the required inputs for the model. For the model to function properly, the teams playing against one another needed to be combined into a single observation.  This required the creation and assignment of a unique game ID to each team playing in a single game. The data was split into home and away teams. A left join was performed based on the unique game ID to create a single observation for each game containing the data for both the home and away teams.  

### Model Creation 
As noted earlier, the model was trained on 50,000 games and validated on 5,000, an atypical split when seeking to avoid overfitting. Because each baseball game is different, I don't believe overfitting is an issue. Furthermore, it was important to validate the model on recent seasons rather than a larger number of randomly selected games over the past 22 years.  I want to ensure the model performs well on how the game is played today. I performed cross validation to determine the optimal number of trees to train. 

### Performance 
When the model is applied to each game, the overall accuracy is 59%. However, when used in a betting context, one does not have to place a bet on every game. The graph below shows the model's accuracy (y-axis) as the probability threhsold for placing a bet increases (x-axis). The diamonds represent the frequency the user can anticipate seeing predictions at each probability threshold.  

![Picture1](https://user-images.githubusercontent.com/73828790/225315425-bd40c180-ee44-4a3b-a216-f7dfdfbcd4fc.png)

Not suprisingly, the accuracy steadily increases as the probability increases.  For example, if a 66% probability is present, accuracy increases by 13 percent.  However, this level of confidence only appears in 17 percent of predictions. 

### Variable Importance

I calculated Shapley Additive Explanation (SHAP) value to quantify the importance of each input, and included the top 10 in the plot below. Very simply, the features with the largest value are the most important.

<img width="525" alt="000010" src="https://user-images.githubusercontent.com/73828790/225316842-aa66c900-06ca-4ebb-8ba4-ae8b09139fd3.png">

In addition to importance, the above plot shows the effect of high and low values of the feature and noted by the color. For example, high values of losses in the last 5 games (RollingOppLosses) produce a high SHAP value suggesting it is largely responsible for the prediction.

### Deployment
To deploy the model, the user needs to execute 4 steps: 
1. Ingest the most recent game data,
2. Input the visiting and home teams playing one another,
3. Input each team's odds (American) of winning, and
4. Run remaining code

#### Ingest data
To execute this step, the user must simply run the code already written.  All required cleaning and feature engineering to ensure the data’s format exactly matches what the model was trained on is included.    

#### Input Teams
All visiting teams should be typed into the opp vector using the team's abbreviation used by Baseball-Reference.com found [here](https://www.baseball-reference.com/about/team_IDs.shtml). The home team's abbreviation should be typed into the team vector.  All abbreviations need quotation marks.   

<img width="981" alt="Capture" src="https://user-images.githubusercontent.com/73828790/225322227-f0b76b00-742b-4111-b97c-58c76df67726.PNG">

#### Input Odds
To determine where the value lies, the user needs to input the American odds for each team to win in the respective odds vector.  The order is critically important. The first number in the Opp_odds vector should correspond to the first team in the opp vector. That team's opponent that day should be the first team in the Tm vector and its odds to win should be the first value in the Tm_odds vector.  

#### Run ramaining code
The output of the code is a csv file containing the model's prediction with the necessary contextual information to determine the "edge" associated with each bet.  ![image](https://user-images.githubusercontent.com/73828790/225327134-191e3e81-22fd-4c2a-a4af-3f1e8d6ac830.png)


