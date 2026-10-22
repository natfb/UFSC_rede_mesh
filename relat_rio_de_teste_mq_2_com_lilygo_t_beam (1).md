# Relatório de Teste de Campo: Integração do Sensor de Gás MQ-2 e Sensor de Temperatura (HTU21D) com LilyGO T-Beam (Meshtastic)

## Visão Geral

Este documento regista o processo de configuração, os passos de resolução de problemas e o ajuste final bem-sucedido para integrar um **sensor de gás e fumo MQ-2** e um **sensor de temperatura/humidade HTU21D** a um **LilyGO T-Beam** a correr o firmware Meshtastic. O objetivo principal foi estabelecer um nó de telemetria ambiental e alerta remoto de gás através de uma malha LoRa privada e encriptada.

---

## A Tentativa com o Sensor de Temperatura (HTU21D) e a Causa da Falha

Antes da conclusão bem-sucedida do MQ-2, foi feita a tentativa de ligar um **sensor ambiental HTU21D** no barramento I2C (`0x40`) do T-Beam para monitorização contínua de temperatura e humidade. No entanto, este componente **não funcionou**.

### O que aconteceu (Diagnóstico do Erro):
1. **Conflito de Prioridade de Sonda no Firmware (Firmware `2.7.x`):** 
   O algoritmo de deteção automática de hardware do Meshtastic no barramento I2C confundiu o endereço `0x40` do HTU21D, identificando-o incorretamente como um **sensor de corrente INA219**.
2. **Sintomas do Erro:**
   * O sistema apresentava leituras de energia falsas (valores absurdos como `8.2V` na bateria).
   * Os dados reais de temperatura e humidade retornavam constantemente o valor **zero (`0`)**.
3. **Soluções Identificadas (mas não aplicadas neste nó focado em segurança):**
   * Desativar a medição de energia por barramento (`power_measurement_enabled`).
   * Fazer o *rollback* (downgrade) para o firmware da versão `2.5.x` através de carregamento de `.zip` para reordenar a prioridade de sondagem do barramento I2C.
   * *Decisão:* Para isolar o problema e garantir a fiabilidade do alarme crítico de segurança, o foco foi totalmente redirecionado para o funcionamento perfeito do sensor de gás MQ-2 via GPIO digital.

---

## Configuração de Hardware do MQ-2 e Correção de Cablagem

Durante as fases iniciais de testes do MQ-2, os alertas não eram disparados corretamente ou entravam num ciclo infinito de repetição. Um problema crítico descoberto e resolvido foi:

* **Problema de Cablagem:** O pino de saída digital (`DO`) do módulo estava incorretamente ligado ao pino **`AO` (Saída Analógica)** em vez da linha correta de comparação digital.
* **Correção:** A linha de sinal digital foi redirecionada com sucesso da saída do comparador do sensor (`DO`) para o **GPIO 13** do LilyGO T-Beam.

### Mapeamento Final de Pinos

| Pino do MQ-2 | Pino do LilyGO T-Beam | Descrição |
| :--- | :--- | :--- |
| **VCC** | **5V** | Obrigatório para alimentar a resistência de aquecimento interna e o comparador LM393 |
| **GND** | **GND** | Referência comum de massa |
| **DO** | **GPIO 13** | Sinal digital de disparo (interrupção por evento) |
| **AO** | *Desconectado* | Não utilizado para a deteção binária de eventos |

---

## Configuração de Firmware e Módulo de Deteção

Para contornar as restrições de segurança do canal público e eliminar o reenvio contínuo de mensagens a cada minuto, foram aplicados os seguintes parâmetros ao módulo **Detection Sensor** do Meshtastic:

### 1. Segurança do Canal (Canal Privado com Chave AES-256)
O firmware do Meshtastic bloqueia o envio automatizado de alertas de sensores em canais públicos abertos (`AQ==`) para evitar spam na rede.
* **Resolução:** Foi gerada e aplicada uma Chave Pré-Partilhada (PSK) privada ao canal primário:
  ```bash
  meshtastic --ch-set psk random --ch-index 0
  ```

### 2. Parâmetros do Sensor de Deteção
Para garantir que o nó apenas transmite um único alerta quando o evento ocorre:
* **Pull-up Interno:** Ativado para estabilizar o pino flutuante do ESP32 em estado repouso.
* **Tipo de Gatilho:** Ajustado para deteção por borda/transição (`FALLING_EDGE` / valor `2`), disparando estritamente no momento em que o fumo/gás é detetado.
* **Desativação de Estado Periódico:** Definido `state_broadcast_secs` como `0` para impedir envios sem alterações reais.

Comandos finais executados no terminal:
```bash
meshtastic --set detection_sensor.enabled true
meshtastic --set detection_sensor.monitor_pin 13
meshtastic --set detection_sensor.use_pullup true
meshtastic --set detection_sensor.detection_trigger_type 2
meshtastic --set detection_sensor.name "Gas/Smoke Alarm detected#"
meshtastic --set detection_sensor.minimum_broadcast_secs 60
meshtastic --set detection_sensor.state_broadcast_secs 0
```

---

## Resultados do Ensaio e Validação

1. **Estado de Repouso (Silencioso):** Com o potenciómetro azul devidamente regulado para manter o LED indicador `DO` apagado em ar limpo, o sistema permanece totalmente silencioso, sem disparos falsos nem mensagens enviadas a cada minuto.
2. **Comportamento do Gatilho:** Ao introduzir gás (ou rodar o potenciómetro para o limiar de disparo), o LED do módulo acende e **exatamente um pacote encriptado** (`Gas/Smoke Alarm detected#`) é enfileirado e transmitido com sucesso (`txGood=2`).
3. **Receção em Outros Nós:** Os restantes nós da rede que importaram o canal encriptado via QR code / URL conseguiram desencriptar e exibir o alerta de segurança no ecrã OLED e no chat em tempo real.