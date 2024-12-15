# Stat-Methods-2024

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

We aggregate some variables to check likely correlations as to apply high dim techniques later on

        data$total_sqft <- data$sqft_living + data$sqft_basement # Total area including basement
        data$bath_per_bed <- ifelse(data$bedrooms > 0, data$bathrooms / data$bedrooms, 0) # Bathroom to bedroom ratio
        data$total_rooms <- data$bedrooms + data$bathrooms # Total number of rooms
        data$sqft_diff_15 <- data$sqft_living - data$sqft_living15 # Difference in living area from nearest neighbors

    data[2:18] <- scale(data)

    
We create the data matrix X with only the original (cleaned) features to use for gglasso. Y is the true price.

    X <- data[,2:18]
    X = as.matrix(X)
        
    Y <- data[,1]
        

# EDA

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

MGLM actual vs pred

        predMGLM <- predict(MGLM.cv, newx = train_glm, s = opt.lam.MGLM)
        plot(test_glm, predMGLM, 
             xlab = "Actual Values", ylab = "Predicted Values", 
             main = "Actual vs Predicted")
        abline(0, 1, col = "black", lty = 2)

# Group Lasso

We try group Lasso with two different sets of groups.

### 1. How real estate professionals evaluate property value

Physical characteristics are the living area size, lot size, above-ground and basement areas, number of bedrooms, bathrooms, and floors. Aesthetic and view features say whether the property has a waterfront view and the quality of its overall view. Condition and grade focus on the property's overall condition and grade ratings. Temporal aspects say about the year the property was built, and its age. Neighborhood context has the average living area and lot size of the 15 nearest properties and the property’s zip code. The geographic location is specified by its latitude and longitude coordinates.

### 2 Variables expected to be highly correlated.

House size metrics capture highly correlated attributes such as the living area size (total, above ground, and basement) and the average size of nearby properties. Lot size metrics include total lot area and that of the 15 nearest properties. Aesthetic and condition features assess property appeal, such as waterfront views, quality and condition ratings, and overall grade. Temporal factors track the property's age, renovation history, and listing date. Geographic features provide location information, including zip code, latitude, and longitude. Lastly, core amenities detail the number of bedrooms, bathrooms, and floors.

## gglasso code

    library(gglasso)

    cor(X) # check correlations to create groupset_2
    
    groupset_1 <- c(1,1,1,1,1,1,2,2,3,3,1,5,6,6,5,5,4)

    groupset_2 <- c(3,3,1,1,3,4,4,4,4,1,1,2,2,2,1,1,3)
                    #gr1 area: sqft_living, sqft_above, sqft_basement, sqft_living15, sqft_lot, sqft_lot15
                    #gr2 spatial dim: lat, long, zipcode
                    #gr3 rooms and age: bedrooms, bathrooms, floors, age
                    #gr4 quality and features: grade, condition, view, waterfront


    fit_1 <- gglasso(x = X, y = Y, group = groupset_1, loss = 'ls')
    fit_2 <- gglasso(x = X, y = Y, group = groupset_2, loss = 'ls')

    coef.mat.1=fit_1$beta
    coef.mat.2=fit_2$beta


    par(mfrow=c(1,2))
    fit.cv.1=cv.gglasso(x=X,y=Y,group=groupset_1, nfolds=100, lambda.factor=0.0001)
    plot(fit.cv.1)
    
    fit.cv.2=cv.gglasso(x=X,y=Y,group=groupset_2, nfolds=100, lambda.factor=0.0001)
    plot(fit.cv.1)


    # lambda choice
    fit.1.lambda <- gglasso(x=X, y=Y, group=groupset_1, loss='ls',lambda.factor=0.0001)
    lambda_0_1 <- fit.1.lambda$lambda.1se
    lambda_1_1 <- fit.1.lambda$lambda.min

    fit.2.lambda <- gglasso(x=X, y=Y, group=groupset_2, loss='ls',lambda.factor=0.0001)
    lambda_0_2 <- fit.2.lambda$lambda.1se
    lambda_1_2 <- fit.2.lambda$lambda.min


