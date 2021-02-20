# Kafdrop Protobuf
### Building the proto descriptor file
In order to view protobuf message, kafdrop needs the protobuf descriptor file. This file can be built using `protoc` the protobuf-compiler. Ubuntu user can install this tool along with the `kafkacat` which will be used to later upload the proto payload using:
```shell
sudo apt update
sudo apt install -y kafkacat protobuf-compiler
```
Mac user with Homebrew:
```shell
brew install kafkacat protobuf
```
Build the descriptor file. For example using the proto from this repo:
```shell
protoc -o path/to/target/Person.desc Person.proto
```
### Running the containers
The easiest way to run kafdrop with the accompanying kafka cluster is to use the `docker-compose.yaml` in this repo, which is a slightly modified version of the [original](https://github.com/obsidiandynamics/kafdrop/blob/master/docker-compose/kafka-kafdrop/docker-compose.yaml) kafdrop's docker-compose yaml.

Please replace the line under the `kafdrop: > volumes:` that says `your-protobufdesc-dir-here` with the correct path. And then, as usual:
```shell
docker compose up -d
```
### Uploading the payload
The kafka broker from the newly build container does not have any message yet. Let's upload a `Person` message:
```shell
cat John.person \
| protoc --encode=Person Person.proto \
| kafkacat -b localhost:9092 -t Person
```
We are done. Open your browser and navigate to `localhost:9000` to browse the message.
### Running the JAR instead
The JAR can be build using Maven. Clone the kafdrop repo, and then from inside the repo directory checkout the latest non-SNAPSHOT version:
```shell
git checkout 3.27.0
```
Then with JDK 11 build the JAR:
```shell
mvn package
```
When the build succeeds, the JAR will be inside the `target` directory. This command will run the  JAR with protobuf support:
```shell
java --add-opens=java.base/sun.nio.ch=ALL-UNNAMED \
    -jar target/kafdrop-3.27.0.jar \
    --kafka.brokerConnect=<host:port,host:port>,... \
    --message.format=PROTOBUF \
    --protobufdesc.directory=/your/proto/desc/dir
```
