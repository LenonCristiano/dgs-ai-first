# AGENTS.md — NovaTech Assistant

> Constitution do projeto. Todo agente de IA (Copilot, Claude Code) lê este arquivo antes de gerar qualquer artefato.
> As seções abaixo são preenchidas por papéis diferentes nos exercícios do Cenário 2.

## Project Overview
<!-- TODO (Tech Lead — Ex. 2.1) -->

## Tech Stack & Architecture
<!-- TODO (Tech Lead — Ex. 2.1): inclui regras de gerenciamento de contexto da ADR-0002 -->

## Coding Standards (Tech Lead)
<!-- TODO (Tech Lead — Ex. 2.1) -->

## Product Rules & Guardrails (Product Specialist)
<!-- TODO (Product Specialist — Ex. 2.3) -->

## Testing Standards (QA)

> Esta seção é prescritiva. Todo agente de IA que gerar código de teste DEVE seguir estas regras sem exceção.
> Desvios exigem aprovação explícita do QA no PR.

### Framework e ferramentas

- **Runner:** Vitest
- **Mocks HTTP:** msw (Mock Service Worker) — obrigatório para qualquer chamada a Azure AI Search, Azure OpenAI ou APIs externas
- **Factories:** funções utilitárias em `tests/fixtures/` para gerar dados de teste — nunca hardcode inline
- **Coverage mínimo:** 80% de linhas; o CI rejeita PRs abaixo disso

### Nomenclatura

Todo teste usa `describe` + `it` com frases em inglês que descrevem comportamento e condição:

```typescript
// ✅ CORRETO
describe('QueryHandler', () => {
  it('should return a response with source_document when question matches a chunk', async () => { ... });
  it('should return a not-found message when no chunk matches the question', async () => { ... });
  it('should return HTTP 400 when request body is missing the question field', async () => { ... });
});

// ❌ ERRADO — não descreve comportamento nem condição
test('query endpoint works', async () => { ... });
test('test1', async () => { ... });
```

Formato obrigatório para o `it`: **`should [comportamento esperado] when [condição]`**

### Estrutura interna de cada teste

Todo teste deve ter as três seções explícitas com comentário:

```typescript
it('should return HTTP 400 when request body is missing the question field', async () => {
  // arrange
  const request = makeRequest({ body: '{}' });

  // act
  const response = await handler(request);

  // assert
  expect(response.status).toBe(400);
  expect(response.body).toMatchObject({ error: 'question is required' });
});
```

Seções misturadas ou implícitas são reprovadas em review.

### O que todo teste DEVE ter

- **Arrange/Act/Assert** explícitos (comentários obrigatórios)
- **Assertions específicas** ao comportamento: verificar status HTTP, campos do body, tipos, valores concretos
- **Dados de domínio realistas**: perguntas sobre logística, chunks da NovaTech, não strings genéricas como `"test"` ou `"hello"`
- **Mocks explícitos**: deixar claro no topo do teste o que está sendo mockado e com qual resposta
- **Um único comportamento por teste**: se o nome do `it` tem "and", provavelmente são dois testes

### O que todo teste NÃO DEVE ter

```typescript
// ❌ Assertions vagas
expect(result).toBeDefined();
expect(result).toBeTruthy();
expect(result).not.toBeNull();

// ❌ Acesso a serviços reais (Azure AI Search, Azure OpenAI, banco)
const result = await realAzureSearchClient.search('...');

// ❌ Dependência de ordem entre testes
let sharedState: string;
it('sets state', () => { sharedState = 'foo'; });
it('uses state', () => { expect(sharedState).toBe('foo'); }); // quebra se rodar isolado

// ❌ Dados hardcoded espalhados no arquivo
const chunk = { id: 'abc123', content: 'texto qualquer', score: 0.9 };

// ❌ Console.log dentro de testes
console.log(result); // remove antes do commit
```

### Padrão de mocking com msw

Use msw para interceptar chamadas HTTP externas. O handler deve ser declarado no nível do `describe`, não dentro do `it`:

```typescript
import { http, HttpResponse } from 'msw';
import { server } from '../fixtures/msw-server';

describe('SearchService', () => {
  it('should return ranked chunks when Azure AI Search responds with results', async () => {
    // arrange
    server.use(
      http.post('https://novatech-search.search.windows.net/*', () =>
        HttpResponse.json(makeAzureSearchResponse([chunkPOL001A, chunkPOL001B]))
      )
    );

    // act
    const result = await searchService.query('Qual o prazo de devolução?');

    // assert
    expect(result.chunks).toHaveLength(2);
    expect(result.chunks[0].id).toBe('POL-001-A');
  });
});
```

Nunca mocke com `jest.fn()` ou `vi.fn()` chamadas HTTP — use msw. Para módulos internos, `vi.mock()` é permitido.

### Padrão de fixtures

Fixtures ficam em `tests/fixtures/` e são importadas — nunca recriadas em cada arquivo de teste.

```
tests/fixtures/
├── chunks.ts          # Chunks simulados do pipeline RAG (baseados no Anexo B)
├── queries.ts         # Perguntas de teste do domínio de logística NovaTech
├── expected-responses.ts  # Respostas esperadas por cenário
└── msw-server.ts      # Instância compartilhada do server msw
```

Exemplo de uso:

```typescript
// tests/fixtures/chunks.ts
export const chunkPOL001A = {
  id: 'POL-001-A',
  content: 'O cliente pode solicitar a devolução de mercadorias em até 7 (sete) dias úteis...',
  source_document: 'POL-001',
  section: '3.1',
  score: 0.95,
};

// tests/fixtures/queries.ts
export const queries = {
  returnDeadline: 'Qual o prazo de devolução de mercadorias?',
  dangerousGoodsReturn: 'Posso devolver carga perigosa?',
  goldSLA: 'Qual o SLA para clientes Gold?',
  noMatch: 'Qual a política de estacionamento da empresa?',
};
```

Ao criar novos chunks ou queries para testes, adicione ao fixture correspondente — não crie inline no arquivo de teste.

### Localização dos testes

| Tipo | Pasta | Regra |
|------|-------|-------|
| Unitário | `tests/unit/` | Sem chamadas externas. Tudo mockado. |
| Integração | `tests/integration/` | APIs externas mockadas com msw. Módulos internos reais. |
| E2E | `tests/e2e/` | Fluxo completo. Uso restrito — consome tokens reais. Aprovação do QA obrigatória para adicionar. |

## Project Management Rules (Delivery Manager)
<!-- TODO (Delivery Manager — Ex. 2.3) -->

## Build & Deploy
<!-- TODO (Tech Lead — Ex. 2.1) -->
