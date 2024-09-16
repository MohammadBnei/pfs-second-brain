`back-connectivity/cache/flight/aes/datadomain/generator.go`

- [[#Generate|Generate]]
	- [[#Generate#Context Setup and Initialization|Context Setup and Initialization]]
	- [[#Generate#Data Loading|Data Loading]]
	- [[#Generate#Sales Data Filtering|Sales Data Filtering]]
	- [[#Generate#Property Reference Loading|Property Reference Loading]]
	- [[#Generate#Place ID Loading|Place ID Loading]]
	- [[#Generate#Route Data Merging|Route Data Merging]]
	- [[#Generate#Curation Rules Loading|Curation Rules Loading]]
	- [[#Generate#Curation Rules Logging|Curation Rules Logging]]
	- [[#Generate#Iterating Over Curation Rules and Shops|Iterating Over Curation Rules and Shops]]
	- [[#Generate#Iterating Over Brand Shop Codes and Destinations|Iterating Over Brand Shop Codes and Destinations]]
	- [[#Generate#Merging Route Data|Merging Route Data]]
	- [[#Generate#Adding SDP Only Routes to Result Map|Adding SDP Only Routes to Result Map]]
	- [[#Generate#Updating Route Durations|Updating Route Durations]]
	- [[#Generate#Computing Computation Cells|Computing Computation Cells]]
	- [[#Generate#Reporting Computed Cells by Key|Reporting Computed Cells by Key]]
	- [[#Generate#Updating Booking Window Offset|Updating Booking Window Offset]]
	- [[#Generate#Writing to CSV File|Writing to CSV File]]
	- [[#Generate#Updating Functional Errors Report|Updating Functional Errors Report]]
	- [[#Generate#Generating Final Report|Generating Final Report]]
	- [[#Generate#Sending Data Domain Report|Sending Data Domain Report]]
	- [[#Generate#Updating CloudWatch Metric|Updating CloudWatch Metric]]
- [[#MergeDataRoute|MergeDataRoute]]
	- [[#MergeDataRoute#Context Setup and Airport Configuration|Context Setup and Airport Configuration]]
	- [[#MergeDataRoute#Airport Code Override|Airport Code Override]]
	- [[#MergeDataRoute#Airports to City Conversion Function|Airports to City Conversion Function]]
	- [[#MergeDataRoute#Destination and Return Origin Processing|Destination and Return Origin Processing]]
	- [[#MergeDataRoute#Office ID and Destination Rule Configuration|Office ID and Destination Rule Configuration]]
	- [[#MergeDataRoute#Route Generation and Data Processing|Route Generation and Data Processing]]
	- [[#MergeDataRoute#Route Data Generation|Route Data Generation]]
	- [[#MergeDataRoute#Duration Processing|Duration Processing]]
	- [[#MergeDataRoute#Route Data Processing and Update|Route Data Processing and Update]]
	- [[#MergeDataRoute#Corporate Codes Processing|Corporate Codes Processing]]
	- [[#MergeDataRoute#Search By FBA Processing|Search By FBA Processing]]
	- [[#MergeDataRoute#Mandatory Carriers Processing|Mandatory Carriers Processing]]
	- [[#MergeDataRoute#Cabin Processing|Cabin Processing]]
	- [[#MergeDataRoute#Route Data Update or Insertion|Route Data Update or Insertion]]
- [[#GenerateAmadeusSilos|GenerateAmadeusSilos]]
	- [[#GenerateAmadeusSilos#Function Purpose|Function Purpose]]
	- [[#GenerateAmadeusSilos#Input Parameters|Input Parameters]]
	- [[#GenerateAmadeusSilos#Special Cases|Special Cases]]
	- [[#GenerateAmadeusSilos#General Case|General Case]]
	- [[#GenerateAmadeusSilos#Return Value|Return Value]]


## Generate 
### Context Setup and Initialization

* The function takes in a `context.Context` object (`ctx`) and an `io.Writer` object (`w`) as input.
* It initializes a logger with a field "action" set to "Generate".
* It loads ondrules and Amadeus config using the `ref.GetConfig()` method.

### Data Loading

* The function iterates over a list of brand shop codes obtained from `ref.GetBrandShopCodes()`.
* For each brand, it logs an info message indicating that it's processing the brand.
* It calls the `catalog.GetSales` function to retrieve sales data for the current brand and context.

### Sales Data Filtering

* The function filters out sales with a start date later than `maxNumberOfDaysBeforeSaleStart` days from now.
* If no sales are found or an error occurs, it logs a warning message and continues to the next brand.

### Property Reference Loading

* For each sale, it loads the property reference using the `tourop_service.FindOneByID` function.
* If the load fails due to a missing document, it logs a warning message and continues to the next sale.
* If any other error occurs during loading, it returns an error.

### Place ID Loading

* It checks if the loaded property reference has a valid place ID.
* If not, it logs an error message and continues to the next sale.

### Route Data Merging

* For each valid sale, it calls the `mergeRouteData` function to merge route data into the `result` map.
* If any error occurs during merging, it returns an error.

### Curation Rules Loading

* The function loads a list of curation rules using the `tourop_service.FindByQuery` function.
* If any error occurs during loading, it returns an error.

### Curation Rules Logging

* It logs an info message with the number of loaded curation rules.

### Iterating Over Curation Rules and Shops

* The function iterates over each curation rule and shop combination.
* For each combination, it gets the corresponding shop reference using `ref.GetShopByID`.

### Iterating Over Brand Shop Codes and Destinations

* It iterates over each brand shop code and destination combination for the current shop.
* For each combination, it gets the corresponding place reference using `ref.GetPlaceById`.
* It creates a temporary result map to store merged route data.

### Merging Route Data

* The function calls the `mergeRouteData` function to merge route data into the temporary result map.
* If any error occurs during merging, it returns an error.

### Adding SDP Only Routes to Result Map

* If the temporary result map is not empty, it adds the merged route data to the main result map.
* It logs a warning message if no routes are found for a given shop and destination.

### Updating Route Durations

* The function iterates over each route in the result map.
* If the route has both min and max durations set, it updates the max duration to be at least 5 nights (4 days) more than the min duration.

### Computing Computation Cells

* The function defines a helper function `getComputedCells` to compute the total number of computation cells for all routes in the result map.
* It initializes two variables `nbInitialComputationCells` and `nbComputationCells` with the computed value.

### Reporting Computed Cells by Key

* The function creates a report string that shows the total number of computation cells for each key (office ID and/or silo) in the result map.
* It uses a map to accumulate the cell count for each key.

### Updating Booking Window Offset

* The function iterates over each route in the result map and reduces its booking end offset by 1 if it exceeds the booking start offset.
* This is done until the total number of computation cells (`nbComputationCells`) is within the maximum allowed limit (`maxComputedCells`).

### Writing to CSV File

* The function writes the updated result map to a CSV file using the `writeToCsv` function.

### Updating Functional Errors Report

* The function updates the functional errors report by deduplicating and sorting the error messages.

### Generating Final Report

* The function generates a final report string that includes:
	+ The total number of computation cells (`nbComputationCells`).
	+ A list of functional errors.
	+ A message indicating whether the booking window was reduced due to exceeding the maximum allowed limit (`maxComputedCells`).

### Sending Data Domain Report

* The function sends an email with the final report using the `sendDataDomainReport` function.

### Updating CloudWatch Metric

* The function updates a CloudWatch metric with the total number of computation cells (`nbComputationCells`) using the `awsutil.PutCloudWatchFloatMetric` function.



## MergeDataRoute

### Context Setup and Airport Configuration

* The function takes in a `context.Context` object (`ctx`) and a logger entry (`logger`).
* It uses the `tourop_entities.Sale` object to override airport codes if specified.
* The function loads an Amadeus configuration using the `amadeusConfig.LoadConfig` method.

### Airport Code Override

* If the sale's arrival airport is not empty, it overrides the airport codes with a single value (the arrival airport).
* Similarly, if the sale's return origin airport is not empty, it overrides the return origin airports with a single value (the return origin airport).

### Airports to City Conversion Function

* The function defines an inner function `airportsToCity` that takes in a slice of destination IATA codes (`dest`) and returns a new slice with city IATA codes instead of airport IATA codes.
* This function is used later in the mergeRouteData function.

### Destination and Return Origin Processing

* The function processes each destination IATA code in the converted airport codes slice.
* For each destination, it updates the return origin airports by applying the same conversion using the `airportsToCity` function.

### Office ID and Destination Rule Configuration

* The function loads office IDs for the specified brand shop code using the `amadeusConfig.GetOfficeIds` method.
* It also loads destination rules from the cache using the `pprules.GetOnDRulesCache` method.

### Route Generation and Data Processing

* For each office ID, the function generates Amadeus silos using the `GenerateAmadeusSilos` function.
* It then processes each rule in the destination rules slice to generate route data.
* The function checks if the from and to airports are the same or if the return from and return to airports are the same, and skips processing if so.

### Route Data Generation

* For each valid rule, the function generates a `RouteData` struct with the following properties:
	+ Booking start offset
	+ Booking end offset
	+ Min duration (initially set to the first duration in the rule's durations slice)
	+ Max duration (initially set to the first duration in the rule's durations slice plus the route's offset days)

### Duration Processing

* The function updates the min and max duration properties of the `RouteData` struct by iterating over each duration in the rule's durations slice.
* It adds the route's offset days to each duration and updates the min and max duration properties accordingly.

### Route Data Processing and Update

* The function processes each rule in the destination rules slice to generate route data.
* It checks if the max stops property of the route is valid (not nil and not negative), and if so, updates the max legs property of the route data.

### Corporate Codes Processing

* The function appends corporate codes from the config to the route data's corpo codes array.
* It removes duplicates from the corpo codes array using the `collectionutil.DedupStringArray` function.
* It sorts the corpo codes array in ascending order using the `sort.Strings` function.

### Search By FBA Processing

* The function checks if free baggage allowance is allowed for the destination IATA code, and updates the route data's search by FBA property accordingly.

### Mandatory Carriers Processing

* The function determines the mandatory carriers based on the sale airline codes and config search carriers.
* If there are no mandatory carriers, it uses the preferred carriers from the config instead.

### Cabin Processing

* The function sets the cabin property of the route data to "BUSINESS" if the sale cabin is BUSINESS.

### Route Data Update or Insertion

* The function checks if a route with the same key already exists in the result map.
* If it does, it updates the existing route data's booking start and end offsets if necessary.
* Otherwise, it inserts the new route data into the result map.

## GenerateAmadeusSilos

### Function Purpose

* The `GenerateAmadeusSilos` function generates a list of Amadeus Silos based on input parameters.
* It takes in four string slices: `cabin`, `airlines`, `arrivalAirportCodes`, and `returnOriginAirportCodes`.
* The function returns a slice of strings representing the generated Amadeus Silos.

### Input Parameters

* `cabin`: A single string representing the cabin type (e.g., "Economy", "Premium Economy").
* `airlines`: A slice of strings representing the airlines involved in the booking.
* `arrivalAirportCodes`: A slice of strings representing the airport codes for arrival destinations.
* `returnOriginAirportCodes`: A slice of strings representing the airport codes for return origin destinations.

### Special Cases

* If there are no return origin airport codes, and either:
	+ The cabin is empty and there are no airlines (returns a single default silo: `DefautSilo`).
	+ The cabin is not empty or there are airlines (returns a single silo with the cabin type and airline codes).

### General Case

* If there are return origin airport codes, the function generates multiple Amadeus Silos based on combinations of arrival and return airports.
* For each combination:
	+ A unique silo code is generated using the "OJ" prefix and the arrival and return airport codes (if different).
	+ The cabin type and airline codes are appended to the silo code.

### Return Value

* The function returns a slice of strings, where each string represents an Amadeus Silo generated based on the input parameters.