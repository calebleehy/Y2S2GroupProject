---
title: "Top 100 Billboard"
author: "Caleb, Cheryl, Sarah, Weijun, Wenchen"
output:
  html_document: 
    toc: true
    toc_float: true
    keep_md: true
urlcolor: blue
---



# Introduction

This data records the top 100 songs rated by Billboard magazine every year from 1958 to 2021. We have 2 datasets: `billboard` and `audio`. `billboard` contains important song properties such as `performer` and `song_id`, which helps us to differentiate one song from another. It also contains numeric and datetime data of each song such as `week_id` and `previous_week_position` to help us see how a song is performing at the Top 100 Billboard chart. The `audio` dataset contains various features of each song such as its `danceability`, `loudness`, and whether it contains explicit lyrics or not, i.e. `spotify_track_explicit`.

# Descriptive statistics

The date, `week_id`, of each song is initially presented in the form “month/day/year” as a string. We convert it to “Date” class so that it can be easily sorted in chronological order, in “yyyy-mm-dd” format. All duplicated data in the audio data is removed before it is combined with the `audio` dataset by `song_id`, `song`, and `performer` as a single data frame, named `billboard_audio` via left_join. The data is then sorted by date in ascending order. We mutate a new column ‘spotify_genre_factor’ and summarize to have an idea of what’s the possible genre to work on in question 2. We then remove the columns `url`, `spotify_track_id` and `spotify_track_preview_url` as they do not contain any useful information in our visualizations. 

One key descriptive statistic we found is that the mean number of performers that appear on the Top 100 Billboard is approximately 349 a year, with the year with the most unique performers being 2020, with 465 performers. We also wanted to see how the duration a song stayed on the top 100 when it first made its debut compared to the total duration it was on the top 100 chart. The median and mean of the total duration were found to be 10.00 and 11.16, which is significantly higher than the median and mean of its debut at 1.000 and 1.005. This suggests that the majority of the songs dropped out of the top 100 and came back subsequently.

## Cleaning data


```r
audio = audio %>%
  distinct(song_id,.keep_all = TRUE)
billboard_audio = billboard %>%
  mutate(week_id = mdy(week_id)) %>%
  left_join(audio, by = c("song_id","song","performer")) %>%
  arrange(week_id) %>%
  mutate(week_number = ceiling(row_number()/100)) %>%
  mutate(spotify_genre_factor = as.factor(spotify_genre))
billboard_audio = billboard_audio[ , -c(1,12,13)]
```

## Key descriptive statistics

### Number of performers that appear on the top 100 chart per year


```r
no_of_performers <- billboard_audio %>%
  select(week_id, song_id, performer) %>%
  group_by(week_id = lubridate::floor_date(week_id, "year")) %>%
  distinct(performer, .keep_all = TRUE) %>%
  rename(Year = week_id) %>%
  filter(format(Year, format="%Y") != 2021) %>%
  count(Year) %>%
  rename(Number_of_Performers = n) %>%
  arrange(desc(Number_of_Performers))

no_of_performers
```

```
## # A tibble: 63 × 2
## # Groups:   Year [63]
##    Year       Number_of_Performers
##    <date>                    <int>
##  1 2020-01-01                  465
##  2 2018-01-01                  446
##  3 1970-01-01                  422
##  4 2019-01-01                  420
##  5 1966-01-01                  413
##  6 2017-01-01                  410
##  7 1971-01-01                  409
##  8 1967-01-01                  405
##  9 1975-01-01                  403
## 10 1976-01-01                  403
## # ℹ 53 more rows
```

```r
no_of_performers %>% 
  ungroup() %>%
  summarise(Mean_Number_of_Performers = mean(Number_of_Performers))
```

```
## # A tibble: 1 × 1
##   Mean_Number_of_Performers
##                       <dbl>
## 1                      349.
```

### Number of weeks a song stays on the top 100 chart


