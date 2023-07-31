# V3

This repo contains all of the code which was used to develop and deploy the OpenAlex v3 Author Name Disambiguation (AND) System.

## Model Basics

This model uses outputs from our concepts (v3) and institutions (v2) models, as well as citations and coauthors data in order to disambiguate 2 works that have names which could be the same author. The output of that model was sent through a custom clustering process that gave us the authors which are now deployed in OpenAlex. Eventually there will be a document created which details the entire system in more detail, but for now this page will contain that high-level information.

## Modeling Process

### Data Used

The main features of our AND system are the author's name, the institutions that are tagged using our v2 institution parser, the concepts that are tagged using our v3 concepts tagger, the citations attached to a work, the coauthors on a work, and an ORCID if available. The data is taken directly from our database so it is possible to pull the data yourself in order to recreate this system. These features are compiled for each author for each work (work_author) and for work_authors that could be the same person based on name alone, the features are compared against each other using a Disambiguator (XGBoost) model. The model gives us a probability that the 2 work_authors are same and we take those probabilities into a custom clustering process. A lot of thought went into creating the training dataset and those considerations will be discussed in the document that will be released at a later date. The datasets were created using ORCID to find author names that were similar or the same but most likely not the same person (hard negatives) whereas the postive pairs were just works chosen within the same ORCID profile. There are a few datasets for this system can be found at Zenodo: 

1. Datasets of ORCID profiles that have the same name or similar name
2. Full dataset of samples before training data curation
3. Actual training/validation/testing datasets used to train Disambiguator model (post-curation)

The datasets can be found at the following link: https://zenodo.org/record/8200679

### Clustering

ORCID is used as a source of truth for this entire process, so our initial clusters were created using ORCID. In addition to using ORCID to form our base clusters, we also use it to compare 2 work_authors and if the ORCIDs exist and do not match, those work_authors are automatically removed from consideration for being a match. We use a clustering method that continously updates cluster information for each "round" of clustering. Work_authors that have a higher matching score from the Disambiguator Model are given preference over lower scoring work_author pairs. Many different clustering methods were attempted and the best result was obtained by making sure that each round of clustering only allowed a single change for each cluster. This allowed us to apply ORCID to new additions in a cluster to make sure we were not creating ORCID clashes within a cluster and so we could (as much as possible) keep names consistent in a cluster. Basically, after each round of clustering we re-ran the function to compare the names of 2 work_authors to make sure all names of the cluster are taken into account. To explain this, we will give a quick example:

If before a round of clustering there are 3 clusters:
* Cluster 1: Work_authors that all contain the initialed name "J. Priem"
* Cluster 2: Work_authors that all contain the name "Jason Priem"
* Cluster 3: Work_authors that all contain the name "Joseph Priem"

At this point, Cluster 1 (and all works contained within) could belong to anyone with the last name "Priem" and the first initial "J". Therefore, both Cluster 2 and Cluster 3 are candidates for merging with it. Let's say through our next round of clustering there is a merge of Cluster 1 and Cluster 2 (because a work_author in Cluster 1 had a high model score with a work_author in Cluster 2). This would mean all of the works in Cluster 1 ("J. Priem") are now associated with "Jason Priem" and for future rounds of clustering, the cluster information must be updated accordingly.
* NEW CLUSTER: Work_authors that all contain the initialed name "J. Priem" and "Jason Priem"
* Cluster 3: Work_authors that all contain the name "Joseph Priem"

Some work_authors that may have matched "J. Priem" in the last round will no longer match since that has been claimed by a full name ("Jason Priem"). Updating the cluster information every round is an expensive task but we do it to try (as much as possible) to keep the names within a cluster related so that there are no clusters where "Jason Priem" and "Joseph Priem" are tied together. The one caveat to this is if an ORCID is used. We join on ORCID no matter the name so if there are multiple names in our system that are attached to an ORCID, that cluster will have multiple names. This does not work out well when the data in our system is not up-to-date or if multiple people are using a single ORCID. However, this is a good way to introduce shortened names, nicknames, and name changes into our author clusters. So if you are trying to make your author profile look as good/complete as possible, the best way to do that is to have an ORCID attached to your works.

### Results

This new AND system resulted in about 92 million author clusters being created. 38 million are authors with multiple works while 54 million are authors with a single work. Over time, we expect the number of authors in our system to decrease as we apply updates and further refine our disambiguation system. But for now, we have a stable set of clusters that have mostly looked good in qualitative testing. The data processing, scoring, and clustering process took over 2 weeks to complete on a large Databricks clusters so be warned, if you want to imitate this system on the full OpenAlex database, it will be expensive. It would be considerably less expensive if you wanted to limit your data to a specific topic or journal or some other filter that greatly reduces the number of total possible authors to disambiguate.

### Deployment

After the clustering was completed, a modified clustering algorithm was deployed in a Databricks notebook that runs every 6 hours. In the future, we plan to change this system so that it can be semi-live (assign authors to new works automatically if there is an author cluster to match to, if there is no match it will go through a longer process) but our only concern was getting the AND system up and running as soon as possible.


## Directories

### 001_Data_Exploration_Transformation

These notebooks show code that was used to manipulate and explore the data and also create the training data.

### 002_Data_Processing_Modeling_Clustering

These notebooks show code that was used to create the Disambiguator Model (XGBoost), transform all of our data for scoring, scoring the data using the model, and clustering the data to create disambiguated clusters of works that we assign an Author ID. The Disambiguator model file is also included in this directory.

### 003_Deployment

Notebooks containing the code used to deploy the model in Databricks.






