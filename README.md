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

        data$total_sqft <- data$sqft_living + data$sqft_basement # Total area including basement
        data$bath_per_bed <- ifelse(data$bedrooms > 0, data$bathrooms / data$bedrooms, 0) # Bathroom to bedroom ratio
        data$total_rooms <- data$bedrooms + data$bathrooms # Total number of rooms
        data$sqft_diff_15 <- data$sqft_living - data$sqft_living15 # Difference in living area from nearest neighbors
        data$age_since_reno <- ifelse(data$yr_built > 0, 2024 - data$yr_built, 0) # Age of the property


We plot them

        # Visualization: Price vs Total Area
        plot(data$total_sqft, data$price,
             main = "Property Price vs Total Area",
             xlab = "Total Area (sqft)",
             ylab = "Price",
             col = "blue",
             pch = 16)
        
        # Visualization: Price vs Age
        plot(data$age_since_reno, data$price,
             main = "Property Price vs Age",
             xlab = "Age of Property (years since built or renovated)",
             ylab = "Price",
             col = "green",
             pch = 16)
        
        # Visualization: Price vs Total Number of Rooms
        plot(data$total_rooms, data$price,
             main = "Property Price vs Total Number of Rooms",
             xlab = "Total Number of Rooms (Bedrooms + Bathrooms)",
             ylab = "Price",
             col = "red",
             pch = 16)
        
        # Visualization: Price vs Bathroom/Bedroom Ratio
        plot(data$bath_per_bed, data$price,
             main = "Property Price vs Bathroom to Bedroom Ratio",
             xlab = "Bathroom to Bedroom Ratio",
             ylab = "Price",
             col = "purple",
             pch = 16)
        
        # Visualization: Price vs Living Area Difference
        plot(data$sqft_diff_15, data$price,
             main = "Property Price vs Living Area Difference",
             xlab = "Living Area Difference (Current vs Neighbors)",
             ylab = "Price",
             col = "orange",
             pch = 16)


We also do a basic correlation analysis


