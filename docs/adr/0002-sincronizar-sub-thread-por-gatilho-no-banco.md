# ADR 0002: Sincronizar a sub-thread por gatilho no banco

- **Status:** aceita
- **Data:** setembro de 2026

## Contexto

O atendimento guarda o departamento em dois lugares: o campo do atendimento, que o painel mostra e que a ação de transferir altera, e a conversa ativa do departamento (sub-thread), que o webhook do WhatsApp consulta para gravar a mensagem nova, montar o aviso de retorno e decidir qual equipe notificar. O fluxo de transferência do painel é anterior ao modelo de sub-threads e atualizava só o campo do atendimento. Depois de uma transferência, o painel mostrava o departamento novo enquanto o WhatsApp seguia o antigo.

Mais de um caminho do painel muda o departamento (transferir, devolver e enviar ao jurídico), e o painel é evoluído em paralelo pela equipe do cliente.

## Decisão

Um gatilho no banco age em toda mudança de departamento de um atendimento: cria a conversa do destino (ou reativa, se o atendimento já passou por lá), marca-a como a mais recente e encerra a de origem. Um ajuste único corrigiu os atendimentos abertos que já estavam divergentes, e a página do atendimento passou a selecionar a conversa nova logo após a transferência.

## Alternativas consideradas

- **Ajustar cada fluxo do painel que muda o departamento.** Rejeitada: a regra teria de ser repetida em cada caminho, e qualquer fluxo novo criado no painel poderia esquecê-la.

## Consequências

- A regra vale para qualquer caminho que mude o departamento, atual ou futuro, e entrou em produção sem depender da publicação do painel.
- Parte da regra de negócio fica no banco. Para isso, criei uma infraestrutura de testes de banco com PostgreSQL descartável em Docker: os cenários de transferência, devolução, fluxo do jurídico, atendimento sem troca, atendimento antigo e ajuste único foram escritos antes da correção e vistos falhar.
- A mesma infraestrutura passou a cobrir o aviso à equipe do destino em transferências sem responsável e o cálculo de atraso de resposta.
