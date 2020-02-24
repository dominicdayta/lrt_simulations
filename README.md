# Using Dynamic, Data-Driven Simulations On MRT Passenger Traffic Data For Optimizing Service Performance Strategies
## Dominic Dayta
#### Originally presented to Accenture Philippines, March 2018 as an entry for the 2018 Big Data Challenge

This repository contains the full R code, dataset, and presentation material for the entry "Using Dynamic Data-Driven Simulations on MRT Passenger Traffic Data For Optimizing Service Performance Strategies" originally presented to Accenture Philippines for their 2018 Big Data Challenge.

For potential contributors or those who would wish to adapt the code to their purposes (but seriously, why would you?) below are some useful information regarding the structure of the code. In retrospect, the code itself was built on too many for loops - at least more than an R developer should be comfortable with - which explains the lengthy comments at the beginning of the code (added at the time of writing) agonizing over the long running times.

### Data

Although completely based on a monte-carlo type simulation, the program makes use of actual passenger traffic data obtained through the Philippine government's Open Data portal (data.gov.ph) giving the number of passengers entering each station of the Metro Rail Transit (MRT-3) running from North Avenue in Quezon City, to Taft Avenue Station in Pasay City. The data itself has been included in the repository for reference, although it may be accessed through the Open Data Portal as well.

Certain assumptions had to be made to further flesh out the data (and these are outlined in the Configurations section below) due to the data representing the overall sum of passenger entrance and exits to the stations regardless of line (the MRT runs on two parallel lines: southbound and northbound, experiencing differing traffic schedules througout the day, though mostly complementary).

At the same time, while the data gives the number of passengers entering and exiting the station at each hour of each day of coverage, this is only indicative of passenger position to as far as the station platforms and gives no information on the number of passengers actually boarding and alighting the coaches. Data with better specificity on the two points mentioned would definitely make for more accurate simulations, and perhaps improve the specificity of the simulations themselves (for instance, one may explore origin-destination patterns as well). However, as these was not available at the time of coding, one would find these to be lacking in the codes as they stand.

### Configurations

To answer the above missing dimensions in the data, some rough estimates have been provided. For instance, one finds, in line 158 of the current version of the code the following line defining the values of an array called `southbound`. This serves as a mutiplier to the actual data trimming the total passenger traffic to a rough estimate of the proportion entering the southbound line (from North to Taft Avenue). 

```r
#generate matrix of proportion of southbound arrivals per station
southbound<-c(1,0.95,0.95,0.95,0.7,0.6,0.5,0.4,0.3,0.05,0.05,0.05,0)
```

Thus, in line 167, when obtaining the number of passengers entering each station, one sees multiplication:

```r
passenger_entry[k,station]<-round(arrives(0,240+arrival[k,3+station]+parking,stations[station],2014,iterate)*southbound[station],digits=0)
```
which trims the returned value from the function `arrives()` (more on this later) the element in `southbound` with index equal to `station` (the proportion of estimated arrivals at that station).

In terms of passengers exiting the trains at each station, a similar solution has been implemented. One finds, in line 204, the array `exit`:

```r
#below is a vector that states the percentage of the train's load that will exit at a given station
exit<-c(0,0,0.01,0.05,0.2,0.3,0.2,0.2,0.4,0.4,0.2,1)
```

And finally, there is an assumption in the simulation that at each station, if there are more people on the platform than there is space inside the coach, only enough people will come in such as to fill 90% of the available space onboard. On the other hand, if there is enough space on the train for everyone waiting on the platform, then everyone boards. This is implemented in line 216, where the value `people_board` is declared as:

```r
people_board<-min(passenger_entry[k,4+station],round(0.9*(1180-load[station]),digits=0))
```

These comprise the more practical configurations specific to the simulations conducted for the Accenture Big Data Challenge. The rest of the configurations to be mentioned in this readme concern the specifics of the system being simulated. The speed (represented in the code with the variable `v` is set to a constant 750 meters per minute (80 km/h), while the length of operating hours `ophours` is set to 1020 minutes. And, because the simulation considers the situation at each station, it is important to specify the number of stations, which in this case is thirteen which, though unnecessary, was named in an array on line 163:

```r
stations<-c("NORTH","QUEZON","KAMUNING","CUBAO","SANTOLAN","ORTIGAS","SHAW","BONI","GUADALUPE","BUENDIA","AYALA","MAGALLANES","TAFT")
```

### Simulation Procedure

The function `train_arrivals()` runs per iteration and gives the number of trains arriving at each station at each hour interval. The output is per hour consistent with the data, although the actual simulation estimates on the level of minutes (which is why the velocity constant is in meters per minute).

One will notice here that in my younger and more vulnerable years I had the annoying tic of resetting the seed to a function of the index of the current iteration (which I really recommend against to anyone looking to writing code that's actually reproducible). See line 59:

```r
set.seed(iteration*k*tau*s)
```

Meanwhile, the function `arrives()` takes in the beginning and end time of the interval in question, and gives the number of passengers arriving within that period. This is my favorite part of the simulation since this is the one that's not mechanical and actually applies some statistics. First it loads the passenger traffic data and constructs a cumulative density function for the arrivals. This is done on lines 118 to 121 on the rather painfully slow codes:

```r
h<-hist(as.numeric(subset(entry,END_HOUR==paste(arrival_time),select=paste(station))[[1]]),main=paste("Distribution of Arrivals, ",station,sep=""),xlab="Number of Arrivals",freq=FALSE)
  cumdensity<-rep(0,NROW=h$counts)
  for(j in 1:NROW(h$counts)){cumdensity[j]<-sum(h$counts[1:j])/sum(h$counts)}
  cdf<-cbind(h$breaks[2:NROW(h$breaks)],cumdensity)
```

The reason why a cumulative density had to be constructed was because I use this to do inverse sampling on the from a Uniform(0,1) distribution on lines 127 to 134, such that I am not simply taking the actual observed passenger traffic on that hour and station, but taking a random sample from a distribution constructed out of that data:

```r
sampled<-runif(1,0,1)
  for(j in 1:nrow(cdf)){
    if(sampled<=cdf[j,2]){
      maxarrival<-cdf[j,1]
      if(j==1) meanarrival<-cdf[j,1]/2 else meanarrival<-(cdf[j,1]+cdf[j-1,1])/2 
      break
    }  
  }
```

This provides the random component of the simulation, in that while the process of trains arriving, parking to collect passengers, and departing at each station is strictly deterministic (a good improvement on this would be to add an additional random component for mechanical breakdowns -- which I had thought of at the time but did not have enough time before the contest deadline to properly account for in my codes), there is a probability for slacks and surges in the arrival of people at the stations.
