---
title: "Bird Strikes Analsis Report"
author: "Akanksha Panwar"
date: "2024-08-25"
output: html_document
---

# Business Task:

A bird strike is strictly defined as a collision between a bird and an aircraft which is in
flight or on a take-off or landing roll. The term is often expanded to cover other wildlife
strikes - with bats or ground animals. Bird Strike is common and can be a significant
threat to aircraft safety. For smaller aircraft, significant damage may be caused to the
aircraft structure and all aircraft, especially jet-engine ones, are vulnerable to the loss
of thrust which can follow the ingestion of birds into engine air intakes. This has
resulted in several fatal accidents.

I am provided with a dataset containing data related to bird strikes from the year 2000 to 2011 and am going to analyse it.

It is a large dataset with 26 columns and more than 25,000 rows

# Preparing Data:

## Loading Packages
```{r}
library(tidyverse)
library(janitor)
library(reshape2)
```

## Importing Datasets
```{r}
bird_strikes <- read_csv("Bird Strikes data.csv")
```

# Cleaning Process:

- Changing column names to lowercase and replacing spaces with underscores in column names
- Changing flight_date column from character format to date format
```{r}
glimpse(bird_strikes)
bird_strikes <- clean_names(bird_strikes)
bird_strikes$flight_date <- as.POSIXct(bird_strikes$flight_date, format="%m/%d/%y", tz=Sys.timezone())
bird_strikes$flight_date <- as.Date(bird_strikes$flight_date, "%d/%m/%Y", tz=Sys.timezone())
```
![glimpse](https://github.com/user-attachments/assets/0106cbde-5d58-4e79-860f-531e3f9709c8)

- Checking if there's any duplicate data 
```{r}
bird_strikes[duplicated(bird_strikes), ]
```
![dup data](https://github.com/user-attachments/assets/7edfe2ff-d371-49ff-a0d5-ea09feab3054)

- Changing some columns to factor format or logical format
```{r}
bird_strikes$altitude_bin <- as.factor(bird_strikes$altitude_bin)
bird_strikes$wildlife_number_struck <- as.factor(bird_strikes$wildlife_number_struck)
bird_strikes$wildlife_size <- as.factor(bird_strikes$wildlife_size)
bird_strikes$effect_indicated_damage <- as.factor(bird_strikes$effect_indicated_damage)

summary(bird_strikes)
bird_strikes %>% select(altitude_bin, wildlife_number_struck, flight_date, aircraft_number_of_engines, wildlife_size, feet_above_ground, aircraft_make_model) %>% filter(!complete.cases(.))

bird_strikes %>% drop_na(flight_date) %>% summary()

```

* Changing to factor format and reordering the levels for 'when_phase_of_flight' column
```{r}
bird_strikes$when_phase_of_flight <- as.factor(bird_strikes$when_phase_of_flight)
levels(bird_strikes$when_phase_of_flight)
bird_strikes$when_phase_of_flight <- factor((bird_strikes$when_phase_of_flight), 
                                             levels = c("Parked",
                                                         "Taxi", 
                                                         "Take-off run", 
                                                         "Climb", 
                                                         "Descent", 
                                                         "Approach", 
                                                         "Landing Roll"))
levels(bird_strikes$when_phase_of_flight)
```
![phase levels](https://github.com/user-attachments/assets/e133e306-a872-4dab-b198-ea75eb370d96)

```{r}
bird_strikes$wildlife_number_struck <- factor(bird_strikes$wildlife_number_struck, 
                                               levels = c("1", "2 to 10", "11 to 100", "Over 100"))

bird_strikes$conditions_precipitation <- factor(bird_strikes$conditions_precipitation,
                                                 levels = c("None",
                                                            "Rain",
                                                            "Fog",
                                                            "Snow",
                                                            "Fog, Rain",
                                                            "Fog, Snow",
                                                            "Rain, Snow",
                                                            "Fog, Rain, Snow"))

bird_strikes$wildlife_size <- factor(bird_strikes$wildlife_size,
                                      levels = c("Small", "Medium", "Large"), exclude = NA)
```


# Analysing Data

```{r}
bird_strikes <- arrange(bird_strikes, flight_date)

glimpse(bird_strikes)
```
![analyse glimpse](https://github.com/user-attachments/assets/78157a67-aa27-472a-9737-3a8f2a91e996)

```{r}
bird_strikes %>% select(wildlife_number_struck_actual, cost_total, feet_above_ground, number_of_people_injured) %>% summary()

```
![summary](https://github.com/user-attachments/assets/82c50320-89f4-4b90-8ba3-1f9da797b0e6)

- On an average, less than 5 birds struck an aircraft, and usually strikes occured around 800 feet above the ground and people rarely got injured.

## Visualising Data

```{r}
bird_strikes %>% 
     drop_na(when_phase_of_flight) %>% 
     ggplot(aes(wildlife_number_struck_actual, when_phase_of_flight))+
     geom_col(aes(fill = wildlife_size))+ 
  labs(title = "Strike rate per flight phase", x = "Number of strikes", y = "Phase of flight")
```
![Strikes per phase](https://github.com/user-attachments/assets/3d887353-f718-4193-be25-d81e063c80c7)

```{r}
bird_strikes %>% drop_na(wildlife_number_struck, pilot_warned_of_birds_or_wildlife) %>% 
          ggplot(aes(wildlife_number_struck, flight_date))+
          geom_boxplot()+
          geom_point(alpha = 0.3, 
                     aes(size = feet_above_ground, colour = conditions_precipitation))+
          coord_flip()+
          facet_wrap(~conditions_sky)+ 
  labs(title = "Strikes over the years", x = "Number of wildlife per strike", y = "Time")
```
![Strikes over the years](https://github.com/user-attachments/assets/e8544ed7-d1ab-4166-985d-8e5b37a9fc50)

# Conclusion:

Based on the visualisation above, I have drawn the following insights, 
* The first graph indicates that the maximum number of strikes happened during the approach phase of the flights, followed by the take-off, climb and landing roll phases. 
* Most of the strikes were due to small size birds or wildlife. Large size wildlife rarely hit.
* The next graph shows that in maximum cases, throughout the years, the number of wildlife per strike was either 1 or less than 10. 
* Most of the strikes also happen in the range of 0-5000ft above ground, followed closely by 5000-10000ft above ground range. 
* It is also safe to conclude that the majority of strike cases happened when there was no precipitation of any kind.

### Thank you for reading!!
