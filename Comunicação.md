# Comunicação MQTT

A comunicação no projeto é feita via protocolo MQTT. O ESP32 publica os dados de temperatura e umidade nos tópicos `home/temperatura` e `home/umidade`. O estado do sistema é enviado para o tópico `home/status`. O controle da lâmpada é feito pelo tópico `home/lamp`.

## Broker MQTT
O broker MQTT utilizado é o **HiveMQ**, porem pode ser usado qualquer broker MQTT.

## Fluxo de Dados
1. O ESP32 envia os dados do sensore.
2. O Node-RED (ou qualquer outro software que use MQTT) se inscreve nos tópicos para receber as atualizações.
3. O controle da lâmpada também é feito através de comandos MQTT para ligar/desligar.
