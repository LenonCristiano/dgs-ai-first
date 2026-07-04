# QA 3.2 — Comparação de Avaliações: QA vs Claude

> Confronto entre a avaliação feita pelo QA e a avaliação feita pelo Claude como co-reviewer.
> Objetivo: identificar divergências, discutir interpretações e consolidar a revisão final.

---

## Teste 1 — Assertions vagas

| Aspecto | QA | Claude | Alinhamento |
|---|---|---|---|
| Problema principal | Não verifica a resposta esperada | Não verifica `answer`, `source_document`, `confidence_score` | ✅ Alinhados — Claude detalha os campos específicos |
| Mocks de serviços externos | Não mencionado | Azure AI Search e Azure OpenAI sem mock | ⚠️ Claude acrescenta |
| Nome do teste | Não mencionado | Nome vago, viola padrão `should [comportamento] when [condição]` | ⚠️ Claude acrescenta |
| Dado de entrada | Não mencionado | `'prazo devolução'` deveria ser fixture de `queries.ts` | ⚠️ Claude acrescenta |
| Risco identificado | Resposta errada ao atendente | Resposta vazia passa no CI sem ser detectada | ✅ Essencialmente o mesmo risco |

**Conclusão:** Avaliações alinhadas no problema central. Claude detalha os campos ausentes e acrescenta problemas de mock e nomenclatura que não foram mencionados pelo QA.

---

## Teste 2 — Dados irreais

| Aspecto | QA | Claude | Alinhamento |
|---|---|---|---|
| Validade do cenário | "Não é uma pergunta sobre logística" — cenário questionado | Cenário válido: validação de input vazio é responsabilidade do endpoint | ❌ Divergência principal |
| Problema identificado | Incompleto, sem documentação para o caso | Falta verificar o body do erro; outros inputs inválidos não cobertos | ⚠️ Parcialmente alinhados |
| Risco identificado | Erro não tratado para o usuário | Stack trace exposta ou mensagem vazia ao atendente | ✅ Essencialmente o mesmo risco |

**Ponto de divergência:** O QA avaliou que o teste não deveria existir porque "não é uma pergunta sobre logística". O Claude entende que testar pergunta vazia é correto e necessário — o endpoint precisa rejeitar inputs inválidos independentemente do domínio. O problema não é o cenário, é que o teste só verifica o status 400 e não o corpo da resposta de erro.

**Recomendação:** Manter o cenário, corrigir a assertion para incluir verificação do body do erro.

---

## Teste 3 — Mock que mascara bug

| Aspecto | QA | Claude | Alinhamento |
|---|---|---|---|
| Problema do mock | Mock não testado com base na entrada | Mock declarado mas não conectado ao handler — provavelmente nunca intercepta nada | ✅ Alinhados — Claude detalha o mecanismo |
| Retorno não verificado | Parâmetros de entrada não verificados no retorno | Body de retorno, ID gerado e campos não validados | ✅ Alinhados |
| Erro de framework | Não mencionado | `jest.fn()` em projeto Vitest — violação do AGENTS.md, deveria ser `vi.fn()` | ⚠️ Claude acrescenta |
| Dados fora do domínio | Não mencionado | `rating: 5`, `comment: 'great'` em inglês, fora do domínio NovaTech | ⚠️ Claude acrescenta |
| Risco identificado | Retorno da API não verificado | Feedback nunca persistido mas teste passa — time vai a produção sem dados reais | ✅ Essencialmente o mesmo risco |

**Conclusão:** Avaliações alinhadas nos problemas centrais. Claude acrescenta o erro de framework (`jest` vs `vi`) como problema adicional relevante — é uma violação de convenção que pode causar comportamento inesperado dependendo da configuração de compatibilidade do Vitest.

---

## Resumo geral das divergências

| # | Divergência | Impacto na conclusão |
|---|---|---|
| Teste 1 | Claude mais detalhado, QA capturou o essencial | Nenhum — mesma conclusão: teste insuficiente |
| Teste 2 | QA questionou a validade do cenário; Claude defende que é válido | Médio — muda a ação: QA removeria o teste, Claude o corrigiria |
| Teste 3 | Claude acrescenta erro de framework (`jest` vs `vi`) | Baixo — mesma conclusão, Claude adiciona problema técnico relevante |

**Divergência mais importante — Teste 2:**
A questão não é se o cenário é do domínio de logística, mas se o endpoint trata corretamente inputs inválidos. Rejeitar perguntas vazias com 400 é comportamento esperado de qualquer API — e deve ser testado. A ação correta é corrigir o teste, não removê-lo.