What we plot (gglasso):


                # Heatmap
                image(1:nrow(coef.mat.1), 1:ncol(coef.mat.1), as.matrix(coef.mat.1), 
                      xlab = "Features", ylab = "Lambda", main = "Coefficient Heatmap: Group Set 1",
                      col = heat.colors(100))
                
                image(1:nrow(coef.mat.2), 1:ncol(coef.mat.2), as.matrix(coef.mat.2), 
                      xlab = "Features", ylab = "Lambda", main = "Coefficient Heatmap: Group Set 2",
                      col = heat.colors(100))
                
                # Cross-validation plot 
                plot(fit.cv.1$lambda, fit.cv.1$cvm, type = "b", log = "x",
                     xlab = "Log(Lambda)", ylab = "Cross-Validation Error",
                     main = "CV Error: Group Set 1")
                
                plot(fit.cv.2$lambda, fit.cv.2$cvm, type = "b", log = "x",
                     xlab = "Log(Lambda)", ylab = "Cross-Validation Error",
                     main = "CV Error: Group Set 2")
                
                # Lambda Path
                matplot(log(fit_1$lambda), t(as.matrix(coef.mat.1)), type = "l", 
                        xlab = "Log(Lambda)", ylab = "Coefficients", 
                        main = "Coefficient Path: Group Set 1", col = 1:nrow(coef.mat.1), lty = 1)
                
                matplot(log(fit_2$lambda), t(as.matrix(coef.mat.2)), type = "l", 
                        xlab = "Log(Lambda)", ylab = "Coefficients", 
                        main = "Coefficient Path: Group Set 2", col = 1:nrow(coef.mat.2), lty = 1)
                
                # Feature importance
                importance_1 <- rowSums(abs(as.matrix(coef.mat.1)))
                barplot(importance_1, main = "Feature Importance: Group Set 1", 
                        xlab = "Groups", ylab = "Sum of Absolute Coefficients", col = "blue")
                
                importance_2 <- rowSums(abs(as.matrix(coef.mat.2)))
                barplot(importance_2, main = "Feature Importance: Group Set 2", 
                        xlab = "Groups", ylab = "Sum of Absolute Coefficients", col = "green")
                
                # Residuals
                preds_1 <- predict(fit.cv.1, s = lambda_1_1, X)
                residuals_1 <- Y - preds_1
                plot(preds_1, residuals_1, main = "Residual Plot: Group Set 1", 
                     xlab = "Predicted", ylab = "Residuals", pch = 16, col = "blue")
                abline(h = 0, col = "red")
                
                preds_2 <- predict(fit.cv.2, s = lambda_1_2, X)
                residuals_2 <- Y - preds_2
                plot(preds_2, residuals_2, main = "Residual Plot: Group Set 2", 
                     xlab = "Predicted", ylab = "Residuals", pch = 16, col = "green")
                abline(h = 0, col = "red")


Prediction plots for Neural Network comparison
        
        pred_1_1se <- predict(fit_1, newx = X, s = lambda_0_1)
        pred_1_min <- predict(fit_1, newx = X, s = lambda_1_1)
        
        pred_2_1se <- predict(fit_2, newx = X, s = lambda_0_2)
        pred_2_min <- predict(fit_2, newx = X, s = lambda_1_2)


MSE

                cat("Model 1 - MSE:", mse_1, "R2:", r2_1, "MAE:", mae_1, "\n")
 Model 1 - MSE: 13572908807 R2: -11.50303 MAE: 48891.6 
 
                cat("Model 2 - MSE:", mse_2, "R2:", r2_2, "MAE:", mae_2, "\n")
 Model 2 - MSE: 11156390644 R2: -9.27699 MAE: 43118.46 


---------------------------------------------------
##### IMPORTAN Edit 
We just now spotted a big typo error in the code, the group lasso lambdas were not used on the results of cross-validated lasso but on the results of the first gglasso fit, which created all the bad fits of the code.
The rightly calculated code snippet is the following:

        lambda_0_1 <- fit.cv.1$lambda.1se  # within 1 standard error
        lambda_1_1 <- fit.cv.1$lambda.min  # minimizing the cross-validation error
        lambda_0_2 <- fit.cv.2$lambda.1se
        lambda_1_2 <- fit.cv.2$lambda.min
        
        # preds
        pred_1_min <- predict(fit.cv.1, newx = X, s = lambda_1_1)
        pred_2_min <- predict(fit.cv.2, newx = X, s = lambda_1_2)


Which yield:

        mse_1_min <- mean((Y - pred_1_min)^2)
        r2_1_min <- 1 - sum((Y - pred_1_min)^2) / sum((Y - mean(Y))^2)

        mse_2_min <- mean((Y - pred_2_min)^2)
        r2_2_min <- 1 - sum((Y - pred_2_min)^2) / sum((Y - mean(Y))^2)



Plots:

                fitted1_1se <- predict(fit.cv.1, newx = X, s = lambda_0_1)
                fitted1_min <- predict(fit.cv.1, newx = X, s = lambda_1_1)

        plt <- cbind(Y, fitted1_1se, fitted1_min)
        matplot(
          plt,
          main = "Predicted vs Actual",
          type = 'l',
          lwd = 2,
          col = 1:3,  # Color for the lines
          ylab = "Response Variable",
          xlab = "Observations"
        )
        
        grid()
        
        legend(
          "topright",
          legend = c("Actual", "Fitted-1se", "Fitted-min"),
          col = 1:3,
          lty = 1:3,  # Line styles
          lwd = 2,
          bty = "n"
        )

                fitted2_1se <- predict(fit.cv.2, newx = X, s = lambda_0_1)
                fitted2_min <- predict(fit.cv.2, newx = X, s = lambda_1_1)

        plt <- cbind(Y, fitted2_1se, fitted2_min)
        matplot(
          plt,
          main = "Predicted vs Actual",
          type = 'l',
          lwd = 2,
          col = 1:3,  # Color for the lines
          ylab = "Response Variable",
          xlab = "Observations"
        )
        
        grid()
        
        legend(
          "topright",
          legend = c("Actual", "Fitted-1se", "Fitted-min"),
          col = 1:3,
          lty = 1:3,  # Line styles
          lwd = 2,
          bty = "n"
        )
