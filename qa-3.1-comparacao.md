# QA 3.1 — Comparação de Avaliações: QA vs Claude

> Confronto entre a avaliação feita pelo QA (item 1) e a avaliação feita pelo Claude (item 2).
> Objetivo: identificar divergências, discutir interpretações e calibrar a rubrica.

---

## Tabela comparativa de scores

| # | Pergunta resumida | Score QA | Score Claude | Diferença | Classificação QA | Classificação Claude |
|---|---|---|---|---|---|---|
| 1 | Prazo de devolução | 3,00 | 2,85 | -0,15 | ✅ Aprovada | ✅ Aprovada |
| 2 | Devolução carga perigosa | 2,45 | 2,30 | -0,15 | ⚠️ Aprovada com ressalvas | ⚠️ Aprovada com ressalvas |
| 3 | SLA Gold resolução | 3,00 | 2,85 | -0,15 | ✅ Aprovada | ✅ Aprovada |
| 4 | SLA Platinum | 2,35 | 2,35 | 0,00 | ⚠️ Aprovada com ressalvas | ⚠️ Aprovada com ressalvas |
| 5 | Frete 600kg Manaus | 3,00 | 2,85 | -0,15 | ✅ Aprovada | ✅ Aprovada |
| 6 | Frete 600kg sem destino | 1,00 | 1,20 | +0,20 | ❌ Reprovada — Falha Crítica | ❌ Reprovada — Falha Crítica |
| 7 | Receita de bolo | 3,00 | 3,00 | 0,00 | ✅ Aprovada | ✅ Aprovada |
| 8 | Política em inglês | 1,00 | 2,00 | +1,00 | ❌ Reprovada | ❌ Reprovada |

**Score médio QA: 2,35 | Score médio Claude: 2,43**

---

## Análise das divergências

### Respostas 1, 3 e 5 — Divergência pequena e sistemática (-0,15)

**O que divergiu:** QA deu D4=3 (completa) nas três respostas. Claude deu D4=2 (parcialmente completa).

**Motivo da divergência no Claude:**
- Resposta 1: menciona exceção de perigosas mas não cita o procedimento completo (abrir chamado, fotos, ramal 4500)
- Resposta 3: informa resolução (24h) mas omite o tempo de resposta (2h úteis), que também compõe o SLA Gold
- Resposta 5: informa o multiplicador regional (1.8) mas não menciona o fator de peso (1.0 para 500-1.000kg)

**Conclusão:** Divergência de interpretação sobre o nível de detalhe exigido pela D4. Ambas as leituras são defensáveis. A rubrica diz "o atendente tem tudo o que precisa para responder sem buscar informação adicional" — a pergunta é se o atendente consegue calcular o frete ou informar o SLA completo com o que foi dado. **Recomendação: alinhar o critério de D4 na próxima revisão da rubrica, especificando o nível de detalhe esperado para perguntas de cálculo e de SLA.**

---

### Resposta 2 — Divergência pequena (-0,15)

**O que divergiu:** QA deu D4=2. Claude deu D4=1.

**Motivo da divergência no Claude:** "Escalar supervisor" é um encaminhamento errado — o correto pela POL-001 §3.2 é encaminhar para Gestão de Riscos no ramal 4500. Um atendente seguindo essa resposta tentaria escalar para o supervisor, que provavelmente não tem acesso ao protocolo de carga perigosa. Claude interpretou isso como omissão de informação crítica que muda o encaminhamento (D4=1). QA interpretou como detalhe impreciso que não inverte o sentido (D4=2).

**Conclusão:** A divergência é de interpretação sobre a gravidade da imprecisão. O encaminhamento errado num contexto de carga perigosa é risco relevante — D4=1 é a interpretação mais conservadora e alinhada com o critério da rubrica ("em caso de dúvida, opte pelo nível mais baixo"). **Recomendação: adotar D4=1 para esta resposta.**

---

