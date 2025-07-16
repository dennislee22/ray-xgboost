# Ray XGBoost

<img width="700" height="637" alt="image" src="https://github.com/user-attachments/assets/5dd49097-dcfc-427c-ba68-6e9bf9c0a585" />

```
NAME               READY   STATUS    RESTARTS   AGE     IP            NODE                          NOMINATED NODE   READINESS GATES
3ewqy1h4spd1sm32   5/5     Running   0          7m21s   10.42.1.163   ecs-w-03.dlee5.cldr.example   <none>           <none>
3x76skywp7cyyzq2   5/5     Running   0          7m20s   10.42.1.164   ecs-w-03.dlee5.cldr.example   <none>           <none>
66nql0aeni2khhav   5/5     Running   0          10m     10.42.3.216   ecs-w-01.dlee5.cldr.example   <none>           <none>
lf5tq4jzw6pf87dx   5/5     Running   0          7m20s   10.42.3.221   ecs-w-01.dlee5.cldr.example   <none>           <none>
mh1bq56vrgv86q2e   5/5     Running   0          7m21s   10.42.2.186   ecs-w-02.dlee5.cldr.example   <none>           <none>
pkye044m88sjcnjs   5/5     Running   0          7m21s   10.42.3.218   ecs-w-01.dlee5.cldr.example   <none>           <none>
```

<img width="800" height="722" alt="image" src="https://github.com/user-attachments/assets/c7babc62-0e93-46e0-b7c4-f0a3d9f69642" />

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


