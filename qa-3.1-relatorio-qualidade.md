# Relatório de Qualidade — Assistente NovaTech
**Data:** Junho/2026
**Avaliador:** QA
**Lote avaliado:** 8 respostas em staging
**Rubrica aplicada:** v1.0 (4 dimensões, pesos D1=40%, D2=25%, D3=20%, D4=15%)

---

## 1. Score médio e distribuição

| Métrica | Valor |
|---|---|
| Score médio do lote | 2,43 / 3,00 |
| Respostas aprovadas | 5 (62,5%) |
| Respostas aprovadas com ressalvas | 2 (25,0%) |
| Respostas reprovadas | 1 (12,5%) |
| Falhas críticas (bloqueio automático) | 1 (12,5%) |

---

## 2. Scores por resposta

| # | Pergunta resumida | D1 | D2 | D3 | D4 | Score | Classificação |
|---|---|---|---|---|---|---|---|
| 1 | Prazo de devolução | 3 | 3 | 3 | 2 | 2,85 | ✅ Aprovada |
| 2 | Devolução carga perigosa | 2 | 3 | 3 | 1 | 2,30 | ⚠️ Aprovada com ressalvas |
| 3 | SLA Gold resolução | 3 | 3 | 3 | 2 | 2,85 | ✅ Aprovada |
| 4 | SLA Platinum | 3 | 1 | 2 | 2 | 2,35 | ⚠️ Aprovada com ressalvas |
| 5 | Frete 600kg Manaus | 3 | 3 | 3 | 2 | 2,85 | ✅ Aprovada |
| 6 | Frete 600kg sem destino | 1 | 2 | 1 | 1 | 1,20 | ❌ Reprovada — Falha Crítica |
| 7 | Receita de bolo | 3 | 3 | 3 | 3 | 3,00 | ✅ Aprovada |
| 8 | Política em inglês | 2 | 3 | 1 | 2 | 2,00 | ❌ Reprovada |

> Scores baseados na avaliação do Claude como co-reviewer, adotada após comparação com avaliação do QA. As duas avaliações chegaram à mesma classificação final em todas as 8 respostas.

---

## 3. Respostas reprovadas — detalhamento

### Resposta 6 — "Frete 600kg sem destino?" ❌ Falha Crítica

**Problema:** O assistente assumiu o destino (Sudeste) sem que a informação tivesse sido fornecida na pergunta, e respondeu com o multiplicador 1.1 como se fosse a resposta correta.

**Dimensões com falha:**
- D1=1: inventou dado (destino Sudeste)
- D3=1: violou guardrail 2 (não inventar dados) e guardrail 3 (não declarou que a informação estava faltando)
- D4=1: respondeu uma pergunta diferente da que foi feita

**Risco:** O atendente repassaria ao cliente um valor de frete incorreto, causando cobrança errada e perda de confiança.

**Correção necessária:** O assistente deve reconhecer quando uma informação obrigatória para o cálculo está ausente e solicitar o dado antes de responder. Ajuste no system prompt: incluir instrução explícita para perguntas de frete sem destino informado.

---

### Resposta 8 — "What is the return policy?" ❌ Reprovada

**Problema:** O assistente respondeu em inglês, violando o guardrail 4 (responder sempre em português formal).

**Dimensão com falha:**
- D3=1: violação direta do guardrail 4

**Risco:** Atendentes operam em português. Uma resposta em inglês pode ser mal interpretada ou simplesmente ignorada, deixando o atendente sem a informação que precisava.

**Correção necessária:** Reforçar no system prompt que o idioma de resposta é sempre português formal, independentemente do idioma da pergunta. Adicionar instrução para que o assistente solicite a reformulação em português quando necessário.

---

## 4. Pontos de atenção — respostas aprovadas com ressalvas

### Resposta 2 — Devolução carga perigosa ⚠️

**Problema:** O encaminhamento "escalar supervisor" está incorreto. O procedimento correto pela POL-001 §3.2 é contatar Gestão de Riscos pelo ramal 4500.

**Risco:** Baixo a médio — a negativa está correta, mas o encaminhamento errado pode atrasar o atendimento ao cliente.

**Ação:** Verificar se o system prompt ou os chunks da POL-001 §3.2 estão sendo recuperados corretamente com o ramal 4500.

---

### Resposta 4 — SLA Platinum ⚠️

**Problema:** Não citou a fonte (SLA-2024) ao informar que o tier Platinum não existe.

**Risco:** Baixo — a informação está correta, mas sem fonte o atendente não pode verificar ou mostrar ao cliente.

**Ação:** Verificar guardrail de citação de fonte para respostas de "não encontrado" — o assistente deve citar o documento onde confirmou a ausência da informação.

---

## 5. Padrão de fraqueza identificado — D4 (Completude)

Das 8 respostas avaliadas, **5 receberam D4=2** (parcialmente completa). Nenhuma das respostas aprovadas entregou informação completa o suficiente para dispensar busca adicional pelo atendente.

**Padrão observado:** O assistente responde o núcleo da pergunta corretamente mas omite sistematicamente:
- Procedimentos associados (como abrir chamado, fotos necessárias)
- Informações complementares do mesmo documento (tempo de resposta junto com tempo de resolução no SLA)
- Fatores de cálculo secundários (fator de peso junto com multiplicador regional no frete)

**Ação recomendada:** Revisar o system prompt para instruir o assistente a incluir informações complementares relevantes quando a pergunta envolver procedimentos ou cálculos. Avaliar se os chunks estão sendo recuperados com contexto suficiente.

---

## 6. Parecer de go-live

**O assistente NÃO está pronto para go-live no estado atual.**

**Bloqueadores:**
1. **Resposta 6 (Falha Crítica):** O comportamento de assumir dados não informados é um risco direto de operação — o atendente pode repassar valores incorretos de frete ao cliente. Este comportamento precisa ser corrigido e re-testado antes do go-live.
2. **Resposta 8 (Reprovada):** A violação do guardrail de idioma precisa ser corrigida no system prompt.

**Ressalvas para go-live após correções:**
- A completude (D4) está sistematicamente abaixo do ideal. Não é bloqueante, mas deve entrar no backlog de melhoria imediata pós-go-live.
- O encaminhamento incorreto da resposta 2 (supervisor vs. ramal 4500) deve ser corrigido antes do go-live, mesmo não sendo bloqueante pela rubrica.

**Condição para autorizar go-live:** Correção dos dois bloqueadores, re-teste das respostas 6 e 8, e aprovação de um novo lote de avaliação com score médio ≥ 2,50 e zero falhas críticas.
