---
layout: post
title: Correlates of police involved homicides
tags: politics
--- 
Police involved homicides in 2015 were strongly correlated with all 2014 homicides and the log of the state's estimated population in 2015.

	Call:
	lm(formula = homi2015$n ~ log(pop) + homi2014$n)

	Residuals:
		Min      1Q  Median      3Q     Max 
	-48.635 -11.487   0.013  10.075  62.087 

	Coefficients:
				 Estimate Std. Error t value Pr(>|t|)    
	(Intercept) 195.98670   59.22581   3.309  0.00178 ** 
	log(pop)    -13.60314    4.09766  -3.320  0.00173 ** 
	homi2014$n    0.10521    0.01176   8.946 8.51e-12 ***
	---
	Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

	Residual standard error: 17.64 on 48 degrees of freedom
	Multiple R-squared:  0.723, Adjusted R-squared:  0.7114 
	F-statistic: 62.63 on 2 and 48 DF,  p-value: 4.176e-14

 ![](http://careaga.s3.amazonaws.com/2016-09-25-RplotModel.png)
    
   
   
   
