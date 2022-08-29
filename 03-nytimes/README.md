
# NYTimes health news

The following is an example of how to use the [NYTimes
API](https://developer.nytimes.com/) to get a list of articles related
to a particular term. Here we are searching articles that include the
term `"health"`, and that were published up to seven days ago since
2022-08-29.

``` r
# Preparing a function to use the GET method with the NYTimes API
library(httr)
f <- function(offset = 0) {
  GET(
    url = "https://api.nytimes.com/",
    path = "svc/search/v2/articlesearch.json",
    query = list(
      fq           = "health",
      facet        = "true",
      "begin_date" = gsub("-", "", Sys.Date() - 7),
      "api-key"    = Sys.getenv("NYT_APIKEY"),
      offset       = offset
      )
    )
}

# Retrieving 40 articles
keywords <- NULL
maxtries <- 5
niters   <- 10
i        <- 1
ntries   <- 1
while ((ntries < maxtries) && (i <= niters)) {
  
  res <- tryCatch(f(10 * (i-1)), error = function(e) e)
  
  # If it returns error
  if (inherits(res, "error")) {
    ntries <- ntries + 1
    next
  }
  
  # If the return code is not 200
  if (httr::status_code(res) != 200) {
    ntries <- ntries + 1
    next
  }
  
  # Parsing the data
  ans <- httr::content(res)
  keywords <- c(
    keywords,
    sapply(ans$response$docs, function(x) sapply(x$keywords, "[[", "value"))
    )
  
  # Incrementing and restarting
  i      <- i + 1
  ntries <- 1L
  
}
```

Now that we got the data, we can proceed to do some visualization

``` r
library(ggplot2)
library(ggwordcloud)

keywords <- as.data.frame(table(unlist(keywords)))

# Just to make sure that we were able to download info!
stopifnot(nrow(keywords) > 1)
ggwordcloud(keywords[, 1], keywords[, 2])
```

![](README_files/figure-gfm/preparing-data-1.png)<!-- -->

Finally, a table with the top 20 articles

``` r
tab <- keywords[order(-keywords[,2]),][1:20,]
colnames(tab) <- c("Keyword", "# Articles")
knitr::kable(tab, row.names = FALSE)
```

| Keyword                                              | \# Articles |
|:-----------------------------------------------------|------------:|
| Midterm Elections (2022)                             |          30 |
| United States Politics and Government                |          30 |
| Democratic Party                                     |          20 |
| Elections, House of Representatives                  |          20 |
| Abortion                                             |          10 |
| Bank of England                                      |          10 |
| Biden, Joseph R Jr                                   |          10 |
| Bolt, Usain                                          |          10 |
| Chernihiv (Ukraine)                                  |          10 |
| Civilian Casualties                                  |          10 |
| Crist, Charlie                                       |          10 |
| Defense and Military Forces                          |          10 |
| DeSantis, Ron                                        |          10 |
| Discrimination                                       |          10 |
| East Hampton (NY)                                    |          10 |
| Education (K-12)                                     |          10 |
| Elections, Governors                                 |          10 |
| Elections, Senate                                    |          10 |
| Endangered and Extinct Species                       |          10 |
| Far East, South and Southeast Asia and Pacific Areas |          10 |
