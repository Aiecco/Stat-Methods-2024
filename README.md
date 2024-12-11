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

We create the data matrix X with only the price and the original (cleaned) features to use for gglasso.

        X <- data[,2:20]
        X=as.matrix(X)
    
        X <- scale(X)

# EDA

We aggregate some variables to check likely correlations as to apply high dim techniques later on

        data$total_sqft <- data$sqft_living + data$sqft_basement # Total area including basement
        data$bath_per_bed <- ifelse(data$bedrooms > 0, data$bathrooms / data$bedrooms, 0) # Bathroom to bedroom ratio
        data$total_rooms <- data$bedrooms + data$bathrooms # Total number of rooms
        data$sqft_diff_15 <- data$sqft_living - data$sqft_living15 # Difference in living area from nearest neighbors
        data$age_since_reno <- ifelse(data$yr_built > 0, 2024 - data$yr_built, 0) # Age of the property


We plot them

        par(mfrow=c(2,2))
        # Visualization: Price vs Total Area
        plot(data$total_sqft, data$price,
             main = "Property Price vs Total Area",
             xlab = "Total Area (sqft)",
             ylab = "Price",
             col = "blue",
             pch = 1,  # smaller points
             cex = 0.5)  # reduce point size
        abline(lm(price ~ total_sqft, data = data), col = "black")
        
        # Visualization: Price vs Total Number of Rooms
        plot(data$total_rooms, data$price,
             main = "Property Price vs Total Number of Rooms",
             xlab = "Total Number of Rooms (Bedrooms + Bathrooms)",
             ylab = "Price",
             col = "red",
             pch = 1,  # smaller points
             cex = 0.5)  # reduce point size
        abline(lm(price ~ total_rooms, data = data), col = "black")
        
        # Visualization: Price vs Bathroom/Bedroom Ratio
        plot(data$bath_per_bed, data$price,
             main = "Property Price vs Bathroom to Bedroom Ratio",
             xlab = "Bathroom to Bedroom Ratio",
             ylab = "Price",
             col = "purple",
             pch = 1,  # smaller points
             cex = 0.5)  # reduce point size
        abline(lm(price ~ bath_per_bed, data = data), col = "black")
        
        # Visualization: Price vs Living Area Difference
        plot(data$sqft_diff_15, data$price,
             main = "Property Price vs Living Area Difference",
             xlab = "Living Area Difference (Current vs Neighbors)",
             ylab = "Price",
             col = "orange",
             pch = 1,  # smaller points
             cex = 0.5)  # reduce point size
        abline(lm(price ~ sqft_diff_15, data = data), col = "black")



We also do a basic correlation analysis
        
        library(ggcorrplot)
        correlation_matrix <- cor(data[, sapply(data, is.numeric)])
        ggcorrplot(correlation_matrix, 
                   method = "square", 
                   lab = TRUE, 
                   lab_size = 3)

                   
# Basic models



        

# Group Lasso

We try group Lasso with two different sets of groups.

### 1. How real estate professionals evaluate property value

Physical characteristics include the living area size, lot size, above-ground and basement areas, number of bedrooms, bathrooms, and floors. Aesthetic and view features detail whether the property has a waterfront view and the quality of its overall view. Condition and grade focus on the property's overall condition and grade ratings. Temporal aspects provide information on the year the property was built, last renovated, and its listing date. Neighborhood context includes the average living area and lot size of the 15 nearest properties and the property’s zip code. Finally, the geographic location is specified by its latitude and longitude coordinates.

### 2 Variables expected to be highly correlated.

House size metrics capture highly correlated attributes such as the living area size (total, above ground, and basement) and the average size of nearby properties. Lot size metrics include total lot area and that of the 15 nearest properties. Aesthetic and condition features assess property appeal, such as waterfront views, quality and condition ratings, and overall grade. Temporal factors track the property's age, renovation history, and listing date. Geographic features provide location information, including zip code, latitude, and longitude. Lastly, core amenities detail the number of bedrooms, bathrooms, and floors.

## gglasso code

    library(gglasso)

    groupset_1 <- c(
      rep(1, 7),  # Group 1: Physical Characteristics
      rep(2, 2),  # Group 2: Aesthetic and View Features
      rep(3, 2),  # Group 3: Property Condition and Grade
      rep(4, 3),  # Group 4: Temporal Aspects
      rep(5, 3),  # Group 5: Neighborhood Context
      rep(6, 2)   # Group 6: Geographic Location)
    
    groupset_2 <- c(
      rep(1, 4), # House Size Metrics
      rep(2, 2), # Lot Size Metrics
      rep(3, 4), # Aesthetic and Condition Features
      rep(4, 3), # Temporal Factors
      rep(5, 3), # Geographic Features
      rep(6, 3)  # Core Amenities)

    fit_1 <- gglasso(x = X, y = Y, group = groupset_1, loss = 'ls')
    fit_2 <- gglasso(x = X, y = Y, group = groupset_2, loss = 'ls')

    coef.mat.1=fit$beta
    coef.mat.2=fit$beta


    par(mfrow=c(1,2))
    fit.cv.1=cv.gglasso(x=X,y=Y,group=groupset_1, nfolds=10, lambda.factor=0.0001)
    plot(fit.cv.1)
    
    fit.cv.2=cv.gglasso(x=X,y=Y,group=groupset_2, nfolds=10, lambda.factor=0.0001)
    plot(fit.cv.1)



    # lambda choice
    fit.1.lambda <- gglasso(x=X, y=Y, group=groupset_1, loss='ls',lambda.factor=0.0001)
    lambda_0_1 <- fit.1.lambda$lambda.1se
    lambda_1_1 <- fit.1.lambda$lambda.min

    fit.2.lambda <- gglasso(x=X, y=Y, group=groupset_2, loss='ls',lambda.factor=0.0001)
    lambda_0_2 <- fit.2.lambda$lambda.1se
    lambda_1_2 <- fit.2.lambda$lambda.min
    
