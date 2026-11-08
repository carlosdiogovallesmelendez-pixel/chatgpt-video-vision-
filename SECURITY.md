# Política de segurança

## Estado do projeto

Esta é uma adaptação de código-fonte em desenvolvimento. A presença de controles de segurança não equivale a uma auditoria independente nem autoriza exposição pública sem configuração apropriada.

## Controles implementados na aplicação

- Autenticação administrativa com senha configurada pelo ambiente.
- Sessão assinada HMAC em cookie `HttpOnly`, `SameSite=Strict`, com `Secure` quando a conexão chega por HTTPS.
- Comparação de credenciais por comparação de hash em tempo constante.
- Limitação de tentativas de login por endereço IP.
- Validação de origem em requisições mutáveis.
- Uploads com identificadores aleatórios, nomes internos gerados, extensões permitidas, limites de arquivo e quota de armazenamento.
- Arquivos em diretório privado, com permissões restritas quando suportadas.
- Validação de caminho para servir/consultar arquivos.
- URLs YouTube limitadas aos domínios oficiais e a formas de URL de vídeo individual; argumentos são passados ao processo sem shell.
- Execução do container sob usuário não-root.
- Chaves de API no ambiente do servidor, nunca no JavaScript do navegador.
- Erros com padrões de chaves comuns redigidos antes de serem apresentados.
- Endpoint MCP remoto desativado sem token forte; sessões MCP têm limite e expiração.
- Fila web limitada a uma tarefa em processamento por instância.

## Requisitos antes de exposição pública

1. Use HTTPS e um reverse proxy confiável.
2. Troque os valores de exemplo por senhas e tokens aleatórios.
3. Restrinja o acesso de rede e proteja o volume persistente.
4. Monitore CPU, memória, disco, logs e limites de API.
5. Atualize dependências e imagens de container.
6. Faça avaliação de segurança do host e das políticas de retenção.
7. Não escale réplicas sem migrar fila e armazenamento para infraestrutura compartilhada.

## Relato de vulnerabilidade

Para a origem do projeto, consulte os canais de segurança do repositório original: https://github.com/jordanrendric/claude-video-vision/security. Para uma cópia derivada, o responsável pela distribuição deve fornecer seu próprio canal de contato seguro. Não publique chaves, vídeos privados, transcrições ou dados pessoais num relato público.

## Limites conhecidos

A primeira versão não implementa multi-tenancy, recuperação de senha, upload retomável, antivírus dedicado, sandbox de sistema operacional independente para FFmpeg ou criptografia ponta a ponta. O MCP remoto deve ser mantido desligado quando não for necessário. Uma revisão independente continua recomendada antes da hospedagem pública.
