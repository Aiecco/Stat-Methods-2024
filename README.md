# Stat-Methods-2024

    High dimensional methods for regression and its regularization

# Dataset
        https://www.kaggle.com/datasets/sukhmandeepsinghbrar/housing-price-dataset/data

### Columns:
        Date of property listing
        Property price in currency
        Number of bedrooms
        Number of bathrooms
        Living area size in square feet
        Total lot size in square feet
        Number of floors
        Indicates if property has waterfront view (0 for no, 1 for yes).
        Quality level of property view (0 to 4)
        Overall condition rating (1 to 5)
        Overall grade rating (1 to 13)
        Living area above ground level in square feet
        Basement area in square feet
        Year property was built
        Year property was last renovated (0 if never)
        Property location zip code
        Latitude coordinate of property location
        Longitude coordinate of property location
        Living area size of 15 nearest properties in square feet
        Lot size of 15 nearest properties in square feet

# Preprocess/clean

We clean the ID and convert the year-built of the house into its age. 

    data$date <- as.Date(substr(data$date, 1, 8), format = "%Y%m%d")
    data$yr_renovated[data$yr_renovated == 0] <- NA
    data <- data[!duplicated(data$id), ]
    current_year <- as.numeric(format(Sys.Date(), "%Y"))
    data$age <- ifelse(is.na(data$yr_renovated), current_year - data$yr_built, current_year - data$yr_renovated)

    data <- data[-c(1)]
    data <- data[-c(15)]

# EDA

We aggregate some variables to check likely correlations as to apply high dim techniques later on

        data$total_sqft <- data$sqft_living + data$sqft_basement
        data$sqft_diff_15 <- data$sqft_living - data$sqft_living15
        data$bath_per_bed <- ifelse(data$bedrooms > 0, data$bathrooms / data$bedrooms, 0)
        data$age_since_reno <- ifelse(data$yr_built > 0, 2024 - data$yr_built, 0)

We plot them


    plot(data$total_sqft, data$price, # Property Price vs. Total Area
         main = "Property Price vs Total Area",
         xlab = "Total Area (sqft)", 
         ylab = "Price",
         col = "blue", 
         pch = 16)
         
    boxplot(data$price ~ data$bedrooms, # Price Distribution by Number of Bedrooms
            main = "Price Distribution by Bedrooms",
            xlab = "Number of Bedrooms",
            ylab = "Price",
            col = "lightgreen")
            
    plot(data$sqft_diff_15, data$price, # Living Area Difference (Current vs. Nearest Neighbors)
         main = "Price vs Living Area Difference",
         xlab = "Living Area Difference (Current vs Neighbors)",
         ylab = "Price",
         col = "darkred", 
         pch = 16)
         
    hist(data$bath_per_bed, # Bathrooms per Bedroom
         main = "Histogram of Bathrooms per Bedroom",
         xlab = "Bathrooms per Bedroom",
         ylab = "Frequency",
         col = "skyblue",
         breaks = 10)

We also do a basic correlation analysis

        cor(data[, c("price", "sqft_living", "total_sqft", "bedrooms", "bathrooms", "sqft_lot")])
