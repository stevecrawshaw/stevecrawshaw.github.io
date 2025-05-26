---
layout: post
title: POSIT Conf 2024 - Learnings from YT videos
date: 2025-05-26 10:00:00 +0100
categories: [R, Data Science]
tags:
- labelling
- tidypredict
- orbital
- duckdb
- models
- packages
---

# [Posit Conf 2024](https://www.youtube.com/playlist?list=PL9HYL-VRX0oSFkdF4fJeY63eGDvgofcbn)

## Shannon Pileggi's talk on [Context is King](https://www.youtube.com/watch?v=eoI9QZdHBMw&list=PL9HYL-VRX0oSFkdF4fJeY63eGDvgofcbn&index=19) (labelling)

1. Label fields in dataframes to provide context. Useful for reproducbility and business continuity.
2. The [labelled package](https://cran.r-project.org/web/packages/labelled/index.html) can help with this.
3. But also labels are just attributes so can be done with `attr()`.
4. Labels are supported in other packages like ggplot2 and GT
5. Use ggeasy to add labels to plots.
6. Use a metadata.csv file to add labels programmatically.
7. Use labelled::generate_dictionary() to create a data dictionary - which can then be saved as a table in your dB.

## [Automating package purpose documentation](https://www.youtube.com/watch?v=q4vmmlUEoQg&list=PL9HYL-VRX0oSFkdF4fJeY63eGDvgofcbn&index=39)

[Annotater package](https://annotater.liomys.mx/) can automatically give comments for the packages required.

Works with pacman.

# [Tidypredict and orbital](https://www.youtube.com/watch?v=Qnm1y0KPxVM&list=PL9HYL-VRX0oSFkdF4fJeY63eGDvgofcbn&index=46)

[Orbital](https://cran.r-project.org/web/packages/orbital/index.html) takes a finished model and decomposes it to a series of instructions (an Orbital object) that is somewhat portable. It minimises the number of variables and process in the model, and works with tree or linear - based models. It also includes the pre - processing steps from a tidymodels workflow, including recipes etc.

The orbital object can be implemented inside a database (MySQL, duckdb) as a SQL statement using the orbital_sql() function. Hence predictions can be made inside the database without using R or other data science languages.

The orbital object can also be converted to an R function and used to, for example, drive a shiny app without the overhead of running a full model in the background. Similarly you can convert that R function to javascript, python etc to use in other applications.

This is an interesting approach to model portability and deployment, allowing for efficient predictions without needing the full model context. This could be implemented in a duckdb database to drive an evidence app for example.