## MapReduce finetuning

### Scenario 1 – Job with 100 mappers and 1 reducer takes a long time for the reducer to start after all the mappers are complete. One of the reasons could be that reduce is spending a lot of time copying the map outputs. So in this case we can try couple of things.

If possible add a combiner to reduce the amount of output from the mapper to be sent to the reducer
Enable map output compression – this will further reduce the size of the outputs to be transferred to the reducer. 

### Scenario 2 – A particular task is using a lot of memory which is causing the slowness or failure, I will look for ways to reduce the memory usage.

Make sure the joins are made in an optimal way with memory usage in mind. For e.g. in Pig joins, the LEFT hand side tables are sent to the reducer first and held in memory and the RIGHT most table in streamed to the reducer. So make sure the RIGHT most table is largest of the datasets in the join.
We can also increase the memory requirements needed by the map and reduce tasks by setting – mapreduce.map.memory.mb and mapreduce.reduce.memory.mb 

### Scenario 3 – Understanding the data helps a lot in optimizing the way we use the datasets in PIG and HIVE scripts.

If you have smaller tables in join, they can be sent to distributed cache and loaded in memory on the Map side and the entire join can be done on the Map side thereby avoiding the shuffle and reduce phase altogether. This will tremendously improve performance. Look up USING REPLICATED in Pig and MAPJOIN or hive.auto.convert.join in Hive
If the data is already sorted you can use USING MERGE which will do a Map Only join
If the data is bucketted in hive, you may use hive.optimize.bucketmapjoin or hive.optimize.bucketmapjoin.sortedmerge depending on the characteristics of the data 

### Scenario 4 – The Shuffle process is the heart of a MapReduce program and it can be tweaked for performance improvement.

If you see lots of records are being spilled to the disk (check for Spilled Records in the counters in your MapReduce output) you can increase the memory available for Map to perform the Shuffle by increasing the value in io.sort.mb. This will reduce the amount of Map Outputs written to the disk so the sorting of the keys can be performed in memory.
On the reduce side the merge operation (merging the output from several mappers) can be done in disk by setting the mapred.inmem.merge.threshold to 0
