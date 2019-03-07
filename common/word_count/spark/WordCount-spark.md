# Spark WordCount



#### pom.xml

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>${spark.version}</version>
</dependency>
```



#### code

```scala
package com.wdlily.spark.wordcount

import org.apache.spark.{SparkConf, SparkContext}

/**
  * Spark WordCount
  *
  * @Author: hakeedra 19-3-7 上午11:51
  */
class WordCount {
    def main(args: Array[String]): Unit = {
        
        val conf = new SparkConf().setMaster("local[2]").setAppName("word_count")
        val sc = new SparkContext(conf)
        sc.setLogLevel("warn")
        
        sc.textFile("/data/input/words")
          .flatMap(_.split(" "))
          .map((_, 1))
          .reduceByKey(_ + _)
          .foreach(println)
        
        sc.stop()
    }
}
```





