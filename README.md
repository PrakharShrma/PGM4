Perfect ✅ — here’s the **ready-to-use `README.md` file content** for your **Program 4 (MovieTagsJoin)**.
It preserves **exact indentation**, **spacing**, and **formatting** — ready to drop into your Cloudera or project folder.

---

````markdown
# 🧩 Program 4 – Movie Tags Join (MapReduce)

## ✅ Objective
Develop a MapReduce program to find the **tags associated with each movie** by analyzing MovieLens data.

---

## 🧠 Concept
This program performs a **reduce-side join** between two datasets:

- `movies.csv`  → contains `movieId,title,genres`  
- `tags.csv`    → contains `userId,movieId,tag,timestamp`  

Both files share the common key **movieId**, which is used for joining.

---

## 💻 Java Program – `MovieTagsJoin.java`

```java
import java.io.IOException;
import java.util.*;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.*;
import org.apache.hadoop.mapreduce.lib.input.*;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MovieTagsJoin {

    // ---------- MAPPER for movies.csv ----------
    public static class MovieMapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {

            String line = value.toString();

            // Skip header line
            if (key.get() == 0 && line.contains("movieId")) return;

            String[] fields = line.split(",", 3);  // movieId,title,genres
            if (fields.length >= 2) {
                String movieId = fields[0].trim();
                String title = fields[1].trim();
                context.write(new Text(movieId), new Text("MOVIE::" + title));
            }
        }
    }

    // ---------- MAPPER for tags.csv ----------
    public static class TagMapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {

            String line = value.toString();

            // Skip header line
            if (key.get() == 0 && line.contains("userId")) return;

            String[] fields = line.split(",", 4);  // userId,movieId,tag,timestamp
            if (fields.length >= 3) {
                String movieId = fields[1].trim();
                String tag = fields[2].trim();
                context.write(new Text(movieId), new Text("TAG::" + tag));
            }
        }
    }

    // ---------- REDUCER ----------
    public static class JoinReducer extends Reducer<Text, Text, Text, Text> {
        @Override
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {

            String movieTitle = null;
            List<String> tags = new ArrayList<>();

            for (Text val : values) {
                String value = val.toString();

                if (value.startsWith("MOVIE::")) {
                    movieTitle = value.substring(7);
                } else if (value.startsWith("TAG::")) {
                    tags.add(value.substring(5));
                }
            }

            // Write only if both movie title and tags exist
            if (movieTitle != null && !tags.isEmpty()) {
                context.write(new Text(movieTitle), new Text(tags.toString()));
            }
        }
    }

    // ---------- DRIVER ----------
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "Movie Tags Join");

        job.setJarByClass(MovieTagsJoin.class);

        // Add multiple input paths
        MultipleInputs.addInputPath(job, new Path(args[0]), TextInputFormat.class, MovieMapper.class);
        MultipleInputs.addInputPath(job, new Path(args[1]), TextInputFormat.class, TagMapper.class);

        job.setReducerClass(JoinReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileOutputFormat.setOutputPath(job, new Path(args[2]));

        job.waitForCompletion(true);
    }
}
````

---

## 📂 Input Files

### `Movie.txt`

```
movieId,title,genres
1,Toy Story (1995),Adventure|Animation|Children
2,Jumanji (1995),Adventure|Children|Fantasy
3,Grumpier Old Men (1995),Comedy|Romance
```

### `Tags.txt`

```
userId,movieId,tag,timestamp
15,1,funny,1139045764
20,1,animated,1139045765
21,2,adventure,1139045770
22,3,romantic,1139045780
```

---

## ⚙️ Execution Steps in Cloudera Quickstart VM

### 1️⃣ Start Hadoop Services

```bash
start-all.sh
```

### 2️⃣ Create Directories in HDFS

```bash
hdfs dfs -mkdir /movie
hdfs dfs -mkdir /tag
```

### 3️⃣ Copy Input Files to HDFS

```bash
hdfs dfs -copyFromLocal Movie.txt /movie
hdfs dfs -copyFromLocal Tags.txt /tag
```

### 4️⃣ Compile and Create the JAR File

```bash
hadoop com.sun.tools.javac.Main MovieTagsJoin.java
jar cf MovieTagsJoin.jar MovieTagsJoin*.class
```

### 5️⃣ Run the MapReduce Job

```bash
hadoop jar MovieTagsJoin.jar MovieTagsJoin /movie /tag /user/root/movietag_output
```

### 6️⃣ View the Output

```bash
hdfs dfs -cat /user/root/movietag_output/part-r-00000
```

---

## 🧾 Sample Output

```
Toy Story (1995)           [funny, animated]
Jumanji (1995)             [adventure]
Grumpier Old Men (1995)    [romantic]
```

---

## 🏁 Notes

* Ensure `MovieTagsJoin.java` and input files are placed in the same working directory.
* Before execution, verify Hadoop NameNode and DataNode services are running.
* Delete the output directory if it already exists:
  `hdfs dfs -rm -r /user/root/movietag_output`

---

© 2025 Big Data Analytics Lab – BIS701 | Bangalore Institute of Technology

```

---

Would you like me to generate a **ready-to-download `README.md` file** with this exact formatting so you can directly place it in your Cloudera project folder (Desktop or `/home/cloudera`)?
```
