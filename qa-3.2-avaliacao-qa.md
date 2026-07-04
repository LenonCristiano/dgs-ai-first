# QA 3.2 — Revisão dos Testes Gerados por IA (QA)

> Avaliação própria aplicada pelo QA antes da comparação com o Claude.
> Contexto: 3 testes de integração gerados pelo Copilot para o projeto NovaTech Assistant.

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

**O que testa:** A resposta para uma pergunta sobre prazo de devolução.

**O que falha em testar:** Não verifica a resposta esperada para a pergunta.

**Risco se passar com código errado:** Corre o risco de dar a resposta errada para o atendente.

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

**O que testa:** A resposta para uma pergunta "vazia".

**O que falha em testar:** Está incompleto. Não é uma pergunta sobre logística e não existe documentação para esse caso.

**Risco se passar com código errado:** Corre o risco de apresentar um erro não tratado para o usuário.

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

**O que testa:** A criação de um mock que valida que a API retornou sucesso e que o mock foi chamado.

**O que falha em testar:** Não testa com base na entrada de dados. São passados parâmetros para a API que não são testados no retorno.

**Risco se passar com código errado:** O teste vai passar sem ser verificado o retorno da API chamada.
