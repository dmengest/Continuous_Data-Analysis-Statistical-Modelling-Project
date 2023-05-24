
# Project on Continuous Data Analysis and Statistical Modeling for the 2016 US Presidential Election

## Introduction:

The project aimed to investigate the factors that influenced the vote rate for the Republican candidate, Donald Trump, in the 2016 US Presidential Election. The study focused on socio-economic and demographic characteristics and their association with the percentage of GOP (Republican Party) votes. The dataset used was obtained from Kaggle, consisting of 3141 observations representing different areas of the United States.

## Research Questions:

- Did the per capita money income obtained in the past 12 months influence the percentage of GOP votes in the 2016 presidential elections?
* What is the impact of socio-economic and demographic characteristics, 
including race, gender, education, household, veterans, and number of firms, on the percentage of GOP votes in the 2016 presidential elections?
- Can we predict the undecided counties (vote rate: 48%-52%) where future campaign efforts should be focused?

## Study Method:

- Creation of a new dataset based on selected variables.
- Examination of the distribution of individual variables through univariate analysis, removal of missing values, and transformation of skewed data.
- Analysis of bivariate relationships using correlation and scatterplot matrix.
- Splitting the dataset into training and testing datasets (50:50) and normalizing them.
- Implementation of forward stepwise selection for variable inclusion in the final model.
- Evaluation of the model's performance using logistic regression and comparison with the linear model.
- Fit the final linear model on the testing dataset for evaluation.
## Data Exploration:
The key variable chosen was per capita money income obtained in the past 12 months, along with variables related to race, household, gender, education, firms, and veterans. Univariate analysis revealed the distribution of variables, and log-transform was applied to skewed variables. No missing values were found. Bivariate relationships were examined through correlation and scatterplot matrix.

## Simple Linear Regression:
Simple linear regression was used to assess the impact of income on GOP vote percentage. The results indicated a marginal effect of income on GOP support (p-value < 0.0001).

## Multiple Linear Regression:
Forward stepwise regression was employed to build the final model, considering variables' significance and multicollinearity. The final model included income, white race, veterans, and firms, along with relevant interaction terms. Assumptions of linearity, normality, and equality of variance were checked and fulfilled.

## Interpretation:
The statistical results indicated that race (white) had the strongest direct influence on GOP votes, followed by income, interaction between white race and education, veterans, and firms. The R-squared value of the final model was 0.5716, explaining approximately 58% of the variance in GOP votes. The model's performance was validated using the testing dataset.

## General Linear Model (GLM):
The GLM was applied to dichotomize the response variable into win and lose categories. Logistic regression was performed based on the final model, providing odds ratios for different variables. White race showed a significant positive effect on the predicted GOP vote rate.

## Discussion:
The study analyzed county-level data for the 2016 US Presidential Election and developed a statistical model to predict GOP vote percentages. The model showed promising results in capturing the influence of socio-economic and demographic factors on voting patterns. Further analysis and comparison of predicted and observed data were conducted to assess the model's accuracy and identify any discrepancies.

Overall, the project provided insights into the factors affecting GOP vote percentages and offered a predictive model for future campaign efforts.

## Software used 

The project utilized R and SAS software for data analysis and modeling. Below is the code used in this project.

