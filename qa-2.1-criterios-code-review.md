# QA 2.1 — Critérios de Code Review para Testes Gerados por IA

> Entregável do item 3 do Exercício 2.1 (QA).
> Critérios objetivos aplicáveis a qualquer teste gerado por Copilot ou outro agente de IA.
> Dois QAs aplicando esta lista independentemente devem chegar à mesma conclusão.

---

## Como usar este documento

Para cada teste em revisão, percorra os critérios em ordem. Qualquer **REPROVAR** encerra a revisão daquele teste — registre o critério violado no PR e solicite correção antes de continuar.

Um teste só passa no code review de QA se atender **todos** os critérios.

---

## Critério 1 — O nome do teste descreve um comportamento verificável

**Como verificar:**
Tente completar a frase: *"Este teste verifica que o sistema [comportamento] quando [condição]."*
Se o nome do `it(...)` não permitir completar a frase de forma específica, o critério falhou.

**Exemplos:**

| Nome do teste | Resultado | Motivo |
|---|---|---|
| `'should return source_document when question matches a chunk'` | ✅ APROVADO | Comportamento e condição claros |
| `'should return HTTP 400 when request body is missing the question field'` | ✅ APROVADO | Comportamento e condição claros |
| `'query endpoint works'` | ❌ REPROVADO | "Works" não define comportamento nem condição |
| `'test1'` | ❌ REPROVADO | Sem qualquer informação de comportamento |
| `'should work correctly'` | ❌ REPROVADO | "Correctly" não é verificável |

**Ação ao reprovar:** Solicitar renomeação seguindo o padrão `should [comportamento] when [condição]`.

---

## Critério 2 — Nenhuma assertion vaga

**Como verificar:**
Buscar no arquivo de teste as seguintes strings:
- `toBeDefined()`
- `toBeTruthy()`
- `toBeFalsy()`
- `toBeNull()`
- `not.toBeUndefined()`
- `not.toBeNull()`

Se qualquer uma aparecer como **única** assertion de um `it`, o critério falhou. Assertions vagas usadas em conjunto com assertions específicas são toleradas apenas se justificadas em comentário.

**Exemplos:**

```typescript
// ❌ REPROVADO — toBeDefined() como única assertion não verifica nada útil
expect(result).toBeDefined();

// ❌ REPROVADO — toBeTruthy() não diz o que deveria ser verdadeiro
expect(response.body).toBeTruthy();

// ✅ APROVADO — assertions específicas ao contrato da resposta
expect(response.status).toBe(200);
expect(body.answer).toBe('O prazo é de 7 dias úteis após o recebimento.');
expect(body.source_document).toBe('POL-001');
```

**Ação ao reprovar:** Solicitar substituição por assertions que verifiquem o contrato real da função — status HTTP, campos obrigatórios, valores esperados.

---

## Critério 3 — Serviços externos estão mockados com msw

**Como verificar:**
Identificar no teste qualquer uso de `SearchService`, `CompletionService`, clientes HTTP do Azure, ou qualquer `fetch`/`axios` direto. Para cada um, verificar se existe um handler `msw` correspondente no `arrange` do teste.

A ausência de qualquer import de `msw` num teste de integração que envolva serviços externos é sinal automático de reprovação.

**Exemplos:**

```typescript
// ❌ REPROVADO — chama serviço real; quebra no CI sem credenciais
const result = await realAzureSearchClient.search('Qual o prazo?');

// ❌ REPROVADO — sem msw; não há garantia de que a chamada HTTP foi interceptada
const response = await handler(makeRequest({ body: { question: 'Qual o prazo?' } }));

// ✅ APROVADO — msw intercepta a chamada antes que ela saia da máquina
server.use(
  http.post('https://novatech-search.search.windows.net/*', () =>
    HttpResponse.json({ value: [chunkPOL001A] })
  )
);
const response = await handler(makeRequest({ body: { question: queries.returnDeadline } }));
```

