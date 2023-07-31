### 002_Data_Processing_Modeling_Clustering

* 001_disambiguator_model_creation: Creating XGBoost model
* 002_disambiguator_model_testing: Testing XGBoost model and clustering methods
* 003_transform_data_for_scoring_PYSPARK: Transforming data for scoring all data
* 004_score_all_data_PYSPARK: Scoring all data with the Disambiguator (XGBoost) model
* 005_initial_clustering_PYSPARK: Initial Clustering of all data to create Author clusters

There is also a "Disambiguator.pkl" file which is the XGBoost model used for disambiguation. The model was created in xgboost==1.7.5 so make sure you have that version installed when trying to load it.