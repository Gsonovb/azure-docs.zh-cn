<!--
Used in:
sql-database-elastic-pool.md   
sql-database-resource-limits.md
sql-database-service-tiers.md  
-->
 
### <a name="basic-elastic-pool-limits"></a>基本弹性池限制

| 池大小 (eDTU)  | **50** | **100** | **200** | **300** | **400** | **800** | **1200** | **1600** |
|:---|---:|---:|---:| ---: | ---: | ---: | ---: | ---: |
| 每个池的最大数据存储空间* | 5 GB | 10 GB | 20 GB | 29 GB | 39 GB | 78 GB | 117 GB | 156 GB |
| 每个池的最大内存中 OLTP 存储* | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 |
| 每个池的最大数据库数 | 100 | 200 | 500 | 500 | 500 | 500 | 500 | 500 |
| 每个池的最大并发工作线程数 | 100 | 200 | 400 | 600 | 800 | 1600 | 2400 | 3200 |
| 每个池的最大并发登录数 | 100 | 200 | 400 | 600 | 800 | 1600 | 2400 | 3200 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 |30000 | 30000 | 30000 | 30000 |
| 每个数据库的最小 eDTU 数 | {0, 5} | {0, 5} | {0, 5} | {0, 5} | {0, 5} | {0, 5} | {0, 5} | {0, 5} | {0, 5} |
| 每个数据库的最大 eDTU 数 | {5} | {5} | {5} | {5} | {5} | {5} | {5} | {5} | {5} |
||||||||

### <a name="standard-elastic-pool-limits"></a>标准弹性池限制

| 池大小 (eDTU)  | **50** | **100** | **200** | **300** | **400** | **800** | 
|:---|---:|---:|---:| ---: | ---: | ---: | 
| 每个池的最大数据存储空间* | 50 GB| 100 GB| 200 GB | 300 GB| 400 GB | 800 GB | 
| 每个池的最大内存中 OLTP 存储* | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 
| 每个池的最大数据库数 | 100 | 200 | 500 | 500 | 500 | 500 | 
| 每个池的最大并发工作线程数 | 100 | 200 | 400 | 600 |  800 | 1600 |
| 每个池的最大并发登录数 | 100 | 200 | 400 | 600 |  800 | 1600 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 | 30000 | 30000 |
| 每个数据库的最小 eDTU 数 | {0,10,20,<br>50} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} |
| 每个数据库的最大 eDTU 数 | {10,20,<br>50} | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | 
||||||||

### <a name="standard-elastic-pool-limits-continued"></a>标准弹性池限制（续） 

| 池大小 (eDTU)  |  **1200** | **1600** | **2000** | **2500** | **3000** |
|:---|---:|---:|---:| ---: | ---: |
| 每个池的最大数据存储空间* | 1.2 TB | 1.6 TB | 2 TB | 2.4 TB | 2.9 TB | 
| 每个池的最大内存中 OLTP 存储* | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 
| 每个池的最大数据库数 | 500 | 500 | 500 | 500 | 500 | 500 |
| 每个池的最大并发工作线程数 |  2400 | 3200 | 4000 | 5000 | 6000 |
| 每个池的最大并发登录数 |  2400 | 3200 | 4000 | 5000 | 6000 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 | 30000 | 
| 每个数据库的最小 eDTU 数 | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} | {0,10,20,<br>50,100} |
| 每个数据库的最大 eDTU 数 | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | {10,20,<br>50,100} | 
||||||||

### <a name="premium-elastic-pool-limits"></a>高级弹性池限制

