+++
title = "Mystery hunt 3"
date = 2019-11-26T19:44:40-08:00
weight = 4
pre = "<b>3. </b>"
+++

Your development team has reported that redis performance has degraded over time. They suspect memory fragmentation may be the root cause. They've asked for an easy way to get visibility into trend of memory fragmentation over a period of time.

#### Hint
Remember we created the ***redis-*** index? Examine the contents of this index and build a visualization in Kibana.

#### Mystery Solved
The ***redis-*** index contains all the key redis metrics  collected by flunetbit. The ```mem_fragmentation_ratio``` field gives the ratio of memory used as seen by the operating system to memory allocated by Redis.

Tracking fragmentation ratio is important for understanding your Redis instance’s performance. A fragmentation ratio greater than 1 indicates fragmentation is occurring. A ratio in excess of 1.5 indicates excessive fragmentation, with your Redis instance consuming 150% of the physical memory it requested. A fragmentation ratio below 1 tells you that Redis needs more memory than is available on your system, which leads to swapping. Swapping to disk will cause significant increases in latency (see used memory). Ideally, the operating system would allocate a contiguous segment in physical memory, with a fragmentation ratio equal to 1 or slightly greater.

**Watch this video to create memory fragmentation visualization**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-mem-fragmentation-video.mp4

**(Optional)**
We've built a custom dashboard to monitor redis metrics. Follow the steps below to import this dashboard and explore.

**Download and unzip dashboard and visualization**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-dashboard-visualization.zip

**Watch this video to import visualizations into Kibana**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-vis-import.mp4
