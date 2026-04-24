# Plano de Implementação: TechStore SupportBot

## Visão Geral

Implementação incremental seguindo Clean Architecture em Python 3.11+. As tarefas progridem da camada de domínio (entidades, interfaces, use case) para a infraestrutura (repositórios em memória) e, por fim, para a API (FastAPI). Cada etapa é integrada à anterior antes de avançar.

## Tarefas

- [ ] 1. Configurar estrutura do projeto e dependências
  - Criar estrutura de diretórios: `src/domain/`, `src/api/`, `src/infrastructure/`, `tests/domain/`, `tests/infrastructure/`, `tests/api/`
  - Criar arquivos `__init__.py` em todos os pacotes
  - Criar `pyproject.toml` (ou `requirements.txt`) com dependências: `fastapi`, `uvicorn`, `pydantic`, `hypothesis`, `pytest`, `httpx`
  - _Requisitos: 6.1_

- [ ] 2. Implementar entidades e value objects do domínio
  - [ ] 2.1 Criar `src/domain/entities.py` com `Intent` (Enum), `FAQ` (dataclass frozen), `Ticket` (dataclass), `Email` (value object com validação), `ConversationState` (dataclass), `HandlerResult` (dataclass)
    - `Email.__post_init__` valida formato com regex `[^@]+@[^@]+\.[^@]+`
    - `Ticket` recebe `id: str`, `customer_name`, `customer_email`, `original_question`, `created_at: datetime`
    - `ConversationState.step` é `Literal["awaiting_name", "awaiting_email", "awaiting_question"]`
    - _Requisitos: 1.1, 3.1, 3.3, 4.2, 4.5_

  - [ ]* 2.2 Escrever testes unitários para `Email` value object
    - Testar formatos válidos (ex.: `user@example.com`) e inválidos (ex.: `nao-e-email`, `@sem-usuario.com`)
    - _Requisitos: 3.3_

- [ ] 3. Implementar interfaces (ports) do domínio
  - Criar `src/domain/ports.py` com `IFAQRepository` (ABC: `find_match`, `get_all`) e `ITicketRepository` (ABC: `save`)
  - Criar `src/domain/exceptions.py` com `TicketPersistenceError`
  - _Requisitos: 2.2, 4.1, 4.4_

- [ ] 4. Implementar `IntentClassifier`
  - [ ] 4.1 Criar `src/domain/intent_classifier.py` com `IntentClassifier.classify(message: str, faqs: list[FAQ]) -> Intent`
    - Lógica: verifica se alguma keyword de qualquer FAQ está contida na mensagem (case-insensitive); retorna `FAQ_MATCH` se sim, `UNKNOWN` caso contrário
    - _Requisitos: 1.1, 1.4_

  - [ ]* 4.2 Escrever teste de propriedade P1 — classificador sempre retorna Intent válida
    - **Propriedade 1: O classificador sempre retorna uma Intent válida**
    - Usar `@given(st.text(min_size=1))` com lista de FAQs fixas
    - Resultado deve estar em `{Intent.FAQ_MATCH, Intent.UNKNOWN}`
    - **Valida: Requisito 1.1**

  - [ ]* 4.3 Escrever testes unitários para `IntentClassifier`
    - Mensagem com keyword de FAQ → `FAQ_MATCH`; mensagem sem match → `UNKNOWN`; mensagem vazia (tratada pelo handler)
    - _Requisitos: 1.1, 1.4_

