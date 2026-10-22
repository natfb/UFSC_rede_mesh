## Heltec
Heltec WiFi LoRa 32 V3 ESP32-S3

## Lilygon T-BeamV1.1
https://wiki.lilygo.cc/products/t-beam-series/t-beam/#pin-diagram

pins to display
vcc -> 3v
gnd -> gnd
SCL -> 22
SDA -> 21

### Range Test ausente no firmware do T-Beam — diagnóstico de 22/10/2026

O T-Beam V1.1 `nl01` está com Meshtastic **2.7.26.54e0d8d**. Nessa versão, a compilação oficial para ESP32 usada pelo T-Beam exclui o módulo de teste de distância com `-DMESHTASTIC_EXCLUDE_RANGETEST=1`.

O CLI ainda aceita e salva `range_test.enabled true` e `range_test.sender 60`, e a leitura posterior mostra esses valores. **Isso não significa que o módulo esteja disponível:** o T-Beam não gera automaticamente os pacotes `seq 1`, `seq 2`, etc. Reiniciar e desativar/reativar a configuração não resolveu.

No diagnóstico, uma mensagem manual enviada pelo T-Beam chegou ao Heltec diretamente por LoRa, sem MQTT. Portanto, o enlace funcionou; a ausência dos pacotes automáticos foi associada à exclusão do módulo no firmware.

O Heltec V3 `nh02` usa ESP32-S3 e sua compilação nessa mesma versão inclui o módulo. Ele já está configurado como receptor (`enabled=true`, `sender=0`, `save=true`). A gravação do CSV dos testes ainda não foi validada, pois o T-Beam não está gerando esses pacotes.

Para usar o T-Beam como emissor autônomo, será necessário instalar uma compilação que inclua Range Test, verificando a compatibilidade antes de gravar. Nenhuma troca de firmware foi feita neste diagnóstico. Essa limitação é da compilação instalada, não do hardware T-Beam em si.

Fontes da versão examinada:

- [Opções de compilação ESP32: exclusão de Range Test](https://github.com/meshtastic/firmware/blob/v2.7.26.54e0d8d/variants/esp32/esp32.ini).
- [Compilação Heltec V3](https://github.com/meshtastic/firmware/blob/v2.7.26.54e0d8d/variants/esp32s3/heltec_v3/platformio.ini) e [base ESP32-S3](https://github.com/meshtastic/firmware/blob/v2.7.26.54e0d8d/variants/esp32s3/esp32s3.ini).


## como ligar lyligo
Método 2: Puxar o pino IO0 ao GND (Garantido)Se a combinação de botões não funcionar, a barra de pinos (headers) da sua placa expõe o pino IO0 (ou 0). O modo de gravação é ativado quando esse pino está em nível lógico baixo ($GND$):1.Fazer um jumper temporário do IO0 ao GND:Com a placa plugada.Localize os pinos IO0 (ou apenas 0) e qualquer pino GND na barra de pinos lateral da LilyGO.Use um fio jumper, clipe de papel metálico ou uma pinça para encostar e fechar contato entre o IO0 e o GND.2.Dar um pulso de Reset:Reset do SoC.Mantendo o jumper entre IO0 e GND firme, pressione e solte o botão RST uma única vez.

video lilygo tbeam with mesh https://www.youtube.com/watch?v=OoxD5pSbibk


# Traking dos Hardwares
#nh01 veryLongSlow   vermelho     recebendo pings(modulo de distancia)
#nh03 veryLongSlow   verde
#nh02 veryLongSlow    azul      router_client
#nl01                              T-Beam: envio Range Test indisponível no firmware 2.7.26.54e0d8d (ver diagnóstico acima)
#nl02               sem display



# Sensores
- MQ-2: detecta a presença de gases inflamáveis e fumaça no ar
- HTU21D: umidade relativa do ar e a temperatura
