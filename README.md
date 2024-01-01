# nosql-challenge

This Repository utilizes MongoDB and Pymongo to manipulate and analyze data from UK restaurants as represented in the file [establishments.json](Resources/establishments.json). This data contains names, local authorities, geolocations, and ratings data (among others) used to analyze the quality of local UK restaurants.

## Part 1: Database Setup

In part one, using the [NoSQL setup starter notebook](NoSQL_setup_starter.ipynb), first, the data was imported from the source file using the following code in the terminal:

<code> mongoimport --type json -d uk_food -c establishments --drop --jsonArray Resources/establishments.json</code>

This code dropped any existing establishments collections and replaced them with the json from <i>establishments.json</i>

Once this data was imported, the Mongo Client was created, and the uk_food database was assigned to a variable "uk_food" as well as the "establishments" collection assigned to an "establishments" variable.

## Part 2: Update the Database

Using the same notebook, a new establishement ('Penang Flavours') was added to the <i>establishments</i> collection. Missing data was added by querying the collection's BusinessTypeID field for "Restaurant/Cafe/Canteen" and the new establishment was updated with this ID number.

Additionally, all documents with the "Dover" LocalAuthorityName were dropped from the collection for exclusion from the analysis in part 3.

Lastly, the datatypes of <i>geocode.latitude</i> and <i>geocode.longitude</i> were changed to decimal and the datatype for <i>RatingValue</i> was changed to an integer format and all non-numerical entries were set to null.

## Part 3: Exploratory Analysis

This section uses the [NoSql analysis starter notebook](NoSQL_analysis_starter.ipynb) to answer the following questions from the database set up in parts 1 and 2:

<ol>
    <li>Which establishments have a hygiene score equal to 20?</li>
    <li>Which establishments in London have a RatingValue greater than or equal to 4</li>
    <li>What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new "Penang Flavours" restaurant?</li>
    <li>How many establishments in each Local Authority area have a hygiene score of 0 (sorted from highest to lowest?)</li>
</ol>

1. This was queried using <code>{'scores.Hygiene': {'$eq':20}}</code> It was discovered that 41 establishments had a rating of 20. This information was printed and converted to a Pandas DataFrame

2. This was queried using <code>{'RatingValue': {'$gte': 4}, 'LocalAuthorityName': {'$regex': 'London'}}</code> It was discovered that 33 establishments in London have a RatingValue >= 4. This was also converted to a Pandas DataFrame

3. This was queried with the $and comparison operator with the criteria of .01 +/- the latitude and longitude of "Penang Flavours" and a RatiingValue of 5. These were then sorted by lowest to highest hygiene score with <code>[('scores.Hygiene', 1)]</code> and cast as a limited list of 5 results with <code>list(establishments.find(query).sort(sort).limit(5))</code> This was also converted to a Pandas DataFrame. The results are as follows:

    <ol>
        <li>Volunteer</li>
        <li>Plumstead Manor Nursery</li>
        <li>Atlantic Fish Bar</li>
        <li>Iceland</li>
        <li>Howe and Co Fish and Chips -Van 17</li>
    </ol>

4. For this query, an aggregation pipeline was built using the following queries: 
    <ul>
        <li>match_query to match hygiene scores to 0</li>
        <li>group_query matches counts of _id grouped by LocalAuthorityName</li>
        <li>sort_values sorts by count in descending order</li> 
    </ul>
This pipeline is then aggregated and cast as a list to be saved to a variable. Finally, the list is converted to a Pandas DataFrame. 

The 55 distinct LocalAuthorities had the following top 10 results:
<ul>
    <li>{'_id': 'Thanet', 'count': 1130}</li>
    <li>{'_id': 'Greenwich', 'count': 882}</li>
    <li>{'_id': 'Maidstone', 'count': 713}</li>
    <li>{'_id': 'Newham', 'count': 711}</li>
    <li>{'_id': 'Swale', 'count': 686}</li>
    <li>{'_id': 'Chelmsford', 'count': 680}</li>
    <li>{'_id': 'Medway', 'count': 672}</li>
    <li>{'_id': 'Bexley', 'count': 607}</li>
    <li>{'_id': 'Southend-On-Sea', 'count': 586}</li>
    <li>{'_id': 'Tendring', 'count': 542}</li>
</ul>
