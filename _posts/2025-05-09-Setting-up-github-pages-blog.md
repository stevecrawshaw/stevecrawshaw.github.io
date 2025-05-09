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

Looks good. I also included reference to a stylesheet vs2015.min.css

```css
  <!-- Begin highlight.js support -->
  <link rel="stylesheet" href="/js/highlightjs/styles/github.css">
  <link rel="stylesheet" href="/js/highlightjs/styles/vs2015.min.css">
  <link rel="stylesheet" href="/js/highlightjs/styles/ssms.css" /> 
```

Which styles any code block in a style that can be trialled on the [highlightjs demo site](https://highlightjs.org/demo)

I think this is enough tinkering with the styles for now.
