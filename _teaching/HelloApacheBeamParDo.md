---
title: "Apache Beam - Using Pardo"
collection: teaching
type: "Data Processing"
permalink: /teaching/HelloApacheBeamParDo
date: 2024-06-01
venue: "Beam"
date: 2024-06-01
location: "Local"
---

We had performed the same demonstration using SimpleFuntions. Refer [Page](https://nuneskris.github.io/teaching/HelloApacheBeam).  We will be using Pardo Function for this.
Refer https://github.com/nuneskris/hello-apache-beam for detailed description of the demo without Pardo.

We will perform the same Apache Beam functions in the previous demo, with the below enhancements
1. Using Pardo Functionss
2. Running it on GCP Cloud using GCP DataFlow Runner
3. Input and Output using GCP Storage
4. Deploy from on-prem using maven

Objectives
1. Input Storage Bucket and file
2. IAM Service Account with Permissions
3. Beam Code

# 1. Input Storage Bucket and file

![image](https://github.com/user-attachments/assets/1a3a3181-1e65-47bf-a0dc-823f921ebe23)

# 2. IAM Service Account with Permissions

![image](https://github.com/user-attachments/assets/b5d678db-2e4a-4271-97ac-1266985ae83c)

I have an addtional roles for BigQuery for another demo. Also created keys and downloaded them.

# 3. Beam Code
* This pipeline reads cricket score data from a GCS file, processes it to filter and count the number of wickets per player and dismissal type, and then writes the results back to a GCS file.
* The DoFn classes are used for specific processing tasks, such as extracting data, filtering wickets, converting to key-value pairs, and summing values by key.

```java
package com.nuneskris.study.beam; 

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.Description;
import org.apache.beam.sdk.options.PipelineOptions;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.values.KV;

/*
 * This Java class, BeamScoreViaOnlyPardo, is an Apache Beam pipeline that processes a cricket score dataset. 
 * The pipeline reads data from a file stored in Google Cloud Storage (GCS), processes it using several transformations (ParDo operations), 
 * and writes the results back to GCS.
 */
public class BeamScoreViaOnlyPardo{
	 
	/*
	 * JobOptions is an interface that extends PipelineOptions. It defines custom options for the pipeline, such as InputFile and OutputFile paths. 
	 * These options are used to specify the location of input and output files.
	 */
	public interface JobOptions extends PipelineOptions {
		
		// this is the JSON option used for providing the location of the input and output file.
	
        @Description("Path of the file to read from")
        String getInputFile();
        void setInputFile(String value);

        @Description("Path of the file to read from")
        String getOutputFile();
        void setOutputFile(String value);
	}
	
	/*	
	 * The main method is the entry point of the program.
		 - It registers the JobOptions interface with PipelineOptionsFactory to handle the command-line arguments.
		 - It then calls processViaPardo with the configured options.
	 */
	
	public static void main(String[] args) throws Exception {
    	PipelineOptionsFactory.register(JobOptions.class);
    	JobOptions options = PipelineOptionsFactory
    	        .fromArgs(args)
    	        .withValidation()
    	        .as(JobOptions.class);
        
    	processViaPardo(options);
    }
    
	/*
	 * This method sets up the pipeline.
		- It reads data from the GCS file specified by getInputFile() using TextIO.read().
		- The data is written to the GCS location specified by getOutputFile() using TextIO.write().
		- withoutSharding() ensures that the output is written to a single file instead of multiple shards.
	 */
    private static void processViaPardo(JobOptions options) {
        Pipeline pipeline = Pipeline.create(options);
        pipeline.apply("Read from GCS", TextIO.read().from(options.getInputFile()))
                .apply("Write To GCS",TextIO.write().to(options.getOutputFile()).withoutSharding())
                ;
        pipeline.run().waitUntilFinish();
    }
    /** A {@link DoFn} that splits lines of text into individual column cells. */
    // The first variable of the doFn is the input and the second variable id the output
    /*
     * ExtractScore is a DoFn (a function applied to elements in a PCollection).
		- It splits each line of text (a string) into an array of strings (columns) by separating on commas.
     */
    public static class ExtractScore extends DoFn<String, String[]> {
		private static final long serialVersionUID = -4296667754676058140L;
		@ProcessElement
        public void processElement(ProcessContext c) {
            String[] words = c.element().split(",");
            c.output(words);
        }
    }
    // We filter by not calling .output for those rows which are filtered out
    /*
     * FilterWickets filters the data to only include rows where the 12th column (index 11) indicates a wicket ("1").
		- Rows that do not represent a wicket are not passed on to the next stage.
     */
    public static class FilterWickets extends  DoFn<String[], String[]> {
		private static final long serialVersionUID = -3042129865531281093L;

		@ProcessElement
        public void processElement(ProcessContext c) {
            if(c.element()[11].equalsIgnoreCase("1")) {
                c.output(c.element());
            }
        }
    }
    
    /*
     * ConvertToKV creates a key-value pair (KV<String, Integer>).
		- The key is a combination of the batsman and the type of dismissal, while the value is always 1, representing a single wicket.
     */
    public static class ConvertToKV extends  DoFn<String[], KV<String, Integer>> {
		private static final long serialVersionUID = -4110309215590766497L;

		@ProcessElement
        public void processElement(ProcessContext c) {
            String key = c.element()[4] + "," + c.element()[12];
            c.output(KV.of(key, Integer.valueOf(1)));
        }
    }
    /*
     * SumUpValuesByKey aggregates the wickets by summing up the values for each key (batsman and dismissal type).
		- It outputs the total number of wickets for each key as a string.
     */
    public static class SumUpValuesByKey extends DoFn<KV<String, Iterable<Integer>>, String>{
		private static final long serialVersionUID = -7808059852703100891L;

		@ProcessElement
        public void processElement(ProcessContext context) {
            int totalWickets = 0;
            String playerAndWicketType = context.element().getKey();
            Iterable<Integer> wickets = context.element().getValue();
            for (Integer amount : wickets) {
                totalWickets += amount.intValue();
            }
            context.output(playerAndWicketType + "," + totalWickets);
        }
    }
}

```