```
%let dir='/folders/myfolders/StatmodProject2020/';
libname moldproj &dir;
proc import out=moldproj.county
datafile="/folders/myfolders/StatmodProject2020/county_facts_dictionary.csv" dbms=csv replace;
getnames=yes;
datarow=2;
run;
proc import out=moldproj.election
datafile="/folders/myfolders/StatmodProject2020/US_County_Level_Presidential_Results_12-16_.csv"
dbms=csv replace;
getnames=yes;
datarow=2;
run;
proc print data=moldproj.election (obs=5);
run;
/*Choose relevant variables and create a new dataset*/
%let x = white income household gender education firms veterans;
data election (keep=county y &x);
set moldproj.election;
rename county_name=county per_gop_2016=y RHI125214=white INC910213=income
HSD310213=household SEX255214=gender EDU635213=education SBO001207=firms
VET605213=veterans;
run;
proc print data=election (obs=10);
run;
*#####################################;
*### STEP 1: Descriptives Analysis ###;
*#####################################;
/*1. Univariate exploration of the data*/
proc univariate plot data=election;
var y &x;
run;
/* Log-transform the right skewed data: */
/* firms (skewness: 16.1494055) */
/* veterans (skewness: 8.26233385) */
data election;
set election;
if firms ne 0 then
firms=log(firms);
veterans=log(veterans);
run;
proc means data=election nmiss;
var y &x;
run;
proc univariate plot data=election;
var &x;
run;
/*2. Bivariate relationships*/
title "Correlation matrix and scatter-plots";
proc sgscatter data=election;
matrix &x / diagonal=(histogram normal);
run;
title;
data election;
set election;
deel=firms/veterans;
run;

proc sgscatter data=election;
matrix &x deel/ diagonal=(histogram normal);
run;
title;
proc corr data=election nosimple;
var &x;
run;
*#####################################;
*### STEP 2: Data spliting 50%:50% ###;
*#####################################;
proc surveyselect data=election out=train_test_split method=srs samprate=0.5
outall seed=123 noprint;
samplingunit y;
run;
data train_set;
set train_test_split;
where Selected=1;
run;
data test_set;
set train_test_split;
where Selected=0;
run;
/* Standardize data! */
proc stdize data=train_set method=mean out=cent_train;
var &x;
run;
proc stdize data=test_set method=mean out=cent_test;
var &x;
run;
proc print data=cent_test;
run;
data cent_train;
set cent_train;
intwhiteincome=white*income;
intwhitehousehold=white*household;
intwhitegender=white*gender;
intwhiteeducation=white*education;
intwhitefirms=white*firms;
intwhiteveterans=white*veterans;
intincomehousehold=income*household;
intincomegender=income*gender;
intincomeeducation=income*education;
intincomefirms=income*firms;
intincomeveterans=income*veterans;
inthouseholdgender=household*gender;
inthouseholdeducation=household*education;
inthouseholdfirms=household*firms;
inthouseholdveterans=household*veterans;
intgendereducation=gender*education;
intgenderfirms=gender*firms;
intgenderveterans=gender*veterans;
inteducationfirms=education*firms;
inteducationveterans=education*veterans;
intfirmsveterans=firms*veterans;
run;
data cent_test;
set cent_test;
intwhiteincome=white*income;
intwhitehousehold=white*household;
intwhitegender=white*gender;
intwhiteeducation=white*education;
intwhitefirms=white*firms;
intwhiteveterans=white*veterans;
intincomehousehold=income*household;
intincomegender=income*gender;
intincomeeducation=income*education;
intincomefirms=income*firms;
intincomeveterans=income*veterans;
inthouseholdgender=household*gender;
inthouseholdeducation=household*education;
inthouseholdfirms=household*firms;

inthouseholdveterans=household*veterans;
intgendereducation=gender*education;
intgenderfirms=gender*firms;
intgenderveterans=gender*veterans;
inteducationfirms=education*firms;
inteducationveterans=education*veterans;
intfirmsveterans=firms*veterans;
run;
%let int_x = intwhiteincome intwhitehousehold intwhitegender intwhiteeducation intwhitefirms intwhiteveterans
intincomehousehold intincomegender intincomeeducation intincomefirms intincomeveterans
inthouseholdgender inthouseholdeducation inthouseholdfirms inthouseholdveterans
intgendereducation intgenderfirms intgenderveterans
inteducationfirms inteducationveterans
intfirmsveterans;
*#####################################;
*### STEP 3: Forward selection ###;
*#####################################;
/* 3.1 Simple Linear regression */
/* t:0.0376 F:61.83 */
proc reg data=train_set;
model y=income / vif;
title "Simple linear regression";
run;
quit;
title;
/* First extra predictor*/
/* income need to stay in the model */
/* 3.2 determine the second fixed variable via t-value and p-value */
/* t:28.40 F:449.92 */
proc reg data=cent_train;
model y=income white;
run;
quit;
/* t:-9.36 F:76.41 */
proc reg data=cent_train;
model y=income household;
run;
quit;
/* t:-5.10 F:44.42 */
proc reg data=cent_train;
model y=income gender;
run;
quit;
/* t:3.22 F:36.28 */
proc reg data=cent_train;
model y=income education;
run;
quit;
/* t:-15.92 F:162.64 */
proc reg data=cent_train;
model y=income firms;
run;
quit;
/* t:-18.27 F:204.34 */
proc reg data=cent_train;
model y=income veterans;
run;
quit;
/* we choose white as a candidate varible, now check for it */
/* r_square:0.3626 vif < 10*/
proc reg data=cent_train;
model y=income white / vif;
run;
quit;
/* 3.3 Add white to model, determine the third fixed variable via t-value and p-value */
/*t:-2.28 F:302.47 */

proc reg data=cent_train;
model y=income white household;
run;
quit;
/*t:-5.05 F:315.56 */
proc reg data=cent_train;
model y=income white gender;
run;
quit;
/*t:-1.25 F:300.57 */
proc reg data=cent_train;
model y=income white education;
run;
quit;
/*t:-14.67 F:412.37 */
proc reg data=cent_train;
model y=income white firms;
run;
quit;
/*t:-17.04 F:451.66 */
proc reg data=cent_train;
model y=income white veterans;
run;
quit;
/* we choose veterans as a candidate varible, now check for it */
/* r_square:0.4605 vif<10*/
proc reg data=cent_train;
model y=income white veterans / vif;
run;
quit;
/* 3.3 Add veterans to model, determine the fourth fixed variable via t-value and p-value */
/*t:0.09 F:338.54 p:0.9249*/
proc reg data=cent_train;
model y=income white veterans household;
run;
quit;
/*t:-1.15 F:339.15 p:0.2485*/
proc reg data=cent_train;
model y=income white veterans gender;
run;
quit;
/*t:0.52 F:338.66 p:0.6029*/
proc reg data=cent_train;
model y=income white veterans education;
run;
quit;
/*t:-3.33 F:343.69 */
proc reg data=cent_train;
model y=income white veterans firms;
run;
quit;
/* we choose firms as a candidate varible, now check for it */
/* r_square:0.4639 vif<10*/
proc reg data=cent_train;
model y=income white veterans firms/vif;
run;
quit;
/* 3.5 Add firms, determine the fifth fixed variable via t-value f-value and p-value (reject */
/*r:0.22 F:274.79 p:0.8236*/
proc reg data=cent_train;
model y=income white veterans firms household;
run;
quit;
/*r:-0.94 F:275.11 p:0.3458*/
proc reg data=cent_train;
model y=income white veterans firms gender;
run;
quit;
/*r:-0.78 F:275.01 p:0.4330*/

proc reg data=cent_train;
model y=income white veterans firms education;
run;
quit;
/* 3.6 fix model with income white veterans and firms, r_square:0.4639 vif < 10 */
/* Add one interaction variable */
proc reg data=cent_train;
model y=income white veterans firms intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intwhitehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intwhiteeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intincomehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intincomefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdveterans;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms intfirmsveterans;
run;
quit;
/* we choose inthouseholdeducation as a candidate interaction varible, now check for it */
/* r_square:0.5173 vif<10 */
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation/ vif;
run;
/* 3.7 add inthouseholdeducation, determine the second interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhitehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomeeducation;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intfirmsveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans Firms inthouseholdeducation intincomeeducation;
run;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intincomeeducation;
run;
/* we choose intwhiteeducation as a candidate interaction varible, now check for it */
/*r:0.5300 vif<10*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation /
vif;
run;
/* 3.8 add intwhiteeducation, determine the third interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intwhitehousehold;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intgenderveterans;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intfirmsveterans;
run;
quit;
/* we choose intincomefirms as a candidate interaction varible, now check for it */
/*r_square:0.5345 vif<10*/
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms / vif;
run;
/* 3.9 add intincomefirms, determine the fourth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intincomehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intincomeveterans;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intfirmsveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans Firms inthouseholdeducation intwhiteeducation
intincomefirms intincomeeducation;
run;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intincomeeducation;
run;
/* we choose intwhitehousehold as a candidate interaction varible, now check for it */
/*r:0.5412 vif<10*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold/ vif;
run;
/* 3.10 add intwhitehousehold, determine the fifth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold inteducationfirms;
run;
quit;
proc reg data=cent_train;

model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intfirmsveterans;
run;
quit;
proc reg data=cent_train;
model y=white Income veterans Firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomeeducation;
run;
/* we choose intincomehousehold as a candidate interaction varible, now check for it */
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold/ vif;
run;
/* 3.11 add intincomehousehold, determine the sisth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold inthouseholdveterans;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intfirmsveterans;
run;
quit;
/* we choose intincomegender as a candidate interaction varible, now check for it */
/*r:0.5531 vif<10*/
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender / vif;
run;
/* 3.12 add intincomegender, determine the seventh interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation

intincomefirms intwhitehousehold intincomehousehold intincomegender
intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
intfirmsveterans;
run;
quit;
/* we choose inteducationveterans as a candidate interaction varible, now check for it */
/*r_square:0.5555 vif<10 */
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans/ vif;
run;
/* 3.13 add inteducationveterans, determine the eighth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender

inteducationveterans inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intfirmsveterans;
run;
quit;
/* we choose intwhitefirms as a candidate interaction varible, now check for it */
/*r_square:0.5577 vif<10*/
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms/ vif;
run;
/* 3.14 add intwhitefirms, determine the ninth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intgendereducation;
run;
quit;

proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms inteducationfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans;
run;
quit;
/* we choose intfirmsveterans as a candidate interaction varible, now check for it */
/*r_square:0.5589 vif<10 */
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans/ vif;
run;
/* 3.15 add intfirmsveterans, determine the tenth interaction variable*/
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intwhiteincome;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intwhitegender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intwhiteveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intincomeeducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intincomeveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans inthouseholdgender;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation

intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans inthouseholdfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans inthouseholdveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intgendereducation;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intgenderfirms;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans intgenderveterans;
run;
quit;
proc reg data=cent_train;
model y=income white veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans inteducationfirms;
run;
quit;
/* all the p value > 0.05 and r_square is 0.5589, so the final model is */
proc reg data=cent_train;
model y=white income veterans firms inthouseholdeducation intwhiteeducation
intincomefirms intwhitehousehold intincomehousehold intincomegender
inteducationveterans intwhitefirms intfirmsveterans/ vif;
run;
*#####################################;
*### STEP 4: Final Model verify ###;
*#####################################;
%let reg_x= white income veterans firms inthouseholdeducation intwhiteeducation intincomefirms intwhitehouseho
/*1. Check multicollinearity*/
proc reg data=cent_train;
model y=&reg_x / vif;
run;
quit;
/*2. Check the assumptions */
/* Normality: verify qqplot of (studentised) residuals */
proc reg data=cent_train;
model y=&reg_x;
output out=resid r=rman p=pman student=student;
run;
quit;
/* Linearity: verify plot of (studentised) residuals vs predicted values*/
/* Homoscedasticity: verify (squared) residuals vs predicted values */
data resid2;
set resid;
rman2=rman**2;
run;
proc sgplot data=resid2;
scatter x=pman y=rman2;
refline 0 / axis=y lineattrs=(color=red);
run;
/*3. Check for outliers */
proc reg data=cent_train noprint;
model y=&reg_x / r;
output out=cookdis cookd=cdist;
run;

quit;
data cookdis2;
set cookdis;
n=_n_;
run;
title "Cook's distance threshold 4/n=0.00255";
proc sgplot data=cookdis2;
scatter x=n y=cdist;
refline 0.0025 / axis=y lineattr=(color=red);
run;
title;
/*Remove outlier according to Cook's distance threshold 4/n=0.00255;*/
data cent_training_outlier_removed;
set cookdis2;
where cdist<0.00255;
/* 93 outlier removed*/
run;
/*Build model after removing outlier*/
/*r_square:0.5714,intwhitehousehold(p=0.4253) and intfirmsveterans(p=0.5833)*/
proc reg data=cent_training_outlier_removed;
model y=&reg_x / vif;
run;
quit;
/*Remove intwhitehousehold and intfirmsveterans and check outlier again*/
/*Only 2 outliers and they are very close to the transhold, r_square:0.5714*/
%let reg_x= white income veterans firms inthouseholdeducation intwhiteeducation intincomefirms intincomehousehold
proc reg data=cent_training_outlier_removed noprint;
model y=&reg_x / r;
output out=cookdis cookd=cdist;
run;
quit;
data cookdis2;
set cookdis;
n=_n_;
run;
title "Cook's distance threshold 4/n=0.00255";
proc sgplot data=cookdis2;
scatter x=n y=cdist;
refline 0.0025 / axis=y lineattr=(color=red);
run;
title;
/*Export the dataset for Logistic Regression*/
proc export data=cent_training_outlier_removed (keep=y county &reg_x)
outfile="/folders/myfolders/StatmodProject2020/cent_train.csv" dbms=csv replace;
run;
proc export data=cent_test (keep=y county &reg_x)
outfile="/folders/myfolders/StatmodProject2020/cent_test.csv" dbms=csv replace;
run;
*#####################################;
*### STEP 5: Testing Final Model ###;
*#####################################;
proc reg data=cent_training_outlier_removed outest=train_estimate noprint;
model y=&reg_x;
run;
proc score data=cent_test score=train_estimate out=test_result type=parms
predict;
var y &reg_x;
run;
ods listing gpath=&dir;
ods graphics on;
ods select FitPlot;

proc reg data=test_result;
model y=model1;
plot model1*y / pred conf;
run;
ods graphics off;
ods listing;
US_election_final
DU Feihong,TangLin,ZhangShengmin
2020/12/11
#import data#
mypath = '.'
setwd(mypath)
cent_train<-read.csv('cent_train.csv')
cent_test<-read.csv('cent_test.csv')
whole_dataset<- rbind(cent_train,cent_test)
str(cent_train)
str(cent_test)
plot(cent_train$y,xlab = 'series',ylab = 'vote_percent')
#---------------------------------------------------------------
#6.categorize y#
cent_train$cat_y <-ifelse(cent_train$y>0.5,1,0)
cent_test$cat_y <-ifelse(cent_test$y>0.5,1,0)
#fit withcategorizedy#statewhatwehavefoundwhencomparetwomodels##
fit.raw.train <-lm(y ~ white+income+veterans+firms+inthouseholdeducation+
intwhiteeducation+intincomefirms+intincomehousehold+
intincomegender+inteducationveterans+intwhitefirms,
data=cent_train)
fit.cat.train <- glm(cat_y ~ white+income+veterans+firms+inthouseholdeducation+
intwhiteeducation+intincomefirms+intincomehousehold+
intincomegender+inteducationveterans+intwhitefirms,
family = binomial,data=cent_train)
fit.raw.train.summary<-summary(fit.raw.train)
fit.cat.train.summary<-summary(fit.cat.train)
beta1 <- fit.cat.train$coefficients[2]
con.odds.white <- exp(beta1)
plot(effects::effect('white', fit.cat.train))
#7.-----------------------------------------------------------------------------
glm.pred.raw <- predict(fit.raw.train, newdata = cent_test, type = "response")
glm.pred.cat <- ifelse(glm.pred.raw >0.5,1,0)
glm.pred.cat<-as.numeric(glm.pred.cat)

#test result#
p_vs_a<-as.matrix(table(actual=cent_test$cat_y,predict=glm.pred.cat))
n = sum(p_vs_a) # numberofinstances
nc = nrow(p_vs_a) # numberofclasses
diag = diag(p_vs_a) # numberofcorrectlyclassifiedinstancesperclass
rowsums = apply(p_vs_a, 1, sum) # numberofinstancesperclass
colsums = apply(p_vs_a, 2, sum) # numberofpredictionsperclass
p = rowsums / n # distributionofinstancesovertheactualclasses
q = colsums / n # distributionofinstancesoverthepredictedclasses
accuracy = sum(diag) / n
precision = diag / colsums
recall = diag / rowsums
#overlap----------------------------------------------------------
overlapEst(cent_test$cat_y, glm.pred.cat)
#difference--------------------------------------
t.test(glm.pred.cat,cent_test$cat_y,paired = T)
#variance------------------------------------
var.test(glm.pred.cat, cent_test$cat_y, alternative = "two.sided")
##########################################################################
fit.raw.whole <-lm(y ~ white+income+veterans+firms+inthouseholdeducation+
intwhiteeducation+intincomefirms+intincomehousehold+
intincomegender+inteducationveterans+intwhitefirms,
data=whole_dataset)
glm.pred.whole <- predict(fit.raw.whole, newdata = whole_dataset, type = "response")
glm.pred.whole<-as.numeric(glm.pred.whole)
glm.pred.whole.cat <- ifelse(glm.pred.whole >0.52,'win',
ifelse(glm.pred.whole<0.48,'lose','undecided'))
whole_dataset$result<-glm.pred.whole.cat
undecided <- whole_dataset%>%
subset(result == 'undecided')
factor(whole_dataset$county)
factor(undecided$county)
table(undecided$county,undecided$result)

```




