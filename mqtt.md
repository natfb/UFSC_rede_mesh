# Guia de Instalação e Execução - MQTT (Mosquitto) via Docker

Guia passo a passo para configurar o ambiente do broker Mosquitto via Docker e executar comandos de publicação/inscrição.

---

## 1. Estrutura de Arquivos e Configuração

Crie o diretório de trabalho e o arquivo de configuração do Mosquitto:

```bash
# Criar diretório para as configurações do Mosquitto
mkdir -p ~/mosquitto/config

# Criar e editar o arquivo de configuração
nano ~/mosquitto/config/mosquitto.conf
```

Cole o seguinte conteúdo no `mosquitto.conf` para permitir conexões anônimas na porta 1883:

```text
listener 1883
allow_anonymous true
```

---

## 2. Executando o Broker Mosquitto via Docker

### Subir o Contêiner do Mosquitto

```bash
docker run -d \
  --name mosquitto \
  --restart unless-stopped \
  -p 1883:1883 \
  -p 9010:9010 \
  -v ~/mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf \
  eclipse-mosquitto
```

### Reiniciar ou Remover o Contêiner (Resolução de Conflitos)

```bash
# Caso precise apenas reiniciar após alterar a configuração
docker restart mosquitto

# Caso queira parar e remover para recriar do zero
docker stop mosquitto
docker rm mosquitto
```

---

## 3. Comandos de Teste do Broker MQTT

Utilize o próprio contêiner do Mosquitto para testar o envio e o recebimento de mensagens no broker sem precisar instalar clientes na máquina hospedeira.

### Escutar Tópicos (Subscriber)

```bash
# Escutar um tópico de teste
docker exec -it mosquitto mosquitto_sub -h localhost -t "teste/topico"

# Escutar todas as mensagens em JSON do Meshtastic
docker exec -it mosquitto mosquitto_sub -h localhost -t "msh/#" -v
```

### Publicar Mensagens (Publisher)

```bash
# Publicar uma mensagem simples em um novo terminal
docker exec -it mosquitto mosquitto_pub -h localhost -t "teste/topico" -m "Ola via Docker!"
```





# Config MEshtatic

# Habilita o módulo MQTT
meshtastic --set mqtt.enabled true

# Define o servidor do HiveMQ (porta 1883 TCP sem SSL)
meshtastic --set mqtt.address "broker.hivemq.com"

# Garante que não usará TLS/SSL nem autenticação
meshtastic --set mqtt.tls_enabled false
meshtastic --set mqtt.username ""
meshtastic --set mqtt.password ""

# Define um Root Topic exclusivo para o seu teste
meshtastic --set mqtt.root "meu-teste-natalia-AeCaIy68QP"

# Permite enviar mensagens do nó para o HiveMQ (Uplink)
meshtastic --ch-index 0 --ch-set uplink_enabled true

# Permite receber mensagens do HiveMQ no nó (Downlink)
meshtastic --ch-index 0 --ch-set downlink_enabled true




# escutar broker hive mq
sudo apt update && sudo apt install mosquitto-clients -y

mosquitto_sub -h broker.hivemq.com -p 1883 -t "meu-teste-natalia-AeCaIy68QP/#" -v
