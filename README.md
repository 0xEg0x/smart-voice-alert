Smart Voice Alert
Um laboratório self-hosted de monitoramento inteligente que combina IoT, IA local, automação e telefonia VoIP para detectar situações críticas e alertar humanos por ligação de voz automática — tudo offline e privado.
Objetivo principal
Sensor detecta algo (MQTT) → IA local decide se é crítico (Ollama) → se sim, Asterisk faz ligação automática para o humano via SIP (Linphone) e fala o alerta (gTTS).
Visão Geral do Fluxo

Sensor (simulado ou real) publica JSON no HiveMQ via MQTT (ex: {"temp":68,"local":"rack-3"}).
n8n captura o dado, envia para Ollama analisar.
Ollama responde "SIM: razão curta" ou "NÃO: razão curta".
Se "SIM", n8n gera áudio TTS e salva em pasta compartilhada.
n8n usa ARI para Asterisk originar a chamada.
Linphone toca → atende → ouve a voz falando o alerta.

Tecnologias Principais

MQTT Broker: HiveMQ CE
Automação & IA: n8n + Ollama (modelo: llama3.2:3b)
VoIP: Asterisk (PJSIP) + Linphone (softphone)
TTS: gTTS (Google TTS)
Interface: Ubuntu LXDE + VNC + SNGREP
VPN (Opcional): WireGuard
Containerização: Docker Compose

Estrutura do Repositório
textsmart-voice-alert/
├── docker-compose.yml          # Todos os containers
├── asterisk_conf/              # Configs Asterisk (pjsip.conf, extensions.conf, etc.)
├── shared/                     # Volume compartilhado (áudios TTS)
├── sounds/en/                  # Sons Asterisk (tt-monkeys, etc.)
├── wireguard/                  # Configs WireGuard (opcional)
└── README.md
Requisitos

Docker e Docker Compose instalados (v2+)
Máquina com pelo menos 8 GB RAM (para Ollama 3B)
Conexão internet para pull inicial de imagens
Celular/PC para testar VPN (opcional)

Tutorial Passo a Passo de Execução
Siga esses passos para clonar, configurar e rodar o projeto 100%.
Passo 1: Clone o Repositório
Abra o terminal e clone o repositório:
Bashgit clone https://github.com/0xEg0x/smart-voice-alert.git
cd smart-voice-alert
Passo 2: Verifique o docker-compose.yml
Certifique-se que o arquivo tem todos os serviços (HiveMQ, n8n, Ollama, Asterisk, desktop). Se quiser VPN, adicione o bloco WireGuard.
Exemplo parcial:
YAMLservices:
  hivemq:
    image: hivemq/hivemq-ce:latest
    # ...  
  n8n:
    image: n8nio/n8n:latest
    # ...
  ollama:
    image: ollama/ollama:latest
    # ...
  asterisk:
    image: andrius/asterisk:latest
    # ...
  desktop:
    image: dorowu/ubuntu-desktop-lxde-vnc
    # ...
  # WireGuard (opcional)
  wireguard:
    image: linuxserver/wireguard:latest
    # ...
volumes:
  n8n_data:
  ollama_data:
  shared:
networks:
  rcon-net:
    driver: bridge
Passo 3: Crie Pastas Necessárias
Crie as pastas para volumes compartilhados e configs:
Bashmkdir -p asterisk_conf shared sounds/en wireguard

Copie configs de Asterisk (pjsip.conf, extensions.conf, etc.) para asterisk_conf/ se não estiverem lá (use os do repo).
Baixe sons para sounds/en/ (ex: tt-monkeys.ulaw do Asterisk downloads).

Passo 4: Inicie os Containers
Bashdocker compose up -d

Aguarde 5-10 min (baixa imagens e modelos Ollama).
Verifique status:textdocker psDeve mostrar todos os containers Up.

Passo 5: Configure Ollama (IA Local)
Entre no container e baixe o modelo:
Bashdocker exec -it rcon-ollama bash
ollama pull llama3.2:3b
exit
Teste:
Bashcurl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Teste rápido."
}'
Passo 6: Configure n8n (Automação)
Acesse http://localhost:5678 e crie o workflow principal:

Trigger: MQTT Subscribe (sensors/#)
Function: Extrai mensagem
HTTP Request: Ollama /api/chat
Function: Parse decisão
IF: Se "sim"
HTTP Request: gTTS TTS
Write Binary File: /shared/alert.mp3
HTTP Request: ARI originate para alert@ai-alert

Salve e ative.

Passo 7: Configure Asterisk (VoIP)

Recarregue configs:textdocker exec -it rcon-asterisk asterisk -rx "dialplan reload"
Teste chamada básica:
No Linphone (via VNC: http://localhost:6080), ligue para 999 → ouça tt-monkeys.

Passo 8: Configure VPN (Opcional, para sensores remotos)

Inicie WireGuard.
Pegue peer.conf de ./wireguard/peer1/peer1.conf.
Importe no app WireGuard do celular/PC.
Conecte → teste MQTT do remoto.

Passo 9: Execute e Teste

Simule sensor:textdocker exec -it rcon-desktop mosquitto_pub -h rcon-hivemq -t sensors/temp -m '{"temp":68,"local":"rack-3"}'
Linphone toca → atende → ouve voz com alerta.

Temp normal (não toca):textdocker exec -it rcon-desktop mosquitto_pub -h rcon-hivemq -t sensors/temp -m '{"temp":28,"local":"sala"}'

Passo 10: Troubleshooting

No áudio: Verifique PulseAudio (instale pulseaudio-utils no desktop).
Erro em n8n: Veja Executions.
Asterisk logs: docker logs rcon-asterisk.
VPN: Verifique logs WireGuard.

Use Cases

Monitoramento servidores: Temp alta → alerta voz.
Casa inteligente: Fumaça → ligação família.
Saúde: Movimento suspeito → chamada cuidador.

Roadmap

TTS offline
DTMF confirmação
Dashboard alertas
Sensores reais (ESP32)

Licença
MIT License
Copyright (c) 2026 Lucas
