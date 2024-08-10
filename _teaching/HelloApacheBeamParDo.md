---
title: "Apache Beam Using Pardo in DataFlow"
collection: teaching
type: "Data Processing"
permalink: /teaching/HelloApacheBeamParDo
date: 2024-06-01
venue: "Beam"
date: 2024-06-01
location: "Local"
---
<img width="581" alt="image" src="https://github.com/user-attachments/assets/651cc19a-a639-4197-b155-b55fc1e2100a">

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
4. Build POM
5. Deploy
6. Run and Validate

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
import org.apache.beam.sdk.transforms.GroupByKey;
import org.apache.beam.sdk.transforms.ParDo;
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
				        .apply(ParDo.of(new BeamScoreViaOnlyPardo.ExtractScore()))
				        .apply(ParDo.of(new BeamScoreViaOnlyPardo.FilterWickets()))
				        .apply(ParDo.of(new BeamScoreViaOnlyPardo.ConvertToKV()))
				        .apply(GroupByKey.<String, Integer>create())
				        .apply(ParDo.of(new BeamScoreViaOnlyPardo.SumUpValuesByKey()))
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

# 4. Build POM
This POM file is set up for an Apache Beam project that can run both locally using the Direct Runner and on Google Cloud Dataflow. It manages dependencies for Beam, SLF4J, and Google Cloud libraries, and it uses Maven plugins to compile the code, package it into a JAR, and bundle all dependencies into a single JAR for easy deployment.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>HelloApacheBeam</groupId>
  <artifactId>HelloApacheBeam</artifactId>
  <version>0.0.1-SNAPSHOT</version>
   <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven-jar-plugin.version>3.1.1</maven-jar-plugin.version>
        <maven-shade-plugin.version>3.2.4</maven-shade-plugin.version>
        <maven-compiler-plugin.version>3.8.0</maven-compiler-plugin.version>
        <slf4j.version>1.7.25</slf4j.version>
        <beam-version>2.58.0</beam-version>
    </properties>
    <dependencies>
        <!-- Apache Beam SDK -->
        <dependency>
            <groupId>org.apache.beam</groupId>
            <artifactId>beam-sdks-java-core</artifactId>
            <version>${beam-version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-jdk14</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
	        <groupId>com.google.cloud</groupId>
	        <artifactId>google-cloud-bigquery</artifactId>
	        <version>2.29.0</version>
    	</dependency>
    	<dependency>
	        <groupId>org.apache.beam</groupId>
	        <artifactId>beam-sdks-java-io-google-cloud-platform</artifactId>
	        <version>2.46.0</version>
	    </dependency>

    </dependencies>
    <profiles>
        <profile>
            <id>direct</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <dependencies>
                <!-- Direct Runner for local execution -->
                <dependency>
                    <groupId>org.apache.beam</groupId>
                    <artifactId>beam-runners-direct-java</artifactId>
                    <version>${beam-version}</version>
                    <scope>runtime</scope>
                </dependency>
            </dependencies>
        </profile>
        <profile>
            <id>dataflow</id>
            <dependencies>
                <!-- Dataflow Runner for execution on GCP -->
                <dependency>
                    <groupId>org.apache.beam</groupId>
                    <artifactId>beam-runners-google-cloud-dataflow-java</artifactId>
                    <version>${beam-version}</version>
                    <!--<scope>runtime</scope>-->
                </dependency>
                <dependency>
                    <groupId>org.apache.beam</groupId>
                    <artifactId>beam-sdks-java-io-google-cloud-platform</artifactId>
                    <version>${beam-version}</version>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven-compiler-plugin.version}</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>${maven-jar-plugin.version}</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>${maven-shade-plugin.version}</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <finalName>${project.artifactId}-bundled-${project.version}</finalName>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/LICENSE</exclude>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

# 5. Deploy

First we would need to set the credetials the service account would need to use.
This command compiles the Java project, then runs the BeamScoreViaOnlyPardo class as a Dataflow job in Google Cloud. The job reads data from a specified input file in GCS, processes it, and writes the transformed data back to another file in GCS, using the DataflowRunner to execute the pipeline on Google Cloud's Dataflow service.

```console
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account-file.json"

mvn compile exec:java \
-Pdataflow \
-Dexec.mainClass=com.nuneskris.study.beam.BeamScoreViaOnlyPardo \
-Dexec.cleanupDaemonThreads=false \
-Dexec.args=" \
--project=durable-pipe-431319-g7 \
--region=us-central1 \
--inputFile=gs://daflow-ingest-kfn-study/IPLMatches2008-2020.csv \
--outputFile=gs://daflow-ingest-kfn-study/IPLMatches2008-2020-Transformed.csv \
--runner=DataflowRunner"
```
1. mvn compile exec:java:
* mvn compile: This command compiles the project, ensuring that all Java source files are compiled into bytecode.
* exec:java: This goal runs a Java program that is part of the project using Maven. It's provided by the exec-maven-plugin.
2. -Pdataflow:
* This flag specifies the Maven profile to use when executing the command. In this case, it's the dataflow profile. This profile likely includes dependencies and configurations specific to running the job on Google Cloud Dataflow.
3. -Dexec.mainClass=com.nuneskris.study.beam.BeamScoreViaOnlyPardo:
  * This flag specifies the fully qualified name of the Java class containing the main method that will be executed. Here, the class is com.nuneskris.study.beam.BeamScoreViaOnlyPardo.
4. -Dexec.cleanupDaemonThreads=false:
* This option prevents Maven from attempting to clean up daemon threads upon completion. This is often necessary for long-running tasks or when using certain libraries that spawn threads.
5. -Dexec.args="...":
This flag passes additional arguments to the Java program. These arguments are specific to the Apache Beam pipeline and configure how it runs. Let's break down the arguments:
* --project=durable-pipe-431319-g7: Specifies the Google Cloud project ID where the Dataflow job will run.
* --region=us-central1: Defines the GCP region where the Dataflow job will be executed.
* --inputFile=gs://daflow-ingest-kfn-study/IPLMatches2008-2020.csv: The input file's location in a Google Cloud Storage (GCS) bucket. The Dataflow job will read data from this file.
* --outputFile=gs://daflow-ingest-kfn-study/IPLMatches2008-2020-Transformed.csv: The output file's location in GCS where the transformed data will be written.
* --runner=DataflowRunner: Specifies that the job should be executed on Google Cloud Dataflow, which is a fully managed service for running Apache Beam pipelines.

# 6. Run and Validate

Run the above command in a terminal
![image](https://github.com/user-attachments/assets/d63b083c-0c97-4d30-b58e-d01120718cc1)

We canw e job was deployed and also executed.

![image](https://github.com/user-attachments/assets/fe171b23-f87b-4734-9b45-65d720759ecf)

![image](https://github.com/user-attachments/assets/edf595b0-0b4b-476e-bbf5-494d1f8a480f)


* File created in the bucket

![image](https://github.com/user-attachments/assets/4bb73c1c-7f3b-49be-aa4c-e39a3f165c4d)

![image](https://github.com/user-attachments/assets/3c1b8888-f914-4847-82f4-f851b06efaef)





