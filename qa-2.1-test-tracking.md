# Board de Rastreabilidade QA — NovaTech Assistant · Query Endpoint
**Versão:** 2.2 · **Atualizado em:** 2026-06-27 · **Projeto:** NovaTech Assistant

---

## Metas de Deploy

| Ambiente    | Critério mínimo                                     |
|-------------|-----------------------------------------------------|
| **Staging** | 100 % de VC-03 aprovados + 80 % dos demais VCs      |
| **Produção**| 100 % de todos os VCs aprovados                     |

---

## Ordem de Prioridade de Implementação

| Prioridade | VC        | Justificativa                              |
|:----------:|-----------|--------------------------------------------|
| 1          | VC-03     | Risco crítico — consultas sobre carga perigosa |
| 2          | VC-04     | Comportamento base do sistema (sem match)  |
| 3          | VC-02     | Rastreabilidade contratual (source_document) |
| 4          | VC-ROB    | Riscos de IA em ambiente de produção       |
| 5          | VC-01     | Performance — depende de infra completa    |

---

## Cenários por Verification Criteria

### VC-01 — Resposta em < 30 s para 95 % das queries

| ID         | Descrição (≤ 10 palavras)                        | VC    | Tipo       | Risco  | Status        | Responsável | Prioridade |
|------------|--------------------------------------------------|-------|------------|--------|---------------|-------------|:----------:|
| TC-01-01   | Query simples com match direto                   | VC-01 | Happy path | Baixo  | Não iniciado  |             | 5          |
| TC-01-02   | Query multi-domínio com 7+ chunks                | VC-01 | Edge case  | Médio  | Não iniciado  |             | 5          |
| TC-01-03   | 10 requisições simultâneas, P95 < 30 s           | VC-01 | Edge case  | Médio  | Não iniciado  |             | 5          |

> **Subtotal VC-01:** 3 cenários · ✅ 0 aprovados · ❌ 0 reprovados · ⏳ 3 não iniciados

---

### VC-02 — 100 % das respostas incluem campo `source_document`

| ID         | Descrição (≤ 10 palavras)                        | VC    | Tipo       | Risco  | Status        | Responsável | Prioridade |
|------------|--------------------------------------------------|-------|------------|--------|---------------|-------------|:----------:|
| TC-02-01   | Match único — source_document correto            | VC-02 | Happy path | Baixo  | Não iniciado  |             | 3          |
| TC-02-02   | Múltiplos documentos — todos listados em sources | VC-02 | Edge case  | Médio  | Não iniciado  |             | 3          |
| TC-02-03   | Sem match — source_document null, não inventado  | VC-02 | Edge case  | Alto   | Não iniciado  |             | 3          |

> **Subtotal VC-02:** 3 cenários · ✅ 0 aprovados · ❌ 0 reprovados · ⏳ 3 não iniciados

---

### VC-03 — Queries sobre carga perigosa + devolução retornam negativa explícita

| ID         | Descrição (≤ 10 palavras)                              | VC    | Tipo         | Risco    | Status        | Responsável | Prioridade |
|------------|--------------------------------------------------------|-------|--------------|----------|---------------|-------------|:----------:|
| TC-03-01   | Pergunta direta — negativa + ramal 4500                | VC-03 | Happy path   | Alto     | Não iniciado  |             | 1          |
| TC-03-02   | Paráfrase — sistema identifica tema sem palavras-chave | VC-03 | Edge case    | Alto     | Não iniciado  |             | 1          |
| TC-03-03   | Carga mista perigosa/normal — processos distintos      | VC-03 | Edge case    | Alto     | Não iniciado  |             | 1          |
| TC-03-04   | Adversarial — sistema não confirma premissa falsa      | VC-03 | Adversarial  | Crítico  | Não iniciado  |             | 1          |

> **Subtotal VC-03:** 4 cenários · ✅ 0 aprovados · ❌ 0 reprovados · ⏳ 4 não iniciados

---

### VC-04 — Queries sem match retornam mensagem padrão "não encontrado"

| ID         | Descrição (≤ 10 palavras)                                | VC    | Tipo       | Risco  | Status        | Responsável | Prioridade |
|------------|----------------------------------------------------------|-------|------------|--------|---------------|-------------|:----------:|
| TC-04-01   | Pergunta fora do domínio — sem invenção                  | VC-04 | Happy path | Baixo  | Não iniciado  |             | 2          |
| TC-04-02   | Domínio correto mas gap documentado — sem invenção       | VC-04 | Edge case  | Médio  | Não iniciado  |             | 2          |
| TC-04-03   | Palavras do domínio sem documento indexado               | VC-04 | Edge case  | Médio  | Não iniciado  |             | 2          |

> **Subtotal VC-04:** 3 cenários · ✅ 0 aprovados · ❌ 0 reprovados · ⏳ 3 não iniciados

---

### VC-ROB — Robustez da IA

| ID         | Descrição (≤ 10 palavras)                                | VC      | Tipo       | Risco  | Status        | Responsável | Prioridade |
|------------|----------------------------------------------------------|---------|------------|--------|---------------|-------------|:----------:|
| TC-ROB-01  | Pergunta ambígua — sistema reconhece sem escolher        | VC-ROB  | Robustez   | Médio  | Não iniciado  |             | 4          |
| TC-ROB-02  | Prompt injection — system prompt não revelado            | VC-ROB  | Robustez   | Alto   | Não iniciado  |             | 4          |
| TC-ROB-03  | Pergunta em idioma estrangeiro — resposta em PT          | VC-ROB  | Robustez   | Baixo  | Não iniciado  |             | 4          |
| TC-ROB-04  | Doc contraditório PROC-042 v1 vs v2 — versão recente     | VC-ROB  | Robustez   | Alto   | Não iniciado  |             | 4          |
| TC-ROB-05  | Tier inexistente Platinum — sem invenção de SLAs         | VC-ROB  | Robustez   | Médio  | Não iniciado  |             | 4          |

> **Subtotal VC-ROB:** 5 cenários · ✅ 0 aprovados · ❌ 0 reprovados · ⏳ 5 não iniciados

---

## Resumo Geral de Cobertura

| VC      | Total de Cenários | ✅ Aprovados | ❌ Reprovados | ⏳ Não iniciados | % Aprovação |
|---------|:-----------------:|:-----------:|:------------:|:---------------:|:-----------:|
| VC-01   | 3                 | 0           | 0            | 3               | 0 %         |
| VC-02   | 3                 | 0           | 0            | 3               | 0 %         |
| VC-03   | 4                 | 0           | 0            | 4               | 0 %         |
| VC-04   | 3                 | 0           | 0            | 3               | 0 %         |
| VC-ROB  | 5                 | 0           | 0            | 5               | 0 %         |
| **Total** | **18**          | **0**       | **0**        | **18**          | **0 %**     |

---

## Legenda

| Símbolo | Significado    |
|:-------:|----------------|
| ✅      | Aprovado        |
| ❌      | Reprovado       |
| ⏳      | Não iniciado    |
| 🔄      | Em execução     |
| ⏸️      | Bloqueado       |

| Risco    | Descrição                                              |
|----------|--------------------------------------------------------|
| Baixo    | Falha tem impacto mínimo na operação                   |
| Médio    | Falha afeta UX mas não compromete segurança            |
| Alto     | Falha afeta rastreabilidade ou compliance              |
| Crítico  | Falha compromete segurança ou induz erro perigoso      |

---

*Gerado automaticamente para o projeto NovaTech Assistant — Query Endpoint QA 2.2*
*Colar em `specs/query-endpoint/test-tracking.md` no repositório do projeto.*
