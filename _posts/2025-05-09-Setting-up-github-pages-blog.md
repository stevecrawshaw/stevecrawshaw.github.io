## Setting up the github pages blog

I wanted to make sure syntax highlighting worked for all the languages I am likely to use, so got Gemini to advise me how to do it. First part was to download language specific highlight JS zip file from [highlightjs](https://highlightjs.org/download). Then replace the highlight.min.js file in the repo with the downloaded one in the zip file. This should enable R and Python highlighting in code blocks. Let's check.

```r
library(tidyverse)
df <- read_csv("test.csv")
df |> glimpse()
```
Now Python

```Python
import polars as pl
df = pl.read_csv('data/test.csv')
df.glimpse()
```
