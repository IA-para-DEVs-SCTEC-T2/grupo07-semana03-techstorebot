# TechStore SupportBot

API REST de suporte ao cliente em Python/FastAPI que responde FAQs sobre entrega, trocas e pagamento — e abre tickets automaticamente quando não souber responder.

## Visão Geral

O TechStore SupportBot é um chatbot de suporte para a plataforma de e-commerce TechStore. Ele classifica mensagens dos clientes, responde perguntas frequentes em português (pt-BR) e, quando não consegue responder, coleta nome, e-mail e pergunta para abrir um ticket de suporte.

## Funcionalidades

- Responde FAQs sobre prazos de entrega, trocas/devoluções e formas de pagamento
- Classifica intenções: `faq_match` ou `unknown`
- Coleta dados do cliente e abre tickets de suporte quando necessário
- Todas as respostas em português (pt-BR)
- Endpoint HTTP `POST /message` com entrada/saída JSON

## Stack

- **Python 3.11+**
- **FastAPI** — camada de API
- **Pydantic** — validação de schemas
- **Hypothesis** — testes baseados em propriedades (PBT)
- **pytest** — runner de testes

## Arquitetura

Clean Architecture com três camadas:

```
src/
├── api/            # FastAPI handlers, schemas, wiring
├── domain/         # Entidades, use cases, interfaces (sem dependências externas)
└── infrastructure/ # Repositórios em memória (FAQ e Ticket)
```

## Instalação

```bash
pip install -r requirements.txt
```

## Executando

```bash
uvicorn src.api.app:app --reload
```

## Testes

```bash
pytest
```

## Uso da API

**POST /message**

```json
// Request
{ "text": "Qual o prazo de entrega?" }

// Response 200
{ "response": "O prazo de entrega padrão é de 5 a 10 dias úteis." }

// Response 400
{ "error": "O campo 'text' é obrigatório." }
```