- [ ] 5. Implementar `MessageHandler` (use case principal)
  - [ ] 5.1 Criar `src/domain/message_handler.py` com `MessageHandler.handle(text: str, state: ConversationState | None) -> HandlerResult`
    - Fluxo: valida texto não-vazio → classifica intent → se `FAQ_MATCH` busca FAQ → se encontrada retorna resposta; se não encontrada reclassifica como `UNKNOWN`
    - Fluxo `UNKNOWN`: inicia coleta (`awaiting_name`) ou avança estado conforme `state.step`
    - Validações de coleta: nome não-vazio, e-mail via `Email` value object, pergunta não-vazia
    - Ao completar coleta, chama `ticket_repo.save()`; captura `TicketPersistenceError` e retorna erro em pt-BR
    - Todas as mensagens ao cliente em pt-BR
    - _Requisitos: 1.2, 1.3, 1.5, 2.5, 3.1–3.5, 4.1, 4.3, 4.4_

  - [ ]* 5.2 Escrever teste de propriedade P2 — FAQ match retorna resposta correta
    - **Propriedade 2: FAQ match retorna a resposta correspondente**
    - Gerar FAQs e mensagens contendo keyword da FAQ; verificar `result.response_text == faq.answer`
    - **Valida: Requisitos 1.2, 2.2**

  - [ ]* 5.3 Escrever teste de propriedade P3 — intent desconhecida inicia coleta
    - **Propriedade 3: Intent desconhecida inicia o fluxo de coleta**
    - Mensagens sem match → `result.new_state.step == "awaiting_name"`
    - **Valida: Requisitos 1.3, 2.5**

  - [ ]* 5.4 Escrever teste de propriedade P4 — mensagens em branco são rejeitadas
    - **Propriedade 4: Mensagens em branco são rejeitadas**
    - Usar `st.text(alphabet=st.characters(whitelist_categories=("Zs",)))` + string vazia
    - Verificar `result.new_state is None` e mensagem de erro em pt-BR
    - **Valida: Requisito 1.5**

  - [ ]* 5.5 Escrever teste de propriedade P5 — dados incompletos nunca criam Ticket
    - **Propriedade 5: Dados incompletos ou inválidos nunca criam um Ticket**
    - Gerar `ConversationState` com pelo menos um campo inválido/vazio; verificar que `ticket_repo.save()` não é chamado
    - **Valida: Requisitos 3.2, 3.3, 3.4, 3.5**

  - [ ]* 5.6 Escrever teste de propriedade P6 — dados válidos sempre criam Ticket completo
    - **Propriedade 6: Dados válidos sempre criam um Ticket com todos os campos**
    - Gerar `(name, email, question)` válidos; verificar `ticket.customer_name`, `ticket.customer_email`, `ticket.original_question`
    - **Valida: Requisitos 4.1, 4.2**

  - [ ]* 5.7 Escrever testes unitários para `MessageHandler`
    - Cada passo do fluxo de coleta com inputs válidos e inválidos; fallback de `faq_match` sem resultado; falha do `ITicketRepository` (mock que lança `TicketPersistenceError`)
    - _Requisitos: 1.2, 1.3, 1.5, 2.5, 3.1–3.5, 4.3, 4.4_

- [ ] 6. Checkpoint — Verificar domínio
  - Garantir que todos os testes do domínio passam. Perguntar ao usuário se houver dúvidas antes de avançar.

- [ ] 7. Implementar serialização de `FAQ` e `Ticket`
  - [ ] 7.1 Criar `src/domain/serialization.py` com funções `serialize_faq(faq: FAQ) -> dict`, `deserialize_faq(data: dict) -> FAQ`, `serialize_ticket(ticket: Ticket) -> dict`, `deserialize_ticket(data: dict) -> Ticket`
    - Usar `dataclasses.asdict()` para serialização
    - Desserializadores devem levantar `ValueError` com mensagem descritiva para campos ausentes ou de tipo incorreto
    - _Requisitos: 5.1, 5.2, 5.5_

  - [ ]* 7.2 Escrever teste de propriedade P8 — round-trip de serialização de FAQ
    - **Propriedade 8: Round-trip de serialização de FAQ**
    - Gerar objetos `FAQ` válidos; verificar `deserialize_faq(serialize_faq(faq)) == faq`
    - **Valida: Requisitos 5.1, 5.3**

  - [ ]* 7.3 Escrever teste de propriedade P9 — round-trip de serialização de Ticket
    - **Propriedade 9: Round-trip de serialização de Ticket**
    - Gerar objetos `Ticket` válidos; verificar `deserialize_ticket(serialize_ticket(ticket)) == ticket`
    - **Valida: Requisitos 5.2, 5.4**

  - [ ]* 7.4 Escrever teste de propriedade P10 — dados corrompidos geram erro descritivo
    - **Propriedade 10: Dados corrompidos geram erro descritivo**
    - Gerar dicts com campos ausentes ou de tipo incorreto; verificar que `ValueError` é levantado com mensagem não-vazia
    - **Valida: Requisito 5.5**

