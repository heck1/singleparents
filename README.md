## singleparents
two approaches to a data wrangling problem in R

# The Problem and approach (i.e.: What is it good for?)

When handling any dataframe on the individual level that includes families of people, you are probably interested in the respective`s person status in that family. In my case, I was interested in the number of single parents in my dataframe. However, as it is the case 99% of the time, I found myself with imperfect data. There is information about the age and gender of the indidivuals + an ID for each family: 
```
Observations: 100
Variables: 3
$ `Familiennummer:` <chr> "475124", "574675", "558110", "439440", "439440", "465883", "528440", "571875", "571921", "571875",...
$ `Geb.-Datum:`     <dttm> 1996-12-31, 1996-12-21, 1989-10-14, 1970-04-20, 1970-01-01, 1985-08-13, 1996-12-31, 1988-04-20, 19...
$ `Geschlecht:`     <chr> "männlich", "männlich", "männlich", "männlich", "weiblich", "männlich", "männlich", "männlich", "mä...
```

What I want is to get to the individuals role in their respective family. So I approached this by looking at the age:
 - Oldest Person in a family with 2+ members AND kids < 18 in that family  is PROBABLY the father in most cases.
 - Second oldest Person in a family is PROBABLY the mother in most cases.
 - Kids under 18.
 
Since I was interested more in figuring out single parents, it was necessary to include an arbitrary defined age-range between the oldest person and the second oldest to minimize the amount of cases where a single parent would have kids over 18 (which would be classified as second oldest) and kids under 18
 

```



Thus, it is arbitrary to filter single persons:
