# ADR 0001: Atendimento único por cliente, com sub-threads por departamento

- **Status:** aceita
- **Data:** junho de 2026

## Contexto

Cada nova interação do cliente criava um atendimento novo, e o mesmo cliente podia existir em mais de um cadastro (o mesmo telefone digitado duas vezes, por exemplo). O histórico ficava fragmentado: a equipe via vários atendimentos da mesma pessoa e precisava correlacionar à mão, um cliente que precisava de dois departamentos acabava com dois atendimentos sem contexto entre eles, e o volume de atendimentos inflava as métricas.

## Decisão

Cada cliente tem no máximo um atendimento ativo. Dentro dele, as mensagens ficam organizadas em uma conversa por departamento (sub-thread), cada uma com status e responsável próprios. Toda mensagem nova do cliente entra na conversa do departamento atual, e só quem não tem atendimento ativo passa pelo menu de departamentos e ganha um protocolo novo.

- O cliente é identificado por telefone ou por e-mail: qualquer um dos dois que coincida aponta para o mesmo cadastro.
- O status do atendimento passa a ser derivado das conversas por um gatilho no banco, e o cliente só abre um atendimento novo quando todas as conversas do atual estão encerradas.
- A migração rodou em produção sem janela de manutenção: unificação dos cadastros duplicados, criação das conversas iniciais para os atendimentos existentes e, só então, índices únicos de telefone e e-mail.

## Alternativas consideradas

- **Manter um atendimento por interação.** Rejeitada: é a causa da fragmentação e das métricas infladas.
- **Status único para o atendimento inteiro.** Rejeitada: um departamento encerrar a sua parte não pode encerrar a de outro que ainda está atendendo.
- **Unir automaticamente cadastros que batem em campos diferentes.** Rejeitada: telefone de um cadastro e e-mail de outro podem ser pessoas distintas. Nesse caso raro, o mais recente fica ativo e o outro é marcado para revisão do gestor.
- **Converter os atendimentos antigos retroativamente.** Rejeitada: o histórico antigo foi preservado como estava, e o modelo novo vale daqui para frente, com a unificação de cadastros feita antes.

## Consequências

- A equipe vê o histórico completo do cliente em um só lugar, e cada departamento trabalha na própria conversa.
- O webhook do WhatsApp, o envio de mensagens, as cadências automáticas e o formulário público passaram a gravar cada mensagem na conversa certa.
- Transferir de departamento passou a envolver duas informações que precisam andar juntas, o departamento do atendimento e a conversa ativa. Isso levou à [ADR 0002](0002-sincronizar-sub-thread-por-gatilho-no-banco.md).
