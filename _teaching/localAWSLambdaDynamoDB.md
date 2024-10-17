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

# 