**Ação ao reprovar:** Solicitar adição dos handlers msw correspondentes. Se o teste for unitário (pasta `tests/unit/`), verificar se os módulos internos estão sendo mockados com `vi.mock()` em vez de chamar implementações reais.

---

## Critério 4 — Dados de teste são do domínio NovaTech e importados de fixtures

**Como verificar:**
Buscar no arquivo as seguintes strings em valores de campos `question`, `content`, `body`, ou similares:
- `"test"`
- `"hello"`
- `"foo"`, `"bar"`, `"baz"`
- `"dummy"`, `"placeholder"`, `"example"`
- `"string"`, `"value"`

Além disso, verificar se chunks, queries e respostas esperadas estão sendo declarados inline no arquivo de teste em vez de importados de `tests/fixtures/`.

**Exemplos:**

```typescript
// ❌ REPROVADO — placeholder sem sentido de domínio
const request = makeRequest({ body: '{"question": "test"}' });

// ❌ REPROVADO — chunk hardcoded inline em vez de fixture
const chunk = { id: 'abc', content: 'texto qualquer', score: 0.9 };

// ✅ APROVADO — dado real do domínio, importado de fixture
import { queries } from '../../fixtures/queries';
import { chunkPOL001A } from '../../fixtures/chunks';

const request = makeRequest({ body: { question: queries.returnDeadline } });
```

**Ação ao reprovar:** Solicitar substituição por perguntas reais do domínio de logística NovaTech e movimentação dos dados para `tests/fixtures/` se ainda não existirem lá.

---

## Critério 5 — Arrange/Act/Assert são seções explícitas com comentários

**Como verificar:**
Os comentários `// arrange`, `// act` e `// assert` devem estar presentes em todo `it(...)`. Testes com lógica colapsada em uma única expressão ou sem separação clara entre as três fases são reprovados.

**Exemplos:**

```typescript
// ❌ REPROVADO — sem separação de fases
it('should return 200 when question matches', async () => {
  expect((await handler(makeRequest({ body: { question: queries.returnDeadline } }))).status).toBe(200);
});

// ❌ REPROVADO — fases presentes mas sem comentários
it('should return 200 when question matches', async () => {
  const request = makeRequest({ body: { question: queries.returnDeadline } });
  const response = await handler(request);
  expect(response.status).toBe(200);
});

// ✅ APROVADO — três fases explícitas
it('should return 200 when question matches a chunk', async () => {
  // arrange
  const request = makeRequest({ body: { question: queries.returnDeadline } });

  // act
  const response = await handler(request);

  // assert
  expect(response.status).toBe(200);
});
```

**Ação ao reprovar:** Solicitar refatoração com as três seções explícitas. Se o `arrange` for vazio (ex: teste de função pura sem setup), o comentário `// arrange — none` deve estar presente para indicar que a omissão foi intencional.

---

## Resumo — Checklist rápido de review

Use esta tabela para agilizar o review de cada teste:

| # | Critério | Verificação rápida | Resultado |
|---|---|---|---|
| 1 | Nome descreve comportamento e condição | O `it(...)` completa: *"verifica que o sistema [x] quando [y]"*? | ☐ APROVADO / ☐ REPROVADO |
| 2 | Sem assertions vagas | Ausência de `toBeDefined()`, `toBeTruthy()` sozinhos | ☐ APROVADO / ☐ REPROVADO |
| 3 | Serviços externos mockados com msw | Handler msw presente para cada chamada HTTP externa | ☐ APROVADO / ☐ REPROVADO |
| 4 | Dados do domínio NovaTech em fixtures | Sem `"test"`, `"foo"`, `"dummy"`; imports de `tests/fixtures/` | ☐ APROVADO / ☐ REPROVADO |
| 5 | Arrange/Act/Assert explícitos | Comentários `// arrange`, `// act`, `// assert` presentes | ☐ APROVADO / ☐ REPROVADO |

> Um único REPROVADO encerra o review daquele teste. Registre o critério violado no comentário do PR.
