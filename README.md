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



Perfect 👍 — here’s the **ready-to-use `README.md`** for **Program 2 – Matrix Multiplication using MapReduce** from your *BDA Manual*, written in the **same layout, indentation, and Markdown spacing style** as Program 4.

---

```markdown
# 🧮 Program 2 – Matrix Multiplication (MapReduce)

## ✅ Objective
Develop a **MapReduce program** to implement **Matrix Multiplication** using Hadoop.

---

## 🧠 Concept
MapReduce divides a large computation task into **Mapper** and **Reducer** phases.

- **Mapper**: emits key–value pairs for partial multiplications of matrices A and B.  
- **Reducer**: collects partial results for each cell `(i,k)` and computes the sum of products.

If **A** is an *m × n* matrix and **B** is an *n × p* matrix,  
then their product **C = A × B** will be an *m × p* matrix where

```

C[i][k] = Σ ( A[i][j] × B[j][k] )   for j = 1 … n

````

---

## 💻 Java Program – `MatMul.java`

```java
import java.io.IOException;
import java.util.HashMap;
import org.apache.hadoop.conf.*;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.*;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class MatMul {

    // ---------- MAPPER ----------
    public static class MatrixMapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {

            Configuration conf = context.getConfiguration();
            int m = Integer.parseInt(conf.get("m"));
            int p = Integer.parseInt(conf.get("p"));

            String line = value.toString();
            String[] parts = line.split(",");

            // parts[0] = Matrix Name (M or N)
            // parts[1] = Row index i or j
            // parts[2] = Column index j or k
            // parts[3] = Value

            if (parts[0].equals("M")) {
                // For every element M[i][j], emit (i,k)
                for (int k = 0; k < p; k++) {
                    context.write(
                        new Text(parts[1] + "," + k),
                        new Text("M," + parts[2] + "," + parts[3])
                    );
                }
            } else {
                // For every element N[j][k], emit (i,k)
                for (int i = 0; i < m; i++) {
                    context.write(
                        new Text(i + "," + parts[2]),
                        new Text("N," + parts[1] + "," + parts[3])
                    );
                }
            }
        }
    }

    // ---------- REDUCER ----------
    public static class MatrixReducer extends Reducer<Text, Text, Text, Text> {
        @Override
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {

            HashMap<Integer, Float> hashA = new HashMap<>();
            HashMap<Integer, Float> hashB = new HashMap<>();

            for (Text val : values) {
                String[] tokens = val.toString().split(",");
                if (tokens[0].equals("M")) {
                    hashA.put(Integer.parseInt(tokens[1]), Float.parseFloat(tokens[2]));
                } else {
                    hashB.put(Integer.parseInt(tokens[1]), Float.parseFloat(tokens[2]));
                }
            }

            int n = Integer.parseInt(context.getConfiguration().get("n"));
            float result = 0.0f;

            for (int j = 0; j < n; j++) {
                float a_ij = hashA.getOrDefault(j, 0.0f);
                float b_jk = hashB.getOrDefault(j, 0.0f);
                result += a_ij * b_jk;
            }

            if (result != 0.0f) {
                context.write(null, new Text(key.toString() + "," + result));
            }
        }
    }

    // ---------- DRIVER ----------
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();

        // M is m×n, N is n×p
        conf.set("m", "2");
        conf.set("n", "2");
        conf.set("p", "2");

        Job job = Job.getInstance(conf, "Matrix Multiplication");

        job.setJarByClass(MatMul.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        job.setMapperClass(MatrixMapper.class);
        job.setReducerClass(MatrixReducer.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        job.waitForCompletion(true);
    }
}
````

---

## 📂 Input File Format

Each line of input identifies the matrix name (M or N) and one element:

```
<MatrixName>,<RowIndex>,<ColumnIndex>,<Value>
```

### Example `matrix.txt`

```
M,0,0,1
M,0,1,2
M,1,0,3
M,1,1,4
N,0,0,5
N,0,1,6
N,1,0,7
N,1,1,8
```

---

## ⚙️ Execution Steps in Cloudera Quickstart VM

### 1️⃣ Start Hadoop Services

```bash
start-all.sh
```

### 2️⃣ Create Input Directory

```bash
hdfs dfs -mkdir /matin
```

### 3️⃣ Copy Input File to HDFS

```bash
hdfs dfs -copyFromLocal matrix.txt /matin
```

### 4️⃣ Compile and Create JAR File

```bash
hadoop com.sun.tools.javac.Main MatMul.java
jar cf MatMul.jar MatMul*.class
```

### 5️⃣ Run the MapReduce Job

```bash
hadoop jar MatMul.jar MatMul /matin /matout
```

### 6️⃣ View the Output

```bash
hdfs dfs -cat /matout/part-r-00000
```

---

## 🧾 Sample Output

```
0,0,19.0
0,1,22.0
1,0,43.0
1,1,50.0
```

---

## 🏁 Notes

* Ensure **HDFS, NameNode, and DataNode** services are active before execution.
* Remove previous outputs if any:

  ```bash
  hdfs dfs -rm -r /matout
  ```
* Adjust `m`, `n`, `p` values in the code for different matrix sizes.

---

© 2025 Big Data Analytics Lab – BIS701 | Bangalore Institute of Technology

```

