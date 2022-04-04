## _Notes embedded with models, and copied below here_

# THIS FILE IS IN THE HANDOUTS FOLDER. COPY IT INTO YOUR CLASS NOTES

- [**Read the chapter on the website!**](https://ledatascifi.github.io/ledatascifi-2022/content/05/02_reg.html) It contains a lot of extra information we won't cover in class extensively.
- After reading that, I recommend [this webpage as a complimentary place to get additional intuition.](https://aeturrell.github.io/coding-for-economists/econmt-regression.html)

## Today

[Finish picking teams and declare initial project interests in the project sheet](https://docs.google.com/spreadsheets/d/1A6oQfhTBHHEb_EWSgfQv2KgsBoZm4V_BQWuvTjxnrdo/edit#gid=1508330834)


# Today is mostly about INTERPRETING COEFFICIENTS (5.2.4 in the book)

1. 25 min reading groups: Talk/read through two regression pages (5.2.3 and 5.2.4) 
    - Assemble your own notes. Perhaps in the "Module 4 notes" file, but you can do this in any file you want.
    - After class, each group will email their notes to Julio/me for participation. (Effort grading.)
1. 10 min: class builds joint "big takeaways and nuanced observations" 
1. 5 min: Interpret models 1-2 as class as practice. 
1. 20 min reading groups: Work through remaining problems below.
1. 10 min: wrap up  

---

## Notes

- Significance analyzed through B - component of variable functions

- Primary X variables
    - Continuous: defined at all points
    - increase = increase 
    - CAN br linear
- Binary
    - output linmited to two (2) resuts
    - 0 and 1, A and B, male and female, etc. 
- Categorical
    - components X1 and X2
    - multiple solutions possible for each model
    - Bv compares value to omitted category (to include in model)
- Controls
    - held all other controls constanmt ---> i.e. B1 estimated change in output (Y) per    change in X
    - "Other" controls and categorical controls
        - could be industry sector, size (rev, mkt cap), ownership (retail vs institutional, strategic vs sponsor)
- Comparing coefficients
    - largely dependent on "size"/incremental value delta of variables 
        - class ex: a change of 1 in a population survey is far less significant than if we were looking at returns, growth rates, etc. (where 1 = 100%)
        - NOT scalar; scale dependent. 
        


- R2: % of variation in y that our model explains
    - "higher is better"
    - when you are vars to a model, R2 cant decrease - tendency to overfut
    - use Adj. R2: 
    
- Transform variables ---> change interpretation of the relationship
    - logs, polynomials, etc. 
    
    
- Categorical variables
    - Binomial outcomes can be 1 and 0, otherwise use new dummy variable

- Interpretation is mechanical
    - A (1 unit) increase in X is associated with a (  ) channge in Y, holding all else constant
    
---    

## Model Notes
### Today. Work in groups. Refer to the lectures. 

You might need to print out a few individual regressions with more decimals.

1. Interpret coefs in model 1-4

##### Model 1
   - Predicted interest rate for borrowers with a credit score of 0 is 11.5%
   - A one unit increase in credit score is associated with a 0.01 decrease in interest rates (1 basis point), holding all other X constant
       - If score goes from 700 to 707, rate falls 7 basis points
       - if score goes from 600 to 606, rate falls 6 basis points
   
##### Model 2
   - A 1% increase in credit score is associated with a 6.07 basis point decrease in interest rates, holding all else constant
       - If credit score goes from 700 to 707, rate falls 6.07 basis points
       - If credit score goes from 600 to 606, rate falls 6.07 basis points 
 
##### Model 3
   - A 1 unit increase in credit score is associated with a 0.17% percent decrease in interest rates, holding all other X constant
       - If credit score from 700 to 707, rate falls by 6 basis points
       
       
##### Model 4
   - a one unbit
       -  If credit score from 700 to 707, rate falls by 
       
      
      
---
1. Interpret coefs in model 5
  - A 1% increase in credit score is associated with a 5.99 basis point decrease in interest rates holding constant log_LTV
    


1. Interpret coefs in model 6 (and visually?)
 - Int
1. Interpret coefs in model 7 (and visually? + comp to table)
1. Interpret coefs in model 8 (and visually? + comp to table)
1. Add l_LTV  to Model 8 and interpret (and visually?)


---



```python

```


```python
import pandas as pd
from statsmodels.formula.api import ols as sm_ols
import numpy as np
import seaborn as sns
from statsmodels.iolib.summary2 import summary_col # nicer tables

```


```python
url = 'https://github.com/LeDataSciFi/ledatascifi-2022/blob/main/data/Fannie_Mae_Plus_Data.gzip?raw=true'
fannie_mae = pd.read_csv(url,compression='gzip') 
```

## Clean the data and create variables you want


```python
fannie_mae = (fannie_mae
                  # create variables
                  .assign(l_credscore = np.log(fannie_mae['Borrower_Credit_Score_at_Origination']),
                          l_LTV = np.log(fannie_mae['Original_LTV_(OLTV)']),
                          l_int = np.log(fannie_mae['Original_Interest_Rate']),
                          Origination_Date = lambda x: pd.to_datetime(x['Origination_Date']),
                          Origination_Year = lambda x: x['Origination_Date'].dt.year,
                          const = 1
                         )
                  .rename(columns={'Original_Interest_Rate':'int'}) # shorter name will help the table formatting
             )

# create a categorical credit bin var with "pd.cut()"
fannie_mae['creditbins']= pd.cut(fannie_mae['Co-borrower_credit_score_at_origination'],
                                 [0,579,669,739,799,850],
                                 labels=['Very Poor','Fair','Good','Very Good','Exceptional'])

```


```python

```

## Statsmodels

As before, the psuedocode:
```python
model = sm_ols(<formula>, data=<dataframe>)
result=model.fit()

# you use result to print summary, get predicted values (.predict) or residuals (.resid)
```

Now, let's save each regression's result with a different name, and below this, output them all in one nice table:


```python
# one var: 'y ~ x' means fit y = a + b*X

reg1 = sm_ols('int ~  Borrower_Credit_Score_at_Origination ', data=fannie_mae).fit()

reg1b= sm_ols('int ~  l_credscore  ',  data=fannie_mae).fit()

reg1c= sm_ols('l_int ~  Borrower_Credit_Score_at_Origination  ',  data=fannie_mae).fit()

reg1d= sm_ols('l_int ~  l_credscore  ',  data=fannie_mae).fit()

# multiple variables: just add them to the formula
# 'y ~ x1 + x2' means fit y = a + b*x1 + c*x2
reg2 = sm_ols('int ~  l_credscore + l_LTV ',  data=fannie_mae).fit()

# interaction terms: Just use *
# Note: always include each variable separately too! (not just x1*x2, but x1+x2+x1*x2)
reg3 = sm_ols('int ~  l_credscore + l_LTV + l_credscore*l_LTV',  data=fannie_mae).fit()
      
# categorical dummies: C() 
reg4 = sm_ols('int ~  C(creditbins)  ',  data=fannie_mae).fit()

reg5 = sm_ols('int ~  C(creditbins)  -1', data=fannie_mae).fit()

```

Ok, time to output them:


```python
# now I'll format an output table
# I'd like to include extra info in the table (not just coefficients)
info_dict={'R-squared' : lambda x: f"{x.rsquared:.2f}",
           'Adj R-squared' : lambda x: f"{x.rsquared_adj:.2f}",
           'No. observations' : lambda x: f"{int(x.nobs):d}"}

# q4b1 and q4b2 name the dummies differently in the table, so this is a silly fix
reg4.model.exog_names[1:] = reg5.model.exog_names[1:]  

# This summary col function combines a bunch of regressions into one nice table
print('='*108)
print('                  y = interest rate if not specified, log(interest rate else)')
print(summary_col(results=[reg1,reg1b,reg1c,reg1d,reg2,reg3,reg4,reg5], # list the result obj here
                  float_format='%0.2f',
                  stars = True, # stars are easy way to see if anything is statistically significant
                  model_names=['1','2',' 3 (log)','4 (log)','5','6','7','8'], # these are bad names, lol. Usually, just use the y variable name
                  info_dict=info_dict,
                  regressor_order=[ 'Intercept','Borrower_Credit_Score_at_Origination','l_credscore','l_LTV','l_credscore:l_LTV',
                                  'C(creditbins)[Very Poor]','C(creditbins)[Fair]','C(creditbins)[Good]','C(creditbins)[Vrey Good]','C(creditbins)[Exceptional]']
                  )
     )
```

    ============================================================================================================
                      y = interest rate if not specified, log(interest rate else)
    
    ============================================================================================================
                                            1        2      3 (log) 4 (log)     5         6        7        8   
    ------------------------------------------------------------------------------------------------------------
    Intercept                            11.58*** 45.37*** 2.87***  9.50***  44.13*** -16.81*** 6.65***         
                                         (0.05)   (0.29)   (0.01)   (0.06)   (0.30)   (4.11)    (0.08)          
    Borrower_Credit_Score_at_Origination -0.01***          -0.00***                                             
                                         (0.00)            (0.00)                                               
    l_credscore                                   -6.07***          -1.19*** -5.99*** 3.22***                   
                                                  (0.04)            (0.01)   (0.04)   (0.62)                    
    l_LTV                                                                    0.15***  14.61***                  
                                                                             (0.01)   (0.97)                    
    l_credscore:l_LTV                                                                 -2.18***                  
                                                                                      (0.15)                    
    C(creditbins)[Very Poor]                                                                             6.65***
                                                                                                         (0.08) 
    C(creditbins)[Fair]                                                                         -0.63*** 6.02***
                                                                                                (0.08)   (0.02) 
    C(creditbins)[Good]                                                                         -1.17*** 5.48***
                                                                                                (0.08)   (0.01) 
    C(creditbins)[Exceptional]                                                                  -2.25*** 4.40***
                                                                                                (0.08)   (0.01) 
    C(creditbins)[Very Good]                                                                    -1.65*** 5.00***
                                                                                                (0.08)   (0.01) 
    R-squared                            0.13     0.12     0.13     0.12     0.13     0.13      0.11     0.11   
    R-squared Adj.                       0.13     0.12     0.13     0.12     0.13     0.13      0.11     0.11   
    R-squared                            0.13     0.12     0.13     0.12     0.13     0.13      0.11     0.11   
    Adj R-squared                        0.13     0.12     0.13     0.12     0.13     0.13      0.11     0.11   
    No. observations                     134481   134481   134481   134481   134481   134481    67366    67366  
    ============================================================================================================
    Standard errors in parentheses.
    * p<.1, ** p<.05, ***p<.01
    


```python
fannie_mae['int'].describe()
```




    count    135038.000000
    mean          5.238376
    std           1.289895
    min           2.250000
    25%           4.250000
    50%           5.250000
    75%           6.125000
    max          11.000000
    Name: int, dtype: float64


