# Experiment 1: New York Taxi Data Analytics
# 
## 1. Prepare test data

This experiment uses the 500, 000 open source New York taxi data. The data can be downloaded from [https://pan.baidu.com/s/1Njtr1316E25vuLzkB4k6RA](https://pan.baidu.com/s/1Njtr1316E25vuLzkB4k6RA). It is recommended that you download the data to `/tmp`.

The following configuration has been tested:

| Component  | Minimum Configuration                  |
| ---------- | ------------------------------- |
| OS         | Ubuntu LTS 18.04                |
| CPU        | Intel Core i5-8250U             |
| GPU        | NVIDIA GeForce MX150, 2GB GDDR5 |
| GPU driver | Driver 430                      |
| Memory     | 8 GB DDR4                       |
| Hard disk    | NVMe SSD 256 GB                 |



## 2. Set MegaWise Parameters

Make sure that you have installed [MegaWise](https://www.zilliz.com/cn/docs/v0.4.0/install_megawise). Navigate to the installed folder of MegaWise. The following steps use `/home/$USER/megawise` as an example. Navigate to this folder and set MegaWise parameters.

1. Open `chewie_main.yaml` in the `conf` folder and make the following changes:

    ```yaml
    cache:  # size in GB
    cpu:
        physical_memory: 16
        partition_memory: 16

    gpu:
        gpu_num: 1
        physical_memory: 2
        partition_memory: 2 
    ```

> Note: In the `cpu` part, `physical_memory` and `partition_memory` must be less than the physical memory;
>
> `gpu_num` must be less than the number of GPUs. `physical_memory` must be less than the global memory of the GPU with the smallest GPU memory. 

2. Open `megawise_config.yaml` in the `conf` folder and make the following changes:

    ```yaml
    worker_num : 1
    gpu:
        physical_memory: 2    # unit: GB
        partition_memory: 2   # unit: GB
    cuda_profile_query_cnt: -1 #-1 means don't profile, positive integer means the number of queries to profile, other value invalid
    ```

> Note: Uodate `worker_num`, `gpu physical_memory`, and `partition_memory`to make sure that they are consistent with settings in `chewie_main.yaml`. `worker_num` corresponds to `gpu_num`.


## 3. Run MegaWise and import test data

1. Check the MegaWise Docker container and get the container ID.

   ```bash
   $ docker ps 
   CONTAINER ID	IMAGE	COMMAND		CREATED		STATUS		PORTS	 NAMES
   4aed62f7f5f1  a1e52c9d94a3  "/docker-entrypoint.…"   15 hours ago   Up 15 hours 0.0.0.0:5433->5432/tcp   fervent_perlman
   ```

   > If you cannot get container ID with `docker ps`, please use `docker ps -a` instead.

2. Run MegaWise Docker.

   ```bash
   $ docker restart <CONTAINER ID>
   ```

3. Enter the bash command of MegaWise Docker.

   ```bash
   $ docker exec -u megawise -it <CONTAINER ID> bash
   megawise@4aed62f7f5f1:/megawise$
   ```

4. Connect to MegaWise.

   ```bash
   megawise@4aed62f7f5f1:/megawise$ cd script
   megawise@4aed62f7f5f1:/megawise/script$ ./connect.sh 
   psql (11.1)
   Type "help" for help.
   postgres=# 
   ```

5. Switch user and import test data.

   ```sql
   postgres=# \c - zilliz
   You are now connected to database "postgres" as user "zilliz".
   postgres=> DROP TABLE IF EXISTS nyc_taxi_50w;
   DROP TABLE
   postgres=> CREATE TABLE nyc_taxi_50w(
       vendor_id               TEXT NOT NULL,
       tpep_pickup_datetime    TIMESTAMP NOT NULL,
       tpep_dropoff_datetime   TIMESTAMP NOT NULL,
       passenger_count         INT4 NOT NULL,
       trip_distance           FLOAT8 NOT NULL,
       old_pickup_longitude    FLOAT8 NOT NULL,
       old_pickup_latitude     FLOAT8 NOT NULL,
       dropoff_longitute       FLOAT8 NOT NULL,
       dropoff_latitute        FLOAT8 NOT NULL,
       fare_amount             FLOAT8 NOT NULL,
       tip_amount              FLOAT8 NOT NULL,
       total_amount            FLOAT8 NOT NULL,
       pickup_longitude        FLOAT8 NOT NULL,
       pickup_latitude         FLOAT8 NOT NULL
   );
   
   postgres=> copy nyc_taxi_50w from '/tmp/nyc_taxi_50w.csv' with csv header delimiter ',';
   COPY 500000
   ```

## 4. Use Infini interactive interface and create dashboards

Mke sure you have installed and run [Infini Interactive Interface](https://www.zilliz.com/cn/docs/v0.4.0/install_infini). Chrome and Firefox are recommended.

```shell
# 192.168.1.60 is the IP address of the server that runs the Infini Docker
http://192.168.1.60
```

1. Enter the login page.


   **Enter username and password to login**

   - Username zilliz
   - Password zilliz

2. Enter information related to MegaWise.

   After login, enter information related to MegaWise and click save to direct to the dashboard.

   > Use the IP address of the server that runs MegaWise Docker.


3. Create analytics interface for New York taxi data.

    1. Click `Add Dashboard` and change the name of the dashboard to `New York Taxi Demo`:


    2. Click `+ ADD Chart`.


4. Create a point map.

    1. Select point map. Specify `nyc_taxi_50w` for `sources`,  add `dropoff_longitude` for `LOT` and add `dropoff_latitude` for `LAT`. Change the name of the point map to `New York Taxi`.

    2. Apply the changes.


   

5. Create a line chart.

    1. Click `+ ADD Chart` and select line chart. In `X AXIS`, add `tpep_dropoff_datetime` grouped by `day`. In `Y AXIS`, add `total_amount`. Change the chart name to `New York Taxi Summary ($ USD)`.


    2. Apply the changes.

   
6. Create a number chart to sum up taxi fee.

   1. Click `+ ADD Chart` and select number chart. In `MEASURE`, add `total_amount` and select `SUM`. Change the `MEASURE` data format to `Sl 1.2k`. Change the chart name to `New York Taxi fare amount($ USD)`.


   2. Apply the changes.

   
7. Create a number chart to sum up taxi journey.

   1. Click `+ ADD Chart` and select number chart. In `MEASURE`, add `trip_distance`and select `SUM`. Change the `MEASURE` data format to `Sl 1.2k`. Change the chart name to `New York taxi mileage (km)`.

	2. Apply the changes.

   

8. Create a pie chart to show taxi market share.

   1. Click `+ ADD Chart` and select pie chart. In `DIMENSION`, add `vendor_id`. In `MEASURE`, select `passenger_count` and select `SUM`. Change the chart name to `New York taxi market share`:

2. Apply the changes.
   
Congratulations. You have completed New York taxi data analytics!