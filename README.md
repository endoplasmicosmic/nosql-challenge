# NoSQL Analysis - Establishments Hygiene Ratings

## Background

This project focuses on analyzing the hygiene ratings of various establishments using MongoDB. The analysis involves querying the data, transforming it into a useful format, and creating meaningful insights based on these queries.

## Project Structure

### Files

- `NoSQL_setup_starter.ipynb`: Jupyter Notebook containing the initial setup and data import into MongoDB.
- `NoSQL_analysis_starter.ipynb`: Jupyter Notebook containing the analysis of the hygiene ratings data.

### Data Analysis

#### Initial Setup and Data Import

The setup involves:
1. **Importing Data**: The data provided in the `establishments.json` file is imported into a MongoDB database. The command used for this import is documented in the notebook to ensure repeatability.
    ```bash
    mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json
    ```
2. **Library Imports**: Necessary libraries like PyMongo and Pretty Print (`pprint`) are imported.
3. **Mongo Client Creation**: An instance of the Mongo Client is created to interact with the MongoDB server.
4. **Database and Collection Check**: Confirming the creation of the database and the correct loading of data.
5. **Assigning Collection to Variable**: The `establishments` collection is assigned to a variable for further use.

#### Database Update

Modifications to the database as per the magazine editors' requests:
1. **Adding a New Restaurant**: Adding a new halal restaurant "Penang Flavours" in Greenwich.
    ```python
    new_restaurant = {
        "BusinessName": "Penang Flavours",
        "BusinessType": "Restaurant/Cafe/Canteen",
        "BusinessTypeID": "",
        "AddressLine1": "Penang Flavours",
        "AddressLine2": "146A Plumstead Rd",
        "AddressLine3": "London",
        "AddressLine4": "",
        "PostCode": "SE18 7DY",
        "Phone": "",
        "LocalAuthorityCode": "511",
        "LocalAuthorityName": "Greenwich",
        "LocalAuthorityWebSite": "http://www.royalgreenwich.gov.uk",
        "LocalAuthorityEmailAddress": "health@royalgreenwich.gov.uk",
        "scores": {
            "Hygiene": "",
            "Structural": "",
            "ConfidenceInManagement": ""
        },
        "SchemeType": "FHRS",
        "geocode": {
            "longitude": "0.08384000",
            "latitude": "51.49014200"
        },
        "RightToReply": "",
        "Distance": 4623.9723280747176,
        "NewRatingPending": True
    }
    establishments.insert_one(new_restaurant)
    ```
2. **Updating Business Type**: Finding and updating the `BusinessTypeID` for the new restaurant.
    ```python
    business_type = establishments.find_one({"BusinessType": "Restaurant/Cafe/Canteen"}, {"BusinessTypeID": 1, "BusinessType": 1})
    establishments.update_one({"BusinessName": "Penang Flavours"}, {"$set": {"BusinessTypeID": business_type["BusinessTypeID"]}})
    ```
3. **Removing Establishments in Dover**: Deleting all establishments in Dover.
    ```python
    establishments.delete_many({"LocalAuthorityName": "Dover"})
    ```
4. **Data Type Correction**: Converting number values stored as strings to appropriate types.
    ```python
    establishments.update_many({}, [{"$set": {"geocode.longitude": {"$toDouble": "$geocode.longitude"}, "geocode.latitude": {"$toDouble": "$geocode.latitude"}}}])
    establishments.update_many({}, [{"$set": {"RatingValue": {"$toInt": "$RatingValue"}}}])
    ```

#### Exploratory Analysis

The exploratory analysis addresses specific questions posed by the magazine editors:
1. **Hygiene Score Equal to 20**:
    ```python
    query = {"scores.Hygiene": 20}
    results = list(establishments.find(query))
    df = pd.DataFrame(results)
    ```
2. **Establishments in London with RatingValue >= 4**:
    ```python
    query = {"LocalAuthorityName": {"$regex": "London", "$options": "i"}, "RatingValue": {"$gte": 4}}
    results = list(establishments.find(query))
    df = pd.DataFrame(results)
    ```
3. **Top 5 Establishments by RatingValue of 5 and Lowest Hygiene Score**:
    ```python
    penang_flavours = establishments.find_one({"BusinessName": "Penang Flavours"})
    latitude = float(penang_flavours["geocode"]["latitude"])
    longitude = float(penang_flavours["geocode"]["longitude"])
    degree_search = 0.01
    query = {
        "geocode.latitude": {"$gte": latitude - degree_search, "$lte": latitude + degree_search},
        "geocode.longitude": {"$gte": longitude - degree_search, "$lte": longitude + degree_search},
        "RatingValue": 5
    }
    results = list(establishments.find(query).sort([("scores.Hygiene", 1)]).limit(5))
    df = pd.DataFrame(results)
    ```
4. **Establishments with Hygiene Score of 0 by Local Authority**:
    ```python
    pipeline = [
        {"$match": {"scores.Hygiene": 0}},
        {"$group": {"_id": "$LocalAuthorityName", "count": {"$sum": 1}}},
        {"$sort": {"count": -1}}
    ]
    results = list(establishments.aggregate(pipeline))
    df = pd.DataFrame(results)
    ```

### Running the Analysis

1. Open `NoSQL_setup_starter.ipynb` and follow the instructions to import the data into MongoDB.
2. Open `NoSQL_analysis_starter.ipynb` to perform the data analysis.

### Sources

- [MongoDB Documentation](https://docs.mongodb.com/)
- [Pandas Documentation](https://pandas.pydata.org/pandas-docs/stable/)
- [Matplotlib Documentation](https://matplotlib.org/stable/contents.html)
