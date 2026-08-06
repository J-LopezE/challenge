# Challenge: API de agente conversacional con memoria

## Objetivo
API en Python (FastAPI) que expone un agente conversacional con memoria por
conversación (id) y una tool de consulta financiera sobre Yahoo Finance, usando
LangChain con un agent_loop definido explícitamente. Expone POST /chat y
GET /chat/{id}, corre en Docker y está lista para funcionar como app productiva.

## Decisiones de diseño (por qué)
- **FastAPI**: validación con Pydantic, docs OpenAPI gratuitas, estándar actual de APIs de IA.
- **Multi-proveedor LLM** (interface + factory): openai/anthropic/ollama por env var; SOLID (OCP + DIP); testeable sin API keys. En local usamos Ollama + qwen2.5:3b.
- **PostgreSQL + Docker Compose**: ACID real, concurrencia, durabilidad (WAL + volumen).
- **UUID como PK**: no expone el volumen de datos ni colisiona entre instancias.
- **yfinance detrás de interface**: API no oficial → cambiable sin tocar la app.
- **Alembic**: migraciones versionadas.
- **Clean Architecture**: el dominio no depende de la infraestructura (regla de dependencias).

## Arquitectura (diagrama)
presentation (routes, schemas) → application (ChatService, agent_loop) → domain (entities, ports) ← infrastructure (SQLAlchemy, LLM, yfinance)
Regla de dependencias: las dependencias apuntan hacia el dominio.

## Endpoints
| Método | Ruta      | Request                  | Response                   | Errores                          |
|--------|-----------|--------------------------|----------------------------|----------------------------------|
| GET    | /chat/{id}| -                        | historial de la conversación | 400 UUID inválido · 404 no existe |
| POST   | /chat     | {conversation_id, message} | respuesta del agente       | 400/422                          |
| GET    | /health   | -                        | {"status": "ok"}           | -                                |

## Modelo de datos
- **conversations**: id (UUID PK), created_at (timestamptz), updated_at (timestamptz)
- **messages**: id (UUID PK), conversation_id (FK → conversations.id, CASCADE, indexada), role (CHECK user|assistant|tool), content (TEXT), created_at
En 3NF (atributos atómicos, sin dependencias parciales ni transitivas) y cumple ACID.

## Roadmap de sesiones
- S1: Setup + FastAPI + Postgres + Docker + GET /chat/{id} (completada)
- S2: agent_loop + POST /chat + tests unit (TDD)
- S3: tool yfinance + tests de integración
- S4: frontend React+TS + README
- S5: CI/CD + simulacro de entrevista

## Banco de preguntas de entrevista
- ¿Por qué FastAPI y no Flask?
- ¿Qué es el agent_loop y cómo evitas loops infinitos?
- ¿Cómo se aplica ACID y 3NF en este schema?
- ¿Cómo pruebas el agente sin gastar en API keys?
- ¿Qué pasa si el LLM falla a mitad del proceso?
- ¿Cómo agregas un proveedor LLM sin tocar código existente?