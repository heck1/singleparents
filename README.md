## singleparents
approach to a data wrangling problem in R

# The Problem and approach (i.e.: What is it good for?)

When handling any dataframe on the individual level that includes families of people, you are probably interested in the respective`s person status in that family. In my case, I was interested in the number of single parents in my dataframe. However, as it is the case 99% of the time, I found myself with imperfect data. There is information about the age and gender of the indidivuals + an ID for each family: 
```
Observations: 100
Variables: 3
$ `ID:` <chr> "475124", "574675", "558110", "439440", "439440", "465883", "528440", "571875", "571921", "571875",...
$ `Geb.-Datum:`     <dttm> 1996-12-31, 1996-12-21, 1989-10-14, 1970-04-20, 1970-01-01, 1985-08-13, 1996-12-31, 1988-04-20, 19...
$ `Geschlecht:`     <chr> "männlich", "männlich", "männlich", "männlich", "weiblich", "männlich", "männlich", "männlich", "mä...
```

What I want is to get to the individuals role in their respective family. So I approached this by looking at the age:
 - Oldest Person in a family with 2+ members AND kids < 18 in that family  is PROBABLY the father in most cases.
 - Second oldest Person in a family is PROBABLY the mother in most cases.
 - Kids under 18.
 
Since I was interested more in figuring out single parents, it was necessary to include an arbitrary defined age-range between the oldest person and the second oldest to minimize the amount of cases where a single parent would have kids over 18 (which would be classified as second oldest) and kids under 18. I set this number to 25 as I wanted to minimize the scenario of single parent + kid under 18 + kid over 18. A single parent, in this case, would be defined as any oldest person without any person in a range of 25 years age difference.

 Some prerequisites need to be done, calculating a numeric age and calculating the occurences of `ID`:

```
df$age <- as.period(interval(df$`Geb.-Datum:`, Sys.Date()),
                    unit = "year")

df <- left_join(df, count(df, `ID:`))
df$family_count <- df$n
```

With this, it becomes arbitrary to filter single persons:
```
df %>% filter(fam_n < 2) %>% tally() %>% summarise(n = sum(n))

Using `n` as weighting variable
# A tibble: 1 x 1
      n
  <int>
1  45
```

Single parents, "regular" parents and the respective Kids are somewhat different:

```
alterunter <- c(0:18)
soz_sum <- soz %>%
  group_by(`Familiennummer:`) %>%
  filter (!is.na(`Familiennummer:`)) %>%
  mutate(
    
    
    erwachsene = ifelse(
      fam_n < 2,
      "Einzelperson",
      "Person in Familie"),
    
    kind_in_familie = ifelse(
      TRUE %in% (alterunter %in% alter),
      "kind im familienverbund",
      "-"),
    
    erwachsene = ifelse(
      fam_n > 1 &
        alter > 18 &
        alter >= max(alter) - 25 &
        kind_in_familie == "kind im familienverbund",
      "Zweitälteste Person im Familienverbund mit Kind",
      erwachsene),
    erwachsene = ifelse(
      fam_n > 1 &
        alter > 18 &
        alter >= max(alter) &
        kind_in_familie == "kind im familienverbund",
      "Älteste Person im Familienverbund mit Kind",
      erwachsene),
    erwachsene = ifelse(
      fam_n > 1 &
        alter > 18 &
        alter >= max(alter) &
        kind_in_familie == "kind im familienverbund" &
        erwachsene == "Älteste Person im Familienverbund mit Kind" & 
        !(any(erwachsene == "Zweitälteste Person im Familienverbund mit Kind" )) ,
      "Alleinerziehend",
      erwachsene),
    
    erwachsene = ifelse(
      fam_n > 1 &
        alter <= 18 ,
      "Kind unter 18 in Familie",
      erwachsene),
    erwachsene = ifelse(
      fam_n < 2 &
        alter <= 18 ,
      "Kind unter 18 - allein",
      erwachsene)
    
  ) 

```
