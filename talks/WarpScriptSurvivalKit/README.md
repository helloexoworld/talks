# Surival kit, Start a Time Series analysis using Warp 10

The slides can be found [here](https://docs.google.com/presentation/d/1WhXAMQ4teoAgs8UTfSh21-d0aYJD3r2hYwDrUJFgJNY/edit?usp=sharing).

## First install Warp 10 

http://www.warp10.io/getting-started/

Adapt X.Y.Z to the Warp 10 version number needed

```
wget https://dl.bintray.com/cityzendata/generic/io/warp10/warp10/X.Y.Z/
tar xf warp10-X.Y.Z.tar.gz
cd warp10-X.Y.Z/bin

```

You will need to set-up a JVM, and then initialise Warp 10 as indicated in getting started. 

## Push Kepler-11 data

Once Warp 10 is running you can add Kepler-11 data that are stored in kepler-11.mc2 file.
To load them, just update the WRITE_TOKEN in the snapshot file (line 18). Then execute.

```
curl -v http://127.0.0.1:8080/api/v0/exec --data-binary @kepler-11.mc2
```

## Replay the demo

Open quantum in your navigator, it comes with Warp 10 standalone at http://127.0.0.1:8090.

### Hello world

Here you can type, some WarpScript commands and click on execute to see the results it provides. Let's try with simple command:

```
NOW
```

Execute will give the current time in micro-seconds (platform unit time).
WarpScript use a (RPN)[https://en.wikipedia.org/wiki/Reverse_Polish_notation] Stack concept.

Let's introduce how variables can be declared in WarpScript as we are going to use it several time for all the next steps.

```
// Push Hello World String on the Stack
'Hello, world!'

// Store it in a variable called hello
'hello' STORE

// Then push back hello vairable on the stack
$hello

// and again
$hello
```


### The FIND operation

Then to verify wich series are available on your local Warp 10, you can use the FIND function.
This will list all the series available for you READ TOKEN.

```
'READ_TOKEN'
'token' STORE

[ $token 'kepler.sap.flux' {} ] FIND
```

### The FETCH operation

To retrive data on Warp 10, it's done using the FETCH operation. 
This will load all the raw as shown during the Meetup.

```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET
```

As only one series is returned by the FETCH, the 0 GET operation extracts the first element of the fetched time series list. 

### Time series operation

Warpcript offers to the user multiples specific Time-series functions as VALUES (which list all the values of a series), ATTICK (get values for a specific timestamp) and also some time manipulation function as TIMEMODULO and TIMESPLIT that were detailed during the presentation.

#### Time modulo utilisation

This WarpScript operation split (singleton or list of) time series according to a given time modulo. The following code will generate a single series for each year of the kepler records.
```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Time modulo
365 d 

// GTS split label
'year' 

TIMEMODULO
```
The generated series starts all on 1 January 1970.

#### Time split utilisation

This WarpScript operation split (singleton or list of) time series according to a given quiesce period and a minimal number of data points per split. The following code will generate a single series for each splits, keeping the time information.
```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT
```

### The Filter framework

WarpScript includes 5 frameworks, the first one we talk a bit about is the filter framework as it allows the user to select some series in a list.

```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT

'gts' STORE

[

    $gts                                // Series list or Singleton
    []                                  // Labels to compute equivalence class
    { 'observation' '~[2-5]' }          // Labels map for selector
    filter.bylabels                     // Filter function operator 
]
FILTER
```

This will load 3 series to compute the next script operations. The fifth observation corresponds to the one shown during the presentation.

### The Bucketize framework

WarpScript includes 5 frameworks, the second one we talk a bit about is the Bucketize framework. This one allow the user to perform some downsampling. In other word, it will split the series in several time buckets, and apply an operation on all the values inside each bucket to generate only one tick per bucket. This allow the user to get a series with less data points and eliminate some local noise. During the presentation the operation applied on the buckets was to keep a min between all values.


```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT

'gts' STORE

[

    $gts                                // Series list or Singleton
    []                                  // Labels to compute equivalence class
    { 'observation' '~[2-5]' }          // Labels map for selector
    filter.bylabels                     // Filter function operator 
]
FILTER

[
    SWAP                                // Series list or Singleton
    bucketizer.min                      // Bucketize function operator
    0                                   // Lastbucket 				
    2 h                                 // Bucketspan
    0                                   // Bucketcount
]
BUCKETIZE

```

### The Mapper framework

The third framework allows the user to apply operation on each series values from a single value operation, to a value window. To try detecting drops in Kepler-11 records, we apply a lower than mapper. It means that we keep only values below a user given threshold.

```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT

'gts' STORE

[

    $gts                                // Series list or Singleton
    []                                  // Labels to compute equivalence class
    { 'observation' '~[2-5]' }          // Labels map for selector
    filter.bylabels                     // Filter function operator 
]
FILTER

[
    SWAP                                // Series list or Singleton
    bucketizer.min                      // Bucketize function operator
    0                                   // Lastbucket 				
    2 h                                 // Bucketspan
    0                                   // Bucketcount
]
BUCKETIZE

[
    SWAP                                // Series list or Singleton
    40150.0                             // Mapper.lt threshold value
    mapper.lt                           // Mapper function operator
    0                                   // Pre elements (For mapping window)
    0                                   // Post elements (For mapping window)
    0                                   // Occurences
]
MAP
```

### The APPLY framework

The last framework seen during this presentation is the Apply one. It computes operation between several gts set. As input the bucketize result is re-used and will substract from it the moving mean of the series (to estimate a trend for all the series).
 
```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT

'gts' STORE

[

    $gts                                // Series list or Singleton
    []                                  // Labels to compute equivalence class
    { 'observation' '~[2-5]' }          // Labels map for selector
    filter.bylabels                     // Filter function operator 
]
FILTER

[
    SWAP                                // Series list or Singleton
    bucketizer.min                      // Bucketize function operator
    0                                   // Lastbucket 				
    2 h                                 // Bucketspan
    0                                   // Bucketcount
]
BUCKETIZE

'bucketizedSeries' STORE

[
    // Compute moving mean 
    [
        $bucketizedSeries
        mapper.mean
        5
        5
        0
    ]
    MAP                                 // Series list or Singleton 0
    $bucketizedSeries                   // Series list or Singleton 1

    [ 'observation' ]                   // Labels to compute equivalence class
    op.sub                              // Apply function operator
]
APPLY
```

### Final result

To get the same result as shown during the talk, use the following script. The APPEND operation was used to merge the bucketize result with the Apply result. 
Then as the value are very different, the final set is standardized, and both results can be compared.

```
'READ_TOKEN'
'token' STORE

[ 
	$token 								// Application authentication
	'kepler.sap.flux' 					// selector for classname
    { 'id' '006541920' }            	// Selector for labels
	'2009-05-02T00:56:10.000000Z' 		// Start date
	'2013-05-11T12:02:06.000000Z' 		// End date
] FETCH

// Get Singleton series
0 GET

// Quiesce period
6 h 

// Minimal numbers of points per series 
100 

// GTS split label
'observation' 

TIMESPLIT

'gts' STORE

[

    $gts                                // Series list or Singleton
    []                                  // Labels to compute equivalence class
    { 'observation' '~[2-5]' }          // Labels map for selector
    filter.bylabels                     // Filter function operator 
]
FILTER

[
    SWAP                                // Series list or Singleton
    bucketizer.min                      // Bucketize function operator
    0                                   // Lastbucket 				
    2 h                                 // Bucketspan
    0                                   // Bucketcount
]
BUCKETIZE

'bucketizedSeries' STORE

[
    // Compute moving mean 
    [
        $bucketizedSeries
        mapper.mean
        5
        5
        0
    ]
    MAP                                 // Series list or Singleton 0
    $bucketizedSeries                   // Series list or Singleton 1

    [ 'observation' ]                   // Labels to compute equivalence class
    op.sub                              // Apply function operator
]
APPLY

// Mergeb bucketize series with previous result
$bucketizedSeries
APPEND

// Standardize the series to compare both results
STANDARDIZE
```
