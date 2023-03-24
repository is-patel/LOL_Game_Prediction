<style>
.table-wrapper {
  overflow-x: scroll;
}
</style>

# Framing the Problem

By analyzing a dataset consisting of League of Legends professional matches from 2022, we want to design a binary classification model to predict whether a given team will win or lose. Our model will use features collected before the 15-minute mark of a match to predict the game’s result. These features include team composition, number of kills, and various resource differentials at 15 minutes. Notice that means we do not consider which team claims a baron as it spawns after 20 minutes, and information about structures on the map, except for which team takes the first tower, as this usually occurs before 15 minutes.

To evaluate our classifier, we chose to calculate its accuracy. With regards to our prediction problem, neither the amount of false positives or negatives is more important than the other. A false positive prediction indicates that a team that lost was predicted to win, and a false negative prediction indicates that a team that won was predicted to lose. There is no serious impact in incorrectly predicting one way or the other. As such, we believed the accuracy metric to be a good measure of the quality of our model. 

The data set contains statistical information about each match played: some of which include: 

*	position : the position played by the player in each row, or 'team' if the row contains team based statistcs
*	result : the outcome of the match, 1 if win, 0 if loss
*	 totalgold : the total gold earned by the player, or in team rows it is total gold earned by the team
*	goldat10 : gold earned at 10 minutes by the player, or in team rows it is the gold earned by the team at 10 minutes
*	opp_goldat10 : similar to the above column, but it is for the enemy play in the same role as the row, or enemy team
*	golddiffat10 : the difference between the players gold, and the enemy player in the same role
*	goldat15 : same as the previous goldat column except for gold at 15 minutes
*	opp_goldat15 : same as the previous goldat column except for gold at 15 minutes
*	golddiffat15 : same as the previous goldat column except for gold at 15 minutes
*	xpat10 : similar to the gold column, however instead of gold it is xp
*	opp_xpat10 : similar to the gold column, however instead of gold it is xp
*	xpdiffat10 : similar to the gold column, however instead of gold it is xp
*	xpat15 : similar to the gold column, however instead of gold it is xp
*	opp_xpat15 : similar to the gold column, however instead of gold it is xp
*	xpdiffat15 : similar to the gold column, however instead of gold it is xp
* firstblood, firstdragon, firstherald, and firsttower, 1 if the team was the first to achieve

With all this information we aim to predict the outcome of a game before it is finished.

Below is a quick table of the information that may be useful for this procedure:
<div class="table-wrapper" markdown="block">
|    |   result |   team_winrate | side   |   firstblood |   firstdragon |   firstherald |   firsttower |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   champion_high_wr |   champion_above_avg_wr |   champion_avg_wr |   champion_below_avg_wr |   champion_low_wr |
|---:|---------:|---------------:|:-------|-------------:|--------------:|--------------:|-------------:|---------------:|-------------:|-------------:|------------:|---------------:|-------------:|-------------:|------------:|-------------------:|------------------------:|------------------:|------------------------:|------------------:|
|  0 |        0 |       0.333333 | Blue   |            1 |             0 |             1 |            1 |           1523 |          137 |           -8 |           3 |            107 |        -1617 |          -23 |           5 |                  0 |                       0 |                 5 |                       0 |                 0 |
|  1 |        0 |       0.333333 | Blue   |            0 |             0 |             1 |            0 |          -3917 |        -1213 |          -47 |           1 |          -4705 |        -1330 |          -56 |           1 |                  0 |                       1 |                 3 |                       1 |                 0 |
|  2 |        1 |       0.333333 | Red    |            1 |             1 |             1 |            1 |           1396 |          883 |           12 |           4 |           2917 |           64 |           -2 |           5 |                  0 |                       3 |                 2 |                       0 |                 0 |
|  3 |        0 |       0.333333 | Red    |            0 |             0 |             0 |            0 |          -1173 |         -561 |           -9 |           3 |          -3637 |        -1641 |          -38 |           6 |                  0 |                       2 |                 3 |                       0 |                 0 |
|  4 |        1 |       0.333333 | Red    |            1 |             1 |             1 |            0 |            255 |          886 |           16 |           1 |             92 |         1076 |           -8 |           4 |                  0 |                       1 |                 4 |                       0 |                 0 |
</div>

# Baseline Model

Our baseline model is a Decision Tree Classifier, and makes predictions based on two features:
* whether or not the team got the first kill
* and the difference in minion kills (creep score) between the team and their opponent.

The first feature is a categorical nominal variable with two unique values:  “1” indicates that the team earned first blood, and “0” indicates otherwise. The second feature is a quantitative variable. Positive values indicate that the team had a “cs” lead, and negative values indicate the opposite. The more extreme a value is in either direction indicates how big the gap between the team and their opponents was. 

Our baseline model was trained on these raw features, that is, they were not further transformed in the actual training of the model. The model achieved 68% accuracy on the training set and on the testing set. Similarly, when trained on the whole dataset, it achieved 68% accuracy. 

We believe that this is a sub-optimal model. However, it is a good baseline model that can be greatly improved. Based only on two variables taken at the 15-minute mark, our classifier accurately predicts 7 out of 10 matches. As such, we believe that with more features (to an extent) and tuning, this model can be much more accurate.

Below is a small chart of the confusion matrix produced:

<iframe src="assets/cfig1.html" width=800 height=600 frameBorder=0></iframe>

Here we can see the reletive predictions, and how well the model predicted wins vs loses. With a quick glance we can see false positives, and false negatives, occur at reletively the same rate.

# Final Model

Our final model is an remains a Decision Tree Classifier. However, to improve upon the baseline model, we will take into account the following features:

1. the team’s overall win-loss record
2. whether or not the team got the first kill
3. how many kills the team had at 15 minutes
4. the difference in gold between the team and their opponent
5. how many objectives (first tower, first dragon, first herald) they obtained first
6. how many champions on the team had overall win rates between 47% and 52%
7. how many champions on the team had overall win rates lower than 46%

Each feature is quantitative. A team’s win-loss record was calculated by dividing their number of wins by their total number of matches. The second, third, and fourth features were taken straight from the dataset. The objectives feature was engineered simply by summing the relevant columns from the dataset. The number of kills and the gold difference at 15 minutes were taken straight from the dataset. The final feature was engineered in the data cleaning process. We first determined each champion's win rate, and divided the champions into five groups at variable win rate thresholds. Then, with the ‘champion’ column now containing 1 of 5 unique values (condensed from 1 of over 140 unique champions), we constructed a One Hot Encoding and concatenated it to our dataset.

There is a clear connection between these features and the outcome of a match. This relationship is obvious with regards to the first feature: teams with better win-loss records are more likely to win a match. The next four features (first blood, kills at 15, gold difference, objectives) are strong indicators of how well a team is doing. With the way League of Legends games and professional playstyle is structured, teams which take the lead and dominate the beginning of a game are generally able to “snowball” their advantage and earn a victory. As such, the clearest ways to quantify a lead is through a team’s kills, the gold differential between them and their opponent, and how many objectives they are able to secure. Taken together, these values accurately describe a team’s performance.

Finally, it is a fact of League of Legends that certain champions outperform others. As mentioned earlier, we chose to quantify this difference by grouping champions into tiers based on their win rate. In analyzing the dataset, we found that teams more commonly picked champions with about average win rates. So, we chose to include the number of champions each team with the relevant win rates in training our model.

To identify the best hyperparameters for the Decision Tree, we performed a Grid Search, training nearly 200 different models and varying the criterion, max_depth, and min_samples_split hyperparameters.
 
The following hyperparameters performed the best when constructing our model:
* criterion: gini
  * The function used to measure the quality of a split
* max_depth: 5   
  * The maximum depth of the decision tree
* min_samples_split: 200
  * The minimum number of samples required to split a node

The accuracy of our model greatly improved, going from 68% to 76%. This is a significant improvement, and means our classifier accurately predicts the outcome in 3 out of 4 matches. One significant issue we repeatedly encountered when building our model was that our train and test accuracy would never surpass 76%. Though we were glad to reach this mark, we built several other models with different features and classifiers as a whole that performed just as well. As mentioned earlier, a team can pick up a lead in a few areas that will translate to leads elsewhere. As such, many of the features in our dataset are highly intertwined with one another and the accuracy of our models plateaued at 76%.

Below is another confusion matrix representative of the final model:

<iframe src="assets/cfig2.html" width=800 height=600 frameBorder=0></iframe>

Here we can see the same trend as earlier, as well as a significant improvment in the performance of the model.

# Fairness Analysis

When looking at the data set, we had seen a slight tendency for the red side to win more often than the blue side. To be more specific it was around 52%. With this information we would like to see if the model tended to also predict better on red side teams than blue side teams.

To do this we will perform a permutation test, with the following parameters:
* Evaluation Metric - We will continue to look at accuracy, this is due to the model not being rewarded for tending to output more wins or loses, and thus false negatives and false positives are equally as bad.
* Null Hypothesis - The created model performs equally on red as it does blue, in the measure of accuracy. 
* Alternative Hypotheses - The created model performs better on red compared to blue, in the measure of accuracy
* Test-Statistic - Red_accuracy - Blue_accuracy
* Significance Level = 0.05 (as is the standard)

We then calculated the observed statistic to be 0.0002497, even though the difference may be small, it could still be significant if there is a general tendency of the randomized permutations to perform with a lower distance.

After the permutation testing we have the resulting histogram:

<iframe src="assets/pfig1.html" width=800 height=600 frameBorder=0></iframe>

As one can see our observed statistic is in the middle, and this is asserted by its p-value of 0.484 on 1000 permutations.

With this we conclude that it is more likely than not, that there is no difference between performance on red and blue sides, and the created model performs equally on red as it does blue, in the measure of accuracy. 

# Conclusion

With this, we conclude the creation of the model; which we have made to predict the outcome of a league of legends game with the information readily avaliable at 15 minutes. With an average performance of 76%, this definitely surpases the many predictions and hopes made by fans during the game, despite the outcome of the game being more likely than not already decided. However, fans don't watch games just for the conclusion, but this model just goes to show how early league games may be decided.