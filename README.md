# Schema Registry

## Gradle
Open and import as gradle project
```shell
gradle generateSchema
```

## Start the Schema Registry Server
```shell
docker-compose up -d
```
Waiting all ready

## gitops
[schema-registry-gitops](https://github.com/domnikl/schema-registry-gitops/tree/main)
```shell
docker run --rm --name schema-registry-gitops -v ./src/main/avdl:/avdl \
  domnikl/schema-registry-gitops:1.9.0 plan --registry http://host.docker.internal:8081 \
  /avdl/com/databius/contractbius/.state.yml
```

```shell
docker run --rm --name schema-registry-gitops -v ./src/main/avdl:/avdl \
  domnikl/schema-registry-gitops:1.9.0 apply --registry http://host.docker.internal:8081 \
  /avdl/com/databius/contractbius/.state.yml
```

Extract the schema and subjects declarations from the contract and load into `.state.yaml`
Requires: [yq](https://mikefarah.gitbook.io/yq/)
```shell
yq e '.schema | .registry[].subjects.schema = (. | .specification) | select(.type == "avro").registry | .[].subjects as $item ireduce ([]; . += $item) | {"subjects": .}' \
  ./src/main/contract/com/databius/contractbius/datacontract.yaml > ./src/main/contract/com/databius/contractbius/.state.yaml
```

Register to schema registry
```shell
docker run --rm --name schema-registry-gitops -v ./src/main/contract:/contract \
  domnikl/schema-registry-gitops:1.9.0 apply --registry http://host.docker.internal:8081 \
  /contract/com/databius/contractbius/.state.yaml
```

## Stop
```shell
docker-compose down
```