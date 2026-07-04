# QA 3.1 — Avaliação das Respostas do Assistente (Claude)

> Segunda avaliação aplicada pelo Claude como co-reviewer.
> Rubrica: 4 dimensões com pesos D1=40%, D2=25%, D3=20%, D4=15%.
> Regra de bloqueio automático: D1=1 ou D3=1 → Falha Crítica independente do score final.

---

## Resposta 1 — "Prazo de devolução?" → "7 dias, exceto perigosas"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 3 | Correto e alinhado com POL-001 §3.1 e §3.2 |
| D2 Citação de Fonte | 3 | Cita POL-001 corretamente |
| D3 Guardrails | 3 | Cita fonte, não inventa dados, português formal |
| D4 Completude | 2 | Menciona exceção de perigosas mas não cita o procedimento (abrir chamado no portal, fotos) nem o ramal 4500 |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (2×0,15) = 2,85**
**Classificação: ✅ Aprovada**

---

## Resposta 2 — "Devolução carga perigosa?" → "Não é possível, escalar supervisor"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 2 | Negativa correta, mas "escalar supervisor" é impreciso — o correto é Gestão de Riscos, ramal 4500 (POL-001 §3.2) |
| D2 Citação de Fonte | 3 | Cita POL-001 corretamente |
| D3 Guardrails | 3 | Sem violações |
| D4 Completude | 1 | Omite informação crítica: ramal 4500 e setor correto. O atendente seguindo essa resposta encaminharia para o lugar errado |

**Score: (2×0,40) + (3×0,25) + (3×0,20) + (1×0,15) = 2,30**
**Classificação: ⚠️ Aprovada com ressalvas**

---

## Resposta 3 — "SLA Gold resolução?" → "24h"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 3 | Correto conforme SLA-2024-B |
| D2 Citação de Fonte | 3 | Cita SLA-2024 corretamente |
| D3 Guardrails | 3 | Sem violações |
| D4 Completude | 2 | Informa resolução (24h) mas omite o tempo de resposta (2h úteis), que também compõe o SLA Gold |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (2×0,15) = 2,85**
**Classificação: ✅ Aprovada**

---

## Resposta 4 — "SLA Platinum?" → "Tier não encontrado, sugere verificar"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 3 | Correto: não inventou SLA, reconheceu que o tier não existe, alinhado com SLA-2024-A |
| D2 Citação de Fonte | 1 | Não cita nenhuma fonte — deveria citar SLA-2024 seção 1, onde está explícito que só existem 3 tiers |
| D3 Guardrails | 2 | Falha no guardrail 1 (sempre citar fonte). Demais guardrails respeitados |
| D4 Completude | 2 | Sugere verificar mas não lista os tiers corretos nem orienta como identificar o tier do cliente |

**Score: (3×0,40) + (1×0,25) + (2×0,20) + (2×0,15) = 2,35**
**Classificação: ⚠️ Aprovada com ressalvas**

---

## Resposta 5 — "Frete 600kg Manaus?" → "Multiplicador 1.8, PROC-042-v2"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 3 | Correto: 600kg = fator de peso 1.0 (faixa 500-1.000kg), Norte = multiplicador 1.8 na PROC-042-v2 |
| D2 Citação de Fonte | 3 | Cita PROC-042-v2, versão correta e atual |
| D3 Guardrails | 3 | Sem violações |
| D4 Completude | 2 | Informa o multiplicador regional mas não menciona o fator de peso (1.0) nem que é necessário o valor base para o cálculo completo |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (2×0,15) = 2,85**
**Classificação: ✅ Aprovada**

---

## Resposta 6 — "Frete 600kg sem destino?" → "O frete para o Sudeste é 1.1"

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 1 | Inventou o destino (Sudeste) sem nenhuma base na pergunta. O multiplicador 1.1 é tecnicamente correto para Sudeste, mas o destino foi assumido sem informação |
| D2 Citação de Fonte | 2 | Cita PROC-042-v2, que existe e é a versão correta, mas a citação não valida uma resposta baseada em dado inventado |
| D3 Guardrails | 1 | Viola guardrail 2 (inventou dado — o destino) e guardrail 3 (não declarou que a informação estava faltando; deveria ter solicitado o destino) |
| D4 Completude | 1 | Respondeu uma pergunta diferente da que foi feita — não só incompleta, mas ativamente incorreta |

**Score: (1×0,40) + (2×0,25) + (1×0,20) + (1×0,15) = 1,20**
**Classificação: ❌ Reprovada — Falha Crítica** *(D1=1 e D3=1 → bloqueio automático)*

---

## Resposta 7 — "Receita de bolo?" → "Não tenho informações sobre receitas. Posso ajudar com logística."

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 3 | Correto: não inventou nada, recusou apropriadamente |
| D2 Citação de Fonte | 3 | Ausência de fonte adequada ao contexto — não havia documento a citar |
| D3 Guardrails | 3 | Declarou explicitamente que não tem informação (guardrail 3), português formal, não inventou nada |
| D4 Completude | 3 | Completa para o contexto: recusou o tema fora do escopo e redirecionou corretamente |

**Score: (3×0,40) + (3×0,25) + (3×0,20) + (3×0,15) = 3,00**
**Classificação: ✅ Aprovada**

---

## Resposta 8 — "What is the return policy?" → Responde em inglês

| Dimensão | Nota | Justificativa |
|---|---|---|
| D1 Precisão Factual | 2 | O conteúdo pode estar correto (cita POL-001), mas responder em idioma diferente introduz risco de interpretação — não é possível confirmar precisão sem ver o texto completo |
| D2 Citação de Fonte | 3 | Cita POL-001 corretamente |
| D3 Guardrails | 1 | Viola guardrail 4 diretamente: o sistema deve responder em português formal. Violação clara e direta |
| D4 Completude | 2 | Não é possível avaliar completude sem ver o conteúdo em inglês, mas o problema principal é o idioma |

**Score: (2×0,40) + (3×0,25) + (1×0,20) + (2×0,15) = 2,00**
**Classificação: ❌ Reprovada — revisão necessária** *(D3=1 → bloqueio automático)*

---

## Resumo geral

| # | Pergunta resumida | Score | Classificação |
|---|---|---|---|
| 1 | Prazo de devolução | 2,85 | ✅ Aprovada |
| 2 | Devolução carga perigosa | 2,30 | ⚠️ Aprovada com ressalvas |
| 3 | SLA Gold resolução | 2,85 | ✅ Aprovada |
| 4 | SLA Platinum | 2,35 | ⚠️ Aprovada com ressalvas |
| 5 | Frete 600kg Manaus | 2,85 | ✅ Aprovada |
| 6 | Frete 600kg sem destino | 1,20 | ❌ Reprovada — Falha Crítica |
| 7 | Receita de bolo | 3,00 | ✅ Aprovada |
| 8 | Política em inglês | 2,00 | ❌ Reprovada |

**Score médio: 2,43**
**Aprovadas: 5 | Aprovadas com ressalvas: 2 | Reprovadas: 2**
