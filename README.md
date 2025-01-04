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
        
        data[2:6] <- scale(data[2:6])
        data[8:22] <- scale(data[8:22])
    
We create the data matrix X with only the original (cleaned) features to use for gglasso. Y is the true price.

    X <- data[,2:18]
    X = as.matrix(X)
        
    Y <- data[,1]
        

# EDA

We plot them (do this before scaling)
        
        plot(data$bedrooms, data$price, main = "Price vs Bedrooms", xlab = "Bedrooms", ylab = "Price", pch = 19, col = "blue", cex = 0.8)
        abline(lm(price ~ bedrooms, data = data), col = "red", lwd = 2)
        
        plot(data$bathrooms, data$price, main = "Price vs Bathrooms", xlab = "Bathrooms", ylab = "Price", pch = 19, col = "purple", cex = 0.8)
        abline(lm(price ~ bathrooms, data = data), col = "red", lwd = 2)
        
        plot(data$sqft_living, data$price, main = "Price vs Sqft Living", xlab = "Sqft Living", ylab = "Price", pch = 19, col = "green", cex = 0.8)
        abline(lm(price ~ sqft_living, data = data), col = "red", lwd = 2)
        
        mean_price_floors <- tapply(data$price, data$floors, mean)
        barplot(mean_price_floors, main = "Average Price by Floors", xlab = "Floors", ylab = "Average Price", col = "orange", border = "black")
        
        plot(data$long, data$lat, main = "Location", xlab = "Longitude", ylab = "Latitude", pch = 19, col = "cyan", cex = 0.8)
        
        mean_price_condition <- tapply(data$price, data$condition, mean)
        barplot(mean_price_condition, main = "Average Price by Condition", xlab = "Condition", ylab = "Average Price", col = "yellow", border = "black")
        
        plot(data$sqft_lot, data$price, main = "Price vs Sqft Lot", xlab = "Sqft Lot", ylab = "Price", pch = 19, col = "orange", cex = 0.8)
        abline(lm(price ~ sqft_lot, data = data), col = "red", lwd = 2)
        
        mean_price_grade <- tapply(data$price, data$grade, mean)
        barplot(mean_price_grade, main = "Average Price by Grade", xlab = "Grade", ylab = "Average Price", col = "pink", border = "black")
        
        plot(data$age, data$price, main = "Price vs Age", xlab = "Age", ylab = "Price", pch = 19, col = "darkgreen", cex = 0.8)
        abline(lm(price ~ age, data = data), col = "red", lwd = 2)


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
                      zipcode + lat + long + sqft_living15 + sqft_lot15 + age + total_sqft + 
                      bath_per_bed + total_rooms + sqft_diff_15, data = data)

Results:

        Residuals:
             Min       1Q   Median       3Q      Max 
        -1177083  -101197   -11144    77629  4586690 
        
        Coefficients: (4 not defined because of singularities)
                      Estimate Std. Error t value Pr(>|t|)    
        (Intercept)     535932       1402 382.131  < 2e-16 ***
        bedrooms        -29948       1819 -16.466  < 2e-16 ***
        bathrooms        14607       2845   5.134 2.87e-07 ***
        sqft_living     147089       4093  35.937  < 2e-16 ***
        sqft_lot          5957       2023   2.945  0.00323 ** 
        floors            3164       1995   1.586  0.11265    
        waterfront      604646      17627  34.303  < 2e-16 ***
        view             44063       1668  26.413  < 2e-16 ***
        condition        18271       1570  11.641  < 2e-16 ***
        grade           107358       2572  41.738  < 2e-16 ***
        sqft_above       19001       3712   5.119 3.10e-07 ***
        sqft_basement       NA         NA      NA       NA    
        zipcode         -29200       1798 -16.239  < 2e-16 ***
        lat              86477       1514  57.121  < 2e-16 ***
        long            -35150       1878 -18.712  < 2e-16 ***
        sqft_living15    10921       2408   4.535 5.80e-06 ***
        sqft_lot15      -10106       2039  -4.957 7.23e-07 ***
        age              59858       2105  28.437  < 2e-16 ***
        total_sqft          NA         NA      NA       NA    
        bath_per_bed     21206       1931  10.983  < 2e-16 ***
        total_rooms         NA         NA      NA       NA    
        sqft_diff_15        NA         NA      NA       NA    
        ---
        Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
        
        Residual standard error: 204400 on 21418 degrees of freedom
        Multiple R-squared:  0.6912,	Adjusted R-squared:  0.691 
        F-statistic:  2820 on 17 and 21418 DF,  p-value: < 2.2e-16

