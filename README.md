# ChatGPT Video Vision

Aplicação web e servidor MCP para análise de vídeos. O projeto preserva o núcleo de processamento do repositório **Claude Video Vision**, de Jordan Vasconcelos, incluindo as seis ferramentas MCP, FFmpeg/FFprobe, cache, manifestos, análise estrutural, YouTube e os backends originais de áudio. A adaptação adiciona uma interface web responsiva, transporte MCP remoto opcional e adaptadores OpenAI separados.

> **Estado desta distribuição:** consulte `docs/RELATORIO-FASE-4.md`. A integração com o SDK MCP oficial está implementada no código, mas o build e os testes de protocolo ainda não foram executados nesta sessão porque o ambiente não conseguiu instalar Node.js 24 e as dependências npm. Não há chaves reais incluídas.

## O que está incluído

- As ferramentas originais `video_watch`, `video_analyze`, `video_detail`, `video_info`, `video_configure` e `video_setup`.
- Extração de quadros/áudio, metadados, filtros estruturais FFmpeg, cache e manifestos.
- Backend Gemini mantido para transcrição e eventos sonoros; Whisper local mantido.
- Adaptador de visão OpenAI via Responses API (`OPENAI_API_KEY`).
- Adaptador de transcrição OpenAI com `gpt-transcribe`; `whisper-1` está disponível somente como rota temporária compatível para timestamps medidos.
- Adaptador experimental de eventos sonoros via `gpt-audio-1.5`, separado do backend de transcrição e desativado por padrão.
- Interface Android/Chrome baseada em PWA, sem dependências de desenvolvimento no celular.
- Login, upload privado, fila de processamento, consulta de resultados e extração de trechos.
- MCP local via STDIO e endpoint remoto opcional `/mcp` com token Bearer.

## Arquitetura

O navegador envia o vídeo ao backend. O backend mantém os arquivos e inicia o servidor MCP original por STDIO. A interface chama as mesmas seis ferramentas MCP — não há uma segunda implementação simplificada dos seus contratos. O FFmpeg/FFprobe executa no servidor. A API OpenAI só é chamada quando há chave no ambiente e a opção correspondente foi ativada.

## Requisitos de execução

- Node.js **24 ou superior**.
- FFmpeg e FFprobe no `PATH`.
- yt-dlp opcional para URLs do YouTube.
- npm com acesso ao registro para instalar as dependências.
- Para recursos OpenAI: uma chave de API com cobrança habilitada separadamente da assinatura ChatGPT.
- Para a interface web: credenciais de aplicação e um segredo de sessão configurados no servidor.

O Moto G04 utiliza apenas Chrome/navegador Android e conexão à internet. Node.js, Python, FFmpeg, FFprobe, Whisper, yt-dlp e chaves de API não são instalados no celular.

## Execução local

1. Instale Node.js 24+ e FFmpeg/FFprobe no computador/servidor que hospedará o processamento.
2. Copie `.env.example` para `.env` e defina `APP_USERNAME`, `APP_PASSWORD` (12+ caracteres) e `SESSION_SECRET` (32+ caracteres). Defina as chaves de API somente se desejar usar provedores externos.
3. Instale e compile o servidor MCP:

   ```bash
   npm ci --prefix mcp-server
   npm --prefix mcp-server run build
   ```

4. Configure as variáveis de ambiente do `.env` no processo do servidor. O Node não carrega `.env` automaticamente; em shells POSIX pode-se usar `set -a; . ./.env; set +a`.
5. Inicie a aplicação:

   ```bash
   node web/server.mjs
   ```

6. Abra `http://localhost:8080` no navegador. Para acesso remoto, use HTTPS em um proxy confiável e não exponha o serviço sem autenticação.

## Docker

```bash
cp .env.example .env
# Edite .env e substitua todos os valores CHANGE_ME.
docker compose up --build -d
```

O Dockerfile usa Node.js 24, instala FFmpeg/FFprobe e yt-dlp, compila o MCP e inicia a interface. O volume nomeado `video-vision-storage` mantém os arquivos entre reinicializações. A publicação pública não é configurada pelo projeto: configure TLS, domínio, backups e firewall no serviço que você escolher.

## Configuração de provedores

### OpenAI

