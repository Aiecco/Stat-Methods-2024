# Application of High-Dimensional Techniques to House Price Prediction


![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0001](https://github.com/user-attachments/assets/7d5d7bcf-05dc-4a44-b944-8765c002a442)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0002](https://github.com/user-attachments/assets/7a035690-a683-4527-b807-bd4f07116c57)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0003](https://github.com/user-attachments/assets/f54f4204-bcfc-4ba7-bc9d-651da25dbb18)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0004](https://github.com/user-attachments/assets/03d190f4-189d-441e-b034-4b3dfa098735)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0005](https://github.com/user-attachments/assets/9f014c26-03c8-4ad4-94f0-3adb89553d5c)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0006](https://github.com/user-attachments/assets/35adfb2e-6802-4dac-9db0-998208e002b3)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0007](https://github.com/user-attachments/assets/7230ce86-0e9e-4de8-a9da-815e2b3d875e)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0008](https://github.com/user-attachments/assets/910a19c0-e4b4-42c1-ba20-f563c7921469)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0009](https://github.com/user-attachments/assets/b9e71c8a-899b-4a02-8026-9e0a2a701872)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0010](https://github.com/user-attachments/assets/b22d0b4c-1622-412c-aa49-5af2cdb3736b)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0011](https://github.com/user-attachments/assets/1c6655f2-48e3-47cd-9145-a05fdb22c076)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0012](https://github.com/user-attachments/assets/271da1d3-50f9-4c65-9ad6-e3db68b7fa27)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0013](https://github.com/user-attachments/assets/da937021-6f3a-4637-92a5-0a600c1b55a4)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0014](https://github.com/user-attachments/assets/245da85c-2164-47bf-a9e7-832431ded06e)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0015](https://github.com/user-attachments/assets/14047e13-af8f-41ab-b6f7-078b095d9deb)
![Application of High Dimensional Techniques to House Price Prediction_pages-to-jpg-0016](https://github.com/user-attachments/assets/fcc1e7ab-5196-4e7c-8521-43194848d2c0)



This study investigates various statistical and machine learning methods to predict house prices using a dataset of real estate properties characterized by physical, locational, and quality attributes. The focus is on understanding the predictive power of different models and assessing the relationship between house features and their market value.
Dataset and Preprocessing

The dataset includes attributes like the number of bedrooms and bathrooms, lot size, scenic features, and locational data such as latitude, longitude, and average neighborhood characteristics. Preprocessing involved cleaning data, handling missing values, creating derived features (e.g., total area, bathroom-to-bedroom ratio), and scaling numerical variables. Features were grouped either based on real-estate domain expertise or statistical correlations to facilitate advanced modeling techniques like Group Lasso.
Models Explored

    Multivariate Linear Regression: A baseline model that explained ~69% of the variance in house prices. Significant predictors included living area, grade, and waterfront view. While the model performed well, some moderate-scale variability in predictions suggested room for improvement.

    Elastic Net Regularized Poisson Regression: Elastic Net was applied to handle multicollinearity and enforce sparsity. Although this approach enabled variable selection, its performance was slightly inferior to linear regression, explaining ~67.5% of the variance, possibly due to the linear nature of the relationships in the data.

    Group Lasso: Designed to select or discard entire groups of related variables, this method performed similarly to linear regression (~60% variance explained). Real-estate-based grouping emphasized physical characteristics, while statistically correlated grouping highlighted quality measures like grade and waterfront. No groups were entirely excluded, indicating their collective importance.

    Deep Neural Network: A medium-depth neural network demonstrated that the dataset's features could effectively predict house prices, achieving an adjusted R2R2 of ~78%. This experiment confirmed the dataset's integrity and the feasibility of achieving accurate predictions with non-linear models.

    Generalized Additive Model (GAM): GAM emerged as the best-performing model, explaining ~85% of the variance. It captured both linear and non-linear relationships by modeling predictors as smooth, flexible functions. This result highlights the complex interplay between house features and prices, justifying GAM's capability to address both linearity and non-linearity in the data.

Key Points

    1 House prices are influenced by both linear and polynomial relationships with features like living area, grade, and location.
    2 GAM's ability to model non-linear effects made it the most effective approach for this dataset.
    3 Despite advanced regularization techniques like Elastic Net and Group Lasso, simpler models like linear regression provided better performance.
    4 The neural network validated the dataset's reliability and offered a benchmark for achievable accuracy.



---------------------------------
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
        
        
        fit.cv.1=cv.gglasso(x=X,y=Y,group=groupset_1, nfolds=100, lambda.factor=0.0001)
        
        fit.cv.2=cv.gglasso(x=X,y=Y,group=groupset_2, nfolds=100, lambda.factor=0.0001)
        
        # lambda choice
        fit.1.lambda <- gglasso(x=X, y=Y, group=groupset_1, loss='ls',lambda.factor=0.0001)
        fit.2.lambda <- gglasso(x=X, y=Y, group=groupset_2, loss='ls',lambda.factor=0.0001)
        
        lambda_0_1 <- fit.cv.1$lambda.1se  # within 1 standard error
        lambda_1_1 <- fit.cv.1$lambda.min  # minimizing the cross-validation error
        lambda_0_2 <- fit.cv.2$lambda.1se
        lambda_1_2 <- fit.cv.2$lambda.min
        
### plotting

        par(mfrow=c(1,2))
        
        plot(fit.cv.1)
        plot(fit.cv.1)
                
        
        # preds
        fitted1_1se <- predict(fit.cv.1, newx = X, s = lambda_0_1)
        fitted1_min <- predict(fit.cv.1, newx = X, s = lambda_1_1)
        
        fitted2_1se <- predict(fit.cv.2, newx = X, s = lambda_0_1)
        fitted2_min <- predict(fit.cv.2, newx = X, s = lambda_1_1)
        
        # residuals
        residuals1_min <- Y - fitted1_min
        residuals2_min <- Y - fitted2_min





    plt <- cbind(Y, fitted1_1se, fitted1_min)
    matplot(
      plt,
      main = "Group Set 1",
      type = 'l',
      lwd = 1,
      col = 1:3,  # Color for the lines
    )
    
    grid()
    
    legend(
      "topright",
      legend = c("Actual", "Fitted-1se", "Fitted-min"),
      col = 1:3,
      lty = 1:3,  # Line styles
      lwd = 1,
      bty = "n"
    )



    plt <- cbind(Y, fitted2_1se, fitted2_min)
    matplot(
      plt,
      main = "Group Set 2",
      type = 'l',
      lwd = 1,
      col = 1:3,  # Color for the lines
    )
    
    grid()
    
    legend(
      "topright",
      legend = c("Actual", "Fitted-1se", "Fitted-min"),
      col = 1:3,
      lty = 1:3,  # Line styles
      lwd = 1,
      bty = "n"
    )



        # actual vs predicted (min lambda)
        
        plot(Y, fitted1_min, 
             main = "Actual vs Predicted (GroupSet 1)", 
             xlab = "Actual Y", ylab = "Predicted Y", 
             pch = 16, col = "black", cex=0.5)
        abline(0, 1, col = "red", lwd = 2)
        
        plot(Y, fitted2_min, 
             main = "Actual vs Predicted (GroupSet 2)", 
             xlab = "Actual Y", ylab = "Predicted Y", 
             pch = 16, col = "black", cex=0.5)
        abline(0, 1, col = "red", lwd = 2)
        
        
        # residuals vs predicted (min lambda)
        
        plot(fitted1_min, residuals1_min, 
             main = "Residuals vs Predicted (GroupSet 1)", 
             xlab = "Predicted Y", ylab = "Residuals", 
             pch = 16, col = "black", cex=0.5)
        abline(h = 0, col = "red", lwd = 2)
        
        plot(fitted2_min, residuals2_min, 
             main = "Residuals vs Predicted (GroupSet 2)", 
             xlab = "Predicted Y", ylab = "Residuals", 
             pch = 16, col = "black", cex=0.5)
        abline(h = 0, col = "red", lwd = 2)

        
        # group contribution
        
        group_contrib_1 <- sapply(unique(groupset_1), function(g) {
          sum(abs(coef.mat.1[groupset_1 == g]))
        })
        names(group_contrib_1) <- unique(groupset_1)
        
        group_contrib_2 <- sapply(unique(groupset_2), function(g) {
          sum(abs(coef.mat.2[groupset_2 == g]))
        })
        names(group_contrib_2) <- unique(groupset_2)
        
        # normalize contributions for better comparison
        group_contrib_1_norm <- group_contrib_1 / sum(group_contrib_1)
        group_contrib_2_norm <- group_contrib_2 / sum(group_contrib_2)
        
        
        barplot(group_contrib_1_norm, 
                main = "Group Contributions - Groupset 1", 
                xlab = "Group", 
                ylab = "Normalized Contribution", 
                col = "blue",
                ylim = c(0, max(group_contrib_1_norm)))
        
        barplot(group_contrib_2_norm, 
                main = "Group Contributions - Groupset 2", 
                xlab = "Group", 
                ylab = "Normalized Contribution", 
                col = "green",
                ylim = c(0, max(group_contrib_2_norm)))



### mse and r2

        n <- length(Y) # Number of observations
        p_1 <- sum(fit.cv.1$glmnet.fit$df[fit.cv.1$lambda == lambda_1_1])
        p_2 <- sum(fit.cv.2$glmnet.fit$df[fit.cv.2$lambda == lambda_1_2])
        
        # Groupset 1
        mse_1 <- mean((Y - fitted1_min)^2)
        ss_res_1 <- sum((Y - fitted1_min)^2)
        ss_tot_1 <- sum((Y - mean(Y))^2)
        r2_1 <- 1 - (ss_res_1 / ss_tot_1)
        adj_r2_1 <- 1 - ((1 - r2_1) * (n - 1)) / (n - p_1 - 1)
        
        # Groupset 2
        mse_2 <- mean((Y - fitted2_min)^2)
        ss_res_2 <- sum((Y - fitted2_min)^2)
        ss_tot_2 <- sum((Y - mean(Y))^2)
        r2_2 <- 1 - (ss_res_2 / ss_tot_2)
        adj_r2_2 <- 1 - ((1 - r2_2) * (n - 1)) / (n - p_2 - 1)

        
        # output
        cat("Groupset 1: \n")
        cat("MSE:", mse_1, "\n")
        cat("Adjusted R2:", adj_r2_1, "\n\n")
        
        cat("Groupset 2: \n")
        cat("MSE:", mse_2, "\n")
        cat("Adjusted R2:", adj_r2_2, "\n")



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


### metrics 

        gampreds <- predict(best_model, newdata = data_gam)
        
        gamres <- data_gam$price - gampreds
        
        ss_total <- sum((data_gam$price - mean(data_gam$price))^2)
        
        ss_residual <- sum(gamres^2)
        
        # metrics
        mse <- ss_residual / length(data_gam$price)
        cat("MSE: ", mse, "\n")
        
        rsq <- 1 - (ss_residual / ss_total)
        n <- nrow(data_gam)
        p <- length(coef(best_model)) - 1  # Excluding intercept
        adj_rsq <- 1 - ((1 - rsq) * (n - 1)) / (n - p - 1)
        cat("Adjusted R-squared: ", adj_rsq, "\n")

        MSE:  20475518275 
        Adjusted R-squared:  0.84781 


        # cross-validation (k-fold, k=100)
        cv_results <- cv.glm(data = data_gam, glmfit = best_model, K = 100)
        
        # cross-validation error
        cat("Cross-validation error: ", cv_results$delta[1], "\n")


### plots

        # actual vs predicted
        plot(data_gam$price, gampreds, main = "Actual vs Predicted",
             xlab = "Actual Price", ylab = "Predicted Price", pch = 16, col = "black", cex=0.5)
        abline(a = 0, b = 1, col = "red")


        # residuals
        plot(gampreds, gamres, main = "Residuals vs Predicted",
     xlab = "Predicted Price", ylab = "Residuals", pch = 16, col = "darkgreen", cex=0.5)
        abline(h = 0, col = "red")
        
        # Histogram of Residuals
        hist(gamres, main = "Histogram of Residuals", xlab = "Residuals", col = "lightblue", border = "black")
