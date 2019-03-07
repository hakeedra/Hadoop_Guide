# Flink WordCount



#### pom.xml

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-clients_2.11</artifactId>
    <version>1.7.2</version>
</dependency>

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-scala_2.11</artifactId>
    <version>1.7.2</version>
</dependency>
```



#### code

```scala
package com.wdlily.flink

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, _}

/**
  * flink word count
  *
  * @Author: hakeedra 19-3-7 下午12:20
  */
object WordCount {
    
    def main(args: Array[String]): Unit = {
        
        val env = ExecutionEnvironment.getExecutionEnvironment
        val text: DataSet[String] = env.readTextFile("/data/input/words")

        val counts: AggregateDataSet[(String, Int)] = text.flatMap {_.toLowerCase.split(" ") filter {_.nonEmpty}}
          .map {(_, 1)}
          .groupBy(0)
          .sum(1)

        counts.writeAsCsv("/data/ouput/flink", "\n", " ")
        env.execute("Scala WordCount")
    
    }
}

```