- [ ] 8. Implementar repositórios de infraestrutura
  - [ ] 8.1 Criar `src/infrastructure/faq_data.py` com lista de FAQs iniciais em pt-BR cobrindo os três tópicos obrigatórios: prazos de entrega, trocas/devoluções e formas de pagamento
    - Cada FAQ deve ter `id`, `keywords`, `question` e `answer` em pt-BR
    - _Requisitos: 2.1, 2.3, 2.4_

  - [ ] 8.2 Criar `src/infrastructure/faq_repository.py` com `InMemoryFAQRepository` implementando `IFAQRepository`
    - `find_match`: itera sobre FAQs e retorna a primeira cujas keywords aparecem na mensagem (case-insensitive); retorna `None` se não encontrar
    - `get_all`: retorna lista completa de FAQs
    - _Requisitos: 2.1, 2.2_

  - [ ] 8.3 Criar `src/infrastructure/ticket_repository.py` com `InMemoryTicketRepository` implementando `ITicketRepository`
    - `save`: gera UUID v4 como `id`, define `created_at = datetime.utcnow()`, armazena em dict interno, retorna `Ticket` persistido
    - Captura erros internos e relança como `TicketPersistenceError`
    - _Requisitos: 4.2, 4.5_

  - [ ]* 8.4 Escrever teste de propriedade P7 — IDs de Ticket são únicos
    - **Propriedade 7: IDs de Ticket são únicos**
    - Criar N ≥ 2 tickets via `InMemoryTicketRepository`; verificar que todos os IDs são distintos
    - **Valida: Requisito 4.5**

  - [ ]* 8.5 Escrever testes de integração para repositórios
    - `InMemoryFAQRepository`: verifica FAQs dos três tópicos, busca por keyword
    - `InMemoryTicketRepository`: save e retrieve, unicidade de IDs
    - _Requisitos: 2.1, 4.2, 4.5_

- [ ] 9. Implementar camada de API (FastAPI)
  - [ ] 9.1 Criar `src/api/schemas.py` com Pydantic models `MessageRequest` (campo `text: str`) e `MessageResponse` (campo `response: str`)
    - _Requisitos: 6.1, 6.4_

  - [ ] 9.2 Criar `src/api/dependencies.py` com função de wiring que instancia `InMemoryFAQRepository`, `InMemoryTicketRepository`, `IntentClassifier` e `MessageHandler` via `fastapi.Depends`
    - _Requisitos: 6.1_

  - [ ] 9.3 Criar `src/api/handlers/message_handler.py` com rota `POST /message`
    - Valida corpo da requisição (Pydantic levanta 422; tratar como 400 com JSON `{"error": "..."}`)
    - Chama `MessageHandler.handle(text, state=None)`
    - Retorna HTTP 200 com `{"response": "..."}` para requisições válidas
    - Retorna HTTP 400 com `{"error": "..."}` para corpo ausente/malformado ou `text` vazio
    - _Requisitos: 6.1, 6.2, 6.3, 6.4_

  - [ ] 9.4 Criar `src/api/app.py` com instância FastAPI e registro da rota `/message`
    - _Requisitos: 6.1_

  - [ ]* 9.5 Escrever teste de propriedade P11 — requisições válidas retornam HTTP 200
    - **Propriedade 11: Requisições válidas retornam HTTP 200**
    - Usar `FastAPI TestClient` + `httpx`; gerar `text` não-vazio; verificar status 200 e campo `"response"` no JSON
    - **Valida: Requisitos 6.1, 6.2**

  - [ ]* 9.6 Escrever teste de propriedade P12 — requisições malformadas retornam HTTP 400
    - **Propriedade 12: Requisições malformadas retornam HTTP 400**
    - Gerar corpos ausentes, não-JSON ou sem campo `"text"`; verificar status 400 e campo `"error"` no JSON
    - **Valida: Requisito 6.3**

  - [ ]* 9.7 Escrever testes de integração para o endpoint
    - Smoke test: POST com mensagem válida → 200; POST sem corpo → 400
    - _Requisitos: 6.1, 6.2, 6.3_

- [ ] 10. Checkpoint final — Garantir que todos os testes passam
  - Executar `pytest` e verificar que todos os testes (unitários, de propriedade e de integração) passam. Perguntar ao usuário se houver dúvidas.

## Notas

- Tarefas marcadas com `*` são opcionais e podem ser puladas para um MVP mais rápido
- Cada tarefa referencia requisitos específicos para rastreabilidade
- Os testes de propriedade usam **Hypothesis** com `@settings(max_examples=100)`
- Cada teste de propriedade deve ser anotado com `# Feature: techstore-supportbot, Property N: <texto>`
- O estado de conversa (`ConversationState`) é gerenciado pelo cliente da API entre requisições (stateless no servidor)
