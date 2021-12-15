---
title: Text Anaylsis ft. Michael Scott
author: ''
date: '2021-02-24'
slug: text-analysis
categories: R Project
tags: 
- text analysis
- visualization
- sentiment analysis
subtitle: 'A text and sentiment analysis on the "Dinner Party" episode of The Office.'
summary: ''
authors: []
lastmod: '2021-12-14T19:26:04-08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---




## The Office - Season 4 x Episode 9: The Dinner Party

![The Dinner Party](featured.jpg)

This exercise is atext and sentiment analysis on the "Dinner Party" episode of The Office. The assignment was given in Dr. Allison Horst's ESM 244 course.

## Read in the text transcript

Source: https://www.officequotes.net/no4-09.php


```r
dinner_text <- read_delim(here("content","post", "2021-12-14-text-analysis", "dinnerparty2.txt"), 
                          "\t", escape_double = FALSE, col_names = FALSE, 
                          trim_ws = TRUE) %>% 
  rename(lines = X1)
```

- Each row is a a line of the transcript for each character.

Example: First line from the episode is from Stanley Hudson: 


```r
dinner_line1 <- dinner_text[1,]

dinner_line1 
```

```
## # A tibble: 1 x 1
##   lines                       
##   <chr>                       
## 1 Stanley: This is ridiculous.
```

## Tidy Data

  1. I separated the character names and their lines into two columns, `character` and `line`, separated by colon. 
  2. Removed the last four rows that are not character's lines but stage direction. 
  3. I  filtered the characters to include the characters that spoke the most words.


```r
dinner_tidy <- dinner_text %>% 
  separate(col = lines, into = c("character", "line"), sep = ":") %>% 
  slice(-(283:288)) %>% 
  filter(character %in% c("Andy", "Angela", "Dwight", "Michael", "Hunter's CD", "Jan", "Jim", "Pam")) 
```

## Tokenize

Here, we get word counts for each character for the episode. 


```r
dinner_tokens <- dinner_tidy %>% 
  unnest_tokens(word, line) 
```


```r
dinner_wordcount <- dinner_tokens %>% 
  count(character, word)
```

## Remove stop words


Most of the words in the transcript are stop words. To remove them, we use `anti_join` and the `stop_words` function. 


```r
dinner_nonstop_words <- dinner_tokens %>% 
  anti_join(stop_words)
```

We then recount with the stopwords removed. 

```r
nonstop_counts <- dinner_nonstop_words %>% 
  count(character, word) 
```

## Top 10 Words

Here, we find the top 10 words that each character said during that episode. 


```r
top_10_words <- nonstop_counts %>% 
  group_by(character) %>% 
  arrange(-n) %>% 
  slice(1:10)


top_10_words %>% 
  group_by(word) %>% 
  ggplot() +
  geom_bar(aes(reorder(word, n), n), stat = 'identity', fill = "red") +
  facet_wrap(~character, scales = "free") +
  coord_flip() +
  theme_calc() +
  labs(x = 'word')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

## Word Cloud

Let's make a word cloud for the top 5 characters who spoke the most:

Michael, Jan, Jim, Pam, Dwight

```r
nonstop_counts %>% 
  group_by(character) %>% 
  count() %>% 
  arrange(desc(n)) 
```

```
## # A tibble: 8 x 2
## # Groups:   character [8]
##   character       n
##   <chr>       <int>
## 1 Michael       275
## 2 Jan           190
## 3 Jim            88
## 4 Pam            72
## 5 Dwight         38
## 6 Angela         21
## 7 Andy           20
## 8 Hunter's CD     4
```



```r
dinner_top5 <- nonstop_counts %>% 
  filter(character %in% c("Michael", "Jan", "Jim", "Pam", "Dwight")) %>% 
  group_by(character) %>% 
  arrange(-n) %>% 
  slice(1:20)
```


```r
cloud <- ggplot(data = dinner_top5, aes(label = word)) +
  geom_text_wordcloud(aes(color = n, size = n), shape = "diamond") +
  scale_size_area(max_size = 6) +
  scale_color_gradientn(colors = c("red","blue","darkgreen")) +
  facet_wrap(~character) +
  theme_calc()

cloud
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

## Sentiment Analysis


```r
#afinn lexicon
get_sentiments(lexicon = "afinn")
```


### Sentiment analysis with afinn: 

First, bind words in `dinner_nonstop_words` to `afinn` lexicon:

```r
dinner_afinn <- dinner_nonstop_words %>% 
  inner_join(get_sentiments("afinn"))
```

Then, we find counts based on `afinn` lexicon and plot them:

```r
afinn_counts <- dinner_afinn %>% 
  count(character, value)

# Plot them: 
ggplot(data = afinn_counts, aes(x = value, y = n)) +
  geom_col(fill = "blue") +
  facet_wrap(~character) +
  theme_calc() +
  labs(y = "", x = "Lexicon Value")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />

```r
# Find the mean afinn score by characeter: 
afinn_means <- dinner_afinn %>% 
  group_by(character) %>% 
  summarize(mean_afinn = mean(value))

ggplot(data = afinn_means, 
       aes(x = fct_rev(as.factor(character)), 
           y = mean_afinn)) +
  geom_col(fill = "blue") +
  coord_flip() +
  theme_calc() +
  labs(x = "Character", y = "Mean")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-2.png" width="672" />
