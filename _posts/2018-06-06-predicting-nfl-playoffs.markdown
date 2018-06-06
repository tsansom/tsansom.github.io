---
layout: post
comments: true
title: "Predicting NFL Playoffs"
subtitle: "Using positional salary spending and previous year stats to predict NFL playoff teams"
data: 2018-06-06 04:31:11 -0500
categories: scraping, lasso, regression
---

## Introduction
Professional sports tend to have a lot of inertia. The best teams in one season tend to be contenders the next season as well. Although players get shuffled around through trades, free agency, drafts, and waivers, the full cast of players on each team doesn't change that much from year to year.

The NFL is a prime example of a high inertia league. The New England Patriots have only missed the playoffs 3 times in the last 17 years whereas the Cleveland Browns last playoff appearance was a Wild Card loss in 2002. To find the last Browns playoff win you have to go all the way back to 1994 (where they beat the Patriots in a Wild Card game). Granted, these teams are on extreme ends of the spectrum. But the point remains that teams tend to mirror performance of the previous year, making gains/losses on the multi-season time scale as opposed to the sub-seasonal (week-to-week) or seasonal time scale.

Recently I stumbled onto [this visualization](https://data.world/dwpeterson/nfl-positional-spending-where-nfl-teams-spend-their/insights/26a43d02-949f-4a5f-8d34-e7f13882a71f) which breaks down offensive positional spending vs in-season success of NFL teams. I saw some interesting things such as the most successful teams (13-16 wins) spent less on wide receivers and quarterbacks and more on running backs and tight ends than teams with 12 wins or less.

This then begged the question: could I use this positional spending data and previous season team performance to **predict** what teams were going to make the playoffs this year? Only one way to find out...

## Getting the Data
The data that I wanted for this project was positional spending, number of players at each position, and offensive/defensive stats from the previous year. I also needed to grab NFL standings for each year which includes features like wins, losses, point differential, made playoffs, etc. This served as my target variable in some way, I'll get to that later.

Unfortunately, this data was not readily available in a tidy csv. Also, since there's a single table for each team, season, and feature, I couldn't simply copy and paste the tables into Excel and create a csv that way. I needed to scrape the data systematically from several websites and then consolidate the data into a single dataset. To do this I used Python's Requests and BeautifulSoup packages. Positional spending came from [Spotrac.com](spotrac.com), offensive/defensive data came from [NFL.com](nfl.com), and season standings data came from [ESPN.com](espn.com).

The code for scraping the data is in notebooks 1.1 - 1.4 [here](https://github.com/tsansom/Springboard-Data-Science/tree/master/capstone_projects/nfl). 
