# Taxize-Workflow
 A workflow to clean the taxonomy for databasing to EcoVault

This script is meant to prepare the **Cameroon data** for **EcoVault**.
Data needs to be formatted into a clean taxonomy.

### 1. Install pacman to load packages
```r
# install.library("pacman")
```

### 2. Use pacman's loading function to load taxize and helper packages tidyverse, and magrittr
```r
pacman::p_load(tidyverse, taxize, magrittr)
```

### 3. Load the data using readxl
```r
messydata <- readxl::read_xlsx("input/messydata.xlsx", sheet = "fortaxize")
```

### 4. Check the data using head()
```r
head(messydata)

group     final_visitor  taxize_name species_code
  <chr>     <chr>          <chr>       <chr>       
1 Blattodea Blattodea sp.1 Blattodea   UP_0001_BLAT
2 Blattodea Blattodea sp.3 Blattodea   UP_0002_BLAT
3 Blattodea Blattodea sp.4 Blattodea   UP_0003_BLAT
4 Blattodea Blattodea sp.5 Blattodea   UP_0004_BLAT
5 Blattodea Blattodea sp.6 Blattodea   UP_0005_BLAT
6 Blattodea Blattodea sp.8 Blattodea   UP_0006_BLAT
```

N.B. In my case, this is real data from Cameroon. Before I readied the data for input, I added a column called "taxise_name", to help run the query against taxonomic databases. Blattodea sp. 1 became Blattodea, for e.g.

### 5. Set the API key
If you are using databases with a limit on the query rate, it might help acquiring an API key to relax the rate limit. For help, check the Taxize manual [Chapter 4 Authentication | taxize book](https://books.ropensci.org/taxize/authentication.html). I acquired one for NCBI.
```r
options(ENTREZ_KEY = "key")
# or
set_entrez_key("key")
```

### 6. In batches, get the taxonomy for the data
```R
# Set a vector for the data you want
tax_for_messydata <- c("kingdom", "phylum", "class",
                            "order", "family", "genus", "species")

# run the tax_name function from taxize. I used NCBI as the database.
# sometimes this requires manual input in cases of conflict, so keep an eye on the console.
                            
tax_batch1 <- tax_name(messydata$taxize_name[1:50], 
                    get = tax_for_messydata, 
                    db = 'ncbi')
tax_batch2 <- tax_name(messydata$taxize_name[51:100], 
                    get = tax_for_messydata, 
                    db = 'ncbi')
tax_batch3 <- tax_name(messydata$taxize_name[101:150], 
                    get = tax_for_messydata, 
                    db = 'ncbi')
# and so on...
```

### 7. Join all the batches together, and output them as a CSV.
```R
taxizeraw <- bind_rows(tax_batch1, tax_batch2, tax_batch3)

write.csv(taxizeraw, "output/taxizeraw.csv", row.names = FALSE)
```


### 8. Now, we switch to Excel. (This can be done in R too.)

- Use VLOOKUP to join the "taxize_query" from the "messydata.xlsx" to the "query" from the "taxizeraw.csv". 
- Save and close.
