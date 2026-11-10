# ADR 0003: Segurar a mensagem do atendente fora da janela de 24 horas

- **Status:** aceita
- **Data:** outubro de 2026

## Contexto

O WhatsApp só permite que a empresa mande texto livre até 24 horas depois da última mensagem do cliente. Fora dessa janela, só um modelo de mensagem aprovado pela Meta pode reabrir a conversa. Quando o atendente respondia depois do prazo, a mensagem falhava e precisava ser reescrita, e o cliente que já estava em atendimento recebia o mesmo modelo usado para cliente novo.

## Decisão

O reengajamento tem duas partes:

1. **Modelo por setor.** Fora da janela, o cliente em atendimento recebe um modelo que cita o setor e o protocolo do atendimento. O caminho de cliente novo continua com o modelo próprio.
2. **Mensagem segurada.** A mensagem do atendente, com texto e anexo, fica guardada como aguardando reabertura. Quando o cliente responde (inclusive tocando em "Continuar atendimento"), o webhook entrega todas as mensagens seguradas, na ordem em que foram escritas, no lugar do aviso automático padrão.

A retenção usa os metadados da própria mensagem, sem tabela nova nem rotina agendada: a expiração de 24 horas é calculada a partir do momento em que a mensagem foi segurada, e o painel mostra "Aguardando resposta" ou "Reenvio expirado".

## Alternativas consideradas

- **Deixar a mensagem falhar.** Rejeitada: obriga o atendente a reescrever e a lembrar de voltar ao atendimento depois.
- **Tabela de mensagens pendentes com rotina de expiração.** Rejeitada: a mensagem já tem onde guardar o estado, e a expiração calculada na leitura dispensa um agendamento a mais.

## Consequências

- O atendente escreve uma vez, e o cliente recebe o texto real do atendente, e não um modelo genérico.
- A mensagem reenviada mostra a hora em que foi escrita e a hora do reenvio, e acompanha o status real do WhatsApp (falha com motivo, ou lida).
- Os testes do QA mostraram dois casos de borda, tratados em seguida: o toque em "Continuar atendimento" chegando por outro atendimento do mesmo telefone (a busca das mensagens seguradas passou a considerar todos os atendimentos abertos do número) e um segundo modelo enviado enquanto o primeiro ainda aguardava resposta (passou a existir a regra de modelo pendente).
