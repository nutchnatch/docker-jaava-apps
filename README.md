# docker-java-apps

* To compile Hello.java:
$docker run -v ${PWD}:/hello -w /hello openjdk:11.0.10-buster javac Hello.java

* To run Hello.class
$ docker run -v ${PWD}:/hello -w /hello openjdk:11.0.10-buster java Hello

* Running Hello with JRE, instead of JDK:
$ docker run -v ${PWD}:/hello -w /hello openjdk:11.0.10-jre-buster java Hello 

* To remove the container after running it, add --rm option:
$ docker run -v ${PWD}:/hello -w /hello --rm openjdk:11.0.10-jre-buster java Hello 

* To delete all data:
$ docker system prune -a

# Build maven image:
$ docker build -f mven.Dockerfile -t my-api-maven .

# Run maven image:
$ docker run -it --rm my-api-maven

# Compiling the project reusing the local dependencies:
$ docker run -it --rm -v ${PWD}:/app -v ${HOME}/.m2:/root/.m2 -w /app maven:3.6.3-jdk-11-slim mvn clean package

# Multi stage builds:
- Dockerfile can contain multiple FROM instructions
- Each FROM instruction starts a new stage with a new buid context (As builder -> defines the stage)
- We can copy artifacts from a stage to another using (builder is previous stage): COPY --from=builder build/libs/app.jar app.jar
- Not copied artifacts are not passed through stages and so, they are discarded, only the final stage is kept

- FROM maven:3.6.3-jdk-11-slim As build
- WORKDIR /app
- COPY pom.xml .
- RUN mvn dependency:go-offline
- COPY src src
- RUN mvn package

- FROM tomcat:9
- COPY --from=build /app/target/web.war ${CATALINA_HOME}/webapps/ROOT.war
- EXPOSE 8080
- ENTRYPOINT ["catalina.sh", "run"]

# Memory option for doker run command:
- docker run -m 200m my-image  (m - memory limit, minimum is 4M)

# CPU options for docker run command (only for java v 8u131):
- docker run --cpu-shares=1024 my-image (--cpu-shares, -c) - charing cpu between containers
- docker run --cpus=1 my-image (number of cpus)
- docker run --cpu-period=50000 --cpu-quota=25000 my-image

# For Java 9 and java 8u131+, for cpu the following options are automatically set:
- XX:ParallelGCThreads
- XX:CICompilerCount
# For memory, use
- XX:+UnlockExperimentalVMOptions
- XX:+UseCGroupMemoryLimitForHeap
# If we don's specify -Xms and -Xmx, then initial and maximum heap size are calculated based on the amount of memory on the machine with the following flags:
- XX:InitialRAMFraction
- XX:MaxRAMFraction (defaults to 4)
* Value  -   Percentage of RAM for heap
-  1   ---->               100%
-  2   ---->               50%
-  3   ---->               33%
-  4   ---->               25%


# Compile the java cpu's application with docker:
$ docker run -it --rm -v ${PWD}:/java openjdk:8u131-slim javac /java/Stats.java     ---> will generate the Stats.class in local dir (mounted by volume)

# Run the application with docker, specifying memory and cpu consumption (java version openjdk:8u131-slim):
$ docker run -it --rm -v ${PWD}:/java -m 100m --cpus=1 openjdk:8u131-slim java -cp /java Stats
- Mem and cpu limits not respected:
Heap size: 31MB
Maximum size of heap: 443MB
Available processors: 2

# Set the memory configurations (java version openjdk:8u131-slim)
$ docker run -it --rm -v ${PWD}:/java -m 100m --cpus=1 openjdk:8u131-slim java -cp /java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap Stats
- Only Mem limit respected:
Heap size: 7MB
Maximum size of heap: 44MB
Available processors: 2

# Set the memory configurations ( changing java version to openjdk:8u191-alpine or openjdk:11.0.10-slim):
$ docker run -it --rm -v ${PWD}:/java -m 100m --cpus=1 openjdk:8u191-alpine java -cp /java Stats
$ docker run -it --rm -v ${PWD}:/java -m 100m --cpus=1 openjdk:11.0.10-slim java -cp /java Stats
- Memory and cpu limits are respected:
Heap size: 7MB
Maximum size of heap: 48MB
Available processors: 1
