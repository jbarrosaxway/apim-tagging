# 📋 Padrão de Nomenclatura para Headers com Selectors no Axway API Manager
## Guia Prático

---

## 📋 Índice Rápido

1. [Visão Geral](#visão-geral)
2. [Padrão de Nomenclatura](#padrão-de-nomenclatura-para-headers-com-selectors)
3. [Sintaxe do Selector](#sintaxe-do-selector)
4. [Processamento Automático de Vaults AWS](#processamento-automático-de-vaults-aws)
5. [Exemplos Práticos](#exemplos-práticos)
6. [Fluxo Visual](#fluxo-visual)

---

## 🎯 Visão Geral

### O que são Outbound Parameters?

Os **Outbound Parameters** no **Axway API Manager** são usados para mapear e transformar parâmetros (headers, query params, etc.) antes de enviar a requisição para o backend.

Quando você precisa de **lógica dinâmica** para escolher valores (por exemplo, selecionar diferentes vaults baseado em um header), você pode usar **selectors** com expressões ternárias.

**Padrão de Nomenclatura:** Para facilitar a identificação e processamento automático, use o padrão `custom-<nome-header>-selector` no nome do **OUTBOUND PARAMETER**.

---

## 🔄 Padrão de Nomenclatura para Headers com Selectors

**Convenção de Nomenclatura:** Use uma **convenção de nomenclatura** nos headers outbound para documentar e processar mapeamentos dinâmicos com selectors.

**Padrão de Nomenclatura:**
```
custom-<nome-header>-selector
```

**Como Funciona:**
1. **Header Outbound no API Manager:** `custom-client_id-selector` (configurado na interface)
2. **Política de Roteamento:** Extrai o nome do header da string (remove `custom-` e `-selector`)
3. **Header Enviado ao Backend:** `client_id` (nome extraído)
4. **Valor:** Resultado do selector configurado no campo `OUTBOUND VALUE`
5. **⚠️ Headers de Trânsito:** O header `custom-client_id-selector` é **removido** após o processamento e **não é enviado** para o backend. Apenas o header final (`client_id`) é enviado.

**Exemplo na Interface do API Manager:**

```
┌─────────────────────────────────────────────────────────┐
│  OUTBOUND PARAMETERS                                    │
├─────────────────────────────────────────────────────────┤
│  OUTBOUND PARAMETER    │ TYPE │ OUTBOUND VALUE         │
├─────────────────────────────────────────────────────────┤
│  custom-client_id-sel  │header│ ${http.headers['Product-│
│                        │      │ Type'] == 'PRODUCT_A' ? │
│                        │      │ 'vault://.../client_   │
│                        │      │ id_a' : ...}            │
└─────────────────────────────────────────────────────────┘
```

**Exemplo Completo:**

**Configuração no API Manager:**
- **OUTBOUND PARAMETER:** `custom-client_id-selector`
- **PARAMETER TYPE:** `header`
- **REQUIRED:** `true` (toggle ativado)
- **OUTBOUND VALUE:** `${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../client_id_a' : 'vault://aws/.../client_id_default'}`

**Processamento na Política de Roteamento:**
1. Lê o header outbound: `custom-client_id-selector`
2. Extrai o nome: remove `custom-` e `-selector` → `client_id`
3. Avalia o selector: `${http.headers['Product-Type'] == 'PRODUCT_A' ? ...}`
4. **Se o valor resultante começar com `vault://aws/`:** A política automaticamente chama o filtro de recuperação de secret da AWS
5. Define header para backend: `client_id = <valor recuperado do vault ou valor literal>`
6. **Remove header de trânsito:** O header `custom-client_id-selector` é **removido** e não é enviado para o backend

**Resultado Final:**
```
Request para Backend:
  Headers:
    - client_id: <valor do vault selecionado pelo selector>
```

---

## 📝 Sintaxe do Selector (Formato Policy Studio)

**Formato:**
```
${http.headers['Header-Name'] == 'valor' ? 'valor_se_verdadeiro' : 'valor_se_falso'}
```

**Exemplos de Selectors:**
- `${http.headers['Product-Type']}` - Valor de um header
- `${http.query.param}` - Valor de query parameter
- `${api.property.name}` - Propriedade customizada da API
- `${environment.variable}` - Variável de ambiente

**⚠️ Importante:** No Policy Studio, use:
- `http.headers['Header-Name']` (com colchetes e aspas) para acessar headers
- Valores do ternário entre aspas simples: `'vault://...'`
- Pode usar ternários aninhados para múltiplas condições

---

## 🔐 Processamento Automático de Vaults AWS

### Como Funciona

Quando o valor resultante do selector **começa com `vault://aws/`**, a política de roteamento **automaticamente** faz a chamada para o **filtro de recuperação de secret da AWS**.

**Fluxo de Processamento:**

```
┌─────────────────────────────────────────────────────────┐
│  1. Avalia o Selector                                    │
│     ${http.headers['Product-Type'] == 'PRODUCT_A' ? ...}│
│     → Resultado: 'vault://aws/.../client_id_a'           │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  2. Detecta Prefixo vault://aws/                          │
│     ✅ Valor começa com 'vault://aws/'                    │
│     → Aciona filtro de recuperação de secret da AWS     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  3. Chama Filtro AWS Secrets Manager                     │
│     ┌────────────────────────────────────────────────┐  │
│     │ Filtro: AWS Secrets Manager                    │  │
│     │ Path: vault://aws/.../client_id_a              │  │
│     │ → Recupera secret do AWS Secrets Manager        │  │
│     │ → Retorna valor do secret                       │  │
│     └────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  4. Define Header para Backend                           │
│     client_id = <valor recuperado do AWS Secrets Manager>│
└─────────────────────────────────────────────────────────┘
```

**Exemplo Prático:**

**Selector:**
```
${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/my-organization/client_id_a' : 'vault://aws/my-organization/client_id_default'}
```

**Processamento:**
1. **Avalia condição:** `Product-Type == 'PRODUCT_A'` → `true`
2. **Resultado do selector:** `'vault://aws/my-organization/client_id_a'`
3. **Detecta prefixo:** Valor começa com `vault://aws/` → **Aciona filtro AWS**
4. **Filtro AWS Secrets Manager:**
   - Conecta ao AWS Secrets Manager
   - Recupera o secret no caminho: `my-organization/client_id_a`
   - Retorna o valor do secret (ex: `abc123xyz`)
5. **Define header:** `client_id = abc123xyz`

**Se o valor NÃO começar com `vault://aws/`:**
- O valor é usado literalmente, sem chamada ao filtro AWS
- Exemplo: Se o selector retornar `'valor-literal'`, o header será `client_id = valor-literal`

**⚠️ Importante:**
- O prefixo `vault://aws/` é **obrigatório** para acionar o filtro de recuperação de secret
- O caminho após `vault://aws/` deve corresponder ao caminho configurado no AWS Secrets Manager
- A política de roteamento deve ter o filtro AWS Secrets Manager configurado e habilitado

---

## 🔄 Headers de Trânsito

### O que são Headers de Trânsito?

Os headers com o padrão `custom-<nome-header>-selector` são **headers de trânsito** (temporary headers) usados apenas para processamento interno na política de roteamento.

### Comportamento

**Antes do Processamento:**
```
Request Interno:
  Headers:
    - custom-client_id-selector: ${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../client_id_a' : 'vault://aws/.../client_id_default'}
    - Product-Type: PRODUCT_A
```

**Durante o Processamento:**
1. A política lê o header `custom-client_id-selector`
2. Extrai o nome do header final: `client_id`
3. Avalia o selector e recupera o valor (do vault ou literal)
4. Define o header final: `client_id = <valor>`
5. **Remove o header de trânsito:** `custom-client_id-selector` é removido

**Após o Processamento (Request para Backend):**
```
Request para Backend:
  Headers:
    ✅ client_id: abc123xyz (valor recuperado do vault)
    ✅ Product-Type: PRODUCT_A
    ❌ custom-client_id-selector: (removido, não enviado)
```

### Por que Remover?

- ✅ **Segurança:** Evita expor informações internas de configuração ao backend
- ✅ **Limpeza:** Mantém apenas os headers necessários na requisição final
- ✅ **Padrão:** Headers de configuração não devem ser enviados ao backend
- ✅ **Clareza:** O backend recebe apenas o header final com o valor correto

### Exemplo Visual

```
┌─────────────────────────────────────────────────────────┐
│  ANTES DO PROCESSAMENTO                                 │
│  Headers:                                               │
│    - custom-client_id-selector: ${...}                  │
│    - Product-Type: PRODUCT_A                             │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼ Processamento
┌─────────────────────────────────────────────────────────┐
│  DURANTE O PROCESSAMENTO                               │
│  1. Lê custom-client_id-selector                        │
│  2. Extrai: client_id                                   │
│  3. Avalia selector → 'vault://aws/.../client_id_a'   │
│  4. Recupera do vault → 'abc123xyz'                    │
│  5. Define: client_id = 'abc123xyz'                    │
│  6. Remove: custom-client_id-selector ❌               │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  APÓS O PROCESSAMENTO (Request para Backend)            │
│  Headers:                                               │
│    ✅ client_id: abc123xyz                              │
│    ✅ Product-Type: PRODUCT_A                            │
│    ❌ custom-client_id-selector: (removido)             │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 Exemplos Práticos

### Exemplo 1: Selector Simples

**Configuração:**
- **OUTBOUND PARAMETER:** `custom-client_id-selector`
- **PARAMETER TYPE:** `header`
- **OUTBOUND VALUE:** `${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../client_id_a' : 'vault://aws/.../client_id_default'}`

**Resultado:**
- **Header enviado ao backend:** `client_id`
- **Valor:** Depende do valor do header `Product-Type` na requisição

**Lógica:**
- Se `Product-Type == 'PRODUCT_A'` → usa `vault://aws/.../client_id_a`
- Senão → usa `vault://aws/.../client_id_default`

---

### Exemplo 2: Ternário Aninhado (Múltiplas Condições)

**Configuração:**
- **OUTBOUND PARAMETER:** `custom-client_id-selector`
- **PARAMETER TYPE:** `header`
- **OUTBOUND VALUE:** `${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../client_id_a' : (http.headers['Product-Type'] == 'PRODUCT_B' ? 'vault://aws/.../client_id_b' : (http.headers['Product-Type'] == 'PRODUCT_C' ? 'vault://aws/.../client_id_c' : 'vault://aws/.../client_id_default'))}`

**Lógica do Ternário Aninhado (equivalente a IF-ELSE IF-ELSE IF-ELSE):**
1. **IF** `Product-Type == 'PRODUCT_A'` → usa `client_id_a`
2. **ELSE IF** `Product-Type == 'PRODUCT_B'` → usa `client_id_b`
3. **ELSE IF** `Product-Type == 'PRODUCT_C'` → usa `client_id_c`
4. **ELSE** → usa `client_id_default`

**Resultado:**
- **Header enviado ao backend:** `client_id`
- **Valor:** Escolhido dinamicamente baseado no `Product-Type`

---

### Exemplo 3: Múltiplos Headers

**Configuração no API Manager:**

```
OUTBOUND PARAMETER: custom-client_id-selector
PARAMETER TYPE: header
OUTBOUND VALUE: ${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../client_id_a' : 'vault://aws/.../client_id_default'}

OUTBOUND PARAMETER: custom-access_token-selector
PARAMETER TYPE: header
OUTBOUND VALUE: ${http.headers['Product-Type'] == 'PRODUCT_A' ? 'vault://aws/.../access_token_a' : 'vault://aws/.../access_token_default'}
```

**Resultado:**
```
Request para Backend:
  Headers:
    - client_id: <valor do selector>
    - access_token: <valor do selector>
```

---

## 🎨 Fluxo Visual

```
┌─────────────────────────────────────────────────────────┐
│  API MANAGER - OUTBOUND PARAMETERS                     │
│  custom-client_id-selector                              │
│  OUTBOUND VALUE: ${http.headers['Product-Type'] == ...} │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  POLÍTICA DE ROTEAMENTO                                 │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 1. Lê: custom-client_id-selector                │  │
│  │ 2. Extrai: client_id (remove custom- e -selector)│  │
│  │ 3. Avalia selector: ${http.headers[...]}        │  │
│  │ 4. Define header: client_id = <valor>           │  │
│  │ 5. Remove header de trânsito:                   │  │
│  │    custom-client_id-selector ❌ (removido)      │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  REQUEST PARA BACKEND                                   │
│  Headers:                                               │
│    ✅ client_id: <valor do vault selecionado>          │
│    ❌ custom-client_id-selector: (NÃO enviado)         │
└─────────────────────────────────────────────────────────┘
```

**⚠️ Importante:** Os headers com padrão `custom-<nome-header>-selector` são **headers de trânsito** usados apenas para processamento interno. Eles são **automaticamente removidos** antes de enviar a requisição para o backend. Apenas o header final (extraído do nome) é enviado.

---

## ✅ Vantagens desta Abordagem

- ✅ **Auto-documentação:** O nome do header já indica que é um selector
- ✅ **Visível na interface:** Fácil de ver na configuração do API Manager
- ✅ **Padrão consistente:** Todos os headers com selector seguem o mesmo padrão
- ✅ **Menos configuração:** Tudo em um lugar (outbound parameters)
- ✅ **Processamento automático:** A política pode extrair o nome do header automaticamente

---

**Versão**: 1.0  
**Última Atualização**: 24/11/2025