```r
song_counts <- billboard_audio %>%
  group_by(song_id) %>%
  count() %>%
  rename(duration = n) %>%
  arrange(desc(duration))
song_counts
```

```
## # A tibble: 29,389 × 2
## # Groups:   song_id [29,389]
##    song_id                                                    duration
##    <chr>                                                         <int>
##  1 RadioactiveImagine Dragons                                       87
##  2 SailAWOLNATION                                                   79
##  3 Blinding LightsThe Weeknd                                        76
##  4 I'm YoursJason Mraz                                              76
##  5 How Do I LiveLeAnn Rimes                                         69
##  6 Counting StarsOneRepublic                                        68
##  7 Party Rock AnthemLMFAO Featuring Lauren Bennett & GoonRock       68
##  8 Foolish Games/You Were Meant For MeJewel                         65
##  9 Rolling In The DeepAdele                                         65
## 10 Before He CheatsCarrie Underwood                                 64
## # ℹ 29,379 more rows
```

```r
song_counts_summary <- summary(song_counts$duration)
song_counts_summary
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    1.00    5.00   10.00   11.16   16.00   87.00
```

### Number of weeks a song stays on the top 100 chart when it debuts


```r
first_instance <- billboard_audio %>%
  group_by(song_id, instance) %>%
  summarise(first_week = min(week_id))
song_counts2 <- billboard_audio %>%
  inner_join(first_instance, by = c("song_id", "instance")) %>%
  group_by(song_id) %>%
  filter(min(week_id) == week_id) %>%
  mutate(length_on_chart = max(weeks_on_chart)) %>%
  select(song_id,length_on_chart)
song_counts2
```

```
## # A tibble: 29,389 × 2
## # Groups:   song_id [29,389]
##    song_id                                                  length_on_chart
##    <chr>                                                              <dbl>
##  1 Come Closer To Me (Acercate Mas)Nat King Cole                          1
##  2 Poor Little FoolRicky Nelson                                           1
##  3 PatriciaPerez Prado And His Orchestra                                  1
##  4 Splish SplashBobby Darin                                               1
##  5 Rebel-'rouserDuane Eddy His Twangy Guitar And The Rebels               1
##  6 OpThe Honeycones                                                       1
##  7 Don't Go HomeThe Playmates                                             1
##  8 Over And OverBobby Day                                                 1
##  9 One Summer NightThe Danleers                                           1
## 10 Do You Want To DanceBobby Freeman                                      1
## # ℹ 29,379 more rows
```

```r
song_counts2_summary <- summary(song_counts2$length_on_chart)
song_counts2_summary
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   1.000   1.000   1.000   1.005   1.000  24.000
```

# Question 1: How did the top 3 performers do on the Top 100 Billboard Chart?

## Introduction

In this question, we aim to compare how the top 3 performers did against each other based on the popularity of their songs. The comparison is made based on key popularity indicators such as `weeks_on_chart` and `peak_position` in qn1_1 and `week_position` in qn1_2. We defined top performers as those with the highest total number of weeks on the Billboard chart. We grouped the data by `song_id` and `performer` in sequence to extract the maximum `weeks_on_chart` of each song and the total number of weeks each performer has spent on the chart, respectively. Subsequently, we arranged them according to descending order, selected the top 3 using the slice function, and extracted the names of the performers into a vector using the pull function. 
 
## Methodology

Visualisation 1 aims to see whether a performer is truly more popular than another by their total `weeks_on_chart`. We will use the `peak_position` obtained by a song to see the popularity of a song from a different angle. `song_id` and `performer` will be used to group our data, so that we only keep the observation with the maximum number of weeks on the chart. From here, we will sum up the total number of weeks a performer has appeared on the chart to get our top performers arranged in descending order. They are: `Taylor Swift`,`Elton John`, then `Madonna`. A 2D-histogram of the songs of each performer will show us the distribution of songs, grouped by the number of weeks and the highest position obtained when it was on the chart.

