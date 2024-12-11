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

We clean the ID and convert the year-built of the house into its age. We also scale the features but not the price.

    data$date <- as.Date(substr(data$date, 1, 8), format = "%Y%m%d")
    data$yr_renovated[data$yr_renovated == 0] <- NA
    data <- data[!duplicated(data$id), ]
    current_year <- as.numeric(format(Sys.Date(), "%Y"))
    data$age <- ifelse(is.na(data$yr_renovated), current_year - data$yr_built, current_year - data$yr_renovated)

    data <- data[-c(1, 2, 15, 16)]

    data[2:18] <- scale(data)

    
We create the data matrix X with only the original (cleaned) features to use for gglasso. Y is the true price.

    X <- data[,2:18]
    X = as.matrix(X)
        
    Y <- data[,1]
        

# EDA

We aggregate some variables to check likely correlations as to apply high dim techniques later on

        data$total_sqft <- data$sqft_living + data$sqft_basement # Total area including basement
        data$bath_per_bed <- ifelse(data$bedrooms > 0, data$bathrooms / data$bedrooms, 0) # Bathroom to bedroom ratio
        data$total_rooms <- data$bedrooms + data$bathrooms # Total number of rooms
        data$sqft_diff_15 <- data$sqft_living - data$sqft_living15 # Difference in living area from nearest neighbors


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


### multivariate linear reg
                MLR <- lm(price ~ bedrooms + bathrooms + sqft_living + sqft_lot + floors +
                       waterfront + view + condition + grade + sqft_above + sqft_basement +
                       zipcode + lat + long + sqft_living15 + sqft_lot15 + age, data = data)

Results:

        Residuals:
              Min        1Q    Median        3Q       Max 
        -7.36e-07 -3.00e-11  6.00e-11  1.50e-10  1.38e-09 
        
        Coefficients: (1 not defined because of singularities)
                        Estimate Std. Error    t value Pr(>|t|)    
        (Intercept)    5.405e+05  4.201e-11  1.287e+16   <2e-16 ***
        bedrooms       3.677e+05  7.398e-11  4.970e+15   <2e-16 ***
        bathrooms      7.485e-11  5.423e-11  1.380e+00    0.168    
        sqft_living    3.946e-11  7.260e-11  5.440e-01    0.587    
        sqft_lot      -1.557e-10  1.267e-10 -1.229e+00    0.219    
        floors         7.671e-12  6.084e-11  1.260e-01    0.900    
        waterfront    -1.241e-11  5.815e-11 -2.130e-01    0.831    
        view          -6.033e-12  4.719e-11 -1.280e-01    0.898    
        condition      1.576e-11  5.097e-11  3.090e-01    0.757    
        grade          3.389e-11  4.487e-11  7.550e-01    0.450    
        sqft_above     7.269e-11  7.782e-11  9.340e-01    0.350    
        sqft_basement -4.531e-11  1.110e-10 -4.080e-01    0.683    
        zipcode               NA         NA         NA       NA    
        lat           -6.270e-11  5.391e-11 -1.163e+00    0.245    
        long          -2.108e-11  4.867e-11 -4.330e-01    0.665    
        sqft_living15 -1.568e-13  5.590e-11 -3.000e-03    0.998    
        sqft_lot15     5.429e-11  7.241e-11  7.500e-01    0.453    
        age            5.543e-12  6.138e-11  9.000e-02    0.928    
        ---
        Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
        
        Residual standard error: 6.15e-09 on 21419 degrees of freedom
        Multiple R-squared:      1,	Adjusted R-squared:      1 
        F-statistic: 4.788e+30 on 16 and 21419 DF,  p-value: < 2.2e-16

### multivariate poisson reg (elastic net)
    train_glm <- data[, 2:18]
    test_glm <- data$price
    MGLM <- glmnet(
    train_glm, test_glm, standardize=FALSE, family="poisson", alpha=0.1)

    train_glm <- as.matrix(train_glm)  # needed for cv
    MGLM.cv <- cv.glmnet(train_glm, test_glm, nfolds=1000)

    opt.lam.MGLM <- c(MGLM.cv$lambda.1se)
    MGLM.coefs <- coef(MGLM.cv, s = opt.lam.MGLM)

Result:

        ****MGLM.coefs
        (Intercept)   540529.7
        bedrooms      356970.6
        bathrooms          .  
        sqft_living        .  
        sqft_lot           .  
        floors             .  
        waterfront         .  
        view               .  
        condition          .  
        grade              .  
        sqft_above         .  
        sqft_basement      .  
        zipcode            .  
        lat                .  
        long               .  
        sqft_living15      .  
        sqft_lot15         .  
        age                .  




### multivariate poisson reg with structural vars (elastic net)
    train_struc <- data[, 2:18]
    test_struc <- data$price
    STRUC <- glmnet(
    train_struc, test_struc, standardize=FALSE, family="poisson", alpha=0.1)

    train_struc <- as.matrix(train_struc)  # needed for cv
    STRUC.cv <- cv.glmnet(train_struc, test_struc, nfolds=1000)

    opt.lam.STRUC <- c(STRUC.cv$lambda.1se)
    STRUC.coefs <- coef(STRUC.cv, s = opt.lam.STRUC)

Result:

    ***STRUC.coefs
    (Intercept)   1.575673e+04
    price         9.708495e-01
    bedrooms      2.231746e-07
    bathrooms     .           
    sqft_living   .           
    sqft_lot      .           
    floors        .           
    waterfront    .           
    view          .           
    condition     .           
    grade         .           
    sqft_above    .           
    sqft_basement .           
    zipcode       .           
    lat           .           
    long          .           
    sqft_living15 .           
    sqft_lot15    .           
    age           . 
    
# Group Lasso

We try group Lasso with two different sets of groups.

### 1. How real estate professionals evaluate property value

Physical characteristics include the living area size, lot size, above-ground and basement areas, number of bedrooms, bathrooms, and floors. Aesthetic and view features detail whether the property has a waterfront view and the quality of its overall view. Condition and grade focus on the property's overall condition and grade ratings. Temporal aspects provide information on the year the property was built, and its age. Neighborhood context includes the average living area and lot size of the 15 nearest properties and the property’s zip code. Finally, the geographic location is specified by its latitude and longitude coordinates.

### 2 Variables expected to be highly correlated.

House size metrics capture highly correlated attributes such as the living area size (total, above ground, and basement) and the average size of nearby properties. Lot size metrics include total lot area and that of the 15 nearest properties. Aesthetic and condition features assess property appeal, such as waterfront views, quality and condition ratings, and overall grade. Temporal factors track the property's age, renovation history, and listing date. Geographic features provide location information, including zip code, latitude, and longitude. Lastly, core amenities detail the number of bedrooms, bathrooms, and floors.

## gglasso code

    library(gglasso)

    groupset_1 <- c(1,1,1,1,1,1,2,2,3,3,1,5,6,6,5,5,4)
    
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
    
