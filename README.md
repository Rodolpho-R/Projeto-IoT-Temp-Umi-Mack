# Projeto-IoT-Temp-Umi-Mack
Medidor de temperatura e umidade

# Sistema de Monitoramento de Temperatura e Umidade

Este projeto utiliza um ESP32 e sensores DHT22 para monitorar a temperatura e a umidade de um ambiente. Os dados são enviados via protocolo MQTT para um servidor de nuvem, onde são exibidos em uma interface gráfica. O sistema também controla um relé para ligar ou desligar um dispositivo com base nos parâmetros de temperatura e umidade.

# Como usar
1. Configure o Wi-Fi e os parâmetros MQTT no código.
2. Carregue o código no ESP32.
3. O sistema irá monitorar a temperatura e umidade e enviar os dados para o broker MQTT.
4. O relé será acionado caso os parâmetros estejam fora da faixa definida.
