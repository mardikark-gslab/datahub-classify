# datahub-classify

Predict InfoTypes for [DataHub](https://datahubproject.io/).

## Installation

`python3 -m pip install --upgrade acryl-datahub-classify`

# API 'check_similarity()'
This API computes the similarity score between a pair of tables and also between all possible pairs of their constituent columns.

API works in two modes "pruning" and "non-pruning". In "pruning" mode only table similarity is calculated with a lenient algorithm and in "non-pruning" mode both table and column similarity are calculated. The table similarity is calculated with a stricter method in non pruning mode.

Following are the input and output contract:
### API Input
API expects following parameters in the input
- `table_info1` & `table_info2` -  These are instances of TableInfo object. Each TableInfo object contains following class variables:
  - `metadata` - Instance of TableMetadata class containing name, description, platform, table-ID, name_embedding(optional) & desc_embedding(optional) of the table 
  - `parent_tables` - List of table IDs of parent tables of the input table 
  - `column_infos` - List of instances of ColumnInfo object for each constituent column of the table. Each ColumnInfo object has following the class variables 
    - `metadata` - Instance of ColumnMetadata class containing name, description, datatype, dataset_name, column-id, name_embedding(optional) & desc_embedding(optional) of the column 
    - `parent_columns` - List of column IDs of parent columns of the column
- `pruning_mode` - This is a boolean flag, it indicates whether API will run the pruning or non-pruning model. Default value is False (non-pruning mode)
- `use_embeddings` - This is a boolean flag, this tells whether to use meaning based similarity for tables and columns. Default value is False. If this flag is set to True then table/column name and description embeddings needs to be passed in the TableInfo object to API (name/description embeddings can be generated using `preprocess_tables()` API).

### API Output
API returns two objects:
- `table_similarity_score` - This is an instance of SimilarityInfo which contains 'score' and 'prediction_factors_scores'. 'score' is the overall similarity score of the input pair and 'prediction_factors_scores' is the instance of SimilarityDebugInfo class describing the confidence and weighted score of each prediction factor.  
- `column_similarity_scores` - This is a dictionary of similarity scores of all column pairs. Key being a tuple of column IDs of the column pair and value being the instance of SimilarityInfo, it contains the semantic similarity score between two columns and confidence & weighted score of each prediction factor.

### Usage of the API
Find usage of the "check_similarity()" API at following link
https://github.com/mardikark-gslab/datahub-classify/blob/stage_2_dev/datahub-classify/tests/demo_similarity_check.py

### Assumptions
Following are the assumptions about input parameters 
- table_name, table_platform & table_schema are the required parameters in TableInfo object.
- col_name, col_datatype are the required parameters in ColumnInfo object.


# API 'preprocess_tables()'
This API generates the embedding for tables (i.e. for name and description of the table and columns)
### API Input
API expects following parameter in the input
- `table_info_list` - This list contains a batch of tables (i.e. instance of TableInfo) for which embedding needs to be populated

### API Output
API returns following parameter
- `table_info_list` - This is a list of TableInfo objects of same length as input. This API populates the embedding of name $ description of the table (and included columns) and updates the same TableInfo and ColumnInfo objects with "name_embedding" and "desc_embedding". Value of "name_embedding" and "desc_embedding" is a list of 'TextEmbeddings' instances.

# Development

### Set up your Python environment

```sh
cd datahub-classify
../gradlew :datahub-classify:installDev # OR pip install -e ".[dev, semantic_similarity]"
source venv/bin/activate
```

### Runnning tests

```sh
pytest tests/ --capture=no --log-cli-level=DEBUG
```

### Sanity check code before committing

```sh
# Assumes: pip install -e ".[dev]" and venv is activated
black src/ tests/
isort src/ tests/
flake8 src/ tests/
mypy src/ tests/
```

### Build and Test

```sh
../gradlew :datahub-classify:build
```

You can also run these steps via the gradle build:

```sh
../gradlew :datahub-classify:lint
../gradlew :datahub-classify:lintFix
../gradlew :datahub-classify:testQuick
```
