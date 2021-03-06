---
date: "2009-07-31 08:21:01"
layout: post
slug: swiid-version-2_0
tags:
- update
- swiid
title: SWIID Version 2.0
---

Version 2.0 of the SWIID is now available, and it is a major upgrade. It introduces two important changes from Version 1.1 (the version described in the *SSQ* article). First, I collected a large number (1500+) of Gini observations that are excluded from the WIID with an eye towards addressing some of the thinner spots in the SWIID's underlying data. Second, I rewrote several parts of the missing-data algorithm. The key change is a switch from multilevel to (flat) linear regression modeling for the imputation of conversion ratios between the 21 categories of available Gini data. Given the patterns of missingness in the data, complete pooling (as occurs in a flat linear regression) proved superior to partial pooling (as occurs in multilevel modeling). The result, along with some minor improvements in coverage, is considerably smaller standard errors in the Gini index estimates, particularly in Latin America and Africa, than in Version 1.1. All SWIID users are encouraged to use these new data in their work.
