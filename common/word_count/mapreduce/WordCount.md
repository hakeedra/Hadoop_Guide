# MapReduce WordCount



#### pom.xml

```xml
<repositories>
    <repository>
        <id>cloudera</id>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>${hadoop.version}-mr1-${cdh.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>${hadoop.version}-${cdh.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>${hadoop.version}-${cdh.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>${hadoop.version}-${cdh.version}</version>
    </dependency>
</dependencies>
```



#### code

```java
package com.wdlily.future;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

/**
 * WordCount by MapReduce
 *
 * @Author: hakeedra 19-3-7 上午11:07
 */
public class WordCount extends Configured implements Tool {

    static class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

        Text text = new Text();
        IntWritable intWritable = new IntWritable(1);

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] split = line.split(" ");
            for (String word : split) {
                text.set(word);
                context.write(text, intWritable);
            }
        }
    }

    static class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

        IntWritable intWritable = new IntWritable();

        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int num = 0;
            for (IntWritable value : values) {
                int i = value.get();
                num += i;
            }
            intWritable.set(num);
            context.write(key, intWritable);
        }
    }

    public int run(String[] args) throws Exception {

        Job job = Job.getInstance(super.getConf(), "WordCount");
        job.setJarByClass(WordCount.class);

        job.setInputFormatClass(TextInputFormat.class);
        //file path
        TextInputFormat.addInputPath(job, new Path("file:///data/input/words.txt"));

        job.setMapperClass(WordCountMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setReducerClass(WordCountReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        job.setOutputFormatClass(TextOutputFormat.class);
        //dir path
        TextOutputFormat.setOutputPath(job, new Path("file:///data/output"));

        boolean b = job.waitForCompletion(true);
        return b ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        int run = ToolRunner.run(new Configuration(), new WordCount(), args);
        System.exit(run);
    }

}

```





















