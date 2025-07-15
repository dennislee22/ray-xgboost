# Ray XGBoost


# Tips
1. Ray Client has architectural limitations and may not work as expected when using Ray for ML workloads (like Ray Tune or Ray Train).
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html

2. `ray.data.read_csv` does support splitting a file into multiple blocks automatically
   - Reference: https://docs.ray.io/en/latest/cluster/running-applications/job-submission/ray-client.html
  
3. 2025-07-15 06:45:46,383	WARNING interfaces.py:140 -- More than 3GB of driver memory used to store Ray Data block data and metadata. This job may exit if driver memory is insufficient.

This can happen when many tiny blocks are created. Check the block size using Dataset.stats() and see https://docs.ray.io/en/latest/data/performance-tips.html for mitigation.
