# QA 3.2 — Revisão dos Testes Gerados por IA (Claude)

> Segunda avaliação aplicada pelo Claude como co-reviewer.
> Contexto: 3 testes de integração gerados pelo Copilot para o projeto NovaTech Assistant.
> Framework do projeto: Vitest (não Jest).

---

## Teste 1 — Assertions vagas

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

**O que testa:** Que o endpoint retorna HTTP 200 e que o body existe para uma pergunta sobre prazo de devolução.

**O que falha em testar:**
- Não verifica o conteúdo da resposta — `answer`, `source_document`, `confidence_score` não são verificados
- Não verifica se a fonte citada é POL-001, que é o documento correto para esse tema
- Não usa mock para Azure AI Search e Azure OpenAI — chama serviços reais ou falha silenciosamente no CI
- O nome `'should return a response'` não descreve comportamento nem condição — viola o padrão `should [comportamento] when [condição]`
- `'prazo devolução'` é uma string literal sem contexto de domínio — deveria usar fixture de `queries.ts`
- Ausência de estrutura arrange/act/assert com comentários explícitos

**Risco se passar com código errado:** O endpoint pode retornar `{ answer: null, source_document: null }` e o teste continua verde. O assistente entrega resposta vazia ao atendente e ninguém detecta no CI.

---

## Teste 2 — Dados irreais

```typescript
describe('query endpoint edge cases', () => {
  it('should handle empty question', async () => {
    const res = await request(app).post('/api/query').send({ question: '' });
    expect(res.status).toBe(400);
  });
});
```

**O que testa:** Que o endpoint rejeita com HTTP 400 uma pergunta vazia — cenário de validação de input válido e importante.

**O que falha em testar:**
- Não verifica o body do erro — o atendente recebe qual mensagem? `{ error: 'question is required' }` ou uma stack trace exposta?
- Não cobre outros inputs inválidos igualmente relevantes: campo `question` ausente no body, tipo errado (número, array), pergunta com só espaços em branco
- O nome `'should handle empty question'` é aceitável mas "handle" é vago — deveria ser `'should return HTTP 400 with error message when question is empty'`

**Risco se passar com código errado:** O endpoint retorna 400 mas com body vazio `{}` ou com stack trace exposta — o teste passa, mas o atendente vê uma mensagem sem sentido ou informação técnica interna exposta ao usuário.

---

## Teste 3 — Mock que mascara bug

```typescript
describe('feedback endpoint', () => {
  it('should save feedback', async () => {
    const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
    const res = await request(app).post('/api/feedback').send({
      queryId: 'q1', rating: 5, comment: 'great'
    });
    expect(res.status).toBe(200);
    expect(mockCreate).toHaveBeenCalled();
  });
});
```

**O que testa:** Que o endpoint retorna 200 e que `mockCreate` foi chamado.

**O que falha em testar:**
- `mockCreate` foi declarado mas nunca conectado ao módulo real — o mock existe mas não está interceptando nada. A assertion `expect(mockCreate).toHaveBeenCalled()` provavelmente sempre falha ou nunca é atingida, tornando o teste inútil ou enganoso
- Usa `jest.fn()` — o projeto usa **Vitest**, não Jest. Violação direta das convenções do AGENTS.md. O correto seria `vi.fn()`
- Não verifica se os dados foram realmente persistidos — body de retorno, ID gerado, nenhum campo validado
- `rating: 5` e `comment: 'great'` são dados em inglês, fora do domínio NovaTech
- Não testa cenários de falha: `queryId` inexistente, `rating` inválido (ex: 10 ou negativo), `comment` ausente

**Risco se passar com código errado:** O feedback nunca é salvo — o mock não está conectado à implementação real — mas o teste passa porque `res.status` retorna 200 por outro motivo. O time conclui que a funcionalidade está funcionando e vai para produção sem feedback real sendo persistido. Nenhum dado de melhoria chega ao time.
