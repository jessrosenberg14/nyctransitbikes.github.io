data_loading_and_tidying
================

``` r
file_names <- list.files(path = "./data")
manhattan_stations <- read_csv("manhattan_stations.csv")

find_manhattan_rides <- function(data_name) {
  
  set.seed(8105)
  
  bike_rides_df <-
    read_csv(str_c("./data/", data_name)) %>% 
    janitor::clean_names() %>%
    filter(gender != 0, 
           year(starttime) - birth_year >= 18,
           year(starttime) - birth_year <= 85, 
           !(start_station_id == end_station_id & tripduration < 180)
    ) %>% 
    mutate(
      start_station_id = as.numeric(start_station_id), 
      end_station_id = as.numeric(end_station_id)
    ) %>% 
    filter(
      start_station_id %in% pull(manhattan_stations, id), 
      end_station_id %in% pull(manhattan_stations, id)
    ) %>% 
    sample_frac(.01)
  
  return(bike_rides_df)
    
}

manhattan_rides_df <- map_df(file_names, find_manhattan_rides)
```

Loading data:

-   Dropped missing values, including where `gender == 0`
-   Dropped riders younger than 18 or older than 85
-   Dropped any rides that were less than 3 minutes and started and
    ended at the same station

``` r
manhattan_rides_df <-
  manhattan_rides_df %>% 
  mutate(
    gender = ifelse(gender == 1, "male", "female") %>% 
      factor(levels = c("male", "female")),
    trip_min = tripduration/60, 
    day_of_week = wday(starttime, label = TRUE),
    start_date = format(starttime, format = "%m-%d"), 
    stop_date = format(stoptime, format = "%m-%d"), 
    year = as.factor(year(starttime)), 
    age = ifelse(year == 2019, 2019 - birth_year, 2020 - birth_year), 
    age_group = cut(age, 
                    breaks = c(-Inf, 25, 35, 45, 55, 65, 85), 
                    labels = c("18-25","26-35", "36-45", "46-55", "56-65", "66-85"))
  )
```

Added variables:

-   Created trip duration variable in minutes instead of seconds
-   Created day of the week variable
-   Created other date-related variables
-   Created age variables and age groups

``` r
write_csv(manhattan_rides_df, "manhattan_rides.csv")
```
