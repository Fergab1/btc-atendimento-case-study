# ADR 0004: Arquivar anexos antigos no Cloudflare R2

- **Status:** aceita
- **Data:** setembro de 2026

## Contexto

No plano atual do Supabase, a cota de armazenamento é por organização e dividida com outro produto do mesmo cliente. Essa cota já tinha estourado uma vez, com uploads falhando. Com fotos, vídeos, áudios e documentos do WhatsApp passando a entrar no chat, o consumo de armazenamento do atendimento ia crescer.

## Decisão

Uma rotina semanal move para o Cloudflare R2 os anexos com mais de 7 dias (prazo configurável). A ordem garante que nenhum link do chat quebre:

1. O arquivo é copiado para o R2 e o tamanho é conferido antes de qualquer outro passo.
2. O link na mensagem é trocado para a cópia arquivada.
3. Só então a cópia original é apagada.

Sem a configuração do endereço público do R2, a rotina não move nada. O cliente de armazenamento compatível com S3 foi escrito na própria função, com a assinatura de requisições implementada sobre a Web Crypto e testada contra vetores de referência.

## Alternativas consideradas

- **Plano pago do Supabase.** Rejeitada: custo mensal recorrente para resolver algo que o arquivamento resolve.
- **Google Drive.** Tentada e descartada: uma política do Google bloqueou o uso pretendido.
- **Manter tudo no armazenamento principal.** Rejeitada: a cota voltaria a estourar com o volume de mídia do WhatsApp.

## Consequências

- O armazenamento principal guarda só os anexos recentes, e o projeto segue no plano atual.
- Os anexos do formulário público entram no mesmo arquivamento.
- Passa a existir um segundo provedor de armazenamento, com credenciais e acesso público próprios, configurados fora do código.
