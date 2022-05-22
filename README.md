# NBA Game Predictor

## Overview

The goal of this project is to create a model trained off NBA team game log data from the 2007 season to the 2020 season to forecast the outcome of an NBA game in the 2021 season.

## Steps

1. Obtain, verify, and modify the data
2. Create the model

## 1. Obtain, verify, and modify the data

### A. Obtain Data

Using the [nba_api](https://github.com/swar/nba_api), I gathered the game data for all regular season NBA games since 2007.

### B. Verify Data

I first verified this data by the number of rows. The amount of rows should equal games played $\times$ 2. There should be a total of [Teams] $\times$ [Games Played]  $\times$ [Seasons] $\div$ [2 Teams in each game] = 
30 $\times$ 82  $\times$ 12  $\div$ 2= 18,450 games played.
In a typical season there is a total of 1230 games = 30 $\times$ 82  $\div$ 2
However some seasons had less than 82 games
- 2011 had 66 games each for a total of 990 games (240 less)
- 2020 was tricky due to covid. Different teams played different amount of games but the total amount for the whole season was 1059 games (171 less)
- 2020-2021 had 72 games each for a total of 1080 games (150 less)

18,450 - 240 - 171 - 150 = 17,899 total games played which matches the number of rows $\div$ 2

Next I verified the total points in each season by grouping the data by season and comparing the total values with those found on [NBA League Averages - Totals | Basketball-Reference.com](https://www.basketball-reference.com/leagues/NBA_stats_totals.html)


Lastly I took a random sample of 15 games and compared them with the box scores found on [Basketball-Refence.com](https://www.basketball-reference.com/)

### C. Modify the data

### The data I have:
- 2 rows for each game (1 for each team)
- Whether the team won or loss
- How many of X stat the team got that game
	- Points
	- Rebounds
	- Assists 
	- Turnovers
	- Shots Made
	 Shots Missed
	
### The data I need:
- 1 row for each game split between home and away stats
- The team's X stat going into the game:
	- Offensive Rating
	- Defensive Rating
	- Assist Rating
	- Assist Rating Allowed
	- Offensive Rebound Rating
	- Offensive Rebound Rating Allowed
	- Turnover Rating
	- Turnover Rating Allowed
	- True Shooting Percent
	- Effective Field Goal Percent Allowed
- Whether the team played the day before

### Why this is the data I chose to use:
I split each row into home/away team so the model can adjust for home court advantage. This does not adjust for each individual team however. I suspect some teams have a stronger home-court advantage than others and that home court advantage varies seasons to season (especially 2019-2021). 

**Offensive/Defensive Rating (ORTG/DRTG)** is Points scored/allowed per 100 possessions. This is more useful than points per game since it adjusts for pace of play and minutes played (overtime).

**Assist Rating** I calculate as Total Assists $\div$ Total Made Shots. I use this since in order to have an assist, you need a made shot. Take the Cleveland Cavalier's loss in game 4 of the 2018 NBA finals to the Golden State Warriors. The Warriors won 108-85 to cap off a 4-0 sweep and take home their 2nd trophy in just as many years. A saddened Cavs fan may look at that game's box score and see the Cavaliers had only 21 assists to the Warrior's 25. They may then come to the conclusion that the Cavaliers lost because they played more selfishly and didn't pass enough. However, the Assist Ratings tell a different story. The Warriors had an Assist Rating of 64.1% and the Cavaliers had 70%.  I understand that the assist rating is not perfect. In theory a team could pass the ball only on 20 possessions but every time those passes led to a basket. All other 80 possessions, they only shot and missed with no passes. This would lead to a 100% assist rating; however, the team obviously still played selfishly and should have passed more. This extreme scenario would never happen, but I would expect a much milder example to have occurred occasionally. I believe this stat can be further enhanced by incorporating potential assists (missed shots that would have counted for an assist had they gone in) and total shot attempts. 

**Offensive Rebound Rating** I calculate as Offensive Rebounds $\div$ Total Missed Shots. Similar reasoning to Assist Rating, except you need a missed shot in order to get an offensive rebound. One issue that may arise with this is free throw misses. There can be an additional rebound opportunity when a player misses a free throw. This is not easily fixable since not all free throw misses are in play; sometimes the player has an additional free throw. I believe this balances out a little with the fact that not every shot results in a rebound. For example, the shot could be a buzzer beater, or the shot could result in a jump ball.

**Offensive Rebound Rating Allowed** accounts for defensive rebounds since when the opponent misses, (usually) a rebound is the only thing that occurs.

**Turnover Rating** is Turnovers per 100 possessions. Just like ORTG, this accounts for pace of play and minutes played.

**Turnover Rating** looks at how many turnovers a team forces.

### Creating the features:

Since I do not want to cause data leakage by using the stats from the game I want to predict, I only use data from the previous games in that season. I also want to split the features up by home and away. To do this, I created both home and away columns and if the team was the home team, I put their values into the home columns and set the away values to 0. Vice versa for away teams. I then grouped the data frame by game ID and summed all the values. I ended up with 17,889 rows which is expected.

## 2. Create the model

I will be using a basic logarithmic model with default parameters. I will then test the accuracy using cross validation.

### Adjusting Data
Since the goal of this project was initially to predict games during the 2021-22 season, I am not using data from that season to train the model and instead will use it as an additional testing set. Also due to covid, the 19-20 and the 20-21 season have a lot of variance with key players missing many more games than usual. There was also less or no fans at all during these seasons which can impact home court advantage. Since the data uses team data rather than individual player data, I suspect the model will do much better for the 2007-2018 seasons than for 2019 and 2020. Because of this, I split the data into two data sets: One including all seasons 2007-2020, and one with only 2007-2018. I expect the model trained on the smaller set will have a higher accuracy and will predict the 2021 season better. Lastly, due to the low sample size of data at the beginning of each season, I\'m going to exclude games if the teams haven\'t played a combined 20 games yet.

### Adjusting Features
The features that have significant coefficients are Back-to-Back, Offensive Rating, Defensive Rating. Therefore, we can exclude all other features in our model. An interesting note about the coefficients, back to back games impact away teams more than home teams.

### Analysis
When excluding the 2019 and 2020 season, the mean accuracy was 67.9% +/- 3.7%. When 2019 and 2020 are included, the overall accuracy drops to 66.8%. This supports my guess that 2019 and 2020 were more unpredictable than prior seasons. However, when using each model to predict the 2021 season, the results were very similar. Both models obtained a 64% accuracy, and they only differ on 12 of the 1,075 games. When using a baseline model of selecting the home team to win every game, we would obtain a 58.3% accuracy when predicting all games between 2007 and 2022. A total of 10,437 home wins and 17,889 games. Another baseline model we could use is selecting the team with better record every game. This will yield an accuracy of 63.9% when you don't include games where teams combined for less than 20 prior games played that season. A total of 10,182 favorite wins and 15,943 games. This drops to just 61.3% for just the 2021-22 season.

## Next Steps

- Create better models
	- Use a grid search to optimize parameters
	- Test other methods such as random forest classifiers or K-nearest neighbors
- Use time series data
	- could transform data frame so each row is a team and their respective Net Rating (Off. Rating - Def. Rating) for each game
	- could aggregate last X number of games stats into a new feature
- Add in analysis using sportsbook data
	- compare how well sportsbooks do to these models
	- create new models to predict the point spread of games
- Add in player analysis
	- explore how much impact star players have when they are playing compared to when injured/resting
	- create new models to predict games using player data

