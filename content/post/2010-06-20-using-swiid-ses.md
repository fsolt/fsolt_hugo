---
date: "2010-06-20 12:05:01"
layout: post
slug: using-swiid-ses
tags:
- note
- swiid
title: Using the SWIID Standard Errors
---

Incorporating the standard errors in the SWIID estimates into one's analyses is the right thing to do, but it is not a trivial exercise.  I myself have left it out of some work where I felt the model was already maxed out on complexity (though in such cases, I advise at least excluding observations with particularly large errors).  The short story is that one generates a bunch of Monte Carlo simulations of the SWIID data from the estimates and standard errors, then analyses each simulation, then combines the results of the multiple analyses as one would in a multiple-imputation setup (this should be easier to do with Stata 11's new multiple-imputation tools, but I won't get my copy of Stata 11 until the fall--oh well).  The code below does the trick.


    **Using the SWIID Standard Errors: An Example**
    //Load SWIID and generate fake data for example
    use "SWIIDv2_0.dta", clear
    set seed 4533160
    gen x1 = 20*rnormal()
    gen x2 = rnormal()
    gen x3 = 3*rnormal()
    gen y = .03*x1 + 3*x2 + .5*x3 + .05*gini_net + 5 + 20*rnormal()
    reg y x1 x2 x3 gini_net
    
    //Generate ten Monte Carlo simulations of the gini_net series
    egen ccode=group(country)				
    tsset ccode year						
    set seed 3166							
    forvalues a = 1/10 {
    	gen e0 = rnormal()
    	quietly tssmooth ma e00 = e0, weight (1 1 <2> 1 1)
    	quietly sum e00
    	quietly gen g`a'=gini_net+e00*(1/r(sd))*gini_net_se
    	drop e0 e00
    }
    
    //Perform analysis using each of the ten simulations, saving the results
    local other_ivs = "x1 x2 x3"		/*to be replaced with your other IVs, that is, not including gini_net or the constant*/
    local n_ivs = 5				/*to be replaced with the number of IVs, now *including* gini_net and the constant*/
    matrix coef = J(`n_ivs', 10, -99)
    matrix se = J(`n_ivs', 10, -99)
    matrix r_sq = J(1, 10, -99)
    forvalues a = 1/10 {
    	quietly reg y `other_ivs' g`a'	/*to be replaced with your analysis*/	
    	matrix coef[1,`a'] = e(b)'
    	matrix A = e(V)
    	forvalues b = 1/`n_ivs' {
    			matrix se[`b', `a'] = (A[`b',`b'])
    	}
    	matrix r_sq[1, `a'] = e(r2)
    }		
    
    local cases = e(N)
    
    svmat coef, names(coef)
    svmat se, names(se)
    svmat r_sq, names(r_sq)
    
    
    //Display results across all simulations
    egen coef_all = rowmean(coef1-coef10)
    
    gen ss_all = 0
    forvalues a = 1/10 {
    	quietly replace ss_all = ss_all + (coef`a'-coef_all)^2
    }
    egen se_all = rowmean(se1-se10)
    replace se_all = se_all + (((1+(1/10)) * ((1/9) * ss_all))) /*Total variance, per Rubin (1987)*/
    replace se_all = (se_all)^.5 /*Total standard error*/
    
    gen t_all = coef_all/se_all
    gen p_all = 2*normal(-abs(t_all))
    
    egen r_sq_all = rowmean(r_sq1-r_sq10)
    
    gen vars = " " in 1/`n_ivs'
    local i = 0
    foreach iv in `other_ivs' "Inequality" "Constant" {
    	local i = `i'+1
    	replace vars = "`iv'" in `i'
    }
    mkmat coef_all se_all p_all if coef_all~=., matrix(res_all) rownames(vars)
    matrix list res_all, format(%9.3f)
    quietly sum r_sq_all
    local r2 = round(`r(mean)', .001)
    di "R-sq = `r2'"
    di "N = `cases'"


Please feel free to drop me an email if you have any questions or comments.
