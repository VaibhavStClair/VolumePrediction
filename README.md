# What is this project about?

This project involves creating a data pipeline that is used for deploying machine learning model.
It uses Spark and Python for data integration and transformations. 

The Project is divided into 4 phases:


>Raw Data Processing

>Feature Engineering

>Integrate ML training

>Model Serving

For phase 1 and 2, PySpark was used to effectively Extract, Transform and Load the data.

For phase 3 and 4, Python and Flask API was used to fit and deploy the model.

# Steps of Execution

This section explains the extract transform load for the part one and two of the project. 
First we'll start by creating the spark session. 
So we have to load the data inside spark for parallel computation and processing, for that we are first defining the schema for both the datasets. 
Next step is to declare the path where the data is residing and where it should be stored after transformations. 

There are 3 main functions have built in Utilities/ETL.py:
1. load_files: Here, a dictionary for which the key will be the name of the file and the value will be the data frame. 
I tried many methods for loading the data sets into memory like repartition or coalesce or changing the config parameters while loading, but they all seem to give me out of memory issue. 
So for now I have taken first 40 files in both the datasets due to my laptop's limited RAM and cores. 

NOTE: ***If you want to run all the files, please uncomment line 90,93, 102 and 103 in ETL.py file***

2. writeparquet: A dictionary with key and value pair, writing the data frames as the value with the name as keys of the dictionary into storage. 

3. transformation: this will take the date column in string format and convert it to timestamp first and then converting it back to the date format. 
After that I am getting the month and year from this date column. We need to define the windows function here so that we can partition the data for calculating the moving average and the ruling median. 
For that the partition is being done by month year and symbol will stop the moving average and the ruling medium are calculated as required. 
All these functions are in a class and to call these I'm just making an object and calling the class. 
The next step is to store the final data which is union of both the stocks and the ETF's data set, and I'm storing it into a different directory. 

The files that are downloaded from the kaggle are being stored in the bronze directory, after transformation and applying rolling median and moving average the data are stored in silver directory, and finally the union of the data sets are stored in the gold directory. 
The next step is to apply the machine learning models as given by the document for that we are importing the final data set from the gold layer and then applying the random forest regressor. 
Here the data was split in training and testing for 80% in training and 20% in testing. 
After fitting the model to the training data set the prediction variable is made on the test data set. 
The machine learning model outputs the prediction variable, the mean absolute error, and the mean squared error. 

Log files and pickle files are saved to the directory.

# How to run on your machine

1. Clone the project and place your ETF and Stocks data files in '../VolumePrediction/bronze/' (for e.g ../VolumePrediction/bronze/etf)
2. Run pip install -r requirements.txt for installing libraries
3. Execute both ETL.py and volume_predictor.py
4. For deployment, please refer next section

# Updated Deployment

First, run the ETL.py file. It will generate a folder structure like this

![image](https://user-images.githubusercontent.com/90218716/236530561-f5974af8-eff4-4196-ada0-c67011a9f478.png)

Then, run the volume_predictor.py file which will give the logs of model and save model in pickle format in the directory

![image](https://user-images.githubusercontent.com/90218716/236531302-fcabc173-22c1-4c20-aff3-a004e8a44e08.png)

![image](https://user-images.githubusercontent.com/90218716/236531699-17d5deb8-551d-4769-8537-6c4d87b43d50.png)


To serve the Machine Learning API, 2 files were made, api_app.py and test.py.
For running the server on localhost, we can still use the file app.py 

api_app.py file will
1. Load the data in pickle format
2. Make a class for inputting the necessary variable's type 
3. Define an endpoint /volume_prediction 
4. Define the variable machine learning feature values and use the predict function on it. 

Meanwhile to import the necessary libraries, requirements.txt was also updated with following libraries: 

pydantic

fastapi

uvicorn

pypi-json

To run api_app.py, we have to go to terminal and give command "uvicorn api_app:app --reload" 
Next step is the run the test.py file which has the output of url given by uvicorn and add the endpoint defined in api_app.py. example - "http://127.0.0.1:8000/volume_prediction"
The test.py contains the input_data which gets converted to json and loaded back to api request.
Running the file will give a single response text which is the predicted volume.

The result generated will look like this

![image](https://user-images.githubusercontent.com/90218716/236528678-441c68e6-26c3-4170-9405-cee5900ccac8.png)

DATA FLOW:

![image](https://user-images.githubusercontent.com/90218716/236104446-e811ea5b-5456-4540-a4ce-60354741c1c1.png)


References:

https://stackoverflow.com/questions/32336915/pyspark-java-lang-outofmemoryerror-java-heap-space
https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/
https://gist.github.com/malanb5/0179157f732c9765c3bf6023ebbf2077
https://sparkbyexamples.com/pyspark/pyspark-window-functions/
https://www.learneasysteps.com/how-to-calculate-median-value-by-group-in-pyspark/
https://www.linkedin.com/pulse/time-series-moving-average-apache-pyspark-laurent-weichberger/
https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html
https://www.geeksforgeeks.org/deploy-machine-learning-model-using-flask/
