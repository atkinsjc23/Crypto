# Intro
We explored predicting the price of Bitcoin (BTC) using macro-economic data from FRED. We considered using OLS, ARIMA, and ARIMAX models to model 
the price of BTC. 

# Variable Selection
A very small selection of variables were considered in the development of the models. Only variables with daily frequencies from FRED were considered. 

From FRED, we pulled
  1. CBBTCUSD (Coin Base's BTC price in USD)
  2. CBETHUSD (Coin Base's ETH price in USD - Not used)
  3. DBAA (Moody's Seasoned Baa Corporate Bond Yield)
  4. DCOILWTICO (Crude Oil Prices: West Texas Intermediate)
  5. OVXCLS (CBOE Crude Oil ETF Volatility Index)
  6. VIXCLS (CBOE Volatility Index: VIX)
  7. T10YIE	(10-Year Breakeven Inflation Rate)
  8. SP500 (S&P 500)

for the periood 2016-05-18 through 2022-03-07. The data from 2016-05-18 through 2022-02-07 was used to train the models. We used 2022-02-08 through 
2022-03-07 as our out-of sample (OOS) data to measure the performance of each model. 

Variables DBAA, VIX, T10YIE, and SP500 were chosen by there relevance to CCAR stress testing. See  
https://www.federalreserve.gov/newsevents/pressreleases/files/bcreg20200206a1.pdf

The remaining two variables OVXCLS and DCOILWTICO were chosen because of the relationship BTC has with energy. To mine a new BTC requires energy; thus,
the available supply of BTC is intermingled with energy prices, which will impact the price of BTC.

# OLS
The simplest models considered were ordinary least square (OLS) models. The script combinatorial_selection.ipynb computes every OLS model using
at least one and at most six dependent variables, where each variable can be lagged by 0, 7, and 14 days. We first enumerated each selection of
un-lagged variables, and then iterated through all possible lags for those selected variables. Selecting variables in this manner avoids models
that have SP500_lag0 and SP500_lag7 as predictors. Total number of models computed was:
  $$\sum_{i=1}^{6} (6 \choose i) (3^i).$$
A total of 4,095 models where computed.

Potential candidate models required that 
  1. the coefficients were significant (p-value of 0.05)
  2. low multicolinearity (variable inflation factor (VIF) less than 5) 
  3. goodness of fit (Jarque–Bera test with p-value of 0.05)
  4. low heteroskedasticity (Breusch–Pagan test with p-value of 0.05)

After applying these requirements, there was a total of 1,194 candidate models. Futher, we only considered the best lags (highest adjusted 
R-square) for each selection of variables. We kept the models with the highest adjusted R-squared value for each of the remainng candidate models. 
A total of 51 models remained.

The surviving candidate models were then sorted by highest adjusted R-squared values. Top models had variables and lags of:
  1. OVXCLS_lag0	DCOILWTICO_lag14	SP500_lag0  T10YIE_lag0 VIXCLS_lag14	
  2. OVXCLS_lag0	DCOILWTICO_lag14	SP500_lag0	T10YIE_lag0	
  3. OVXCLS_lag0	DCOILWTICO_lag0	  T10YIE_lag0	DBAA_lag0	  VIXCLS_lag7	
  4. OVXCLS_lag0	SP500_lag0	      T10YIE_lag0
  5. OVXCLS_lag0	DCOILWTICO_lag0	  T10YIE_lag0	DBAA_lag0	

We note that the difference between the top two OLS models is the incorporation of VIX as a dependent variable.

# ARIMAX
We then used the variables from the top two OLS models above and computed a pool of ARIMAX models.  For each choice of exogenous variables, we
considered orders (p,d,q) where d was either 0,1, or 2 and p and q were any non-empty subset of [1,2,3]. This amounts to
  $$ 3(2^{3} - 1)^2 = 147 $$
and a total of 2*147 = 294 ARIMAX models computed.

If an LU Decomposition error arose when training a model, we then omitted that model from consideration. Additionally, we required each model to 
satisfy
  1. the coefficients were significant (p-value of 0.05)
  2. goodness of fit (Jarque–Bera test with p-value of 0.05)
  3. low auto-correlation (ljung-box test with lags 10 through 15 and a p-value greater than .05)

Lastly, we chose the best model per exogenous variables by selecting the model with the lowest akaike information criterion (AIC). The models were:
  1. OVXCLS_lag0	DCOILWTICO_lag14	SP500_lag0  T10YIE_lag0 VIXCLS_lag14	with order (3,0,[2,3])
  2. OVXCLS_lag0	DCOILWTICO_lag14	SP500_lag0	T10YIE_lag0	              with order (3,0,[3])

# ARIMA
Lastly, we consider ARIMA models. We considered orders (p,d,q) where d was either 0,1,2, or 3 and p and q were any non-empty subset of [1,2,3,4].
This produced a total of 4 * (2^4 - 1)^2 = 900 models. We required that
  1. the coefficients were significant (p-value of 0.05)
  2. low auto-correlation (ljung-box test with lags 10 through 15 and a p-value greater than .02)
and then chose the model with the lowest AIC. The best order was (4,0,3)

# Conclusion
The best model was chosen using the script OOS_preformance.ipynb. The script predicted BTC during the OOS time period, and compares the predicted values
aginst the actual BTC values. We plotted and computed RMSE to determine the best model. The 13 candidate models were:
  1. top 10 OLS models
  2. best ARIMAX model for the given exogenous variables (see ARIMAX section above)
  3. best ARIMA model (see ARIMA section above)

The top 3 models with the lowest RMSE were:
  1. OLS: [DCOILWTICO_lag0, T10YIE_lag0, SP500] with RMSE of 8,703.88
  6. OLS: [DCOILWTICO_lag0, T10YIE_lag0, SP500, VIXCLS_lag0] with RMSE of 14,551.41
  7. ARIMA: order (4,0,3) with RMSE of 19,023.55

