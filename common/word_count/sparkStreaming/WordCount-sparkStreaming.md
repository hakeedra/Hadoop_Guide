# SparkStreaming WordCount

word count for socket source

```shell
nc -l 12580
```



#### pom.xml

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>${spark.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.11</artifactId>
    <version>${spark.version}</version>
</dependency>
```



#### code

```scala
package com.wdlily.spark.streaming

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.dstream.ReceiverInputDStream

/**
  * WordCount for socket
  *
  * @Author: hakeedra 19-3-7 下午12:13
  */
object WordCountForSocket {
    
    def main(args: Array[String]): Unit = {
        
        val conf = new SparkConf().setAppName("WordCountForSocket").setMaster("local[2]")
        val ssc: StreamingContext = new StreamingContext(conf, Seconds(3))
        ssc.sparkContext.setLogLevel("WARN")
        
        val socketValue: ReceiverInputDStream[String] = ssc.socketTextStream("future", 12580)
        socketValue.flatMap(_.split(" ")).map(x => (x, 1)).reduceByKey(_ + _).print()
        
        ssc.start()
        ssc.awaitTermination()
    }
}

```





