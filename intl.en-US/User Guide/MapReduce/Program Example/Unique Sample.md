# Unique Sample {#concept_mzd_52l_vdb .concept}

## Preparation {#section_e3n_syg_vdb .section}

1.  Prepare the JAR package of the test program. Assume the package is named mapreduce-examples.jar, The local storage path is data\\resources.
2.  Prepare tables and resources for testing the Unique operation.
    -   Create tables.

        ```
        create table ss_in(key bigint, value bigint);
        create table ss_out(key bigint, value bigint);
        ```

    -   Add resources.

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  Use the tunnel command to import the data.

    ```
    tunnel upload data ss_in;
    ```

    The contents of data file imported into the table ss\_in.

    ```
     1,1
     1,1
     2,2
     2,2
    ```


## Procedure {#section_rlv_bzg_vdb .section}

Run Unique on the odpscmd, as follows:

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.Unique ss_in ss_out key;
```

## Result {#section_hzz_dzg_vdb .section}

The content of output table ss\_out is as follows:

```

| key | value |

| 1 | 1 |
| 2 | 2 |

```

## Sample code {#section_jgb_gzg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    Import java. util. iterator;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.data.TableInfo;
    import com.aliyun.odps.mapred.JobClient;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.ReducerBase;
    import com.aliyun.odps.mapred.TaskContext;
    import com.aliyun.odps.mapred.conf.JobConf;
    import com.aliyun.odps.mapred.utils.InputUtils;
    import com.aliyun.odps.mapred.utils.OutputUtils;
    import com.aliyun.odps.mapred.utils.SchemaUtils;
    
     * Unique Remove duplicate words
     
     
    public class Unique {
      public static class OutputSchemaMapper extends MapperBase {
        private Record key;
        private Record value;
        @Override
        public void setup(TaskContext context) throws IOException {
          key = context.createMapOutputKeyRecord();
          value = context.createMapOutputValueRecord();
        
        @ Override
        public void map(long recordNum, Record record, TaskContext context)
            throws IOException {
          long left = 0;
          long right = 0;
          if (record.getColumnCount() > 0) {
            left = (Long) record.get(0);
            if (record.getColumnCount() > 1) {
              right = (Long) record.get(1);
            
            key.set(new Object[] { (Long) left, (Long) right });
            value.set(new Object[] { (Long) left, (Long) right });
            context.write(key, value);
          
        
      
      public static class OutputSchemaReducer extends ReducerBase {
        private Record result = null;
        @Override
        public void setup(TaskContext context) throws IOException {
          result = context.createOutputRecord();
        
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          result.set(0, key.get(0));
          while (values.hasNext()) {
            Record value = values.next();
            result.set(1, value.get(1));
          
          context.write(result);
        
      
      public static void main(String[] args) throws Exception {
        if (args.length > 3 || args.length < 2) {
          System.err.println("Usage: unique <in> <out> [key|value|all]");
          System.exit(2);
        
        String ops = "all";
        if (args.length == 3) {
          Ops = ARGs [2];
        
        // Key Unique
        if (ops.equals("key")) {
          JobConf job = new JobConf();
          job.setMapperClass(OutputSchemaMapper.class);
          job.setReducerClass(OutputSchemaReducer.class);
          job.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setMapOutputValueSchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setPartitionColumns(new String[] { "key" });
          job.setOutputKeySortColumns(new String[] { "key", "value" });
          job.setOutputGroupingColumns(new String[] { "key" });
          job.set("tablename2", args[1]);
          job.setNumReduceTasks(1);
          job.setInt("table.counter", 0);
          InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
          OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
          JobClient.runJob(job);
        
        // Key&Value Unique
        if (ops.equals("all")) {
          JobConf job = new JobConf();
          job.setMapperClass(OutputSchemaMapper.class);
          job.setReducerClass(OutputSchemaReducer.class);
          job.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setMapOutputValueSchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setPartitionColumns(new String[] { "key" });
          job.setOutputKeySortColumns(new String[] { "key", "value" });
          job.setOutputGroupingColumns(new String[] { "key", "value" });
          Job. Set ("tablename2", argS [1]);
          job.setNumReduceTasks(1);
          job.setInt("table.counter", 0);
          InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
          OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
          Jobclient. runjob (job );
        
        // Value Unique
        if (ops.equals("value")) {
          JobConf job = new JobConf();
          job.setMapperClass(OutputSchemaMapper.class);
          job.setReducerClass(OutputSchemaReducer.class);
          job.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setMapOutputValueSchema(SchemaUtils.fromString("key:bigint,value:bigint"));
          job.setPartitionColumns(new String[] { "value" });
          job.setOutputKeySortColumns(new String[] { "value" });
          job.setOutputGroupingColumns(new String[] { "value" });
          job.set("tablename2", args[1]);
          job.setNumReduceTasks(1);
          job.setInt("table.counter", 0);
          InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
          OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
          JobClient.runJob(job);
        
      
    

```

