# Demonstração do Quarkus: Amazon SSM Client

Este exemplo mostra como usar o cliente AWS SSM com o Quarkus. Como pré-requisito, instale a [interface de linha de comando da AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

# Instância local do AWS SSM

Basta executá-lo da seguinte maneira para iniciar o SSM localmente:

`docker run --rm --name local-ssn -p 8014:4583 -e SERVICES=ssm -e START_WEB=0 -d localstack/localstack:0.11.1`

O SSM escuta em `localhost:8014` para endpoints REST.

Crie um perfil da AWS para sua instância local usando a AWS CLI:

```
$ aws configure --profile localstack
AWS Access Key ID [None]: test-key
AWS Secret Access Key [None]: test-secret
Default region name [None]: us-east-1
Default output format [None]:
```

# Execute a demonstração no modo dev

- Run `./mvnw clean package` and then `java -Dparameters.path=/quarkus/is/awesome/ -jar ./target/quarkus-app/quarkus-run.jar`
- In dev mode `./mvnw clean quarkus:dev -Dparameters.path=/quarkus/is/awesome/`

## Defina alguns parâmetros
Primeiro, adicione quantos parâmetros quiser usando os seguintes padrões para parâmetros simples e seguros:

```
curl -XPUT -H"Content-type: text/plain" "http://localhost:8080/sync/secure?secure=true" -d"stored as cipher text"
curl -XPUT -H"Content-type: text/plain" "http://localhost:8080/sync/plain" -d"stored as plain text"
```

## Listar todos os parâmetros
Agora você pode listar os parâmetros adicionados:

```
curl http://localhost:8080/sync
```

Você deve ver uma saída como esta:
```
{"plain":"stored as plain text","secure":"stored as cipher text"}
```

## Testar os endpoints assíncronos
Substitua `sync` por `async` nos exemplos acima para testar os endpoints assíncronos.

# Executnado em nativo

Você pode compilar o aplicativo em um binário nativo usando:

`./mvnw clean install -Pnative`

e executar com:

`./target/amazon-ssm-quickstart-1.0.0-SNAPSHOT-runner -Dparameters.path=/quarkus/is/awesome/` 


# Executando nativo no contêiner

Crie uma imagem nativa no contêiner executando:

`./mvnw clean package -Pnative -Dnative-image.docker-build=true`

Crie uma imagem do docker:

`docker build -f src/main/docker/Dockerfile.native -t quarkus/amazon-ssm-quickstart .`

Crie uma rede que conecte seu container com localstack:

`docker network create localstack`

Stop your localstack container you started at the beginning:

`docker stop local-ssm`

Start localstack and connect to the network:

`docker run --rm --network=localstack --name localstack -p 8014:4583 -e SERVICES=ssm -e START_WEB=0 -d localstack/localstack:0.11.1`

Run quickstart container connected to that network (note that we're using internal port of the localstack):

`docker run -i --rm --network=localstack -p 8080:8080 -e QUARKUS_SSM_ENDPOINT_OVERRIDE="http://localstack:4583" -e PARAMETERS_PATH="/quarkus/is/awesome/" quarkus/amazon-ssm-quickstart`
