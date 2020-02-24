# Using Dynamic, Data-Driven Simulations On MRT Passenger Traffic Data For Optimizing Service Performance Strategies
## Dominic Dayta
#### Originally presented to Accenture Philippines, March 2018 as an entry for the 2018 Big Data Challenge

This repository contains the full R code, dataset, and presentation material for the entry "Using Dynamic Data-Driven Simulations on MRT Passenger Traffic Data For Optimizing Service Performance Strategies" originally presented to Accenture Philippines for their 2018 Big Data Challenge.

For potential contributors or those who would wish to adapt the code to their purposes (but seriously, why would you?) below are some useful information regarding the structure of the code. In retrospect, the code itself was built on too many for loops - at least more than an R developer should be comfortable with - which explains the lengthy comments at the beginning of the code (added at the time of writing) agonizing over the long running times.

### Data

Although completely based on a monte-carlo type simulation, the program makes use of actual passenger traffic data obtained through the Philippine government's Open Data portal (data.gov.ph) giving the number of passengers entering each station of the Metro Rail Transit (MRT-3) running from North Avenue in Quezon City, to Taft Avenue Station in Pasay City. The data itself has been included in the repository for reference, although it may be accessed through the Open Data Portal as well.

Certain assumptions had to be made to further flesh out the data (and these are outliend in the Configurations section below) due to the data representing the overall sum of passenger entrance and exits to the stations regardless of line (the MRT runs on two parallel lines: southbound and northbound, experiencing differing traffic schedules througout the day, though mostly complementary).

At the same time, while the data gives the number of passengers entering and exiting the station at each hour of each day of coverage, this is only indicative of passenger position to as far as the station platforms and gives no information on the number of passengers actually boarding and alighting the coaches. Data with better specificity on the two points mentioned would definitely make for more accurate simulations, and perhaps improve the specificity of the simulations themselves (for instance, one may explore origin-destination patterns as well). However, as these was not available at the time of coding, one would find these to be lacking in the codes as they stand.

### Configurations

### Simulation Procedure
