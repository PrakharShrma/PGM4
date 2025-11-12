movieId	title	genres
1,Toy Story (1995),Adventure|Animation|Children|Comedy|Fantasy
2,Jumanji (1995),Adventure|Children|Fantasy
3,Grumpier Old Men (1995),Comedy|Romanc

userId	movieId	tag	timestamp
15,1,funny,1139045764
15,2,childish,1139045874
20,1,pixar,1139045984
20,3,oldie,1139046064

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
                    movieTitle = value.substring(7);  // extract movie title
                } else if (value.startsWith("TAG::")) {
                    tags.add(value.substring(5));  // extract tag
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
