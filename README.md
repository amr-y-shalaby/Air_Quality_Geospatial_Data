<!-- TOC -->
* [Objective](#objective)
* [Pipeline](#pipeline)
  * [§0.1 Flow](#01-flow)
  * [§0.2 Execution Time](#02-execution-time)
  * [§1.Setup: Config.ini](#1setup-configini)
  * [§2. Python Requirements](#2-python-requirements)
  * [§3. Execution](#3-execution)
* [Pipeline Design](#pipeline-design)
  * [1. Staging (Extraction) Layer](#1-staging-extraction-layer)
    * [§1. Monthly Data Web Scraping](#1-monthly-data-web-scraping)
    * [§2. Monthly Forecasts Data Web Scraping](#2-monthly-forecasts-data-web-scraping)
    * [§3. Traffic Volume](#3-traffic-volume)
    * [§4. Geographical Names Data](#4-geographical-names-data)
    * [§5. ArcGIS Toronto and Peel Traffic Count](#5-arcgis-toronto-and-peel-traffic-count)
  * [2. Transformation Layer](#2-transformation-layer)
    * [§1. Monthly Air Data Transpose](#1-monthly-air-data-transpose)
  * [3. Production (Loading) Layer:](#3-production-loading-layer)
    * [§1. Monthly Air Data](#1-monthly-air-data)
    * [§2. Monthly Forecasts](#2-monthly-forecasts)
    * [§3. Traffic Volume](#3-traffic-volume-1)
    * [§4. Geographical Names Data](#4-geographical-names-data-1)
    * [§5. ArcGIS Toronto and Peel Traffic Count](#5-arcgis-toronto-and-peel-traffic-count-1)
    * [§6. Data Model Performance](#6-data-model-performance)
    * [§7. fact_air_data_proj](#7-fact_air_data_proj)
    * [§8. fact_gta_traffic_proj](#8-fact_gta_traffic_proj)
    * [§9. fact_hourly_avg](#9-fact_hourly_avg)
    * [§10. fact_weekdays_avg](#10-fact_weekdays_avg)
<!-- TOC -->


# Objective
To complement Government of Canada Weather Stations Data, both actual and forecasted, with ArcGIS Traffic for a unified, geospatial analysis powered by a single, automated pipeline.

# Pipeline
## §0.1 Flow
![Report](Report/pipeline_steps.png)

## §0.2 Execution Time
![Report](Report/pipeline_execution_time.png)

## §1.Setup: Config.ini
The only file that requires user input is _**Config.ini**_ in _**Pipeline**_ folder.  Update the database login credentials and the [parent_dir](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/config.ini) **_parent_dir_** without quotations as below.

Congfig.ini Contents:

|    [postgres_db]     |                  [save_files]                  |
|:--------------------:|------------------------------------------------|
|   host = localhost   |           save_locally_flag = False            |
| db_name   = postgres | parent_dir = /Users/amr/PycharmProjects/ggr473 |
|   user = postgres    |                                                |
| password = postgres  |                                                |
|     port = 5432      |                                                |

## §2. Python Requirements

You need to have the following packages installed.
* pandas~=2.1.3
* requests~=2.31.0
* SQLAlchemy~=2.0.23
* rpy2~=3.5.14
* wget~=3.2
* beautifulsoup4~=4.12.2
* geopandas~=0.14.1


## §3. Execution
After modifying [Config.ini](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/config.ini), run the python script [execute_pipeline.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/execute_pipeline.py).

# Pipeline Design

## 1. Staging (Extraction) Layer
### §1. Monthly Data Web Scraping
To download the Ontario monthly air quality data from https://dd.weather.gc.ca/air_quality/aqhi/ont/observation/monthly/csv/, function _extract_monthly_data()_in the Data Extractor [data_extractor.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_extractor.py) File.
which ingests the public data into the **first** staging table "**stg_monthly_air_data**."

| Column         | Data Type        |
|----------------|------------------|
| the_date       | date             |
| hour_utc       | bigint           |
| FAFFD          | double precision |
| FALIF          | double precision |
| FALJI          | double precision |
| FAMXK          | double precision |
| FAYJG          | double precision |
| FAZKI          | double precision |
| FBKKK          | double precision |
| FBLJL          | double precision |
| FBLKS          | double precision |
| FCAEN          | double precision |
| FCCOT          | double precision |
| FCFUU          | double precision |
| FCGKZ          | double precision |
| FCIBD          | double precision |
| FCKTB          | double precision |
| FCNJT          | double precision |
| FCTOV          | double precision |
| FCWFX          | double precision |
| FCWOV          | double precision |
| FCWYG          | double precision |
| FDATE          | double precision |
| FDCHU          | double precision |
| FDEGT          | double precision |
| FDGED          | double precision |
| FDGEJ          | double precision |
| FDGEM          | double precision |
| FDJFN          | double precision |
| FDMOP          | double precision |
| FDQBU          | double precision |
| FDQBX          | double precision |
| FDSUS          | double precision |
| FDZCP          | double precision |
| FEAKO          | double precision |
| FEARV          | double precision |
| FEBWC          | double precision |
| FEUTC          | double precision |
| FEUZB          | double precision |
| FEVJR          | double precision |
| FEVJS          | double precision |
| FEVJV          | double precision |
| FEVNS          | double precision |
| FEVNT          | double precision |
| last_updated   | timestamp        |
| download_link | text             |
| src_filename   | text             |


### §2. Monthly Forecasts Data Web Scraping
To download the Ontario monthly air quality data from https://dd.weather.gc.ca/air_quality/aqhi/ont/forecast/model/csv/, the function _extract_monthly_forecasts()_ in the Data Extractor [data_extractor.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_extractor.py) File.
which ingests the public data into the **third** staging table "**stg_monthly_forecasts**" with the option to locally appending all the individual CSV files into one file with a prefix **'FORECAST_'** to differentiate it from the actual monthly data.


|      Column       | Data Type |
|:-----------------:|:---------:|
|    cgndb_code     |   text    |
|  community_name   |   text    |
|   validity_date   |   date    |
| validity_time_utc |   text    |
|      period       |  bigint   |
|       value       |  bigint   |
|      amended      |  bigint   |
|   last_updated    | timestamp |
|  donwnload_link   |   text    |
|   src_filename    |   text    |


### §3. Traffic Volume
Using the REST API provided for Toronto Traffic Volume https://open.toronto.ca/dataset/traffic-volumes-at-intersections-for-all-modes/, the function _extract_traffic_volumes()_ in the [data_extractor.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_extractor.py) File.
which ingests the traffic volume data into the **fourth** staging table "**stg_traffic_volume**" with the option to locally save the CSV file to _['./Data/traffic_volume.csv'](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Data/traffic_volume.csv).

|      Column       |    Data Type     |
|:-----------------:|:----------------:|
|        _id        |      bigint      |
|    location_id    |      bigint      |
|     location      |       text       |
|        lng        | double precision |
|        lat        | double precision |
|  centreline_type  | double precision |
|   centreline_id   | double precision |
|        px         | double precision |
| latest_count_date |       text       |
|   download_link   |       text       |
|     filename      |       text       |
|   last_updated    |    timestamp     |

### §4. Geographical Names Data
Downloads and extracts the zip file of the Geographical Names Data from https://natural-resources.canada.ca/earth-sciences/geography/download-geographical-names-data/9245, the function _extract_geo_names()_ in the [data_extractor.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_extractor.py).
which ingests the geographical names data containing the coordinates of the air quality stations into the **fifth** staging table "**stg_geo_names**" with the option to retain the extracted CSV file to _'./Data/cgn_canada_csv_eng.csv'_.

|        Column        |    Data Type     |
|:--------------------:|:----------------:|
|       cgndb_id       |       text       |
|  geographical_name   |       text       |
|       language       |       text       |
|    syllabic_form     |       text       |
|     generic_term     |       text       |
|   generic_category   |       text       |
|     concise_code     |       text       |
| toponymic_feature_id |       text       |
|       latitude       | double precision |
|      longitude       | double precision |
|       location       |       text       |
|  province_territory  |       text       |
|  relevance_at_scale  |      bigint      |
|    decision_date     |       date       |
|        source        |       text       |
|     last_updated     |    timestamp     |
|    download_link    |       text       |
|     src_filename     |       text       |

### §5. ArcGIS Toronto and Peel Traffic Count
Downloads Toronto and Peel Traffic Count from https://www.arcgis.com/home/item.html?id=4964801ff5de475a80c51c5d54a9c8da, the function _extract_geo_names()_ in the [data_extractor.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_extractor.py).
which ingests the Toronto and Peel Traffic Counts with latitude and longitude provided by ArcGIS  into the **sixth** staging table "**load_gta_traffic_arcgis**" with the option to create CSV file to _'./Data/ArcGIS_Toronto_and_Peel_Traffic.csv'_.

|         Column         |    Data Type     |
|:----------------------:|:----------------:|
|        objectid        |      bigint      |
|         tcs__          | double precision |
|          main          |       text       |
|     midblock_route     |       text       |
|      side_1_route      |       text       |
|      side_2_route      |       text       |
|    activation_date     |       date       |
|        latitude        | double precision |
|       longitude        | double precision |
|        latitude        | double precision |
|       longitude        | double precision |
|       count_date       |       date       |
|  f8hr_vehicle_volume   | double precision |
| f8hr_pedestrian_volume | double precision |
|      last_updated      |    timestamp     |
|     download_link     |       text       |
|      src_filename      |       text       |


## 2. Transformation Layer
### §1. Monthly Air Data Transpose
The staging table _stg_monthly_air_data_transpose_ created from transposing the Ontario monthly air quality (https://dd.weather.gc.ca/air_quality/aqhi/ont/observation/monthly/csv/) to be ingested and converted 
into the **second** production table "**FACT_MONTHLY_AIR_DATA_TRANSPOSE** in _**"PUBLIC"**_ Schema via the execution of [data_transformation.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_transformation.py) with the following two conditions:

* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() OVER(PARTITION BY "the_date","hours_utc" ORDER BY hours_utc DESC) = 1_to eliminate duplicate records within any given date.


| Column            |    Data Type     |
|-------------------|:----------------:|
| the_date          |       date       |
| hour_utc          |      bigint      |
| cgndb_id          |       text       |
| air_quality_value | double precision |
| donwnload_link    |       text       |
| src_filename      |       text       |
| last_updated      |    timestamp     |
| last_inserted     |    timestamp     |


## 3. Production (Loading) Layer:
### §1. Monthly Air Data
The staging table _stg_monthly_air_data_ created from the Ontario monthly air quality (https://dd.weather.gc.ca/air_quality/aqhi/ont/observation/monthly/csv/) ingested and converted into the **first** production table "**FACT_MONTHLY_AIR_DATA** in _**"PUBLIC"**_ Schema via [data_loader.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_loader.py) with the following two conditions:

* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() OVER(PARTITION BY "Date","hours_utc" ORDER BY hours_utc DESC) = 1_to eliminate duplicate records within any given date.

|    Column     |    Data Type     |
|:-------------:|:----------------:|
|     date      |       date       |
|   hour_utc    |      bigint      |
|     FAFFD     | double precision |
|     FALIF     | double precision |
|     FALJI     | double precision |
|     FAMXK     | double precision |
|     FAYJG     | double precision |
|     FAZKI     | double precision |
|     FBKKK     | double precision |
|     FBLJL     | double precision |
|     FBLKS     | double precision |
|     FCAEN     | double precision |
|     FCCOT     | double precision |
|     FCFUU     | double precision |
|     FCGKZ     | double precision |
|     FCIBD     | double precision |
|     FCKTB     | double precision |
|     FCNJT     | double precision |
|     FCTOV     | double precision |
|     FCWFX     | double precision |
|     FCWOV     | double precision |
|     FCWYG     | double precision |
|     FDATE     | double precision |
|     FDCHU     | double precision |
|     FDEGT     | double precision |
|     FDGED     | double precision |
|     FDGEJ     | double precision |
|     FDGEM     | double precision |
|     FDJFN     | double precision |
|     FDMOP     | double precision |
|     FDQBU     | double precision |
|     FDQBX     | double precision |
|     FDSUS     | double precision |
|     FDZCP     | double precision |
|     FEAKO     | double precision |
|     FEARV     | double precision |
|     FEBWC     | double precision |
|     FEUTC     | double precision |
|     FEUZB     | double precision |
|     FEVJR     | double precision |
|     FEVJS     | double precision |
|     FEVJV     | double precision |
|     FEVNS     | double precision |
|     FEVNT     | double precision |
| download_link |       text       |
| src_filename  |       text       |
| last_updated  |    timestamp     |
| last_inserted |    timestamp     |


### §2. Monthly Forecasts
The staging table _stg_monthly_forecasts_ acquired from the Ontario monthly air quality data(https://dd.weather.gc.ca/air_quality/aqhi/ont/forecast/model/csv/) is ingested into the production table _fact_monthly_forecasts_ in _PUBLIC_ schema by [data_loader.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_loader.py) with the following two conditions:

* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() OVER(PARTITION BY "cgndb_code", "validity_date" ORDER BY validity_time_utc DESC)=1_ to eliminate the duplicated records within the provided dates.


|      Column       | Data Type |
|:-----------------:|:---------:|
|    cgndb_code     |   text    |
|  community_name   |   text    |
|   validity_date   |   date    |
| validity_time_utc |   text    |
|      period       |  bigint   |
|       value       |  bigint   |
|      amended      |  bigint   |
|  donwnload_link   |   text    |
|   src_filename    |   text    |
|   last_updated    | timestamp |
|   last_inserted   | timestamp |

### §3. Traffic Volume
The staging table _stg_traffic_volume_ constructed from the REST API provided for Toronto Traffic Volume https://open.toronto.ca/dataset/traffic-volumes-at-intersections-for-all-modes/, is ingested into production _Public_ Schema as _FACT_TRAFFIC_VOLUME_ by [data_loader.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_loader.py) with the following two conditions:

* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() OVER(PARTITION BY location_id ORDER BY latest_count_date DESC)=1_ to eliminate the duplicated records within the provided dates.


|      Column       |    Data Type     |
|:-----------------:|:----------------:|
|        _id        |      bigint      |
|    location_id    |      bigint      |
|     location      |       text       |
|        lng        | double precision |
|        lat        | double precision |
|  centreline_type  | double precision |
|   centreline_id   | double precision |
|        px         | double precision |
| latest_count_date |       date       |
|   download_link   |       text       |
|     filename      |       text       |
|   last_updated    |    timestamp     |
|   last_inserted   |    timestamp     |

### §4. Geographical Names Data
The staging table _stg_geo_names_from the downloaded and extracted zip file of the Geographical Names Data(https://natural-resources.canada.ca/earth-sciences/geography/download-geographical-names-data/9245) was ingested into the production table _DIM_GEO_NAMES_ with the following two conditions:
* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() OVER(PARTITION BY cgndb_id ORDER BY decision_date DESC) =1_ to eliminate the duplicated records within the provided dates.


|        Column        |    Data Type     |
|:--------------------:|:----------------:|
|       cgndb_id       |       text       |
|  geographical_name   |       text       |
|       language       |       text       |
|    syllabic_form     |       text       |
|     generic_term     |       text       |
|   generic_category   |       text       |
|     concise_code     |       text       |
| toponymic_feature_id |       text       |
|       latitude       | double precision |
|      longitude       | double precision |
|       location       |       text       |
|  province_territory  |       text       |
|  relevance_at_scale  |      bigint      |
|    decision_date     |       date       |
|        source        |       text       |
|    download_link     |       text       |
|     src_filename     |       text       |
|     last_updated     |    timestamp     |
|    last_inserted     |    timestamp     |


### §5. ArcGIS Toronto and Peel Traffic Count
The staging table _stg_gta_traffic_arcgis_ obtained from ArcGIS Toronto and Peel Traffic Data(https://www.arcgis.com/home/item.html?id=4964801ff5de475a80c51c5d54a9c8da) was ingested into the production table _FACT_GTA_TRAFFIC_ARCGIS_ in PUBLIC schema by [data_loader.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_loader.py) with the following two conditions:

* Added column "_last_inserted_" converted from UTC to EST to capture the time of insertion into production schema
* The condition _ROW_NUMBER() over (PARTITION BY objectid ORDER BY COUNT_DATE DESC) =1_ to eliminate the duplicated records within the provided dates.

|         Column         |    Data Type     |
|:----------------------:|:----------------:|
|        objectid        |      bigint      |
|         tcs__          | double precision |
|          main          |       text       |
|     midblock_route     |       text       |
|      side_1_route      |       text       |
|      side_2_route      |       text       |
|    activation_date     |       date       |
|        latitude        | double precision |
|       longitude        | double precision |
|        latitude        | double precision |
|       longitude        | double precision |
|       count_date       |       date       |
|  f8hr_vehicle_volume   | double precision |
| f8hr_pedestrian_volume | double precision |
|     download_link      |       text       |
|      src_filename      |       text       |
|      last_updated      |    timestamp     |
|     last_inserted      |    timestamp     |


### §6. Data Model Performance
After all the staging tables, sql scripts, and hard extractions made, _data_model_performance_tbl_ in _PUBLIC_ Schema provides insight on duration, scope, and complexity of the execution steps within each of the Extraction, Transformation, and Loading Phases which are tracked by [execute_pipeline.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/execute_pipeline.py).

| Column           |    Data Type     |
|------------------|:----------------:|
| Phase            |       text       |
| step_name        |       text       |
| duration_seconds | double precision |
| start_time       |    timestamp     |
| end_time         |    timestamp     |
| files_processed  | double precision |

| **phase**  |          **step_name**          | **duration_seconds** | **start_time** | **end_time** | **files_processed** |
|:----------:|:-------------------------------:|:--------------------:|:--------------:|:------------:|:-------------------:|
|   stage    |      extract_monthly_data       |        23.47         |    17:19.6     |   17:43.0    |         26          |
|   stage    |    extract_monthly_forecasts    |        283.59        |    17:43.0     |   22:26.6    |         534         |
|   stage    |     extract_traffic_volumes     |         7.27         |    22:26.6     |   22:33.9    |          1          |
|   stage    |     extract_geo_names_data      |        27.11         |    22:33.9     |   23:01.0    |          2          |
|   stage    |   extract_gta_traffic_arcgis    |         0.19         |    23:01.0     |   23:01.2    |          1          |
|   stage    |     transform_monthly_data      |        26.99         |    23:01.2     |   23:28.2    |          1          |
| production |       traffic_volume.sql        |        0.033         |    23:28.3     |   23:28.3    |          1          |
| production |     gta_traffic_arcgis.sql      |        0.009         |    23:28.3     |   23:28.3    |          1          |
| production |      monthly_air_data.sql       |        0.108         |    23:28.3     |   23:28.4    |          1          |
| production |   fact_monthly_forecasts.sql    |        0.193         |    23:28.4     |   23:28.6    |          1          |
| production | monthly_air_data_transposed.sql |        0.732         |    23:28.6     |   23:29.3    |          1          |
| production |          geo_names.sql          |        3.587         |    23:29.3     |   23:32.9    |          1          |
| production |       traffic_volume.sql        |        0.045         |    23:32.9     |   23:33.0    |          1          |
| production |      monthly_air_data.sql       |        0.079         |    23:33.0     |   23:33.1    |          1          |
| production |   fact_monthly_forecasts.sql    |        0.178         |    23:33.1     |   23:33.2    |          1          |
| production | monthly_air_data_transposed.sql |        0.681         |    23:33.2     |   23:33.9    |          1          |
| production |          geo_names.sql          |        3.638         |    23:33.9     |   23:37.6    |          1          |
| production |       traffic_volume.sql        |        0.055         |    23:37.6     |   23:37.6    |          1          |
| production |   fact_monthly_forecasts.sql    |        0.162         |    23:37.6     |   23:37.8    |          1          |
| production | monthly_air_data_transposed.sql |        0.676         |    23:37.8     |   23:38.4    |          1          |
| production |          geo_names.sql          |        3.696         |    23:38.4     |   23:42.1    |          1          |
| production |       traffic_volume.sql        |        0.055         |    23:42.1     |   23:42.2    |          1          |
| production | monthly_air_data_transposed.sql |        0.697         |    23:42.2     |   23:42.9    |          1          |
| production |          geo_names.sql          |        2.760         |    23:42.9     |   23:45.7    |          1          |
| production |       traffic_volume.sql        |        0.036         |    23:45.7     |   23:45.7    |          1          |
| production |          geo_names.sql          |        2.973         |    23:45.7     |   23:48.7    |          1          |
| production |       traffic_volume.sql        |        0.067         |    23:48.7     |   23:48.7    |          1          |
| production |       traffic_volume.sql        |        0.015         |    23:48.7     |   23:48.7    |          1          |
| production |      combine_air_data.sql       |        0.521         |    23:48.8     |   23:49.3    |          1          |

### §7. fact_air_data_proj
The table _fact_air_data_proj_ in _PUBLIC_ schema serves as the POSTGIS Version of fact_air_data with additional geometry column 'geom' created in Python via [data_transformation.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_transformation.py) with the inherited properties of the geolocation name identifiers from the curated table _dim_geo_names_.

| Column                           |         Data Type          |
|----------------------------------|:--------------------------:|
| **geom**                         | **geometry(Point, 26917)** |
| the_date                         |            date            |
| hour_utc                         |           bigint           |
| cgndb_id                         |            text            |
| air_quality_value                |      double precision      |
| donwnload_link                   |            text            |
| src_filename                     |            text            |
| last_updated                     |         timestamp          |
| last_inserted                    |         timestamp          |
| geo_lat                          |      double precision      |
| geo_long                         |      double precision      |
| province_territory               |            text            |
| geo_location                     |            text            |
| geo_decision_date                |         timestamp          |
| concise_code                     |            text            |
| generic_category                 |            text            |
| generic_term                     |            text            |
| geographical_name                |            text            |
| geo_names_second_from_extraction |      double precision      |
| weekday                          |            text            |


### §8. fact_gta_traffic_proj
The table _fact_gta_traffic_proj_ in _PUBLIC_ schema serves as the POSTGIS Version of fact_gta_traffic_arcgis via [data_transformation.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_transformation.py) with additional geometry column 'geom' created in Python via [create_proj_tables.py](Pipeline/create_proj_tables.py).

|         Column         |         Data Type          |
|:----------------------:|:--------------------------:|
|        **geom**        | **geometry(Point, 26917)** |
|        objectid        |           bigint           |
|         tcs__          |      double precision      |
|          main          |            text            |
|     midblock_route     |            text            |
|      side_1_route      |            text            |
|      side_2_route      |            text            |
|    activation_date     |            date            |
|        latitude        |      double precision      |
|       longitude        |      double precision      |
|        latitude        |      double precision      |
|       longitude        |      double precision      |
|       count_date       |            date            |
|  f8hr_vehicle_volume   |      double precision      |
| f8hr_pedestrian_volume |      double precision      |
|     download_link      |            text            |
|      src_filename      |            text            |
|      last_updated      |         timestamp          |
|     last_inserted      |         timestamp          |

### §9. fact_hourly_avg
The table _fact_hourly_avg_ contains the calculated means for the hourly segments per station and converted POSTGIS table via [create_proj_tables.py](Pipeline/Fcreate_proj_tables.py).

|   Column    |         Data Type          |
|:-----------:|:--------------------------:|
|  **geom**   | **geometry(Point, 26917)** |
|  cgndb_id   |            text            |
|   geo_lat   |      double precision      |
|  geo_long   |      double precision      |
|  dawn_avg   |      double precision      |
| morning_avg |      double precision      |
|  noon_avg   |      double precision      |
| evening_avg |      double precision      |

### §10. fact_weekdays_avg
The table _fact_weekdays_avg_ contains the calculated means for the weekdays per station and converted POSTGIS table via [data_loader.py](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/Pipeline/data_loader.py) that executes the query [create_post_proj_tbl.sql](https://github.com/amr-y-shalaby/GGR473_Project/blob/main/SQL/create_postgis_proj_tbl.sql).

|    Column    |       Data Type        |
|:------------:|:----------------------:|
|     geom     | geometry(Point, 26917) |
|   cgndb_id   |          text          |
|   latitude   |     numeric(10, 2)     |
|  longitude   |     numeric(10, 2)     |
|  monday_avg  |     numeric(10, 2)     |
| tuesday_avg  |     numeric(10, 2)     |
| thursday_avg |     numeric(10, 2)     |
|  friday_avg  |     numeric(10, 2)     |
| saturday_avg |     numeric(10, 2)     |
|  sunday_avg  |     numeric(10, 2)     |

