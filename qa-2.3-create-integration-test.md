# SKILL.md — create-integration-test

**Nível:** Artifact
**Hierarquia:** Foundation → Domain → **Artifact**
**Slug:** `create-integration-test`

---

## Dependências — leia antes de usar esta skill

Esta skill pressupõe conhecimento das skills abaixo. Leia-as na ordem indicada antes de gerar qualquer teste:

1. `skills/foundation/typescript-conventions.md` — convenções de tipagem, imports e estrutura de arquivos
2. `skills/foundation/error-handling.md` — como erros são modelados no projeto (não use `try/catch` genérico)
3. `skills/domain/azure-functions-endpoint.md` — contrato de request/response das Azure Functions
4. `skills/domain/azure-ai-search-integration.md` — formato de resposta do Azure AI Search (necessário para montar mocks realistas)

---

## Quando usar esta skill

Use esta skill sempre que precisar gerar um teste de integração para qualquer módulo do projeto `novatech-assistant`.

**Frases-ativação — se o pedido contiver qualquer uma delas, esta skill se aplica:**
- "escreva um teste de integração para..."
- "crie um integration test para..."
- "adicione testes para o handler/service/endpoint..."
- "cubra este cenário com um teste..."
- "teste que o endpoint retorna X quando Y..."

**Esta skill NÃO se aplica a:**
- Testes unitários (sem chamadas entre módulos) → use `create-unit-test` quando disponível
- Testes E2E (fluxo completo com serviços reais) → requerem aprovação explícita do QA
- Testes de performance/carga → fora do escopo desta skill

**Localização dos arquivos gerados:** `tests/integration/`

---

## Template base

Copie e preencha os placeholders marcados com `<PLACEHOLDER>`. Não remova os comentários de seção.

```typescript
import { describe, it, expect, beforeAll, afterAll, afterEach } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from '../fixtures/msw-server';
import { <FIXTURE_IMPORTS> } from '../fixtures/chunks';
import { queries } from '../fixtures/queries';
import { makeRequest } from '../fixtures/factories';
import { <MODULE_UNDER_TEST> } from '../../src/<MODULE_PATH>';

describe('<ModuleName>', () => {
  afterEach(() => {
    server.resetHandlers();
  });

  describe('<method or behavior group>', () => {
    it('should <expected behavior> when <condition>', async () => {
      // arrange
      server.use(
        http.post('<AZURE_SEARCH_ENDPOINT>', () =>
          HttpResponse.json(<MOCK_SEARCH_RESPONSE>)
        ),
        http.post('<AZURE_OPENAI_ENDPOINT>', () =>
          HttpResponse.json(<MOCK_COMPLETION_RESPONSE>)
        )
      );
      const request = makeRequest({ body: { question: queries.<QUERY_KEY> } });

      // act
      const response = await <MODULE_UNDER_TEST>(request);
      const body = JSON.parse(response.body);

      // assert
      expect(response.status).toBe(<EXPECTED_STATUS>);
      expect(body.<FIELD>).toBe(<EXPECTED_VALUE>);
      expect(body.source_document).toBe('<EXPECTED_SOURCE>');
    });
  });
});
```

**Regras de preenchimento:**
- `<ModuleName>` — nome da classe ou módulo sendo testado (ex: `QueryHandler`, `SearchService`)
- `<FIXTURE_IMPORTS>` — importe apenas os chunks necessários para o cenário (ex: `chunkPOL001A`, `chunkSLA2024B`)
- `<QUERY_KEY>` — use sempre uma chave de `queries.ts` (ex: `queries.returnDeadline`), nunca string literal
- `<EXPECTED_SOURCE>` — o ID do documento de origem real (ex: `"POL-001"`, `"SLA-2024"`)
- `afterEach(() => server.resetHandlers())` — obrigatório em todo `describe` que usa msw

---

## Exemplos completos

