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

Positional spending data was only available for the 2013-2018 seasons so that is the time scope of this project. Previous year stats were taken from 2012-2017, where one year was added to match with the current year's positional salary.

After scraping the data, each set was saved into separate csv files. Merging the csv files together yielded a dataset of 39 independent variables and 192 observations (32 teams \* 6 years) as well as a set of potential dependent variables.  

## Feature Descriptions
Below is a description of each of the independent features that was scraped from the websites mentioned above.

**Team** - Team abbreviation (2 or 3 characters)

**Year** - Year of start of season

**Team Stats** (Previous Year)
* **TO** - Turnover differential

**Offensive Stats** (Previous Year)
* **Yds/G_rush_off** - Yards per game rushing by offense
* **TD_rush_off** - Number of rushing touchdowns by offense
* **Yds/G_pass_off** - Yards per game passing by offense
* **Pct_off** - Completion percentage by offense
* **TD_pass_off** - Number of passing touchdowns by offense
* **Sck_off** - Number of sacks allowed by offense
* **Rate_off** - Quarterback rating by offense
* **Pts/G_off** - Points per game by the offense
* **Pen Yds_off** - Penalty yards by offense

**Defensive Stats** (Previous Year)
* **Yds/G_rush_def** - Yards per game rushing allowed by defense
* **TD_rush_def** - Number of rushing touchdowns allowed by defense
* **Yds/G_pass_def** - Yards per game passing allowed by defense
* **TD_pass_def** - Number of passing touchdowns allowed by defense
* **Rate_def** - Quarterback rating allowed by defense
* **Sck_def** - Number of sacks by defense
* **Pct_def** - Completion percentage allowed by defense
* **Pts/G_def** - Points per game allowed by defense
* **Pen Yds_def** - Penalty yards by defense

**Salary Spending by Position** (Current Year)
* **QB** - Quarterback salary
* **RB** - Running back salary
* **WR** - Wide receiver salary
* **TE** - Tight end salary
* **OL** - Offensive line salary
* **DL** - Defensive line salary
* **LB** - Linebacker salary
* **S** - Secondary salary
* **ST** - Special teams salary

**Number of Players per Position** (Current Year)
* **nQB** - Number of quarterbacks
* **nRB** - Number of running backs
* **nWR** - Number of wide receivers
* **nTE** - Number of tight ends
* **nOL** - Number of offensive linemen
* **nDL** - Number of defensive linemen
* **nLB** - Number of linebackers
* **nS** - Number of secondary players
* **nST** - Number of special teams players


## Exploratory Data Analysis