Visualisation 2 is a facetted scatterplot with loess smoothing lines that aims to explore the mean weekly position of songs by the top three performers on the Billboard chart. Each point represents the mean weekly position of songs from each album by the top three performers, with the x-axis representing the week_id and the y-axis representing the mean_week_position. To get the visualisation, we grouped the data by `spotify_track_album`, and selected relevant variables, including `week_id`, `week_position`, `song_id`, `performer`, and `spotify_track_album`. We calculated the `mean_week_position` via mutate, `mean_week_position`  is useful to answer the question as the facetted charts based on it can provide us information on the range and distribution of weekly positions for each performer over time.

## Visualisations


```r
billboard_audio_by_performer <- billboard_audio %>%
  group_by(song_id) %>%
  slice_max(weeks_on_chart) %>%
  ungroup() %>%
  group_by(performer)

top_3_performers <- billboard_audio_by_performer %>%
  summarise(weeks_on_chart_per_song = sum(weeks_on_chart)) %>%
  arrange(desc(weeks_on_chart_per_song)) %>%
  slice_head(n = 3) %>%
  pull(performer)

qn1_1 = billboard_audio_by_performer %>%
  filter(performer %in% top_3_performers) %>%
  group_by(song_id) %>%
  slice_max(weeks_on_chart) %>%
  ungroup() %>%
  group_by(performer) %>%
  mutate(performer=factor(performer, levels=c("Taylor Swift","Elton John", "Madonna"))) %>%
  mutate(breaks = cut(weeks_on_chart, breaks = seq(0,50,5)))

ggplot(data=qn1_1, aes(x=weeks_on_chart, y=peak_position)) +
  geom_bin2d(binwidth=c(5,10)) +
  facet_wrap(~ performer) +
  scale_fill_gradient(name = "Count", low= "mistyrose", high = "red4") +
  expand_limits(x = 0, y = 0) +
  scale_x_continuous(expand = c(0, 0)) +
  scale_y_reverse(expand = c(0, 0), breaks=seq(0,100,10), minor_breaks=NULL) +
  labs(title = "Distribution of Songs of the Top 3 Performers", x = "Number of Weeks Song was on Top 100 Billboard Chart", y = "Highest Position reached by Song") +
  theme_bw() +
  theme(panel.spacing.x = unit(1.5, "lines"))
```

<img src="Top-100-Billboard_files/figure-html/unnamed-chunk-5-1.png" width="80%" style="display: block; margin: auto;" />


```r
qn1_2 <- billboard_audio_by_performer %>%
  filter(performer %in% top_3_performers) %>%
  group_by(spotify_track_album, add=TRUE) %>%
  select(week_id,week_position,song_id,performer,spotify_track_album) %>%
  mutate(mean_week_position = mean(week_position)) %>%
  mutate(spotify_track_album = factor(spotify_track_album)) %>%
  arrange(performer, week_id, spotify_track_album)
qn1_2
```

```
## # A tibble: 230 × 6
## # Groups:   performer, spotify_track_album [81]
##    week_id    week_position song_id                performer spotify_track_album
##    <date>             <dbl> <chr>                  <chr>     <fct>              
##  1 1970-09-26            93 Border SongElton John  Elton Jo… Elton John         
##  2 1971-02-27            26 Your SongElton John    Elton Jo… Elton John         
##  3 1971-05-15            44 FriendsElton John      Elton Jo… The Road To El Dor…
##  4 1972-02-19            42 LevonElton John        Elton Jo… Madman Across The …
##  5 1972-04-15            46 Tiny DancerElton John  Elton Jo… Madman Across The …
##  6 1972-08-12            35 Rocket ManElton John   Elton Jo… Honky Chateau      
##  7 1972-10-14            42 Honky CatElton John    Elton Jo… Honky Chateau      
##  8 1973-03-31            45 Crocodile RockElton J… Elton Jo… Don't Shoot Me I'm…
##  9 1973-07-14            58 DanielElton John       Elton Jo… Don't Shoot Me I'm…
## 10 1973-10-20            49 Saturday Night's Alri… Elton Jo… To Be Continued... 
## # ℹ 220 more rows
## # ℹ 1 more variable: mean_week_position <dbl>
```

