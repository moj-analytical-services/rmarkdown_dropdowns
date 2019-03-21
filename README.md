Displaying Multiple Charts in RMarkdown
================
21 March 2019 (11:31:12)

-   [Introduction](#introduction)
-   [Generate multiple plot PNG files](#generate-multiple-plot-png-files)
-   [Display the plots in R Markdown via drop down menu](#display-the-plots-in-r-markdown-via-drop-down-menu)

Introduction
============

This takes the example from here <https://walkerke.github.io/2016/12/rmd-dropdowns/> and creates a working example using other open data that you can more easily adapt.

First we install the package with devtools as it's not on CRAN

``` r
# devtools::install_github("walkerke/bsselectR")
```

Generate multiple plot PNG files
================================

Then we generate multiple plots using the excellent functional programming with plotting example here: <https://b-rodrigues.github.io/modern_R/functional-programming.html#functional-programming-and-plotting>

``` r
library(bsselectR)
library(tidyverse)
library(RCurl)
url_csv <- RCurl::getURL("https://raw.githubusercontent.com/b-rodrigues/modern_R/master/datasets/unemployment/all/unemployment_lux_all.csv")

unemp_lux_data <- read.csv(text = url_csv) %>%
  dplyr::filter(division %in% c('Beaufort','Bech','Berdorf'))

plots_tibble = unemp_lux_data %>%
  dplyr::group_by(division) %>%
  tidyr::nest() %>%
  dplyr::mutate(plot = purrr::map2(.x = data, .y = division, ~ggplot2::ggplot(data = .x) +
       ggplot2::theme_minimal() +
       ggplot2::geom_line(aes(year, unemployment_rate_in_percent, group = 1)) +
       ggplot2::labs(title = paste("Unemployment in", .y))))

purrr::map2(paste0(plots_tibble$division, ".png"), plots_tibble$plot, ggplot2::ggsave,path = "plots", dpi = 300)
```

Display the plots in R Markdown via drop down menu
==================================================

``` r
division_plots <- paste0(list.files("plots", full.names = TRUE))

names(division_plots) <- str_replace_all(division_plots, 
                                      c("\\.png" = "", 
                                        "plots/" = ""))

bsselect(division_plots, type = "img", selected = "Beaufort", 
         live_search = TRUE, show_tick = TRUE)
```

![](README_files/figure-markdown_github/unnamed-chunk-3-1.png)
