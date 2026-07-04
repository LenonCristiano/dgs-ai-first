# QA 3.2 — Teste 1 Reescrito

> Entregável do item 3 do Exercício 3.2 (QA).
> O Teste 1 original foi reescrito seguindo os Testing Standards definidos no AGENTS.md.

---

## ANTES — Teste original gerado pelo Copilot

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

**Problemas:**
- Nome vago — não descreve comportamento nem condição
- `expect(res.body).toBeDefined()` — assertion vaga, passa com body vazio ou nulo
- Sem mock de Azure AI Search e Azure OpenAI — chama serviços reais ou falha silenciosamente
- Dado de entrada `'prazo devolução'` é string literal sem contexto de domínio
- Sem estrutura arrange/act/assert
- Não verifica `answer`, `source_document` nem `confidence_score`

---

## DEPOIS — Teste reescrito seguindo os Testing Standards

```typescript
import { describe, it, expect, afterEach } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from '../fixtures/msw-server';
import { chunkPOL001A, chunkPOL001B } from '../fixtures/chunks';
import { queries } from '../fixtures/queries';
import { makeRequest } from '../fixtures/factories';
import { handler } from '../../src/functions/query/handler';

describe('QueryHandler', () => {
  afterEach(() => {
    server.resetHandlers();
  });

  describe('when question matches a chunk', () => {
    it('should return HTTP 200 with answer and source_document POL-001 when question is about return deadline', async () => {
      // arrange
      server.use(
        http.post('https://novatech-search.search.windows.net/*', () =>
          HttpResponse.json({ value: [chunkPOL001A, chunkPOL001B] })
        ),
        http.post('https://novatech-openai.openai.azure.com/*', () =>
          HttpResponse.json({
            choices: [{
              message: {
                content: 'O prazo de devolução é de 7 dias úteis após o recebimento confirmado no sistema de tracking.',
              },
            }],
          })
        )
      );
      const request = makeRequest({ body: { question: queries.returnDeadline } });

      // act
      const response = await handler(request);
      const body = JSON.parse(response.body);

      // assert
      expect(response.status).toBe(200);
      expect(body.answer).toBe('O prazo de devolução é de 7 dias úteis após o recebimento confirmado no sistema de tracking.');
      expect(body.source_document).toBe('POL-001');
      expect(body.confidence_score).toBeGreaterThan(0.7);
    });
  });
});
```

---

## O que mudou e por quê

| Problema original | Correção aplicada |
|---|---|
| Nome `'should return a response'` — vago | Renomeado para `'should return HTTP 200 with answer and source_document POL-001 when question is about return deadline'` — descreve comportamento e condição |
| `'prazo devolução'` como string literal | Substituído por `queries.returnDeadline` importado de `tests/fixtures/queries.ts` |
| Sem mocks declarados | msw intercepta Azure AI Search e Azure OpenAI com respostas controladas e determinísticas |
| `expect(res.body).toBeDefined()` | Substituído por três assertions específicas: status 200, conteúdo de `body.answer`, `body.source_document` e `body.confidence_score` |
| Sem arrange/act/assert | Três seções explícitas com comentários obrigatórios |
| Chunks de resposta não especificados | `chunkPOL001A` e `chunkPOL001B` importados de `tests/fixtures/chunks.ts` |
| `afterEach` ausente | Adicionado `server.resetHandlers()` para garantir isolamento entre testes |

---

## Fixtures referenciadas

Para que o teste compile e rode, as fixtures abaixo precisam existir em `tests/fixtures/`:

```typescript
// tests/fixtures/queries.ts
export const queries = {
  returnDeadline: 'Qual o prazo de devolução de mercadorias?',
  // ...demais queries
};

// tests/fixtures/chunks.ts
export const chunkPOL001A = {
  id: 'POL-001-A',
  content: 'O cliente pode solicitar a devolução de mercadorias em até 7 (sete) dias úteis após a data de recebimento confirmada no sistema de tracking.',
  source_document: 'POL-001',
  section: '3.1',
  score: 0.95,
};

export const chunkPOL001B = {
  id: 'POL-001-B',
  content: 'As seguintes categorias de carga NÃO são elegíveis para devolução pelo processo padrão: Cargas perigosas classificadas nas classes 1 a 6 da ANTT...',
  source_document: 'POL-001',
  section: '3.2',
  score: 0.88,
};
```
