# smart-voice-alert

Smart Voice Alert
Um laboratório self-hosted de monitoramento inteligente que combina IoT, IA local, automação e telefonia VoIP para detectar situações críticas e alertar humanos por ligação de voz automática — tudo offline e privado.
Objetivo principal
Sensor detecta algo (MQTT) → IA local decide se é crítico (Ollama) → se sim, Asterisk faz ligação automática para o humano via SIP (Linphone) e fala o alerta (gTTS).
Visão geral do fluxo

Sensor (simulado ou real) publica JSON no HiveMQ via MQTT (ex: {"temp":68,"local":"rack-3"})
n8n escuta topic sensors/#
Extrai mensagem → envia para Ollama analisar
IA responde "SIM: razão curta" ou "NÃO: razão curta"
Se "SIM" → gera áudio TTS (Google gTTS) com a razão
Salva MP3 em volume compartilhado
Asterisk origina ligação para ramal 1000 (Linphone)
Linphone toca → atende → ouve a voz falando o alerta

Tecnologias principais

MQTT Broker — HiveMQ CE
Automação & IA — n8n + Ollama (llama3.2:3b)
Telefonia VoIP — Asterisk (PJSIP) + Linphone (softphone)
Interface gráfica & debug — Ubuntu LXDE + VNC + SNGREP
TTS — Google TTS (gTTS via HTTP)
VPN (opcional) — WireGuard (para sensores remotos)
Containerização — Docker Compose

Estrutura do repositório
textsmart-voice-alert/
├── docker-compose.yml          # Todos os containers
├── asterisk_conf/              # Configs Asterisk (pjsip.conf, extensions.conf, etc.)
├── shared/                     # Volume compartilhado (áudios TTS)
├── sounds/                     # Sons Asterisk (tt-monkeys, etc.)
└── README.md
Como rodar (quick start)

Clone o repositórioBashgit clone https://github.com/SeuUsuario/smart-voice-alert.git
cd smart-voice-alert
Inicie tudoBashdocker compose up -d
Acesse as interfaces
VNC + Linphone: http://localhost:6080 (senha padrão ou vazia)
n8n: http://localhost:5678
HiveMQ dashboard: http://localhost:8080 (opcional)
Asterisk CLI: docker exec -it rcon-asterisk asterisk -rvvv

Teste rápidoBashdocker exec -it rcon-desktop mosquitto_pub -h rcon-hivemq -t sensors/temp -m '{"temp":68,"local":"rack-3"}'→ Verifique se Linphone toca e fala o alerta.

Configurações importantes

Asterisk
Ramal: 1000 (Linphone)
Context de alerta: [ai-alert] em extensions.conf
Playback do arquivo: /shared/alert.mp3

n8n Workflow
Trigger: MQTT subscribe sensors/#
IA: Ollama llama3.2:3b
TTS: gTTS (Google) via HTTP
Originate: ARI para PJSIP/1000 @ ai-alert

Ollama
Modelo recomendado: llama3.2:3b (leve e bom para decisões simples)

Testes recomendados

Temperatura alta (crítico)Bashmosquitto_pub -h rcon-hivemq -t sensors/temp -m '{"temp":68,"local":"rack-3"}'
Temperatura normal (não crítico)Bashmosquitto_pub -h rcon-hivemq -t sensors/temp -m '{"temp":28,"local":"sala"}'
Simulação de fumaçaBashmosquitto_pub -h rcon-hivemq -t sensors/fumaca -m '{"fumaca":true,"local":"cozinha"}'

Roadmap / Melhorias futuras

TTS offline (Piper ou Coqui)
Confirmação interativa (DTMF: pressione 1 para confirmar)
Dashboard de alertas (MQTT → Grafana ou app no celular)
Sensores reais (ESP32 com MQTT-SN)
Autenticação HiveMQ + VPN obrigatória
Gravação das ligações de alerta
Notificação fallback (Telegram/SMS se ligação falhar)

Contribuições
Sinta-se à vontade para abrir issues, pull requests ou forks. Qualquer melhoria em prompt da IA, voz TTS, segurança ou novos sensores é bem-vinda!
Licença
MIT License
Copyright (c) 2026 Lucas