### ✅ DO — Teste bem escrito

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
    it('should return HTTP 200 with answer and source_document when question is about return deadline', async () => {
      // arrange
      server.use(
        http.post('https://novatech-search.search.windows.net/*', () =>
          HttpResponse.json({ value: [chunkPOL001A, chunkPOL001B] })
        ),
        http.post('https://novatech-openai.openai.azure.com/*', () =>
          HttpResponse.json({
            choices: [{ message: { content: 'O prazo é de 7 dias úteis após o recebimento.' } }],
          })
        )
      );
      const request = makeRequest({ body: { question: queries.returnDeadline } });

      // act
      const response = await handler(request);
      const body = JSON.parse(response.body);

      // assert
      expect(response.status).toBe(200);
      expect(body.answer).toBe('O prazo é de 7 dias úteis após o recebimento.');
      expect(body.source_document).toBe('POL-001');
      expect(body.confidence).toBeGreaterThan(0.7);
    });

    it('should return explicit negation and source_document POL-001 when question is about dangerous goods return', async () => {
      // arrange
      server.use(
        http.post('https://novatech-search.search.windows.net/*', () =>
          HttpResponse.json({ value: [chunkPOL001B] })
        ),
        http.post('https://novatech-openai.openai.azure.com/*', () =>
          HttpResponse.json({
            choices: [{ message: { content: 'Cargas perigosas não são elegíveis para devolução pelo processo padrão. Entre em contato com Gestão de Riscos pelo ramal 4500.' } }],
          })
        )
      );
      const request = makeRequest({ body: { question: queries.dangerousGoodsReturn } });

      // act
      const response = await handler(request);
      const body = JSON.parse(response.body);

      // assert
      expect(response.status).toBe(200);
      expect(body.answer).toContain('não');
      expect(body.answer).toContain('4500');
      expect(body.source_document).toBe('POL-001');
      expect(body.dangerous_goods_return).toBe(false);
    });
  });

  describe('when no chunk matches the question', () => {
    it('should return HTTP 200 with not-found message and null source_document', async () => {
      // arrange
      server.use(
        http.post('https://novatech-search.search.windows.net/*', () =>
          HttpResponse.json({ value: [] })
        )
      );
      const request = makeRequest({ body: { question: queries.noMatch } });

      // act
      const response = await handler(request);
      const body = JSON.parse(response.body);

      // assert
      expect(response.status).toBe(200);
      expect(body.answer).toContain('não encontrei informações');
      expect(body.source_document).toBeNull();
    });
  });
});
```

**Por que este teste é correto:**
- Nome descreve comportamento e condição em cada `it`
- Arrange/Act/Assert explícitos com comentários
- Dados do domínio NovaTech importados de fixtures (`queries.returnDeadline`, `chunkPOL001A`)
- msw intercepta todas as chamadas externas
- Assertions verificam campos específicos do contrato (`answer`, `source_document`, `confidence`, `dangerous_goods_return`)
- `afterEach` garante isolamento entre testes
- Campo `source_document` verificado como `null` quando não há match — não como string vazia ou valor inventado

---

### ❌ DON'T — Teste com problemas comuns gerados por IA

```typescript
// ❌ PROBLEMAS: nome vago, sem mocks, dado placeholder, assertion vaga,
//              sem arrange/act/assert, acesso a serviço real implícito

import { handler } from '../../src/functions/query/handler';

test('handler works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
  expect(result.statusCode).toBeTruthy();
});

test('handler returns source', async () => {
  const result = await handler({ body: '{"question": "devolução"}' });
  expect(result.body).toContain('source');
});

test('handler fails gracefully', async () => {
  const result = await handler({ body: '{}' });
  expect(result).not.toBeNull();
});
```

**Por que estes testes são problemáticos:**

| Problema | Onde aparece | Impacto |
|---|---|---|
| Nome vago — não descreve comportamento nem condição | `'handler works'`, `'handler returns source'` | Falha não indica o que quebrou |
| Sem mocks msw — chama Azure AI Search e OpenAI reais | Todos os testes | Quebra no CI, consome tokens, não determinístico |
| Dado placeholder `"test"` e `"devolução"` sem fixture | `body: '{"question": "test"}'` | Não representa cenário real; pode passar por código morto |
| `toBeDefined()` e `toBeTruthy()` como única assertion | `expect(result).toBeDefined()` | Passa com qualquer retorno, inclusive erro silencioso |
| `toContain('source')` sem verificar valor | `expect(result.body).toContain('source')` | Passa se `source_document` for `null`, string vazia ou valor inventado |
| Sem estrutura arrange/act/assert | Todos os testes | Impossível identificar setup vs ação vs verificação |
| Sem `describe` agrupando por módulo | Arquivo inteiro | Testes soltos sem contexto de qual módulo estão testando |

---

## Anti-padrões específicos de testes gerados por IA

Os itens abaixo são comportamentos recorrentes de LLMs ao gerar testes. Ao revisar código gerado, procure ativamente por cada um.

**Anti-padrão 1 — Assertion de existência no lugar de assertion de valor**
O modelo gera `toBeDefined()` ou `toBeTruthy()` porque é a assertion mais "segura" — nunca falha. Sinal: o único `expect` do teste não verifica nenhum campo específico.

**Anti-padrão 2 — Dado genérico no lugar de dado de domínio**
O modelo usa `"test"`, `"hello"`, `"foo"` em campos de entrada porque não conhece o domínio. Sinal: strings de entrada não fazem sentido como perguntas de atendente de transportadora.

**Anti-padrão 3 — Mock incompleto: mocka um serviço, esquece o outro**
O modelo adiciona mock para Azure AI Search mas esquece o Azure OpenAI (ou vice-versa), deixando uma chamada real escapar. Sinal: o teste funciona localmente com credenciais mas quebra no CI.

**Anti-padrão 4 — Teste que verifica implementação, não comportamento**
O modelo verifica se uma função interna foi chamada (`expect(searchService.query).toHaveBeenCalled()`) em vez de verificar o resultado observável pelo cliente (`expect(response.status).toBe(200)`). Sinal: o teste quebra quando o código é refatorado mesmo sem mudar o comportamento externo.

**Anti-padrão 5 — Estado compartilhado entre testes**
O modelo declara variáveis no escopo do `describe` e as modifica dentro de `it`, criando dependência de ordem. Sinal: testes passam quando rodados juntos mas falham quando rodados isoladamente com `vitest run --testNamePattern`.

**Anti-padrão 6 — `describe` aninhados sem propósito**
O modelo cria múltiplos níveis de `describe` sem critério claro (ex: `describe > describe > describe > it`), tornando o output do runner ilegível. Sinal: mais de 2 níveis de aninhamento sem que cada nível agrupe por critério distinto (módulo → método → condição).

**Anti-padrão 7 — Teste que nunca pode falhar**
O modelo gera um teste cujo critério de aprovação é tão amplo que não existe implementação possível que o quebre. Sinal: remover toda a lógica do handler e substituir por `return {}` ainda faz o teste passar.
