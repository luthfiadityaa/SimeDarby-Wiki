# **Palletize LAYOUT**

![image.png](/.attachments/image-32e1e76d-8510-472e-9299-aaa8b33d063f.png)

**Processes:**

1. No Operations (Empty Pallet Supply)
2. Batch Start
3. Batch End
4. Update To Host

Those processes are related to DNStoragePlan status_flag.

STATUS_FLAG | Meaning | Explain
-------------------------------
0 | No Started | Plan just register and do not have any Pallet
1 | Working | When the 1st Pallet store in update to 1
2 | Waiting for last pallet | Batch end submitted (last_pallet_qty > 0) and waiting for ID25.force_completion flag
3 | Batch End | Batch end submitted (qty = 0)
4 | Completed | Updated to 4, will inserted do DNHostsend for reporting to host

