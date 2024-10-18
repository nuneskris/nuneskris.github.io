---
title: "Local Development: AWS Lambda deployed in Docker with DynamoDB in NoSQLWorkbench"
collection: teaching
type: "Application Service"
permalink: /teaching/localAWSLambdaDynamoDB
venue: "AWS"
location: "Local"
date: 2024-03-01
---

# Install DynamoDB locally
The installables can be found at the aws wbesite:https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html

      java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb

I had an issue which I will get into with Java Versions. I ensure that DynamoDB will always use Java 22 by running the below commands:

      export JAVA_HOME=$(/usr/libexec/java_home -v 22)
      export PATH=$JAVA_HOME/bin:$PATH

# Install NOSQL Workbench

![image](https://github.com/user-attachments/assets/0cb8b064-e6bf-4f81-8e9b-a2a7ff8d7e23)

## create a table which we can connect to
![image](https://github.com/user-attachments/assets/1442ee5f-da7e-4ff4-8dc5-1be5437be7c8)

# create a java maven project

1. The maven project should include libraries for Lamda and DynamoDB Services
2. plugins to ensure Java 11 is used [Note: we would need to use Java 11 to compile the lambda function]
3. Plugin for Docker to include dependencies

```xml

      <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>local-lambda-dataservices</groupId>
        <artifactId>local-lambda-dataservices</artifactId>
       <version>0.0.1-SNAPSHOT</version>
         <dependencies>
              <!-- AWS Lambda Core dependency -->
              <dependency>
                  <groupId>com.amazonaws</groupId>
                  <artifactId>aws-lambda-java-core</artifactId>
                  <version>1.2.1</version>
              </dependency>
              <dependency>
      	        <groupId>com.fasterxml.jackson.core</groupId>
      	        <artifactId>jackson-databind</artifactId>
      	        <version>2.5.3</version>
      	</dependency>
          <!-- AWS SDK for DynamoDB -->
          <dependency>
              <groupId>software.amazon.awssdk</groupId>
              <artifactId>dynamodb</artifactId>
              <version>2.20.0</version> <!-- Make sure to use a consistent and recent version -->
          </dependency>
      
          <!-- AWS SDK Core -->
          <dependency>
              <groupId>software.amazon.awssdk</groupId>
              <artifactId>sdk-core</artifactId>
              <version>2.20.0</version> <!-- Ensure this version matches the DynamoDB SDK -->
          </dependency>
      
          <!-- AWS SDK for HTTP URL connection client -->
          <dependency>
              <groupId>software.amazon.awssdk</groupId>
              <artifactId>url-connection-client</artifactId>
              <version>2.20.0</version>
          </dependency>
      
          <!-- Jackson dependency for JSON processing (sometimes needed if using older Java versions) -->
          <dependency>
              <groupId>com.fasterxml.jackson.core</groupId>
              <artifactId>jackson-databind</artifactId>
              <version>2.14.0</version>
          </dependency>
          </dependencies>
          <properties>
          <maven.compiler.source>11</maven.compiler.source>
          <maven.compiler.target>11</maven.compiler.target>
      </properties>
        <build>
          <sourceDirectory>var/task/</sourceDirectory>
          <plugins>
      	    <plugin>
      	       <groupId>org.apache.maven.plugins</groupId>
      	       <artifactId>maven-dependency-plugin</artifactId>
      	       <version>3.1.2</version>
      	       <executions>
      	         <execution>
      	           <id>copy-dependencies</id>
      	           <phase>package</phase>
      	           <goals>
      	             <goal>copy-dependencies</goal>
      	           </goals>
      	            <configuration>
                      <source>11</source> <!-- Set source compatibility to Java 11 -->
                      <target>11</target> <!-- Set target compatibility to Java 11 -->
                  </configuration>
      
      	         </execution>
      	       </executions>
      	     </plugin>
          </plugins>
        </build>
      </project>
```

# Connecting Java Client to Local DynamoDB

```java
package com.kfn.study;

import java.net.URI;
import java.net.URISyntaxException;
import java.util.HashMap;
import java.util.Map;

import software.amazon.awssdk.http.urlconnection.UrlConnectionHttpClient;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;
import software.amazon.awssdk.services.dynamodb.model.AttributeValue;
import software.amazon.awssdk.services.dynamodb.model.PutItemRequest;

public class DynamoDBClient {
	private static final String LOCAL_DYNAMODB_ENDPOINT = "http://host.docker.internal:8000";

	public static DynamoDbClient createClient() throws URISyntaxException {

		// Set up the DynamoDB client to point to the local DynamoDB instance
		DynamoDbClient dynamoDbClient = DynamoDbClient.builder().region(Region.US_WEST_2) // Region is ignored in local
																							// dynamo
				.endpointOverride(new URI(LOCAL_DYNAMODB_ENDPOINT)) // Point to local DynamoDB
				.httpClient(UrlConnectionHttpClient.create()) // Use UrlConnectionHttpClient
				.build();
		System.out.println("established connection");
		return dynamoDbClient;

	}

	public void insertData(String name) {
		try {
			Map<String, AttributeValue> item = new HashMap<>();
			// primary key
			item.put("DBPK", AttributeValue.builder().s("ENTITY#" + name).build());
			// sort key
			item.put("DBSK", AttributeValue.builder().s("ENTITY#" + name).build());
			// attribute
			item.put("name", AttributeValue.builder().s("Kris Nunes").build());

			PutItemRequest request = PutItemRequest.builder().tableName("HelloDynamoDB").item(item).build();

			createClient().putItem(request);

		} catch (Exception e) {
			// TODO Auto-generated catch block

		}

	}

}
```

# Lambda Request Handler

> We will call the below POST invocation which will define as 3 properties. httpMethod, path and body 

            curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" \
        -d '{
              "httpMethod": "POST",
              "path": "/Entity", 
              "body": "{\"name\":\"KFNEntity\"}"
            }'

The Java Lambda RequestHandler

```java
package com.kfn.study;

import java.util.Map;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class LambdaHandler implements RequestHandler<Map<String, Object>, String> {

	private final ObjectMapper objectMapper = new ObjectMapper(); // Jackson for JSON parsing

	@Override
	public String handleRequest(Map<String, Object> event, Context context) {
		// Get HTTP method and path
		String httpMethod = (String) event.get("httpMethod");
		String path = (String) event.get("path");
		System.out.println(httpMethod);
		System.out.println(path);

		// Dispatch based on path and method
		switch (path) {
		case "/Entity":
			return hanEntityRequest(httpMethod, event);
		case "/People":
			return "Unsupported path: " + path;
		default:
			return "Unsupported path: " + path;
		}
	}

	// Handler for /Customer
	private String hanEntityRequest(String method, Map<String, Object> event) {
		switch (method) {
		case "GET":

			return "Fetching customer details";
		case "POST":
			// Extract and process the request body for POST
			String body = (String) event.get("body");
			// Assuming the payload is a JSON object with fields

			try {
				Map<String, String> bodyMap = objectMapper.readValue(body, Map.class);
				String name = bodyMap.get("name");
				DynamoDBClient client = new DynamoDBClient();
				client.insertData(name);
			} catch (JsonMappingException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (JsonProcessingException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

		default:
			return "Unsupported HTTP method for /Customer";
		}
	}
}
```


# Building Lamdda in Docker

## Dockerfile

Note: we use java 11

            FROM public.ecr.aws/lambda/java:11
            
            # Copy function code and runtime dependencies from Maven layout
            COPY target/classes ${LAMBDA_TASK_ROOT}
            COPY target/dependency/* ${LAMBDA_TASK_ROOT}/lib/
            
            # Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
            CMD [ "com.kfn.study.LambdaHandler::handleRequest" ]

# Build and run docker

Note we need provide dummy AWS_ACCESS_KEY_ID and dummy AWS_SECRET_ACCESS_KEY for lambda to call DynamoDB. not sure why

       mvn clean compile dependency:copy-dependencies -DincludeScope=runtime
      docker build -t java-lambda .   
      docker run --rm -p 9000:8080 -e AWS_ACCESS_KEY_ID=dummyAccessKey -e AWS_SECRET_ACCESS_KEY=dummySecretKey java-lambda

![image](https://github.com/user-attachments/assets/f446bf72-3a47-4d13-9e9b-582ae44993be)

# Running the Lambda Funciton

![image](https://github.com/user-attachments/assets/c818ba32-a4ce-46a8-b476-e2a51c6406f7)

![image](https://github.com/user-attachments/assets/4f3f1c5d-f30b-471c-94ad-d81ecc875363)

An insert is made in the DynamoDB database

![image](https://github.com/user-attachments/assets/bf81d0cd-0c58-4dc6-9c1e-3741812612df)







