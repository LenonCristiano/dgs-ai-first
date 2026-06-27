# QA 2.1 — Reescrita do Teste: Antes e Depois

> Entregável do item 2 do Exercício 2.1 (QA).
> Demonstra na prática os padrões definidos na seção Testing Standards do AGENTS.md.

---

## ANTES — Teste gerado pelo Copilot sem guidance

```typescript
// Teste gerado pelo Copilot sem guidance
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

### Problemas identificados

**Problema 1 — Nome não descreve comportamento nem condição**

`'query endpoint works'` não diz o que "funcionar" significa. Se o teste falhar, a mensagem de erro não ajuda a entender o que quebrou. Um bom nome deve completar a frase: *"Este teste verifica que o sistema [comportamento] quando [condição]."*

**Problema 2 — Dado de entrada é um placeholder sem significado de domínio**

`"test"` não é uma pergunta válida num sistema de atendimento de transportadora. Se o endpoint depende do conteúdo da pergunta para acionar o pipeline RAG, uma entrada sem sentido pode passar por caminhos de código que nunca seriam ativados em produção — ou simplesmente retornar erro silenciosamente.

**Problema 3 — Assertion vaga: `toBeDefined()`**

`expect(result).toBeDefined()` passa se o retorno for `{}`, `0`, `false`, ou qualquer coisa que não seja `undefined`. Não verifica status HTTP, não verifica se a resposta contém os campos obrigatórios (`answer`, `source_document`), não verifica se o conteúdo faz sentido. O teste dá falsa segurança: o sistema pode estar retornando lixo e o teste continua verde.

**Problema 4 — Sem estrutura Arrange/Act/Assert**

Setup, execução e verificação estão colapsados em duas linhas. É impossível identificar rapidamente o que é contexto, o que é a ação sendo testada, e o que está sendo verificado.

**Problema 5 — Sem mocks declarados**

O teste chama `handler` diretamente sem nenhuma indicação de que Azure AI Search e Azure OpenAI estão mockados. Se não estiverem, o teste faz chamadas reais a serviços externos — o que é lento, caro, não determinístico e quebra no CI sem credenciais configuradas.

---

## DEPOIS — Teste reescrito seguindo os Testing Standards

```typescript
import { describe, it, expect } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from '../../fixtures/msw-server';
import { chunkPOL001A, chunkPOL001B } from '../../fixtures/chunks';
import { queries } from '../../fixtures/queries';
import { makeRequest } from '../../fixtures/factories';
import { handler } from '../../../src/functions/query/handler';

describe('QueryHandler', () => {
  it('should return a response with source_document when question matches a chunk', async () => {
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
});
```

---

## Mapeamento das melhorias

| # | Problema no teste ruim | Correção aplicada | Padrão do AGENTS.md |
|---|---|---|---|
| 1 | Nome `'query endpoint works'` não descreve nada | Renomeado para `'should return a response with source_document when question matches a chunk'` — descreve comportamento e condição | Nomenclatura: `should [comportamento] when [condição]` |
| 2 | `"test"` como pergunta — placeholder sem domínio | Substituído por `queries.returnDeadline` (`'Qual o prazo de devolução de mercadorias?'`), importado de `tests/fixtures/queries.ts` | Dados de domínio realistas; fixtures em `tests/fixtures/` |
| 3 | `expect(result).toBeDefined()` — não verifica nada útil | Substituído por três assertions: status HTTP `200`, conteúdo de `body.answer`, presença e valor de `body.source_document`, e `body.confidence > 0.7` | Assertions específicas ao comportamento |
| 4 | Sem estrutura arrange/act/assert | Três seções explícitas com comentários obrigatórios | Arrange/Act/Assert explícitos em todo teste |
| 5 | Sem mocks — serviços externos potencialmente reais | msw intercepta Azure AI Search e Azure OpenAI com respostas controladas e determinísticas | msw para HTTP externo; nunca acessar serviços reais em testes |
| 6 | Chunk de resposta não especificado | `chunkPOL001A` e `chunkPOL001B` importados de `tests/fixtures/chunks.ts` — baseados nos chunks reais do Anexo B | Factories e fixtures para dados de teste; nunca hardcode inline |

---

## Fixtures referenciadas (conteúdo esperado)

Para que o teste reescrito compile e rode, as fixtures abaixo precisam existir em `tests/fixtures/`:

```typescript
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

// tests/fixtures/queries.ts
export const queries = {
  returnDeadline: 'Qual o prazo de devolução de mercadorias?',
  dangerousGoodsReturn: 'Posso devolver carga perigosa?',
  goldSLA: 'Qual o SLA para clientes Gold?',
  noMatch: 'Qual a política de estacionamento da empresa?',
};
```

> **Nota:** Os valores de `content` acima são baseados nos chunks reais do Anexo B (Chunks POL-001-A e POL-001-B), garantindo que os dados de teste reflitam o domínio real da NovaTech.
