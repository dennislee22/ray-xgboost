# Ray XGBoost

<img width="700" height="637" alt="image" src="https://github.com/user-attachments/assets/5dd49097-dcfc-427c-ba68-6e9bf9c0a585" />

# Tips
1. Ray Client has architectural limitations and may not work as expected when using Ray for ML workloads (like Ray Tune or Ray Train).
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html

2. `ray.data.read_csv` does support splitting a file into multiple blocks automatically
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html
  
3. This often happens when running Ray in a container or on a system with limited shared memory. Ray uses this shared memory (often /dev/shm) to store large objects, and when it's too small, Ray will spill objects to /tmp, which can significantly impact performance.

4. 2025-07-15 06:45:46,383	WARNING interfaces.py:140 -- More than 3GB of driver memory used to store Ray Data block data and metadata. This job may exit if driver memory is insufficient.

This can happen when many tiny blocks are created. Check the block size using Dataset.stats() and see https://docs.ray.io/en/latest/data/performance-tips.html for mitigation.

https://docs.ray.io/en/latest/ray-core/scheduling/memory-management.html

3
