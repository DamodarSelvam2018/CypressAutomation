<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>gatling-java17-sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <scala.version>2.13.8</scala.version>
        <gatling.version>3.9.0</gatling.version>
    </properties>

    <dependencies>
        <!-- Gatling Dependencies -->
        <dependency>
            <groupId>io.gatling</groupId>
            <artifactId>gatling-test-framework</artifactId>
            <version>${gatling.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.gatling.highcharts</groupId>
            <artifactId>gatling-charts-highcharts</artifactId>
            <version>${gatling.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.gatling</groupId>
            <artifactId>gatling-app</artifactId>
            <version>${gatling.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Scala Library -->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>

        <!-- Java Dependencies -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.36</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>

            <!-- Scala Maven Plugin -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>4.5.6</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- Gatling Maven Plugin -->
            <plugin>
                <groupId>io.gatling</groupId>
                <artifactId>gatling-maven-plugin</artifactId>
                <version>3.0.3</version>
                <executions>
                    <execution>
                        <phase>test</phase>
                        <goals>
                            <goal>execute</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>sonatype</id>
            <url>https://oss.sonatype.org/content/groups/public</url>
        </repository>
    </repositories>
</project>

package simulations

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class PerformanceTest extends Simulation {

  // Define the base URL
  val httpProtocol = http
    .baseUrl("https://your.api.endpoint") // Replace with your API base URL
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")

  // Define the CSV feeder
  val csvFeeder = csv("data/input_paths.csv").circular

  // Define the scenario
  val scn = scenario("Path Parameter Test")
    .feed(csvFeeder) // Feed the scenario with CSV data
    .exec(http("Post to Path Endpoint")
      .post("/your/path/${pathParam}") // Use the path parameter from the CSV
      .check(status.is(200)) // Check if the status is 200
      .check(jsonPath("$.token").saveAs("authToken"))) // Extract token and save it as 'authToken'
    
    .pause(2) // Wait for 2 seconds

    .exec(http("GET with Token")
      .get("/another/endpoint")
      .header("Authorization", "Bearer ${authToken}") // Use the extracted token
      .check(status.is(200))) // Check if the status is 200

  // Set up the simulation
  setUp(
    scn.inject(
      atOnceUsers(10) // Adjust the number of users
    )
  ).protocols(httpProtocol)
}

