
<style>
.reveal section p {
  color: black;
  font-size: .7em;
  font-family: 'Helvetica'; #this is the font/color of text in slides
}


.section .reveal .state-background {
    background: white;}
.section .reveal h1,
.section .reveal p {
    color: black;
    position: relative;
    top: 4%;}



</style>


Topic Modeling
========================================================
author: Chris Bail 
date: Duke University
autosize: true
transition: fade  
  website: https://www.chrisbail.net  
  github: https://github.com/cbail  
  Twitter: https://www.twitter.com/chris_bail

What is Topic Modeling?
========================================================

Latent Dirichlet Allocation
========================================================

Example: LDA of Scientific Abstracts
========================================================

<img src="LDA-concept.png" height="400" />


Running Your First Topic Model
========================================================

```r
library(topicmodels)
data("AssociatedPress")
```


Running Your First Topic Model
========================================================

```r
AP_topic_model<-LDA(AssociatedPress, k=10, control = list(seed = 321))
```



Running Your First Topic Model
========================================================

```r
library(tidytext)
library(dplyr)
library(ggplot2)

AP_topics <- tidy(AP_topic_model, matrix = "beta")

ap_top_terms <- 
  AP_topics %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)
```



Plot
========================================================

```r
ap_top_terms %>%
  mutate(term = reorder(term, beta)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip()
```

Plot
========================================================
<img src="AP plot.png" height="400" />

Now YOU try it
========================================================
1. Pick any of the datasets we’ve collected thus far (or one of the ones listed on the list of crowd-sourced datasets in the first lecture: http://bit.ly/1JA1CF3);

2. Prepare the data so that it can be analyzed in the `topicmodels` package

3. Run three models and try to identify an appropriate value for `k` (the number of topics)

Reading Tea Leaves
========================================================


Structural Topic Modeling
========================================================

<img src="stm_diagram.png" height="400" />


Political Blogs Data
========================================================


```r
google_doc_id <- "1LcX-JnpGB0lU1iDnXnxB6WFqBywUKpew" # google file ID
poliblogs<-read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", google_doc_id), stringsAsFactors = FALSE)
```


Pre-Process
========================================================


```r
library(stm)
processed <- textProcessor(poliblogs$documents, metadata = poliblogs)
```

Pre-Process
========================================================


```r
out <- prepDocuments(processed$documents, processed$vocab, processed$meta)
docs <- out$documents
vocab <- out$vocab
meta <-out$meta
```


Running a Structural Topic Model
========================================================


```r
First_STM <- stm(documents = out$documents, vocab = out$vocab,
              K = 10, prevalence =~ rating + s(day) ,
              max.em.its = 75, data = out$meta,
              init.type = "Spectral", verbose = FALSE)
```

Plot Top Words
========================================================


```r
plot(First_STM)
```

Plot
========================================================
<img src="topic plot.png" height="400" />

Find Exemplary Passages
========================================================


```r
findThoughts(First_STM, texts = poliblogs$documents,
     n = 2, topics = 3)
```



Choosing k
========================================================


```r
findingk <- searchK(out$documents, out$vocab, K = c(10, 30),
 prevalence =~ rating + s(day), data = meta, verbose=FALSE)

plot(findingk)
```


Working with meta-data
========================================================

Working with meta-data
========================================================

```r
predict_topics<-estimateEffect(formula = 1:10 ~ rating + s(day), stmobj = First_STM, metadata = out$meta, uncertainty = "Global")
```

Plot
========================================================

```r
plot(predict_topics, covariate = "rating", topics = c(3, 5, 9),
 model = First_STM, method = "difference",
 cov.value1 = "Liberal", cov.value2 = "Conservative",
 xlab = "More Conservative ... More Liberal",
 main = "Effect of Liberal vs. Conservative",
 xlim = c(-.1, .1), labeltype = "custom",
 custom.labels = c('Topic 3', 'Topic 5','Topic 9'))
```


Plot
========================================================
<img src="lib con.png" height="400" />

Plot Topic Prevalence over TIme
========================================================


```r
plot(predict_topics, "day", method = "continuous", topics = 3,
model = z, printlegend = FALSE, xaxt = "n", xlab = "Time (2008)")
monthseq <- seq(from = as.Date("2008-01-01"),
to = as.Date("2008-12-01"), by = "month")
monthnames <- months(monthseq)
axis(1,at = as.numeric(monthseq) - min(as.numeric(monthseq)),
labels = monthnames)
```


Plot
========================================================
<img src="topic over time.png" height="400" />

Limitations of topic models
========================================================