### multivariate poisson reg (elastic net)
    x_glm <- data[, 2:22]
    y_glm <- data$price
    MGLM <- glmnet(
    x_glm, y_glm, standardize=FALSE, family="poisson", alpha=0.5)

    x_glm <- as.matrix(x_glm)  # needed for cv
    MGLM.cv <- cv.glmnet(x_glm, y_glm, nfolds=1000)

    opt.lam.MGLM <- c(MGLM.cv$lambda.1se)
    MGLM.coefs <- coef(MGLM.cv, s = opt.lam.MGLM)

Result:

        ****MGLM.coefs
        (Intercept)   536525.639
        bedrooms       -1009.958
        bathrooms          .    
        sqft_living   145781.906
        sqft_lot           .    
        floors             .    
        waterfront    526574.268
        view           40996.640
        condition      11319.671
        grade         109439.811
        sqft_above         .    
        sqft_basement      .    
        zipcode            .    
        lat            74933.499
        long          -11647.660
        sqft_living15   6652.210
        sqft_lot15         .    
        age            38344.499
        total_sqft         .    
        bath_per_bed   17377.920
        total_rooms        .    
        sqft_diff_15       .    


MGLM actual vs pred

        predMGLM <- predict(MGLM.cv, newx = x_glm, s = opt.lam.MGLM)
        plot(y_glm, predMGLM, 
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
        fit.2.lambda <- gglasso(x=X, y=Y, group=groupset_2, loss='ls',lambda.factor=0.0001)
        
        lambda_0_1 <- fit.cv.1$lambda.1se  # within 1 standard error
        lambda_1_1 <- fit.cv.1$lambda.min  # minimizing the cross-validation error
        lambda_0_2 <- fit.cv.2$lambda.1se
        lambda_1_2 <- fit.cv.2$lambda.min
        
    
        # preds
        pred_1_min <- predict(fit.cv.1, newx = X, s = lambda_1_1)
        pred_2_min <- predict(fit.cv.2, newx = X, s = lambda_1_2)
        

        
        mse_1_min <- mean((Y - pred_1_min)^2)
        r2_1_min <- 1 - sum((Y - pred_1_min)^2) / sum((Y - mean(Y))^2)
        
        mse_2_min <- mean((Y - pred_2_min)^2)
        r2_2_min <- 1 - sum((Y - pred_2_min)^2) / sum((Y - mean(Y))^2)

### plots

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



# GAM

        data_gam <- data
        
        data_gam$waterfront <- as.factor(data_gam$waterfront)
        data_gam$view <- as.factor(data_gam$view)
        data_gam$condition <- as.factor(data_gam$condition)
        data_gam$grade <- as.factor(data_gam$grade)
        data_gam$zipcode <- as.factor(data_gam$zipcode)
        
        
        
        gam_full <- gam(price ~ ., data = data_gam) # all predictors
        gam_null <- gam(price ~ 1, data = data_gam) # null model
        
        scope <- gam.scope(data_gam[, -1], arg = c("df=2", "df=3", "df=4"))
        
        gam_selected <- step.Gam(gam_null, scope = scope, trace = TRUE)



### best model
                
                #    gam(formula = price ~ s(bathrooms, df = 4) + s(sqft_living, df = 4) + 
                #    s(sqft_lot, df = 4) + s(floors, df = 4) + waterfront + view + 
                #    condition + grade + s(sqft_basement, df = 4) + zipcode + 
                #    s(lat, df = 4) + s(long, df = 4) + sqft_living15 + s(sqft_lot15, 
                #    df = 4) + s(age, df = 4) + s(total_sqft, df = 4) + bath_per_bed + 
                #    s(total_rooms, df = 4) + sqft_diff_15, data = data, trace = FALSE)


### GAM results
        
        Deviance Residuals:
             Min       1Q   Median       3Q      Max 
        -2365025   -56686     1803    52102  2598400 
        
        (Dispersion Parameter for gaussian family taken to be 20566563643)
        
            Null Deviance: 2.897908e+15 on 21435 degrees of freedom
        Residual Deviance: 4.380472e+14 on 21299 degrees of freedom
        AIC: 570010.5 
        
        Number of Local Scoring Iterations: NA 
        
        Anova for Parametric Effects
                                    Df     Sum Sq    Mean Sq    F value    Pr(>F)    
        s(bathrooms, df = 4)         1 8.0319e+14 8.0319e+14 39053.2306 < 2.2e-16 ***
        s(sqft_living, df = 4)       1 5.9577e+14 5.9577e+14 28967.8386 < 2.2e-16 ***
        s(sqft_lot, df = 4)          1 1.5139e+12 1.5139e+12    73.6079 < 2.2e-16 ***
        s(floors, df = 4)            1 1.3586e+11 1.3586e+11     6.6060 0.0101702 *  
        waterfront                   1 1.0205e+14 1.0205e+14  4962.1560 < 2.2e-16 ***
        view                         4 5.7806e+13 1.4452e+13   702.6710 < 2.2e-16 ***
        condition                    4 1.5844e+13 3.9609e+12   192.5913 < 2.2e-16 ***
        grade                       11 1.8924e+14 1.7203e+13   836.4770 < 2.2e-16 ***
        s(sqft_basement, df = 4)     1 3.9841e+12 3.9841e+12   193.7151 < 2.2e-16 ***
        zipcode                     69 4.7367e+14 6.8648e+12   333.7857 < 2.2e-16 ***
        s(lat, df = 4)               1 8.4291e+11 8.4291e+11    40.9846 1.566e-10 ***
        s(long, df = 4)              1 7.3167e+11 7.3167e+11    35.5759 2.492e-09 ***
        sqft_living15                1 1.6544e+12 1.6544e+12    80.4418 < 2.2e-16 ***
        s(sqft_lot15, df = 4)        1 2.3513e+11 2.3513e+11    11.4327 0.0007229 ***
        s(age, df = 4)               1 4.4281e+11 4.4281e+11    21.5306 3.503e-06 ***
        s(total_sqft, df = 4)        1 3.0901e+11 3.0901e+11    15.0249 0.0001064 ***
        bath_per_bed                 1 2.0479e+09 2.0479e+09     0.0996 0.7523422    
        s(total_rooms, df = 4)       1 5.9723e+11 5.9723e+11    29.0388 7.170e-08 ***
        sqft_diff_15                 1 5.4159e+11 5.4159e+11    26.3333 2.898e-07 ***
        Residuals                21299 4.3805e+14 2.0567e+10                         
        ---
        Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
        
        Anova for Nonparametric Effects
                                 Npar Df Npar F     Pr(F)    
        (Intercept)                                          
        s(bathrooms, df = 4)           3  22.99 7.438e-15 ***
        s(sqft_living, df = 4)         3  76.91 < 2.2e-16 ***
        s(sqft_lot, df = 4)            3  21.60 5.840e-14 ***
        s(floors, df = 4)              3  24.35 9.992e-16 ***
        waterfront                                           
        view                                                 
        condition                                            
        grade                                                
        s(sqft_basement, df = 4)       3  70.34 < 2.2e-16 ***
        zipcode                                              
        s(lat, df = 4)                 3  51.48 < 2.2e-16 ***
        s(long, df = 4)                3  15.16 7.523e-10 ***
        sqft_living15                                        
        s(sqft_lot15, df = 4)          3  14.56 1.815e-09 ***
        s(age, df = 4)                 3  61.05 < 2.2e-16 ***
        s(total_sqft, df = 4)          3 333.48 < 2.2e-16 ***
        bath_per_bed                                         
        s(total_rooms, df = 4)         3  24.54 7.772e-16 ***
        sqft_diff_15                                         
        ---
        Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
