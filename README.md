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

        plot_correlation_matrix <- function(cor_matrix, title) {
          n <- nrow(cor_matrix)
          par(mar = c(4, 4, 4, 2)) # Adjust margins
          image(1:n, 1:n, t(cor_matrix[n:1, ]), 
                col = colorRampPalette(c("blue", "white", "red"))(20), 
                axes = FALSE, 
                main = title)
          
          axis(1, at = 1:n, labels = colnames(cor_matrix), las = 2, cex.axis = 0.8)
          axis(2, at = 1:n, labels = rev(rownames(cor_matrix)), las = 2, cex.axis = 0.8)
          
          for (i in 1:n) {
            for (j in 1:n) {
              text(i, n - j + 1, 
                   labels = round(cor_matrix[i, j], 2), 
                   cex = 0.8, 
                   col = ifelse(abs(cor_matrix[i, j]) > 0.5, "white", "black"))
            }
          }
        }
        
        cor_matrix_full <- cor(data[, sapply(data, is.numeric)])
        
        plot_correlation_matrix(cor_matrix_full, "Full Correlation Matrix")
        
        cool_vars <- c("price", "total_sqft", "age_since_reno", "total_rooms", "bath_per_bed", "sqft_diff_15")
        cor_matrix_cool <- cor(data[, cool_vars])
        
        plot_correlation_matrix(cor_matrix_cool, "Correlation Matrix for Cool Variables")


We want to plot the correlations higher than 0.75 (to fix)

        plot_filtered_correlation_matrix <- function(cor_matrix, threshold, title) {
          filtered_matrix <- ifelse(abs(cor_matrix) > threshold, cor_matrix, NA)
          
          par(mar = c(5, 5, 4, 4)) # Large margins to avoid errors
          
          # Set up the plot area
          n <- nrow(cor_matrix)
          image(1:n, 1:n, t(filtered_matrix[n:1, ]), 
                col = colorRampPalette(c("blue", "white", "red"))(20), 
                axes = FALSE, 
                main = title)
          
          axis(1, at = 1:n, labels = colnames(cor_matrix), las = 2, cex.axis = 0.8)
          axis(2, at = 1:n, labels = rev(rownames(cor_matrix)), las = 2, cex.axis = 0.8)
          
          for (i in 1:n) {
            for (j in 1:n) {
              if (!is.na(filtered_matrix[i, j])) {
                text(i, n - j + 1, 
                     labels = round(filtered_matrix[i, j], 2), 
                     cex = 0.8, 
                     col = ifelse(abs(filtered_matrix[i, j]) > 0.5, "white", "black"))
              }
            }
          }
        }
        
        cor_matrix_full <- cor(data[, sapply(data, is.numeric)])
        
        plot_filtered_correlation_matrix(cor_matrix_full, 0.75, "Filtered Correlation Matrix (|r| > 0.75)")


# Basic models

        Lorem Ipsum


# Group Lasso

We try group Lasso with two different sets of groups.

### 1. How real estate professionals evaluate property value

    Physical Characteristics of the Property:
    
        Living area size in square feet
        Total lot size in square feet
        Living area above ground level in square feet
        Basement area in square feet
        Number of bedrooms
        Number of bathrooms
        Number of floors
    
    Aesthetic and View Features:
    
        Indicates if property has waterfront view
        Quality level of property view
    
    Property Condition and Grade:
    
        Overall condition rating
        Overall grade rating
    
    Temporal Aspects:
    
        Year property was built
        Year property was last renovated
        Date of property listing
    
    Neighborhood Context:
    
        Living area size of 15 nearest properties in square feet
        Lot size of 15 nearest properties in square feet
        Property location zip code
    
    Geographic Location:
    
        Latitude coordinate of property location
        Longitude coordinate of property location

### 2 Variables expected to be highly correlated.

    House Size Metrics (Highly Correlated Features):
    
        Living area size in square feet
        Living area above ground level in square feet
        Basement area in square feet
        Living area size of 15 nearest properties in square feet
    
    Lot Size Metrics:
    
        Total lot size in square feet
        Lot size of 15 nearest properties in square feet
    
    Aesthetic and Condition Features:
    
        Indicates if property has waterfront view
        Quality level of property view
        Overall condition rating
        Overall grade rating
    
    Temporal Factors:
    
        Year property was built
        Year property was last renovated
        Date of property listing
    
    Geographic Features:
    
        Property location zip code
        Latitude coordinate of property location
        Longitude coordinate of property location
    
    Core Amenities:
    
        Number of bedrooms
        Number of bathrooms
        Number of floors
