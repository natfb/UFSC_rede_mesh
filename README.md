## Instalação do Meshtastic CLI (Ubuntu)

### Opção 1: Com `pipx`

```bash
sudo apt update && sudo apt install -y pipx
pipx install meshtastic
```

### Opção 2: Com ambiente virtual (`venv`)

```bash
python3 -m venv ~/.meshtastic-venv
source ~/.meshtastic-venv/bin/activate
pip install meshtastic
```

### Checar conexão
Após conectar o dispositivo via usb no computador:
```bash
meshtastic --info
```

## Configurações para aumentar distancia de transmição

Mudar para LongSlow:
```bash
meshtastic --set lora.modem_preset VERY_LONG_SLOW
```

enviar mensagem:
```bash
meshtastic --send "LOS Test 1"
```

Enable RX Boosted Gain (adds ~2-3 dB sensitivity on SX1262 chips)
```bash
meshtastic --set lora.sx126x_rx_boosted_gain true
```
# Disable automated broadcast chatter to keep the narrow channel clear
```bash
meshtastic --set telemetry.device_update_interval 0
meshtastic --set telemetry.environment_update_interval 0
```

## Configurações de localização
heltec para pegar gps do celular:
```bash
meshtastic --set position.gps_enabled false

# set GPS mode to NOT_PRESENT (lets the phone push coordinates)
meshtastic --set position.gps_mode NOT_PRESENT

# set a reasonable broadcast interval (e.g. 300s = 5 min)
meshtastic --set position.position_broadcast_secs 300

meshtastic --set position.position_flags 35

```




# Setar para cliente:

meshtastic --set device.role ROUTER_CLIENT



# setar wifi

# 1. Habilita o uso da interface de rede Wi-Fi
meshtastic --set network.wifi_enabled true

# 2. Define o nome da sua rede Wi-Fi (SSID)
meshtastic --set network.wifi_ssid "FRANCISCO"

# 3. Define a senha da rede
meshtastic --set network.wifi_psk "Bomelula13"



Não confiavel daqui pra baixo
---------------------------------
## Ler dados de sensores

Alguns sensores são detectados automaticamente. Caso queira usar um fora dessa lista é possivel fazer um fork do firmaware com platform.io.

Link do repo: https://github.com/meshtastic/firmware

## Nós
Da para configurar nós como 
- ROUTER (Roteador): O dispositivo se dedica a retransmitir o tráfego da malha e tem prioridade alta no roteamento de pacotes. Ele continua transmitindo pacotes sobre si mesmo (NodeInfo, telemetria) para que outros usuários vejam que o nó de infraestrutura existe na lista deles. É recomendado apenas se você tiver energia contínua e abundante (ligado na tomada).

- REPEATER (Repetidor): É um roteador "invisível". Ele repassa o tráfego de todo mundo, mas não transmite informações sobre si mesmo e não aparece na lista de nós (a menos que seja consultado diretamente). O objetivo do Repeater é economizar bateria e não poluir as frequências de rádio com pacotes desnecessários. É a melhor opção para nós movidos a energia solar com baterias menores.

## Canais

é possivel criar canais privados com chaves assimetricas criptografando.



# Teste de distância: T-Beam emissor e Heltec receptor

**Limitação confirmada em 22/10/2026:** o firmware `2.7.26.54e0d8d` instalado no T-Beam `nl01` exclui o módulo Range Test. Os comandos abaixo salvam a configuração, mas não fazem esse firmware enviar pacotes de teste automaticamente. O Heltec V3 `nh02` inclui o módulo e já está configurado como receptor. Consulte o [diagnóstico em hardware.md](hardware.md#range-test-ausente-no-firmware-do-t-beam--diagnóstico-de-22102026).

Use `--port` sempre que os dois dispositivos estiverem conectados ao mesmo computador. Neste par, o T-Beam `nl01` usa a porta USB Serial e o Heltec `nh02` usa a porta CP2102. Os nomes em `/dev/serial/by-id/` são mais estáveis que `/dev/ttyACM1` e `/dev/ttyUSB0`.

```bash
# T-Beam: configura envio a cada 60 segundos; exige firmware com Range Test incluído
meshtastic --port /dev/serial/by-id/usb-1a86_USB_Single_Serial_5657011105-if00 \
  --set range_test.enabled true --set range_test.sender 60

# Heltec: recebe e grava os pacotes recebidos em rangetest.csv
meshtastic --port /dev/serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0 \
  --set range_test.enabled true --set range_test.sender 0 --set range_test.save true
```

O `save` fica no receptor, onde os pacotes chegam. Para encerrar o teste, configure `range_test.sender 0` no T-Beam e `range_test.enabled false` nos dois nós.


# Nomes
meshtastic --set-owner "Meshtastic nl01"
meshtastic --set-owner-short "nl01"
