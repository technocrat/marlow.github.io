---
layout: post
title: Combination of know items, taken n at a time
tags: Haskell, Python
---
Very functional

    -- combo.hs
	-- problem: C(k,n), where k = the integers from 1 to 9, inclusive
	-- and n = 3, without regard to order, then sum the subsets
	import Data.List
	combinations 0 lst = [[]]
	combinations n lst = do
	    (x:xs) <- tails lst
	    rest <- combinations (n-1) xs
	    return $ x : rest
	result = ( map sum (combinations 3 [1..9]))

Python alternative

    import itertools
    result = map(sum, itertools.combinations([1,2,3,4,5,6,7,8,9], 3))
    for i in result: print(i)