### Resposta 4 — Nenhuma divergência de score (0,00)

**Score idêntico (2,35), mas por caminhos diferentes:**

| Dimensão | QA | Claude |
|---|---|---|
| D1 | 3 | 3 |
| D2 | 1 | 1 |
| D3 | 3 | 2 |
| D4 | 2 | 2 |

**Motivo:** QA deu D3=3 (todos os guardrails respeitados). Claude deu D3=2 (falha no guardrail 1 — citar fonte). O guardrail 1 diz "sempre citar fonte". A resposta não citou o SLA-2024 onde está registrado que só existem 3 tiers. Claude interpretou isso como violação do guardrail 1.

**Conclusão:** Como D2 já penalizou a ausência de fonte, há uma sobreposição entre D2 e D3 neste caso. O score final é o mesmo, mas a interpretação do Claude é mais rigorosa. **Recomendação: a rubrica pode esclarecer se a penalidade de fonte ausente deve aparecer em D2, D3 ou ambas.**

---

### Resposta 6 — Pequena divergência (+0,20)

**O que divergiu:** QA deu D2=1. Claude deu D2=2.

**Motivo da divergência no Claude:** A fonte citada (PROC-042-v2) existe e é a versão correta — o documento em si é válido. O problema não é a fonte, é que a resposta usou um dado inventado (destino Sudeste). Claude separou os dois problemas: fonte real mas resposta errada = D2=2. QA avaliou o conjunto como totalmente errado = D2=1.

**Conclusão:** Ambas as classificações chegam ao mesmo resultado final (Falha Crítica por D1=1 e D3=1). A diferença de interpretação em D2 não muda a ação. Para fins de registro, a leitura do Claude é mais precisa — a fonte existe, o problema é outro.

---

### Resposta 8 — Maior divergência (+1,00)

**O que divergiu:** QA deu nota 1 em todas as dimensões (score 1,00). Claude deu D1=2, D2=3, D3=1, D4=2 (score 2,00).

**Motivo da divergência no Claude:** O erro da resposta 8 é específico: o idioma. A fonte (POL-001) está correta, e o conteúdo factual pode estar certo — não sabemos porque o enunciado não mostra o texto em inglês. Dar D1=1 significaria afirmar que os fatos estão errados, o que não há evidência para concluir. O problema real e verificável é o idioma, que está em D3 (guardrail 4).

**Conclusão:** Ambas chegam à mesma classificação (Reprovada com bloqueio automático por D3=1). Mas a causa registrada importa para a correção: se D1=1, a equipe vai investigar precisão factual. Se D3=1, vai corrigir o guardrail de idioma — que é o problema real. **Recomendação: registrar D3=1 como causa primária. Adotar o score do Claude (2,00) para esta resposta.**

---

## Resumo das recomendações

| # | Recomendação |
|---|---|
| 1, 3, 5 | Alinhar critério de D4 na rubrica: especificar nível de detalhe esperado para respostas de cálculo e SLA |
| 2 | Adotar D4=1 (encaminhamento errado em contexto crítico = informação crítica ausente) |
| 4 | Esclarecer na rubrica se ausência de fonte penaliza D2, D3 ou ambas |
| 6 | Adotar D2=2 (fonte existe e é correta; o problema é o dado inventado, não a fonte) |
| 8 | Registrar causa como D3=1 (idioma), não D1=1. Adotar score 2,00 |

---

## Conclusão geral

As duas avaliações chegaram à **mesma classificação final em todas as 8 respostas** — nenhuma divergência mudou o resultado de aprovada para reprovada ou vice-versa. Isso indica que a rubrica está bem calibrada para as decisões mais importantes.

As divergências existentes são de interpretação sobre nível de detalhe (D4) e sobre como distribuir penalidades entre dimensões (D2 vs D3). Esses pontos devem ser esclarecidos na próxima revisão da rubrica para aumentar a consistência entre avaliadores.
