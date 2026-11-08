# Política de privacidade — ChatGPT Video Vision

**Última atualização:** 6 de novembro de 2026

Esta política descreve o comportamento da aplicação web e do servidor MCP nesta distribuição. O administrador que hospeda uma instância pode configurar retenções e provedores diferentes; informe-se sobre a configuração efetiva antes de enviar conteúdo sensível.

## Dados processados

- Vídeo enviado pelo navegador ou baixado de um link individual do YouTube.
- Quadros, áudio extraído, metadados técnicos, transcrição, eventos sonoros e resultados de análise.
- Credenciais de acesso da aplicação e tokens de sessão.
- Configurações operacionais e logs mínimos de falhas.

## Onde os dados ficam

Os vídeos e resultados ficam no armazenamento privado definido por `VIDEO_VISION_HOME`. Eles não são servidos como arquivos públicos. A rota do player exige sessão autenticada e suporta pedidos HTTP Range. No Docker Compose, o armazenamento fica no volume nomeado `video-vision-storage`.

O padrão configurado para limpeza é `RETENTION_HOURS=24`. A limpeza automática remove uploads, metadados, tarefas e resultados antigos. Cache e manifestos também têm as regras próprias do projeto original; revise `session_max_age_days` e `downloads_max_age_days` ao configurar o servidor. Faça backup somente se isso estiver previsto na sua política de dados.

## Envio para serviços externos

Os seguintes dados podem sair do servidor quando o usuário ativa o recurso correspondente e o administrador configura credenciais:

- OpenAI Vision: quadros selecionados do vídeo e instruções/timestamps são enviados à API OpenAI.
- OpenAI Transcription: arquivos de áudio extraídos ou segmentos são enviados à API OpenAI.
- OpenAI Audio Events: trechos de áudio são enviados ao modelo GPT-Audio, se habilitado.
- Gemini: o backend original pode enviar áudio à API Gemini para transcrição e classificação de eventos.
- YouTube: `yt-dlp` acessa o link fornecido e recebe dados do serviço YouTube.

O processamento local FFmpeg/FFprobe e o Whisper local não exigem envio à OpenAI/Gemini. A aplicação não possui telemetria de produto ou analytics configurados.

Os dados enviados a terceiros ficam sujeitos às políticas, termos, retenção e configurações de cada provedor. A política da OpenAI API informa que os dados não são usados para treinamento por padrão, mas registros de monitoramento de abuso podem ter retenção limitada. Não prometa eliminação imediata pelo provedor externo.

## Credenciais

As chaves de API e segredos devem ficar somente em variáveis de ambiente do servidor. Elas não são inseridas no bundle do navegador. Não envie `.env` a terceiros nem inclua credenciais em issues, logs ou ZIPs.

## Responsabilidades do administrador

O administrador deve configurar HTTPS, firewall, credenciais fortes, acesso ao volume e backups. A primeira versão utiliza uma conta administrativa compartilhada por instância (`APP_USERNAME`/`APP_PASSWORD`), não uma plataforma multiusuário com papéis e isolamento por conta. Não hospede conteúdo sensível sem uma avaliação de conformidade específica.

## Exclusão

A aplicação remove arquivos locais por retenção ou pelo procedimento administrativo de exclusão do volume. A exclusão local não apaga cópias eventualmente retidas por provedores externos.