| 池大小 (eDTU)  | **125** | **250** | **500** | **1000** | **1500** | 
|:---|---:|---:|---:| ---: | ---: | 
| 每个池的最大数据存储空间* | 250 GB| 500 GB| 750 GB| 750 GB| 750 GB| 
| 每个池的最大内存中 OLTP 存储* | 1 GB| 2 GB| 4 GB| 10 GB| 12 GB| 
| 每个池的最大数据库数 | 50 | 100 | 100 | 100 | 100 |  
| 每个池的最大并发工作线程数 | 200 | 400 | 800 | 1600 |  2400 | 
| 每个池的最大并发登录数 | 200 | 400 | 800 | 1600 |  2400 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 | 30000 | 
| 每个数据库的最小 eDTU 数 | {0,25,50,75,<br>125} | {0,25,50,75,<br>125,250} | {0,25,50,75,<br>125,250,500} | {0,25,50,75,<br>125,250,500,<br>1000} | {0,25,50,75,<br>125,250,500,<br>1000,1500} | 
| 每个数据库的最大 eDTU 数 | {25,50,75,<br>125} | {25,50,75,<br>125,250} | {25,50,75,<br>125,250,500} | {25,50,75,<br>125,250,500,<br>1000} | {25,50,75,<br>125,250,500,<br>1000,1500} |  
||||||||

### <a name="premium-elastic-pool-limits-continued"></a>高级弹性池限制（续） 

| 池大小 (eDTU)  |  **2000** | **2500** | **3000** | **3500** | **4000** |
|:---|---:|---:|---:| ---: | ---: | 
| 每个池的最大数据存储空间* | 750 GB | 750 GB | 750 GB | 750 GB | 750 GB |
| 每个池的最大内存中 OLTP 存储* | 16 GB | 20 GB | 24 GB | 28 GB | 32 GB |
| 每个池的最大数据库数 | 100 | 100 | 100 | 100 | 100 | 
| 每个池的最大并发工作线程数 |  3200 | 4000 | 4800 | 5600 | 6400 |
| 每个池的最大并发登录数 |  3200 | 4000 | 4800 | 5600 | 6400 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 | 30000 | 
| 每个数据库的最小 eDTU 数 | {0,25,50,75,<br>125,250,500,<br>1000,1750} | {0,25,50,75,<br>125,250,500,<br>1000,1750} | {0,25,50,75,<br>125,250,500,<br>1000,1750} | {0,25,50,75,<br>125,250,500,<br>1000,1750} |  {0,25,50,75,<br>125,250,500,<br>1000,1750,4000} | 
| 每个数据库的最大 eDTU 数 | {25,50,75,<br>125,250,500,<br>1000,1750} | {25,50,75,<br>125,250,500,<br>1000,1750} | {25,50,75,<br>125,250,500,<br>1000,1750} | {25,50,75,<br>125,250,500,<br>1000,1750} | {25,50,75,<br>125,250,500,<br>1000,1750,4000} | 
||||||||

### <a name="premium-rs-elastic-pool-limits"></a>高级 RS 弹性池限制

| 池大小 (eDTU)  | **125** | **250** | **500** | **1000** |
|:---|---:|---:|---:| ---: | ---: | 
| 每个池的最大数据存储空间* | 250 GB| 500 GB | 750 GB | 750 GB |
| 每个池的最大内存中 OLTP 存储* | 1 GB | 2 GB | 4 GB | 10 GB |
| 每个池的最大数据库数 | 50 | 100 | 100 | 100 |
| 每个池的最大并发工作线程数 | 200 | 400 | 800 | 1600 |
| 每个池的最大并发登录数 | 200 | 400 | 800 | 1600 |
| 每个池的最大并发会话数 | 30000 | 30000 | 30000 | 30000 |
| 每个数据库的最小 eDTU 数 | {0,25,50,75,<br>125} | {0,25,50,75,<br>125,250} | {0,25,50,75,<br>125,250,500} | {0,25,50,75,<br>125,250,500,<br>1000} |
| 每个数据库的最大 eDTU 数 | {25,50,75,<br>125} | {25,50,75,<br>125,250} | {25,50,75,<br>125,250,500} | {25,50,75,<br>125,250,500,<br>1000} | 
||||||||

> [!IMPORTANT]
>\*入池数据库共享池存储空间，因此弹性池中的数据存储空间被限制为池的剩余存储空间或每个数据库的最大存储空间（取二者之中的较小者）。
>
