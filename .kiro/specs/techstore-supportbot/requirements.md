# Documento de Requisitos

## Introdução

O TechStore SupportBot é um chatbot de suporte ao cliente para a plataforma de e-commerce TechStore. O bot responde perguntas frequentes (FAQs) sobre prazos de entrega, trocas/devoluções e formas de pagamento. Quando não consegue responder, coleta nome, e-mail e pergunta do cliente para abrir um ticket de suporte. Todas as respostas ao cliente devem ser em português (pt-BR).

## Glossário

- **SupportBot**: O sistema de chatbot de suporte ao cliente da TechStore.
- **FAQ**: Par pergunta-resposta predefinido cobrindo tópicos de suporte conhecidos.
- **Ticket**: Solicitação de suporte criada quando o bot não consegue resolver a dúvida do cliente; contém nome, e-mail e a pergunta original.
- **Intent**: Classificação de uma mensagem do cliente (ex.: correspondência com FAQ ou desconhecida).
- **Intent_Classifier**: Componente responsável por classificar a mensagem do cliente em uma Intent.
- **FAQ_Repository**: Componente responsável por armazenar e recuperar FAQs.
- **Ticket_Repository**: Componente responsável por persistir Tickets de suporte.
- **Message_Handler**: Componente de domínio que orquestra o fluxo de atendimento de uma mensagem.
- **Customer**: Usuário final que interage com o SupportBot.

---

## Requisitos

### Requisito 1: Classificação de Intenção da Mensagem

**User Story:** Como cliente, quero que o bot entenda minha pergunta, para que eu receba uma resposta relevante ou seja encaminhado ao suporte humano.

#### Critérios de Aceitação

1. WHEN uma mensagem do Customer é recebida, THE Intent_Classifier SHALL classificar a mensagem em uma Intent com um dos dois valores: `faq_match` ou `unknown`.
2. WHEN a Intent resultante é `faq_match`, THE Message_Handler SHALL retornar a resposta da FAQ correspondente ao Customer.
3. WHEN a Intent resultante é `unknown`, THE Message_Handler SHALL iniciar o fluxo de coleta de dados para abertura de Ticket.
4. THE Intent_Classifier SHALL sempre tentar encontrar uma correspondência com uma FAQ antes de classificar a Intent como `unknown`.
5. IF a mensagem do Customer estiver vazia, THEN THE Message_Handler SHALL retornar uma mensagem de erro solicitando que o Customer envie uma pergunta válida.

---

### Requisito 2: Consulta e Resposta de FAQs

**User Story:** Como cliente, quero receber respostas sobre entrega, trocas/devoluções e formas de pagamento, para que eu resolva minhas dúvidas sem precisar falar com um atendente.

#### Critérios de Aceitação

1. THE FAQ_Repository SHALL armazenar FAQs cobrindo os tópicos: prazos de entrega, trocas e devoluções, e formas de pagamento.
2. WHEN a Intent é `faq_match`, THE FAQ_Repository SHALL retornar a resposta associada à FAQ correspondente.
3. THE SupportBot SHALL retornar respostas ao Customer exclusivamente em português (pt-BR).
4. THE SupportBot SHALL retornar respostas concisas e sem jargão técnico nas mensagens voltadas ao Customer.
5. IF nenhuma FAQ correspondente for encontrada no FAQ_Repository para uma Intent classificada como `faq_match`, THEN THE Message_Handler SHALL reclassificar a Intent como `unknown` e iniciar o fluxo de coleta de dados para Ticket.

---

### Requisito 3: Coleta de Dados para Abertura de Ticket

**User Story:** Como cliente, quero poder registrar minha dúvida quando o bot não souber responder, para que um atendente humano entre em contato comigo.

#### Critérios de Aceitação

1. WHEN a Intent é `unknown`, THE Message_Handler SHALL solicitar ao Customer seu nome completo, endereço de e-mail e a pergunta original.
2. THE Message_Handler SHALL coletar os três campos — nome, e-mail e pergunta — antes de criar um Ticket.
3. IF o Customer fornecer um endereço de e-mail em formato inválido, THEN THE Message_Handler SHALL solicitar novamente um e-mail válido sem criar o Ticket.
4. IF o Customer fornecer um nome vazio, THEN THE Message_Handler SHALL solicitar novamente o nome sem criar o Ticket.
5. IF a pergunta original do Customer estiver vazia, THEN THE Message_Handler SHALL solicitar novamente a pergunta sem criar o Ticket.

---

### Requisito 4: Criação de Ticket de Suporte

**User Story:** Como atendente de suporte, quero receber tickets com nome, e-mail e pergunta do cliente, para que eu possa responder de forma eficiente.

#### Critérios de Aceitação

1. WHEN todos os três campos (nome, e-mail e pergunta) forem coletados e validados, THE Message_Handler SHALL criar um Ticket via Ticket_Repository.
2. THE Ticket_Repository SHALL persistir o Ticket contendo: nome do Customer, e-mail do Customer e a pergunta original.
3. WHEN o Ticket for criado com sucesso, THE Message_Handler SHALL retornar uma mensagem de confirmação ao Customer em português (pt-BR) informando que o ticket foi aberto.
4. IF o Ticket_Repository falhar ao persistir o Ticket, THEN THE Message_Handler SHALL retornar uma mensagem de erro ao Customer em português (pt-BR) e não confirmar a abertura do ticket.
5. THE Ticket_Repository SHALL atribuir um identificador único a cada Ticket no momento da criação.

---

### Requisito 5: Serialização e Persistência de Dados

**User Story:** Como desenvolvedor, quero que os dados de FAQ e Ticket sejam serializados e desserializados de forma confiável, para que não haja perda ou corrupção de informações.

#### Critérios de Aceitação

1. THE FAQ_Repository SHALL serializar e desserializar objetos FAQ de e para o formato de armazenamento definido.
2. THE Ticket_Repository SHALL serializar e desserializar objetos Ticket de e para o formato de armazenamento definido.
3. FOR ALL objetos FAQ válidos, serializar e depois desserializar SHALL produzir um objeto equivalente ao original (propriedade de round-trip).
4. FOR ALL objetos Ticket válidos, serializar e depois desserializar SHALL produzir um objeto equivalente ao original (propriedade de round-trip).
5. IF um objeto FAQ ou Ticket corrompido ou inválido for fornecido ao desserializador, THEN THE FAQ_Repository ou Ticket_Repository SHALL retornar um erro descritivo.

---

### Requisito 6: Interface de Entrada via API

**User Story:** Como integrador, quero enviar mensagens ao SupportBot via API, para que o bot possa ser integrado a diferentes canais de atendimento.

#### Critérios de Aceitação

1. THE SupportBot SHALL expor um endpoint de API que receba a mensagem do Customer e retorne a resposta do bot.
2. WHEN uma requisição válida é recebida no endpoint, THE SupportBot SHALL retornar uma resposta HTTP com status 200 e o conteúdo da resposta do bot.
3. IF uma requisição com corpo ausente ou malformado for recebida, THEN THE SupportBot SHALL retornar uma resposta HTTP com status 400 e uma mensagem de erro descritiva.
4. THE SupportBot SHALL aceitar e retornar dados no formato JSON.
