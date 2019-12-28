---
layout: post
title: MR task read/write parquet file
description: "Describe how to read/write parquet file"
modified: 2016-06-09
tags: [mr, parquet]
image:
  feature: abstract-5.jpg
---

# Parquet
Parquet project is published by **Twitter** and **Cloudera**, which is an open-source cloumnar storage format library for Apache Hadoop. Column storage have many priorities, for example, useless information can be throwed away and compression algo can be used for storage since  uniform structure. You can get more information in [Parquet Website][1] and [Cloudera blog][2] . Here, no more desciption about what is Parquet, or how effective to store data.


> Currently, parquet provides a state of the art columnar storage layer that can be taken advantage of by existing Hadoop frameworks, and can enable a new generation of Hadoop data processing architectures such as Impala, Drill, and parts of the Hive 'Stinger' initiative.

## MR task read/write parquet file

### write parquet file
 

We suppose to use the simplest mr task with no reducer.


1.set input path and input format

```Java
TextInputFormat.addInputPath(writeToParquetJob, inputPath);
writeToParquetJob.setInputFormatClass(TextInputFormat.class);
writeToParquetJob.setMapperClass(ParquetMapper.class);
```
 
 
2.define ParquetMapper

```Java
public static class ParquetMapper extends Mapper<LongWritable, Text, Void, Person> {
	public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
    	String[] array = value.toString().split("\t");
    	if (array.length != 2) return;
     	Person person = new Person();
        person.setId(Long.parseLong(array[0]));
        person.setName(array[1]);
        context.write(null, person);
    }
}
```


3.set output format with thrift clas and output path
 
```Java
writeToParquetJob.setNumReduceTasks(0);
writeToParquetJob.setOutputFormatClass(ParquetThriftOutputFormat.class);
ParquetThriftOutputFormat.setCompression(writeToParquetJob, CompressionCodecName.GZIP);
ParquetThriftOutputFormat.setOutputPath(writeToParquetJob, parquetPath);
ParquetThriftOutputFormat.setThriftClass(writeToParquetJob, Person.class);
```

4.run mapreduce task

```Java
boolean ret = writeToParquetJob.waitForCompletion(true);
```


### **read parquet file**

1.set input path and input format

```Java
MultipleInputs.addInputPath(readParquetJob, parquetPath, ParquetThriftInputFormat.class, ParquetReader.class);
```
 
2.define ParquetReader

```Java
public static class ParquetReader extends Mapper<Void, Person, LongWritable, Text> {
	public void map(Void key, Person value, Context context) throws IOException, InterruptedException {
    	context.write(new LongWritable(value.getId()), new Text(value.getName()));
    }
}
```

3.set output format and output path
 
```Java
readParquetJob.setNumReduceTasks(0);
readParquetJob.setOutputFormatClass(TextOutputFormat.class);
TextOutputFormat.setOutputPath(readParquetJob, outputPath);
```

4.run mapreduce task

```Java
boolean ret = readParquetJob.waitForCompletion(true);
```

In addition, thrift class Person is defined as follows:

```Java
struct Person {
    1:optional i64 id;
    2:optional string name;
}
```




  [1]: https://parquet.apache.org
  [2]: https://blog.cloudera.com/blog/2013/07/announcing-parquet-1-0-columnar-storage-for-hadoop/
  [3]: https://blog.cloudera.com/blog/2013/07/announcing-parquet-1-0-columnar-storage-for-hadoop/