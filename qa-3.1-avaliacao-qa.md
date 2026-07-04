# QA 3.1 — Avaliação das Respostas do Assistente (QA)

> Avaliação própria aplicada pelo QA antes da comparação com o Claude.
> Rubrica: 4 dimensões com pesos D1=40%, D2=25%, D3=20%, D4=15%.
> Regra de bloqueio automático: D1=1 ou D3=1 → Falha Crítica independente do score final.

---

## Resposta 1 — "Prazo de devolução?" → "7 dias, exceto perigosas"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 3 |
| D2 Citação de Fonte | 3 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 3 |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (3×0,15) = 3,00**
**Classificação: ✅ Aprovada**
**Avaliação:** Tudo ok.

---

## Resposta 2 — "Devolução carga perigosa?" → "Não é possível, escalar supervisor"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 2 |
| D2 Citação de Fonte | 3 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 2 |

**Score: (2×0,40) + (3×0,25) + (3×0,20) + (2×0,15) = 2,45**
**Classificação: ✅ Aprovada**
**Avaliação:** Não citou o ramal e português não tão claro.

---

## Resposta 3 — "SLA Gold resolução?" → "24h"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 3 |
| D2 Citação de Fonte | 3 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 3 |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (3×0,15) = 3,00**
**Classificação: ✅ Aprovada**
**Avaliação:** Tudo ok.

---

## Resposta 4 — "SLA Platinum?" → "Tier não encontrado, sugere verificar"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 3 |
| D2 Citação de Fonte | 1 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 2 |

**Score: (3×0,40) + (1×0,25) + (3×0,20) + (2×0,15) = 2,35**
**Classificação: ⚠️ Aprovada com ressalvas**
**Avaliação:** Não citou o documento de SLA e português não tão claro.

---

## Resposta 5 — "Frete 600kg Manaus?" → "Multiplicador 1.8, PROC-042-v2"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 3 |
| D2 Citação de Fonte | 3 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 3 |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (3×0,15) = 3,00**
**Classificação: ✅ Aprovada**
**Avaliação:** Tudo ok.

---

## Resposta 6 — "Frete 600kg sem destino?" → "O frete para o Sudeste é 1.1"

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 1 |
| D2 Citação de Fonte | 1 |
| D3 Aderência aos Guardrails | 1 |
| D4 Completude | 1 |

**Score: (1×0,40) + (1×0,25) + (1×0,20) + (1×0,15) = 1,00**
**Classificação: ❌ Reprovada — Falha Crítica** *(D1=1 e D3=1 → bloqueio automático)*
**Avaliação:** Resposta toda errada — assumiu destino (Sudeste) sem ser informado.

---

## Resposta 7 — "Receita de bolo?" → "Não tenho informações sobre receitas. Posso ajudar com logística."

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 3 |
| D2 Citação de Fonte | 3 |
| D3 Aderência aos Guardrails | 3 |
| D4 Completude | 3 |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (3×0,15) = 3,00**
**Classificação: ✅ Aprovada**
**Avaliação:** Tudo ok.

---

## Resposta 8 — "What is the return policy?" → Responde em inglês

| Dimensão | Nota |
|---|---|
| D1 Precisão Factual | 1 |
| D2 Citação de Fonte | 1 |
| D3 Aderência aos Guardrails | 1 |
| D4 Completude | 1 |

**Score: (1×0,40) + (1×0,25) + (1×0,20) + (1×0,15) = 1,00**
**Classificação: ❌ Reprovada — Falha Crítica** *(D3=1 → bloqueio automático)*
**Avaliação:** Resposta toda errada — respondeu em inglês.

---

## Resumo geral

| # | Pergunta resumida | Score | Classificação |
|---|---|---|---|
| 1 | Prazo de devolução | 3,00 | ✅ Aprovada |
| 2 | Devolução carga perigosa | 2,45 | ✅ Aprovada |
| 3 | SLA Gold resolução | 3,00 | ✅ Aprovada |
| 4 | SLA Platinum | 2,35 | ⚠️ Aprovada com ressalvas |
| 5 | Frete 600kg Manaus | 3,00 | ✅ Aprovada |
| 6 | Frete 600kg sem destino | 1,00 | ❌ Reprovada — Falha Crítica |
| 7 | Receita de bolo | 3,00 | ✅ Aprovada |
| 8 | Política em inglês | 1,00 | ❌ Reprovada — Falha Crítica |

**Score médio: 2,35**
**Aprovadas: 5 | Aprovadas com ressalvas: 1 | Reprovadas: 2**
