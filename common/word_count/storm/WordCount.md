# Storm WordCount



#### pom.xml

```xml
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>1.1.3</version>
    <!--<scope>provided</scope>-->
</dependency>
```



#### code

```java
package com.wdlily.storm.wordcount;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.generated.StormTopology;
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.HashMap;
import java.util.Map;
import java.util.Random;

/**
 * storm word count
 *
 * @Author: hakeedra 19-1-19 下午4:09
 */
public class WordCount {

    static class WordSourceSpout extends BaseRichSpout {

        private SpoutOutputCollector spoutOutputCollectorl;
        private Random random;
        private String[] arr = {"aa", "bb", "cc", "dd", "ee", "ff"};

        public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
            this.spoutOutputCollectorl = spoutOutputCollector;
            this.random = new Random();
        }

        public void nextTuple() {
            spoutOutputCollectorl.emit(new Values(arr[random.nextInt(arr.length)]));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
            outputFieldsDeclarer.declare(new Fields("word"));
        }
    }

    static class WordBolt extends BaseRichBolt {

        private Map<String, Integer> map;

        public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
            this.map = new HashMap<>();
        }

        public void execute(Tuple input) {

            String word = input.getStringByField("word");
            map.put(word, map.get(word) == null ? 1 : map.get(word) + 1);

            System.out.println("===========================");
            System.out.println(map);
            System.out.println();
        }

        public void declareOutputFields(OutputFieldsDeclarer declarer) {

        }
    }

    public static void main(String[] args) {

        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout("WordSourceSpout", new WordSourceSpout());
        builder.setBolt("WordBolt", new WordBolt()).globalGrouping("WordSourceSpout");
        StormTopology topology = builder.createTopology();

        Config config = new Config();

        LocalCluster cluster = new LocalCluster();
        cluster.submitTopology("WordCount", config, topology);
    }


}

```