```r
ggplot(data = qn1_2, aes(group = performer)) +
  geom_point(aes(x=week_id, y = mean_week_position),alpha=0.2) +
  geom_smooth(aes(x=week_id, y =mean_week_position),method="loess", color="red4", fill="mistyrose3", show.legend=FALSE) +
  facet_wrap(~ performer) +
  scale_y_reverse() +
  labs(title = "Weekly Positions of Songs by Top 3 Performers",
       x = "Year",
       y = "Mean Week Position") +
  theme_bw()
```

<img src="Top-100-Billboard_files/figure-html/unnamed-chunk-6-1.png" width="80%" style="display: block; margin: auto;" />

## Discussions

The 2D-histogram is arranged from left to right in descending order of total weeks on chart by performer. We can see that `Taylor Swift`,`Elton John` and `Madonna` had the majority of their songs that appeared on the chart for 0-5, 10-15 and 15-20 weeks respectively. This means that `Elton John` and `Madonna` had more songs that stayed longer on the chart than `Taylor Swift`. We can also observe that `Madonna` and `Elton John` had the most number of songs that reached the top 10 on the chart, while the bulk of `Taylor Swift`'s songs, especially the ones that appeared on the chart for 1-5 weeks, fluctuated between the top 10-100. So even though `Taylor Swift` had more songs on the chart than `Elton John` and `Madonna`, the songs of `Elton John` and `Madonna` were popular for a longer time, and achieved a higher peak position than the songs of `Taylor Swift`.

The scatterplot shows that the top three performers came from different eras whereby `Elton John` dominated the Billboard Top 100 in the 1960s to 2000s, `Madonna` came into the picture in the 1990s till the 2010s, and `Taylor Swift` entered the latest in the late 2000s till current. The smoothing line also shows that `Elton John` and `Taylor Swift` fluctuated over their careers while `Madonna` experienced an upwards trend in her average position. It is also observed that `Taylor Swift`'s rank is, on average, higher than `Madonna`'s. Although Visualisation 1 showed us that the majority of `Taylor Swift`’s songs are short-lived compared to `Elton John` and `Madonna`, and the peak position of `Taylor Swift`’s songs are lower than the other two artists. visualisation 2 helped us to understand that `Taylor Swift`'s songs on average, hit the Billboard list high and dropped out fast, while the others on average, hit a lower Billboard position and stayed in the rank.

# Question 2: How have the popularity and attributes of genres changed over the years?

## Introduction

We want to find out how the popularity and attributes of the 5 genres (pop, hip-hop, country, r&b and rock) have changed over the years between 1960 and 2020. For a particular genre such as pop, there are many different kinds of pop such as dance pop, indie pop and acoustic pop. In this question, we will be categorizing all the types of pop as pop and similarly for the other genres. The attributes of the songs that we will be comparing are the  `acousticness` and `valence`. The popularity of the genres will be determined by how many times songs from that genre appeared on the billboard top 100 in a particular year. 

## Methodology

For visualisation 1, we want to find out how the popularity of the 5 genres have changed between 1960 and 2020. First, we filter out songs whose `spotify_genre` contain the genre name for each of the genres, and then count how many times that genre occurs for each year. Then, using full_join, we join all the counts of the 5 genres together, by year. As some genres do not have any songs for some years, they will have `NA` values after the join. Hence, the mutate_all function replaces all `NA`s with 0. Then, using gather, we put all the columns of different genres under the new column `Genre` and the counts to `counts_by_year`. Using ggplot, we plot line graphs for each genre for the `counts_by_year` over the years.

