# Ray XGBoost

In my earlier article, I discussed how the combination of [Dask and XGBoost](https://github.com/dennislee22/dask-xgboost) can effectively overcome limited RAM when training models on massive datasets. Building on the same use case—training a telco fraud detection model, I now explore an alternative distributed framework: [Ray](https://docs.ray.io/), used in conjunction with XGBoost.

## What is Ray and its XGBoost Integration?
Before diving into the results, it's important to understand what Ray is. Unlike Dask, which primarily extends the familiar Pandas and Scikit-Learn APIs for parallel execution, Ray is a more comprehensive framework designed to build and scale ML and Python applications in a distributed fashion.At its core, Ray uses a shared-memory object store that allows different processes and nodes in a cluster to access data with incredible efficiency. For AI/ML, Ray is leveraged by a suite of key libraries:

   - Ray Data: Distributed data processing.
   - Ray Train: Distributed model training that integrates with frameworks like PyTorch, TensorFlow, and, of course, XGBoost.
   - Ray Tune: Distributed hyperparameter tuning.

This article describes the steps to use Ray with XGBoost (on a Kubernetes cluster) to build a model for helping telco to check if a particular MSISDN/user is fradulent based on the captured CDR. The steps to achieve this use case include:

1. Create synthetic dataset (in batch to prevent running into OOM problem).
2. Use dataframe to create feature engineering of the dataset (dask-train-xgboost.ipynb).
3. Train and test the model using XGBoost (dask-train-xgboost.ipynb).

## Prerequisites
1. Configure sufficient shared memory size (/dev/shm). 

<img width="700" height="637" alt="image" src="https://github.com/user-attachments/assets/5dd49097-dcfc-427c-ba68-6e9bf9c0a585" />

## Step 1: Create a Ray Cluster

1. Run first cell in to start a Ray cluster with its dashboard. The 'head' of the Ray cluster can be configured not to take part in the actual task/actor computation but merely managing the Ray cluster.

## Step 2: Train the Model with CSV Dataset
1. Run `ray.data` to ReadCSV the dataset input and split the dataset into training, validation and test portions.
2. Use `pandas` to create feature engineering dataFrame based on the csv dataset.
3. Run `ray.data` to perform feature engineering.
4. Train the model using `ray.train.xgboost.XGBoostTrainer`

## The Result
The outcome describes three crucial trade-offs: speed vs. complexity, generalization vs. thoroughness, and performance vs. resource cost.

1. `ray.train.xgboost.XGBoostTrainer` is more than just a parallel .fit() command. It’s an "opinionated" tool, designed with production best practices—like fault tolerance and automated early stopping—built right in.

The Head-to-Head Experiment
I ran the telco fraud model training on both Dask and Ray clusters. The results were startlingly different across every metric.

Metric

Dask + XGBoost

Ray + XGBoost

Processing Time

346.85 seconds

237.07 seconds

Test Accuracy

99.99% (3 errors)

100% (0 errors)

Memory Observation

Low I/O overhead

High I/O overhead (OOM on <20GB workers)

Top Feature

mobility (Score: 178)

total_calls (Score: 20)




1. The Hidden Cost: Memory Overhead

My first attempt to run the Ray job failed. Workers with less than 20GB of RAM were consistently crashing with Out-of-Memory (OOM) errors during the initial read_csv phase. Dask, by contrast, handled this with much less memory per worker.

This happens because Ray's powerful shared-memory object store, which enables its high-speed performance later in the pipeline, requires a significant memory "buy-in" upfront. It manages data as shared objects for the entire cluster, which has a higher initial resource cost than Dask's more direct approach of having each worker manage its own chunk of a DataFrame.

2. The Training Philosophy: Fixed vs. Flexible

The most telling difference is in the feature importances. The Dask model is complex, using all available features with high scores. The Ray model is dramatically simpler, with much lower scores and completely ignoring the mobility feature, which Dask considered paramount.

This is a direct result of early stopping being a default practice in Ray Train.

Dask built the exact model I asked for: a forest with a fixed number of trees. It was thorough and used every piece of information it could.

Ray had a different goal: build the best possible model without overfitting. It used a validation set to monitor performance, and as soon as the model stopped improving, it stopped training. This resulted in a leaner, more generalized model that was faster to train and, in this case, more accurate on the test set.


```
NAME               READY   STATUS    RESTARTS   AGE    IP            NODE                          NOMINATED NODE   READINESS GATES
8ztd8uxig3jibk0g   5/5     Running   0          5m9s   10.42.2.217   ecs-w-02.dlee5.cldr.example   <none>           <none>
i2pxb2t4gmy8ruxe   5/5     Running   0          5m9s   10.42.3.24    ecs-w-01.dlee5.cldr.example   <none>           <none>
i7xow2ebdqly7870   5/5     Running   0          8m     10.42.1.212   ecs-w-03.dlee5.cldr.example   <none>           <none>
kr3t20zuhy4p8gg1   5/5     Running   0          5m9s   10.42.3.25    ecs-w-01.dlee5.cldr.example   <none>           <none>
mpwk3j51zc30vv5w   5/5     Running   0          5m9s   10.42.2.218   ecs-w-02.dlee5.cldr.example   <none>           <none>
rzsr3vyehkxe53nu   5/5     Running   0          5m9s   10.42.1.214   ecs-w-03.dlee5.cldr.example   <none>           <none>
```

<img width="700" height="725" alt="image" src="https://github.com/user-attachments/assets/9468e715-8a7c-46be-a892-07eae5f2be9f" />


# Tips
1. Ray Client has architectural limitations and may not work as expected when using Ray for ML workloads (like Ray Tune or Ray Train).
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html

2. In Ray, shared memory is primarily used by the Ray object store to efficiently manage and transfer large objects between different actors and tasks. Using shared memory for large objects significantly improves performance by reducing the time required to transfer data between tasks and actors. This is particularly important when working with machine learning models or large datasets that don't fit into local memory. If the /dev/shm partition is too small or unavailable, Ray automatically falls back to using the /tmp directory, which is less performant.
   
2. `ray.data.read_csv` does support splitting a file into multiple blocks automatically
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html
  
3. This often happens when running Ray in a container or on a system with limited shared memory. Ray uses this shared memory (often /dev/shm) to store large objects, and when it's too small, Ray will spill objects to /tmp, which can significantly impact performance.

4. 2025-07-15 06:45:46,383	WARNING interfaces.py:140 -- More than 3GB of driver memory used to store Ray Data block data and metadata. This job may exit if driver memory is insufficient.

<img width="700" height="725" alt="image" src="https://github.com/user-attachments/assets/38620427-1f7c-45e3-8f87-015371d74f6d" />


This can happen when many tiny blocks are created. Check the block size using Dataset.stats() and see https://docs.ray.io/en/latest/data/performance-tips.html for mitigation.

https://docs.ray.io/en/latest/ray-core/scheduling/memory-management.html