---

Would you like me to prepare a **ready-to-download `README_Program2.md`** file with this exact formatting (so you can copy it directly into your Cloudera project folder)?
```

Excellent 👌 — here’s your **`README.md`** for **Program 3 – Weather Data Analysis using MapReduce**, written in the *exact same style, indentation, and Markdown structure* as the previous two (Program 2 and Program 4).

This file is fully ready to be copied into your project or opened in Cloudera as-is.

---

````markdown
# 🌦️ Program 3 – Weather Data Analysis (MapReduce)

## ✅ Objective
Develop a **MapReduce program** that mines weather data and displays appropriate messages indicating **Hot Days** and **Cold Days** based on temperature thresholds.

---

## 🧠 Concept
This MapReduce program reads a weather dataset and classifies each day as:
- **Hot Day** → Maximum temperature > 35 °C  
- **Cold Day** → Minimum temperature < 10 °C  

The Mapper extracts temperature information from each line,  
and the Reducer outputs the categorized results.

---

## 💻 Java Program – `MyMaxMin.java`

```java
import java.io.IOException;
import java.util.Iterator;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.conf.Configuration;

public class MyMaxMin {

    // ---------- MAPPER ----------
    public static class MaxTemperatureMapper extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {

            String line = value.toString();

            // Skip empty lines
            if (line.length() == 0) return;

            // Extract date and temperatures from line
            String date = line.substring(6, 14);                // YYYYMMDD
            float tempMax = Float.parseFloat(line.substring(39, 45).trim());
            float tempMin = Float.parseFloat(line.substring(47, 53).trim());

            // Conditions
            if (tempMax > 35.0) {
                context.write(new Text("Hot Day " + date), new Text(String.valueOf(tempMax)));
            }

            if (tempMin < 10.0) {
                context.write(new Text("Cold Day " + date), new Text(String.valueOf(tempMin)));
            }
        }
    }

    // ---------- REDUCER ----------
    public static class MaxTemperatureReducer extends Reducer<Text, Text, Text, Text> {
        @Override
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {

            Iterator<Text> it = values.iterator();
            String temperature = it.next().toString();
            context.write(key, new Text(temperature));
        }
    }

    // ---------- DRIVER ----------
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = new Job(conf, "Weather Data Analysis");

        job.setJarByClass(MyMaxMin.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        job.setMapperClass(MaxTemperatureMapper.class);
        job.setReducerClass(MaxTemperatureReducer.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        Path outputPath = new Path(args[1]);
        outputPath.getFileSystem(conf).delete(outputPath, true);

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
````

---

## 📂 Input File – `datafileofweather.txt`

Each line represents one day’s record (fixed-width format).
Columns include station ID, date, temperature readings, etc.

Example snippet:

```
USC0010 20150101     ...     36.2    12.4
USC0010 20150102     ...     39.0     9.8
USC0010 20150103     ...     33.5     7.5
```

*(Note: actual dataset lines are much longer; only positions 6–14, 39–45, and 47–53 are used.)*

---

## ⚙️ Execution Steps in Cloudera Quickstart VM

### 1️⃣ Start Hadoop Services

```bash
start-all.sh
```

### 2️⃣ Create Input Directory

```bash
hdfs dfs -mkdir /maxinput
```

### 3️⃣ Copy Input File to HDFS

```bash
hdfs dfs -copyFromLocal datafileofweather.txt /maxinput
```

### 4️⃣ Compile and Create JAR File

```bash
hadoop com.sun.tools.javac.Main MyMaxMin.java
jar cf MyMaxMin.jar MyMaxMin*.class
```

### 5️⃣ Run the MapReduce Job

```bash
hadoop jar MyMaxMin.jar MyMaxMin /maxinput /maxoutput
```

### 6️⃣ View the Output

```bash
hdfs dfs -cat /maxoutput/part-r-00000
```

---

## 🧾 Sample Output

```
Cold Day 20150101    -21.8
Cold Day 20150102    -24.9
Cold Day 20150103    -28.2
Hot Day 20150720      39.0
```

---

## 🏁 Notes

* Make sure Hadoop services (`NameNode`, `DataNode`, `YARN`) are running before execution.
* If the output directory already exists, delete it:

  ```bash
  hdfs dfs -rm -r /maxoutput
  ```
* The substring positions (6–14, 39–45, 47–53) are based on **NOAA weather data** fixed-width format.
  Adjust them if your input file differs in layout.

---

© 2025 Big Data Analytics Lab – BIS701 | Bangalore Institute of Technology

```

---

Would you like me to now create a **combined ZIP** containing `README_Program2.md`, `README_Program3.md`, and `README_Program4.md` — all neatly formatted and ready to import into your Cloudera project folder?
```