For visualisation 2, to find out how the attributes of the 5 selected genres have changed, we used a bar plot with a facet_wrap by genre. The attributes that we will be analysing are the `acousticness` and `valence`. The x-axis represents the years - 1960, 1980, 2000 and 2020, and the y-axis represents the average of the attributes. As we are comparing the attributes, the bars are coloured according to the attributes. Bar plots are effective because in this case we are comparing categorical variables which are the song attributes. Together with a facet_wrap, it allows us to make comparisons across and within the genres easily. 

## Visualisations


```r
billboard_audio2 = billboard_audio %>%
  separate(week_id, into = c("year","month","day"),convert = TRUE)

billboard_pop = billboard_audio2 %>%
  filter(str_detect(spotify_genre, "pop")) %>%
  group_by(year) %>%
  arrange(year) %>%
  count() %>%
  rename(Pop = n)

billboard_country = billboard_audio2 %>%
  filter(str_detect(spotify_genre, "country")) %>%
  group_by(year) %>%
  arrange(year) %>%
  count() %>%
  rename(Country = n)

billboard_hiphop = billboard_audio2 %>%
  filter(str_detect(spotify_genre, "hip hop")) %>%
  group_by(year) %>%
  arrange(year) %>%
  count() %>%
  rename(Hiphop = n)

billboard_randb = billboard_audio2 %>%
  filter(str_detect(spotify_genre, "r&b") | str_detect(spotify_genre, "rhythm and blues")) %>%
  group_by(year) %>%
  arrange(year) %>%
  count() %>%
  rename("R&B" = n)

billboard_rock = billboard_audio2 %>%
  filter(str_detect(spotify_genre, "rock")) %>%
  group_by(year) %>%
  arrange(year) %>%
  count() %>%
  rename(Rock = n)

billboard_genres_count = billboard_pop %>%
  full_join(billboard_country, by = "year") %>%
  full_join(billboard_hiphop, by = "year") %>%
  full_join(billboard_randb, by = "year") %>%
  full_join(billboard_rock, by = "year") %>%
  mutate_all(~replace(., is.na(.), 0)) %>%
  gather(`Pop`:`Rock`, key = "Genre", value = "count_by_year") %>%
  filter((year != 1958) & (year != 1959) & (year != 2021))

ggplot(billboard_genres_count, aes(x = year, y = count_by_year, 
                                   group = Genre, color = Genre)) +
  geom_line(lwd = 1.1) +
  labs(x = "Year", y = "Number of times genre appeared in top 100 (by year)", 
       title = "Popularity of genres over the years") +
  scale_x_continuous(expand = c(0,0), limits = c(1960, 2021)) +
  scale_color_brewer(type = "qual", palette = "Set2") +
  theme(plot.title = element_text(hjust = 0.5, size = 20), legend.position = "top") +
  theme_hc()
```

<img src="Top-100-Billboard_files/figure-html/unnamed-chunk-7-1.png" width="80%" style="display: block; margin: auto;" />