- `OPENAI_API_KEY`: chave somente no servidor.
- `OPENAI_TRANSCRIPTION_MODEL=gpt-transcribe`: padrão de transcrição; não fornece granularidade medida de timestamps na documentação atual.
- `OPENAI_TRANSCRIPTION_MODEL=whisper-1`: compatibilidade legada para `verbose_json`/segmentos enquanto o modelo estiver disponível. A saída tem timestamps medidos, mas o modelo possui encerramento anunciado; não dependa dele a longo prazo.
- `OPENAI_AUDIO_EVENTS_MODEL=gpt-audio-1.5`: classificador experimental de sons não verbais. É opcional e pode falhar por limites de entrada ou resposta não estruturada.
- O modelo visual padrão é `gpt-6.1-terra`; pode ser alterado pela configuração `vision_model`.

### Gemini e Whisper local

- `GEMINI_API_KEY`: preserva o backend original Gemini, inclusive transcrição e `audio_tags`.
- Whisper local continua dependendo dos programas/ambiente indicados pelo `video_setup`.
- Sem credenciais, o sistema ainda pode executar metadados, extração de quadros e filtros FFmpeg; recursos sem backend são omitidos com avisos.

A API OpenAI e a assinatura ChatGPT são faturadas separadamente. O projeto não habilita cobrança e não cria credenciais.

## Ferramentas MCP

| Ferramenta | Comportamento preservado |
|---|---|
| `video_watch` | Extração de quadros, áudio, segmentos e cache; opção `visual_analysis` chama OpenAI Vision |
| `video_analyze` | Filtros estruturais FFmpeg e transcrição opcional |
| `video_detail` | Extração/consulta de trechos, cache e opção de análise visual |
| `video_info` | Metadados FFprobe; para YouTube, tenta ler metadados via yt-dlp sem baixar o vídeo inteiro |
| `video_configure` | Atualiza preferências persistidas |
| `video_setup` | Verifica dependências do ambiente servidor |

Os parâmetros originais foram mantidos; novos parâmetros são opcionais. O `frame_describer_model` permanece por compatibilidade com configuração antiga, mas não controla o adaptador OpenAI.

## Segurança e retenção

- A interface exige autenticação; `APP_PASSWORD` deve ser alterada.
- Cookies HttpOnly, SameSite=Strict e Secure quando o acesso chega por HTTPS.
- Uploads com extensão permitida, nome interno aleatório, limite por arquivo e quota de armazenamento.
- Um worker executa uma tarefa por vez; a fila tem limite configurável.
- Arquivos ficam fora da pasta pública; logs não registram vídeo ou chaves.
- `/mcp` fica desabilitado sem `MCP_AUTH_TOKEN` com pelo menos 24 caracteres.
- A limpeza padrão remove os arquivos internos após `RETENTION_HOURS` (24 horas por padrão). Faça backups e revise essa política antes de hospedar.
- Não há criptografia ponta a ponta implementada pelo aplicativo. Use HTTPS, volume/disco criptografado e controles de acesso do provedor.
- Dados enviados a serviços externos ficam sujeitos às políticas e retenção desses provedores.

## Testes

Com Node.js 24+ e dependências instaladas:

```bash
npm ci --prefix mcp-server
npm test
```

`npm test` compila o servidor, executa Vitest, executa `tools/list` e `tools/call` das seis ferramentas por meio do SDK MCP oficial em STDIO e roda a integração web. A integração cria um vídeo audiovisual local, faz upload, aguarda o worker, consulta um timestamp e testa sessões Streamable HTTP oficiais. Consulte `docs/TESTES-MCP-OFICIAL.md`.

Nesta distribuição, `node --test web/tests/*.test.mjs` foi executado em Node 22.16.0: quatro testes web passaram e o teste de integração MCP oficial foi corretamente marcado como skipped porque Node 24, o build e o SDK não estavam disponíveis. Não confunda esse resultado com aprovação do MCP oficial. Nenhuma API externa foi chamada.

## Materiais originais e licença

Os arquivos específicos do Claude Code foram preservados em `legacy/claude-code/`. A licença MIT original e os créditos de Jordan Vasconcelos permanecem em `LICENSE` e `docs/ATRIBUICAO.md`. A adaptação não é um produto oficial da OpenAI.
