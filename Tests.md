`ldsc` uses the `nose` package for testing. There are two sorts of tests. The fast tests take about 2 seconds to run and test basic functionality. The slow tests check empirically that the estimators of h<sup>2</sup> and r<sub>g</sub> have sensible statistical properties (unbiasedness and accurate standard errors) by running 1000 simulations.

You can run the fast tests with the command

	nosetests -A 'not slow'
This should take about 2 seconds. You can run the slow tests with the command 

	nosetests -A 'slow'
This should take about five minutes.
   
 
