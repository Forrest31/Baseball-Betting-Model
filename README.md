# Baseball-Betting-Model


![mlbbetting](https://user-images.githubusercontent.com/73828790/225275823-ba4e5eb7-e423-4611-9aa4-59b345a9b747.jpeg)

### Overview
### Can we used advanced statistics and machine learning to predict winners of MLB games?

The repository contains two R markdown files: 
#### 1. Creating, Training and Validating Model (MLB Model)
The XG Boost model is trained and validated on over 50,000 and 5,000, respectively, MLB games. As a logit model, its output is the probability of the home team winning.

#### 2. Deploying the Model for Upcoming Games (Daily Run for Baseball Picks)
This file allows the user to deploy the model against the day’s games by after entering the teams playing one another and the posted odds for each team winning. The output is a csv file which shows the “edge” of each pick as defined by the implied probability of the given odds to win minus the probability of winning provided by the model.  

### Data Collection
The data used for this comes from www.baseball-reference.com and www.retrosheet.com via the baseballr and retrosheet packages in R. 

### Data Cleaning 
Data cleaning was require to created consistency for both the dependent and indpendent variables. Specific efforts included:
-Creating consistent team names 
-Creating consistent results
-Removing incomplete/unfinished games (NA values)

### Feature Engineering
The goal for this project was to create a single observation for each game. A significant feature engineering effort was required to create season-to-date and recent (defined as last 5 games based off performance testing) for two key statistics with predictive power:
*Pythagorean record
Pythagorean record is a well-understood metric for determining what a team's record "should be" based on the numbers of runs scored and allowed. To create this metric, I added up each team's runs scored and allowed for every game prior to the one being played in that season as well as the 5 games. All four inputs (home and visitor runs scored and allowed) as well as the recent and season to date Pythagorean records were independent variables in the model. 

*Log 5
Log 5 is the probability that a team with a certain winning percentage will defeat a team with a particular winning percentage. To calculate this, I added up each team's wins for every game prior to the one being played in that season as well as the 5 games and divided it by the total number of games played to create winning percentages. All six inputs (home and visitor wins, losses, and winning percentages) as well as the recent and season to date Log 5 for the home team were independent variables in the model. 

### Data Leakage
In training a model based on games already played, it was important to avoid any data leakage. Season to date and recent totals were caclculated using cumsum and roll away formulas, respectively. In so doing, these formulas would include data from that day's games, data not available when making a predicition. As such, I subtracted all of the data from that day's games from each total.  This resulted in choosing a value of k for recent games one higher than the number of games you want to evaluate (e.g. if you want to look at the last 7 games, the value of k should be 8).

### Data Transformation
Heretofore, each observation in the data was for a team at each point in the season to enable calculations of the required inputs for the model. For the model to function properly, the teams playing against one another needed to be combined into a single observation to produce a single winner.  This required the creation of a unique game ID. The data was then split into home and away teams. A left join was performed based on the unique game ID to create a single observation for each game containing the data for both the home and away teams.  

### Model Creation 
As noted earlier, the model was trained on 50,000 games and validated on 5,000, an typical split and potentially allowing for overfitting. Because each baseball game is different, I don't believe overfitting is an issue in this setting. Furthermore, it was important to validate the model on recent seasons rather than a larger number of randomly selected games over the past 22 years.  I want to ensure the model performs well on how the game is played today. I performed cross validation to determine the optimal number of trees to train. After fitting the model, I validated it on 2021-22 games. 

### Performance 
When the model is applied to each game, the overall accuracy is 59%. However, when used in a betting context, one does not have to place a bet on every game. The graph below shows the model's accuracy (y-axis) as the proability threhsold for placing a bet increases (x-axis). The line on the secondary axis shows how often the user can expect to have games have a particular threhsold.  



The accuracy steadily increases as the threshold for predicting a home team win increases.  For example, if a 66% probability is present for either team, accuracy increases by 13 percent.  However, this level of confidence only appears in 17 percent of predictions. I’ve also included a SHAP analysis to show which inputs play the biggest role in the prediction. 

The deployment file first ingests the most recent data and cleans it followed by performing the necessary feature engineering to ensure the data’s format exactly matches what the model was trained on.  To identify where any potential value lies, the user will need to manually input the home and away teams as well as the odds for each team to win. The output will be a table with each game and, the model’s probability of each team winning, the implied probability from the odds, as well as the expected value of each pick. 