```r
billboard_genres = billboard_audio2 %>%
  select(year, song_id,spotify_genre, acousticness, valence) %>%
  mutate(spotify_genre = str_sub(spotify_genre, 2, -2)) %>%
  mutate(spotify_genre = gsub("'", "", spotify_genre))
selected_genres = c("pop", "country", "hip hop", "r&b", "rhythm and blues", "rock")
billboard_genres <- billboard_genres %>%
  filter(str_detect(spotify_genre, paste(selected_genres, collapse = "|"))) %>%
  distinct(song_id, .keep_all = TRUE) %>%
  arrange(year) %>%
  mutate(spotify_genre = case_when(str_detect(spotify_genre, "hip hop") ~ "hip hop",
                                   str_detect(spotify_genre, "country") ~ "country",
                                   str_detect(spotify_genre, "r&b|rhythm and blues") ~ "r&b",
                                   str_detect(spotify_genre, "pop") ~ "pop",
                                   str_detect(spotify_genre, "rock") ~ "rock")) %>%
  group_by(year, spotify_genre) %>%
  summarise(mean_acousticness = mean(acousticness, na.rm = T),
            mean_valence = mean(valence, na.rm = T), .groups = "drop") %>%
  filter(year %in% c(1960, 1980, 2000, 2020))

  
billboard_genres = billboard_genres %>%
  gather(`mean_acousticness`: `mean_valence`, key = "characteristics", value = "value")
ggplot(billboard_genres, aes(year, value, fill = characteristics)) +
  geom_col(position = "dodge2") +
  geom_text(aes(year, value + 0.05, label = format(round(value, 2), nsmall = 2)), 
                position = position_dodge(width = 20), size = 2.4, hjust = 0.5) +
  facet_wrap(~ spotify_genre) +
  labs(title = "Attributes of Genres from 1960 to 2020",
       subtitle = "The higher the value, the greater the intensity of the attribute",
       x = "", y = "Value", fill = "") +
  scale_x_continuous(breaks=seq(1960,2020,20)) +
  scale_fill_manual(values = c("lightsalmon","goldenrod1"), labels = c("Acousticness", "Valence"), name = "Attributes") +
  theme(plot.title = element_text(hjust = 0.5, size = 15), legend.position = "bottom") +
  theme_few()
```

<img src="Top-100-Billboard_files/figure-html/unnamed-chunk-8-1.png" width="80%" style="display: block; margin: auto;" />

## Discussions

From visualisation 1, popularity of all 5 genres dropped for the year 2020, the most obvious drop being for pop, which could be because of the number of NA entries in `spotify_genre` for 2020 being 1876, significantly higher than the rest of the years, which have around 50 to 300 entries. `pop`, generally the most popular genre, has seen three general peaks, the highest being in recent years, from 1993 to 2016. `rock` has seen a surge in popularity from 1963 to the 1980s, with peak popularity in 1982, and after that, it has been generally declining in popularity. `country`, `hiphop` and `r&b` are relatively less popular, but rising to moderate popularity in the 1990s. Besides this, `country` music had another peak in the 1970s and `r&b` music had its peak around 1960. Between 1990 and 2000, all genres were increasing in popularity except `rock`, thus it can be said that the general taste for music switched from `rock` music to `pop` largely, but also `country`, `r&b`, and `hiphop` music.

From the bar plots, the `acousticness` of all 5 genres generally dropped from 1960 and 2020, which could be attributed to the increasing popularity of electronic music today. Advancements in technology have made it possible to produce music digitally using digital instruments such as tablets and MDI controllers. This could have led to a shift in preference from acoustic instruments such as the acoustic guitar to electric instruments such as the electric guitar, synthesizer and electric bass. The `valence` of the genres generally dropped as well with the exception of `rock`. This could be due to the COVID-19 pandemic which has amplified feelings of anxiety, sadness and worry. Thus, the songs that people listen to may be a reflection of their feelings, explaining the drop in `valency` of the genres in 2020.

# Reference

Electric vs. Acoustic instruments—Audio and sound. (2014, July 13). http://www.noiseaddicts.com/2014/07/electric-vs-acoustic-instruments/

Jthomasmock. (2021). Readme.md. https://github.com/rfordatascience/tidytuesday/blob/master/data/2021/2021-09-14/readme.md

Li, Y., Luan, S., Li, Y., & Hertwig, R. (2021). Changing emotions in the COVID-19 pandemic: A four-wave longitudinal study in the United States and China. Social Science & Medicine (1982), 285, 114222. https://doi.org/10.1016/j.socscimed.2021.114222

Makers, M. M. (2020, August 23). The difference between acoustic, electric and digital instruments. Metro Music Makers. https://www.metromusicmakers.com/2020/08/the-difference-between-acoustic-electric-and-digital-instruments/

Syed, A. (2020, October 26). Hot or not: Analyzing 60 years of billboard hot 100 data. Medium. https://towardsdatascience.com/hot-or-not-analyzing-60-years-of-billboard-hot-100-data-21e1a02cf304
