
<!-- README.md is generated from README.Rmd. Please edit the README.Rmd file -->

# Lab report \#4 - instructions

# Lab 4 Group Members: Analyn Seeman, Devon Katragadda, Ryan Jensen, Grace Wu

Follow the instructions posted at
<https://ds202-at-isu.github.io/labs.html> for the lab assignment. The
work is meant to be finished during the lab time, but you have time
until Monday (after Thanksgiving) to polish things.

All submissions to the github repo will be automatically uploaded for
grading once the due date is passed. Submit a link to your repository on
Canvas (only one submission per team) to signal to the instructors that
you are done with your submission.

# Lab 4: Scraping (into) the Hall of Fame

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
library(rvest)
```

    ## Warning: package 'rvest' was built under R version 4.4.3

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
url <- "https://www.baseball-reference.com/awards/hof_2024.shtml"
html <- read_html(url)
tables <- html_table(html, fill = TRUE)
data <- tables[[1]]
```

``` r
actual_col_names <- data[1, ]
colnames(data) <- actual_col_names
data <- data[-1, ]
```

``` r
library(dplyr)

data <- data %>%
  select(Name, Votes) %>%
  mutate(
    Votes = as.numeric(Votes),
    Name = gsub("X-", "", Name)
  )
```

``` r
library(Lahman)
```

    ## Warning: package 'Lahman' was built under R version 4.4.3

``` r
peopleneeded <- People %>%
  mutate(Name = paste(nameFirst, nameLast)) %>%
  select(playerID, Name)

unmatched <- anti_join(data, peopleneeded, by = "Name")
print(unmatched)
```

    ## # A tibble: 9 × 2
    ##   Name                Votes
    ##   <chr>               <dbl>
    ## 1 Adrian Beltré         366
    ## 2 Billy Wagner HOF      284
    ## 3 Carlos Beltrán        220
    ## 4 Francisco Rodríguez    30
    ## 5 Víctor Martínez         6
    ## 6 José Bautista           6
    ## 7 Bartolo Colón           5
    ## 8 Adrián González         3
    ## 9 José Reyes              0

``` r
data$Name <- iconv(data$Name, to = "ASCII//TRANSLIT")

data_joined <- left_join(data, peopleneeded, by = "Name")

hof_2024 <- data_joined %>%
  mutate(
    yearID = 2024,
    votedBy = "BBWAA",
    ballots = NA,  # Replace with actual number if known
    needed = NA,   # Replace with actual number if known
    inducted = ifelse(Votes >= 75, "Y", "N"),  # Assuming 75% is needed
    category = "Player",
    needed_note = NA
  ) %>%
  select(playerID, yearID, votedBy, ballots, needed, Votes, inducted, category, needed_note)

HallOfFame_updated <- bind_rows(HallOfFame, hof_2024)
```

``` r
write.csv(HallOfFame_updated, "HallOfFame.csv", row.names = FALSE)
```
