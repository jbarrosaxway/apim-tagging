# Melhores Práticas de Tagging de Recursos
## Amplify API Management e Amplify Engage

---

## 1. Introdução

A implementação de uma estratégia eficaz de tagging (etiquetagem) de recursos é fundamental para maximizar o valor das plataformas **Amplify API Management** e **Amplify Engage**. Esta documentação apresenta as melhores práticas para organizar, governar e monitorar recursos por meio de tags estruturadas.

### 1.1 Objetivos do Tagging

- **Organização**: Facilitar a descoberta e categorização de recursos
- **Governança**: Aplicar políticas e controles de forma consistente
- **Monitoramento**: Habilitar relatórios e análises baseados em tags
- **Segurança**: Identificar e proteger recursos críticos
- **Conformidade**: Garantir aderência a padrões organizacionais

### 1.2 Metadados vs Tags: Evitando Redundância

**IMPORTANTE**: Antes de criar tags, identifique quais informações já estão disponíveis como metadados nas plataformas. Tags devem complementar metadados, não duplicá-los.

#### 1.2.1 Metadados Padrão Disponíveis

As plataformas **Amplify API Management** e **Amplify Engage** já armazenam os seguintes metadados padrão (não devem ser duplicados como tags):

**Metadados de Identificação da API (Amplify API Management):**
- ✅ **Nome da API** (`name`) - **NÃO usado para identificação única**
- ✅ **Path/Basepath** (`path`) - Usado para identificação única
- ✅ **Versão** (`version`) - Usado para identificação única
- ✅ **Virtual Host** (`vhost`) - Usado para identificação única (opcional)
- ✅ **API Routing Key** (`apiRoutingKey`) - Query string versioning, tem prioridade sobre `version` para identificação
- ✅ **Organização** (`organization`)

**Metadados Técnicos (do OpenAPI/Swagger quando `apiSpecification` é usado):**
- ✅ **Título da API** (`info.title`)
- ✅ **Versão da API** (`info.version`)
- ✅ **Descrição** (`info.description`)
- ✅ **Contato/Proprietário** (`info.contact`, `info.contact.name`, `info.contact.email`)
- ✅ **Endpoints e Paths** (`paths`)
- ✅ **Métodos HTTP** (`get`, `post`, `put`, `delete`, etc.)
- ✅ **Esquemas de Autenticação** (`security`, `securitySchemes`)
- ✅ **Parâmetros e Schemas** (`parameters`, `components.schemas`)
- ✅ **Servidores/Base URLs** (`servers`)
- ⚠️ **Tags do OpenAPI** (`tags` no spec) - São para agrupamento de **endpoints**, não para classificação de recursos

**Metadados de Configuração (Amplify API Management):**
- ✅ **Summary** (`summary`) - Resumo curto da API
- ✅ **Descrição Manual** (`descriptionManual`) - Descrição em Markdown
- ✅ **Tipo de Descrição** (`descriptionType`) - `original` ou `manual`
- ✅ **Backend Basepath** (`backendBasepath`) - URL do backend
- ✅ **Imagem** (`image`) - Ícone/imagem da API
- ✅ **Custom Properties** (`customProperties`) - Propriedades customizadas (chave-valor)

**Metadados de Segurança e Políticas:**
- ✅ **Security Profiles** (`securityProfiles`) - Perfis de segurança (API Key, OAuth, etc.)
- ✅ **CORS Profiles** (`corsProfiles`) - Configurações CORS
- ✅ **Outbound Profiles** (`outboundProfiles`) - Políticas de roteamento e autenticação outbound
- ✅ **Authentication Profiles** (`authenticationProfiles`) - Autenticação com backend

**Metadados de Ciclo de Vida:**
- ✅ **State** (`state`) - `unpublished`, `published`, `deprecated`
- ✅ **Retirement Date** (`retirementDate`) - Data de aposentadoria (quando `state: deprecated`)
- ✅ **Data de Criação/Atualização** - Gerenciada automaticamente pela plataforma

**Metadados de Quotas:**
- ✅ **Application Quota** (`applicationQuota`) - Limites por aplicação
- ✅ **System Quota** (`systemQuota`) - Limites do sistema

**Metadados de Governança (Amplify Engage):**
- ✅ **Categoria do Produto** (já categorizado no catálogo)
- ✅ **Documentação** (links e descrições)
- ✅ **Termos de Uso**
- ✅ **Informações de Monetização**

**⚠️ IMPORTANTE - Identificação de APIs:**
O Amplify API Management identifica APIs únicas usando: `path + "###" + vhost + "###" + versão`
- Onde `versão` = `apiRoutingKey` (se existir) OU `version` (se `apiRoutingKey` não existir)
- O campo `name` **NÃO** é usado para identificação!
- APIs com mesmo path, vhost e versão são consideradas a mesma API (atualização)
- APIs com qualquer diferença nesses campos são consideradas diferentes (criação de nova API)

#### 1.2.2 O que DEVE ser Tag (Informações Complementares)

Tags devem ser usadas para informações que **NÃO** estão nos metadados padrão:

- 🔖 **Ambiente de Implantação** (dev, test, prod) - geralmente não é metadado padrão
- 🔖 **Projeto/Iniciativa** - não é metadado padrão
- 🔖 **Criticidade/Nível de Importância** - não é metadado padrão
- 🔖 **Centro de Custo/Departamento** - pode estar em contato, mas tag facilita agrupamento
- 🔖 **Conformidade Regulatória** (LGPD, GDPR, PCI-DSS) - não é metadado padrão
- 🔖 **SLA Tier** (gold, silver, bronze) - não é metadado padrão
- 🔖 **Status de Governança** (compliant, non-compliant) - não é metadado padrão
- 🔖 **Classificação de Segurança** (public, internal, confidential) - pode complementar security schemes
- 🔖 **Região/Data Center** - não é metadado padrão
- 🔖 **Tags de Negócio** (campanhas, promoções, segmentação) - não são metadados técnicos

#### 1.2.3 Formato de Tags no Amplify API Management

**Formato YAML** (lista simples):
```yaml
tags:
  - environment:production
  - criticality:critical
  - project:api-modernizacao
```

**Formato JSON** (objeto com chaves e arrays de valores):
```json
"tags": {
  "environment": ["production"],
  "criticality": ["critical"],
  "project": ["api-modernizacao"]
}
```

**Nota**: O CLI converte automaticamente entre os formatos. Use o formato YAML (lista) para simplicidade.

#### 1.2.4 Princípio: Tags Complementam, Não Duplicam

**❌ EVITE:**
- Tag `version:v2` quando a versão já está em `version` ou `info.version`
- Tag `name:Minha API` quando o nome já está em `name`
- Tag `path:/api/v1` quando o path já está em `path`
- Tag `owner:team-backend` quando o proprietário já está em `info.contact.name`
- Tag `description:API de pagamentos` quando já existe `summary` ou `descriptionManual`
- Tag `organization:Minha Org` quando a organização já está em `organization`
- Tag `state:published` quando o estado já está em `state`

**✅ USE:**
- Tag `environment:production` (ambiente não é metadado padrão)
- Tag `project:api-modernizacao` (projeto não é metadado padrão)
- Tag `criticality:critical` (criticidade não é metadado padrão)
- Tag `compliance:lgpd` (conformidade não é metadado padrão)
- Tag `business-unit:financeiro` (área de negócio não é metadado padrão)

---

## 2. Taxonomia de Tags Padronizada

### 2.1 Princípios Fundamentais

1. **Consistência**: Use a mesma nomenclatura em todos os recursos
2. **Clareza**: Tags devem ser autoexplicativas
3. **Hierarquia**: Defina categorias e subcategorias quando necessário
4. **Manutenibilidade**: Facilite atualizações e revisões periódicas

### 2.2 Categorias Recomendadas de Tags

#### 2.2.1 Ambiente (Environment)
```
Valores sugeridos:
- development / dev
- testing / test / qa
- staging / stage
- production / prod
```

**Exemplo**: `environment:production`

#### 2.2.2 Departamento/Área de Negócio (Business Unit)
```
Valores sugeridos:
- financeiro
- vendas
- marketing
- operacoes
- rh
- ti
```

**Exemplo**: `business-unit:financeiro`

#### 2.2.3 Projeto/Iniciativa (Project)
```
Valores sugeridos:
- projeto-digitalizacao
- api-modernizacao
- integracao-erp
- migracao-cloud
```

**Exemplo**: `project:api-modernizacao`

#### 2.2.4 Criticidade (Criticality)
```
Valores sugeridos:
- critical / critico
- high / alto
- medium / medio
- low / baixo
```

**Exemplo**: `criticality:critical`

#### 2.2.5 Tipo de API (API Type) - *Opcional*
> **Nota**: O tipo de API pode ser inferido dos metadados (OpenAPI spec), mas a tag é útil para classificação e filtros rápidos.

```
Valores sugeridos:
- rest
- soap
- graphql
- event-driven
- internal
- external
- partner
```

**Exemplo**: `api-type:rest`

**Quando usar**: Quando você precisa filtrar/agrupar APIs por tipo de forma rápida, especialmente quando o tipo não está claro nos metadados.

#### 2.2.6 Domínio Funcional (Domain) - *Opcional*
> **Nota**: O domínio pode estar na descrição da API, mas a tag facilita busca e agrupamento por área de negócio.

```
Valores sugeridos:
- customer
- order
- payment
- inventory
- notification
- authentication
```

**Exemplo**: `domain:payment`

**Quando usar**: Quando você precisa agrupar APIs por domínio funcional para relatórios de negócio ou organização do catálogo.

#### 2.2.7 Status de Conformidade (Compliance)
```
Valores sugeridos:
- compliant
- non-compliant
- pending-review
- deprecated
```

**Exemplo**: `compliance:compliant`

#### 2.2.8 Proprietário/Responsável (Owner) - *Opcional*
> **Nota**: O proprietário pode estar em `info.contact.name`, mas a tag facilita agrupamento por equipe e relatórios organizacionais.

```
Valores sugeridos:
- team-backend
- team-integration
- team-frontend
- vendor-externo
```

**Exemplo**: `owner:team-integration`

**Quando usar**: Quando você precisa agrupar APIs por equipe responsável para relatórios, alocação de recursos ou governança organizacional. Se o contato já está nos metadados e atende suas necessidades, a tag é opcional.

#### 2.2.9 Versão (Version) - *❌ NÃO RECOMENDADO*
> **⚠️ ATENÇÃO**: A versão da API já está disponível no metadado `info.version` do OpenAPI/Swagger. **NÃO** duplique como tag.

**Exceção**: Use apenas se precisar de uma classificação adicional que não corresponde à versão técnica:
- `version:legacy` para APIs antigas independente da versão numérica
- `version:beta` para APIs em beta independente da versão numérica

**Recomendação**: Utilize o campo de versão dos metadados em vez de criar uma tag.

#### 2.2.10 Região/Data Center (Region)
```
Valores sugeridos:
- us-east
- us-west
- sa-east
- eu-west
```

**Exemplo**: `region:sa-east`

---

## 3. Melhores Práticas para Amplify API Management

### 3.1 Formato de Tags em Arquivos de Configuração

> **IMPORTANTE**: Baseado na implementação do APIM-CLI, `TagMap` é um `Map<String, String[]>` onde:
> - A **chave** é o nome do grupo de tags (ex: "environment", "criticality")
> - O **valor** é um array de strings (ex: ["production"], ["critical", "high"])

#### 3.1.1 Formato YAML (Recomendado)

No formato YAML, tags devem ser um objeto com chaves e arrays de valores:

```yaml
name: PetStore API
path: /petstore/v1
version: 1.0.0
state: published
organization: API Development

# Tags como objeto (formato nativo do TagMap)
tags:
  environment:
    - production
  criticality:
    - critical
  project:
    - api-modernizacao
  business-unit:
    - ti
  compliance:
    - compliant
  api-type:
    - rest
  domain:
    - petstore
```

**Vantagens do formato YAML:**
- Mais legível e fácil de manter
- Suporta comentários
- Formato preferido para versionamento em Git
- Estrutura clara para múltiplos valores

#### 3.1.2 Formato JSON

No formato JSON, tags são um objeto onde cada chave é um grupo de tags e o valor é um array:

```json
{
  "name": "PetStore API",
  "path": "/petstore/v1",
  "version": "1.0.0",
  "state": "published",
  "organization": "API Development",
  "tags": {
    "environment": ["production"],
    "criticality": ["critical"],
    "project": ["api-modernizacao"],
    "business-unit": ["ti"],
    "compliance": ["compliant"],
    "api-type": ["rest"],
    "domain": ["petstore"]
  }
}
```

**Nota**: Este é o formato nativo usado internamente pelo APIM-CLI (`TagMap extends LinkedHashMap<String, String[]>`).

#### 3.1.3 Tags com Múltiplos Valores

Se uma categoria de tag pode ter múltiplos valores, use arrays:

**YAML:**
```yaml
tags:
  environment:
    - production
  criticality:
    - critical
  regulation:
    - lgpd
    - pci-dss  # Múltiplas regulamentações
  domain:
    - payment
    - checkout  # Múltiplos domínios
```

**JSON:**
```json
"tags": {
  "environment": ["production"],
  "criticality": ["critical"],
  "regulation": ["lgpd", "pci-dss"],
  "domain": ["payment", "checkout"]
}
```

#### 3.1.4 Formato Alternativo (Lista Simples)

Alguns exemplos mostram tags como lista simples, mas o APIM-CLI converte para o formato objeto:

```yaml
# Formato alternativo (será convertido para objeto)
tags:
  - environment:production
  - criticality:critical
```

**Recomendação**: Use sempre o formato objeto (chave: array) para garantir consistência e clareza.

### 3.2 Tagging de APIs

#### 3.2.1 Tags Obrigatórias
Todas as APIs devem conter, no mínimo (informações que **NÃO** estão nos metadados padrão):
- `environment`: Identificação do ambiente (dev, test, prod) - **obrigatório**
- `criticality`: Nível de criticidade - **obrigatório**

#### 3.2.2 Tags Altamente Recomendadas
- `project`: Projeto/Iniciativa associada - facilita rastreabilidade
- `business-unit`: Área de negócio responsável - para relatórios organizacionais
- `compliance`: Status de conformidade - essencial para governança

#### 3.2.3 Tags Opcionais (Use conforme necessidade)
- `api-type`: Tipo de API (útil se não está claro nos metadados)
- `domain`: Domínio funcional (útil para agrupamento por área de negócio)
- `owner`: Equipe responsável (útil se precisa agrupar por equipe além do contato)
- `security-level`: Nível de segurança (complementa security schemes)
- `sla-tier`: Nível de SLA (gold, silver, bronze)
- `region`: Região/Data center
- `regulation`: Regulamentações aplicáveis (LGPD, GDPR, PCI-DSS)

#### 3.2.4 Tags a EVITAR (já são metadados)
- ❌ `version`: Use o campo `info.version` dos metadados
- ❌ `name` ou `title`: Use o campo `info.title` dos metadados
- ❌ `description`: Use o campo `info.description` dos metadados
- ❌ `contact`: Use os campos `info.contact.*` dos metadados

#### 3.2.5 Exemplo de Conjunto de Tags (Otimizado)
```
# Tags Obrigatórias
environment:production
criticality:critical

# Tags Altamente Recomendadas
project:api-modernizacao
business-unit:financeiro
compliance:compliant

# Tags Opcionais (conforme necessidade)
api-type:rest
domain:payment
security-level:confidential
sla-tier:gold
region:sa-east
regulation:pci-dss
```

**Nota**: A versão (`v2`) está nos metadados (`info.version`), não precisa ser tag.

### 3.3 Tagging para Agrupamento Lógico

No **Amplify API Management**, utilize tags para agrupar APIs relacionadas que serão publicadas no Marketplace como produtos completos:

**Exemplo**: APIs de e-commerce
```
product:ecommerce
product-component:checkout
product-component:inventory
product-component:shipping
```

### 3.4 Tagging para Governança e Segurança

#### 3.3.1 Identificação de APIs Não Gerenciadas
```
compliance:non-compliant
governance:ungoverned
security-review:required
```

#### 3.3.2 Classificação de Segurança
```
security-level:public
security-level:internal
security-level:confidential
security-level:restricted
```

#### 3.3.3 Políticas de Acesso
```
access-policy:open
access-policy:authenticated
access-policy:authorized
access-policy:restricted
```

### 3.5 Tagging para Descoberta e Reutilização

Facilite a descoberta de APIs através de tags descritivas:

```
use-case:customer-onboarding
use-case:order-processing
use-case:payment-processing
integration-pattern:request-response
integration-pattern:event-driven
```

### 3.6 Tagging para Monitoramento e Análise

Habilite relatórios e dashboards baseados em tags:

```
monitoring:enabled
alerting:enabled
sla-tier:gold
sla-tier:silver
sla-tier:bronze
```

### 3.7 Tags em Arquivos de Configuração por Ambiente

O Amplify API Management CLI suporta arquivos de configuração por ambiente (stage files). Use tags para diferenciar configurações entre ambientes:

**Arquivo Base**: `petstore-api.yaml`
```yaml
name: PetStore API
path: /petstore/v1
version: 1.0.0
state: unpublished
organization: API Development
apiSpecification:
  resource: petstore.json

# Tags comuns a todos os ambientes
tags:
  - api-type:rest
  - domain:petstore
  - project:api-modernizacao
  - business-unit:ti
```

**Arquivo de Produção**: `petstore-api.prod.yaml`
```yaml
name: PetStore API - Produção
state: published
vhost: api.prod.example.com
image: petstore-icon.jpg

# Tags específicas de produção
tags:
  - environment:production
  - criticality:critical
  - compliance:compliant
  - sla-tier:gold
  - region:sa-east
```

**Arquivo de Desenvolvimento**: `petstore-api.dev.yaml`
```yaml
name: PetStore API - Dev
state: unpublished
vhost: api.dev.example.com

# Tags específicas de desenvolvimento
tags:
  - environment:development
  - criticality:medium
  - compliance:pending-review
  - sla-tier:bronze
```

**Comando para importar:**
```bash
# O CLI automaticamente mescla o arquivo base com o arquivo do stage
apim api import -c petstore-api.yaml -s prod
# Carrega: petstore-api.yaml + petstore-api.prod.yaml
```

**Vantagens:**
- Tags de ambiente aplicadas automaticamente
- Configuração base reutilizável
- Overrides específicos por ambiente
- Facilita gerenciamento de múltiplos ambientes

---

## 4. Melhores Práticas para Amplify Engage

### 4.1 Conceito de Assets no Amplify Engage

No **Amplify Engage**, a hierarquia de organização é:

**Hierarquia:**
```
Produtos (Products)
  └── Assets (Agrupamentos de Serviços/APIs)
        └── Serviços/APIs (APIs individuais)
```

**Conceitos:**

1. **Serviços/APIs**: APIs individuais publicadas no API Management
2. **Assets**: Agrupamentos lógicos de serviços/APIs relacionados que formam uma solução ou capacidade
3. **Produtos**: Agrupamentos de assets que formam um produto completo no Marketplace

**Exemplo Prático:**

```
Produto: E-Commerce Platform
  ├── Asset: Payment Services
  │     ├── API: Payment Processing API
  │     ├── API: Refund API
  │     └── API: Payment Status API
  ├── Asset: Order Management
  │     ├── API: Order Creation API
  │     ├── API: Order Status API
  │     └── API: Order Cancellation API
  └── Asset: Inventory Services
        ├── API: Stock Check API
        └── API: Inventory Update API
```

**Por que usar Assets?**
- **Agrupamento Lógico**: Reúne APIs relacionadas em uma solução coesa
- **Governança**: Facilita aplicação de políticas e controles em grupos de APIs
- **Descoberta**: Desenvolvedores encontram soluções completas, não apenas APIs individuais
- **Monetização**: Permite monetizar grupos de APIs como uma solução
- **Versionamento**: Gerencia versões de soluções completas

### 4.2 Tagging de Assets (Agrupamentos de Serviços/APIs)

#### 4.2.1 Tags Essenciais para Assets

Para assets no Amplify Engage (que são agrupamentos de serviços/APIs), use as seguintes tags:

**Obrigatórias:**
- `product`: Produto ao qual o asset pertence
- `asset-name` ou `asset-id`: Identificador do asset (se aplicável)

**Altamente Recomendadas:**
- `business-unit`: Área de negócio responsável
- `domain`: Domínio funcional do asset
- `compliance`: Status de conformidade

**Opcionais:**
- `asset-owner`: Equipe responsável pelo asset
- `asset-status`: Status no ciclo de vida de governança

#### 4.2.2 Exemplo: Asset "Payment Services" (Agrupamento de APIs)

Este asset agrupa múltiplas APIs relacionadas a pagamentos:

```yaml
# Asset: Payment Services
# Agrupa: Payment Processing API, Refund API, Payment Status API

tags:
  product:
    - ecommerce-platform
  domain:
    - payment
  business-unit:
    - financeiro
  compliance:
    - compliant
  regulation:
    - pci-dss
  asset-owner:
    - payment-team
  criticality:
    - critical
```

**APIs que compõem este Asset:**
```yaml
# API 1: Payment Processing API
name: Payment Processing API
path: /api/v1/payments
tags:
  product:
    - ecommerce-platform
  domain:
    - payment
  asset:
    - payment-services  # Referência ao asset
  api-type:
    - rest
  criticality:
    - critical

# API 2: Refund API
name: Refund API
path: /api/v1/refunds
tags:
  product:
    - ecommerce-platform
  domain:
    - payment
  asset:
    - payment-services  # Mesmo asset
  api-type:
    - rest
  criticality:
    - high

# API 3: Payment Status API
name: Payment Status API
path: /api/v1/payment-status
tags:
  product:
    - ecommerce-platform
  domain:
    - payment
  asset:
    - payment-services  # Mesmo asset
  api-type:
    - rest
  criticality:
    - medium
```

#### 4.2.3 Exemplo: Asset "Order Management" (Agrupamento de APIs)

```yaml
# Asset: Order Management
# Agrupa: Order Creation API, Order Status API, Order Cancellation API

tags:
  product:
    - ecommerce-platform
  domain:
    - order
  business-unit:
    - operacoes
  compliance:
    - compliant
  asset-owner:
    - order-management-team
  criticality:
    - high
```

**APIs que compõem este Asset:**
```yaml
# Todas as APIs de Order Management compartilham:
tags:
  product:
    - ecommerce-platform
  domain:
    - order
  asset:
    - order-management  # Referência ao asset
```

#### 4.2.4 Exemplo: Produto Completo "E-Commerce Platform"

O produto agrupa múltiplos assets:

```yaml
# Produto: E-Commerce Platform
# Agrupa Assets:
#   - Payment Services (asset)
#   - Order Management (asset)
#   - Inventory Services (asset)
#   - Customer Services (asset)

# Tags aplicadas ao nível do Produto
tags:
  product:
    - ecommerce-platform
  business-unit:
    - vendas
  compliance:
    - compliant
  regulation:
    - lgpd
    - pci-dss
  criticality:
    - critical
```

#### 4.2.5 Estratégia de Tagging para Hierarquia

**Nível Produto:**
- Tags de alto nível: `product`, `business-unit`, `compliance`, `regulation`
- Aplicadas a todos os assets e APIs do produto

**Nível Asset:**
- Tags específicas do asset: `domain`, `asset-owner`, `asset-status`
- Herda tags do produto
- Pode ter tags adicionais específicas

**Nível API/Serviço:**
- Tags técnicas: `api-type`, `environment`, `criticality`
- Referência ao asset: `asset: [nome-do-asset]`
- Herda tags do produto e do asset

#### 4.2.6 Exemplo Completo: Hierarquia de Tags

```
Produto: E-Commerce Platform
├── Tags do Produto:
│   - product: ecommerce-platform
│   - business-unit: vendas
│   - compliance: compliant
│
├── Asset: Payment Services
│   ├── Tags do Asset:
│   │   - product: ecommerce-platform (herdado)
│   │   - domain: payment
│   │   - asset-owner: payment-team
│   │
│   ├── API: Payment Processing API
│   │   ├── Tags da API:
│   │   │   - product: ecommerce-platform (herdado)
│   │   │   - domain: payment (herdado)
│   │   │   - asset: payment-services
│   │   │   - api-type: rest
│   │   │   - environment: production
│   │   │   - criticality: critical
│   │
│   └── API: Refund API
│       └── Tags similares, mesma referência ao asset
│
└── Asset: Order Management
    └── Estrutura similar
```

#### 4.2.7 Boas Práticas para Tagging de Assets

1. **Consistência**: Todas as APIs de um asset devem ter a tag `asset: [nome-do-asset]`
2. **Herança**: APIs herdam tags do produto e do asset
3. **Especificidade**: APIs podem ter tags adicionais específicas
4. **Rastreabilidade**: A tag `asset` permite rastrear qual asset uma API pertence
5. **Agrupamento**: Use `product` para agrupar assets relacionados

### 4.3 Tagging de Recursos de Governança

#### 4.1.1 Políticas e Regras
```
policy-type:security
policy-type:compliance
policy-type:quality
policy-category:authentication
policy-category:rate-limiting
policy-category:data-protection
```

#### 4.1.2 Processos de Governança
```
governance-stage:design
governance-stage:development
governance-stage:testing
governance-stage:deployment
governance-stage:retirement
```

#### 4.1.3 Aprovações e Revisões
```
approval-status:pending
approval-status:approved
approval-status:rejected
review-cycle:quarterly
review-cycle:annual
```

### 4.2 Tagging para Rastreabilidade

Mantenha rastreabilidade entre recursos:

```
trace-id:PROJ-2024-001
requirement-id:REQ-API-001
design-doc:DD-API-001
test-plan:TP-API-001
```

### 4.3 Tagging para Conformidade Regulatória

Identifique requisitos de conformidade:

```
regulation:lgpd
regulation:gdpr
regulation:pci-dss
regulation:sox
compliance-framework:iso27001
```

---

## 5. Estratégia de Implementação

### 5.1 Fase 1: Planejamento

1. **Defina a Taxonomia**: Estabeleça as categorias e valores permitidos
2. **Documente as Convenções**: Crie um guia de referência
3. **Comunique**: Compartilhe com todas as equipes envolvidas
4. **Valide**: Obtenha aprovação dos stakeholders

### 5.2 Fase 2: Implementação Gradual

1. **Piloto**: Comece com um projeto ou departamento
2. **Refine**: Ajuste a taxonomia baseado em feedback
3. **Expanda**: Aplique gradualmente em toda a organização
4. **Automatize**: Crie processos para aplicação automática de tags

### 5.3 Fase 3: Manutenção Contínua

1. **Revisão Periódica**: Revise tags trimestralmente ou semestralmente
2. **Limpeza**: Remova tags obsoletas ou não utilizadas
3. **Atualização**: Mantenha a documentação atualizada
4. **Treinamento**: Capacite novas equipes nas convenções

---

## 6. Exemplos Práticos

### 6.1 API de Pagamento (Produção)

**Contexto**: API REST de processamento de pagamentos em produção

**Metadados Disponíveis** (não duplicar como tags):
- Nome: "Payment API"
- Versão: "2.0.0" (em `info.version`)
- Descrição: "API para processamento de pagamentos"
- Contato: "Team Integration" (em `info.contact.name`)

**Tags Aplicadas** (informações complementares):
```
# Tags Obrigatórias
environment:production
criticality:critical

# Tags Altamente Recomendadas
project:api-modernizacao
business-unit:financeiro
compliance:compliant

# Tags Opcionais
api-type:rest
domain:payment
security-level:confidential
access-policy:authenticated
sla-tier:gold
monitoring:enabled
alerting:enabled
region:sa-east
regulation:pci-dss
```

**Nota**: A versão `v2` está nos metadados (`info.version: "2.0.0"`), não precisa ser tag.

### 6.2 API de Notificações (Desenvolvimento)

**Contexto**: API de notificações em desenvolvimento

**Metadados Disponíveis**:
- Nome: "Notification Service API"
- Versão: "1.0.0" (em `info.version`)
- Contato: "Backend Team" (em `info.contact.name`)

**Tags Aplicadas**:
```
# Tags Obrigatórias
environment:development
criticality:medium

# Tags Altamente Recomendadas
project:api-modernizacao
business-unit:ti
compliance:pending-review

# Tags Opcionais
api-type:rest
domain:notification
security-level:internal
access-policy:open
sla-tier:bronze
```

**Nota**: A versão `v1` está nos metadados, não precisa ser tag.

### 6.3 API Legacy (Staging)

**Contexto**: API legada em processo de depreciação

**Metadados Disponíveis**:
- Nome: "Legacy Order API"
- Versão: "1.5.2" (em `info.version`)
- Status: "deprecated" (pode estar em metadados de ciclo de vida)

**Tags Aplicadas**:
```
# Tags Obrigatórias
environment:staging
criticality:low

# Tags Altamente Recomendadas
project:api-modernizacao
business-unit:operacoes
compliance:non-compliant

# Tags Opcionais
api-type:soap
domain:order
version:legacy  # Exceção: classificação adicional, não duplica versão técnica
status:deprecated
migration-target:api-order-v2
```

**Nota**: A tag `version:legacy` é uma exceção válida, pois classifica a API como legada independente da versão técnica (1.5.2).

---

## 7. Automação e Governança de Tags

### 7.1 Aplicação Automática de Tags

Configure regras para aplicação automática de tags baseadas em:
- **Metadados da API**: Nome, descrição, endpoints
- **Ambiente**: Baseado na configuração de deployment
- **Políticas**: Aplicação de tags de conformidade
- **Integrações**: Sincronização com sistemas externos (CMDB, ServiceNow)

### 7.2 Validação de Tags

Implemente validações para garantir:
- **Tags Obrigatórias**: Verificar presença de tags essenciais
- **Valores Válidos**: Validar contra taxonomia definida
- **Consistência**: Detectar inconsistências entre recursos relacionados

### 7.3 Relatórios e Dashboards

Crie visualizações baseadas em tags:
- **Inventário por Ambiente**: Distribuição de APIs por ambiente
- **Conformidade**: Status de conformidade por business unit
- **Criticidade**: Distribuição por nível de criticidade
- **Projetos**: Recursos agrupados por projeto

---

## 8. Benefícios do Tagging Estruturado

### 8.1 Operacionais
- ✅ Redução de tempo na localização de recursos
- ✅ Facilita troubleshooting e análise de incidentes
- ✅ Melhora a organização e manutenção do catálogo

### 8.2 Governança
- ✅ Aplicação consistente de políticas
- ✅ Identificação rápida de recursos não conformes
- ✅ Rastreabilidade e auditoria facilitadas

### 8.3 Negócio
- ✅ Visibilidade clara dos recursos por área de negócio
- ✅ Facilita planejamento e alocação de recursos
- ✅ Suporta decisões baseadas em dados

### 8.4 Segurança
- ✅ Identificação de recursos críticos
- ✅ Aplicação de controles de segurança apropriados
- ✅ Monitoramento proativo de ameaças

---

## 9. Checklist de Implementação

### 9.1 Preparação
- [ ] Taxonomia de tags definida e documentada
- [ ] Convenções de nomenclatura estabelecidas
- [ ] Stakeholders alinhados e aprovados
- [ ] Ferramentas e processos configurados

### 9.2 Implementação
- [ ] Tags obrigatórias identificadas
- [ ] Tags aplicadas em recursos existentes
- [ ] Processo de aplicação de tags em novos recursos definido
- [ ] Automação configurada (quando aplicável)

### 9.3 Validação
- [ ] Validação de tags implementada
- [ ] Relatórios e dashboards criados
- [ ] Equipes treinadas nas convenções
- [ ] Documentação disponível e acessível

### 9.4 Manutenção
- [ ] Processo de revisão periódica estabelecido
- [ ] Responsável pela manutenção da taxonomia definido
- [ ] Canal de comunicação para sugestões e feedback criado
- [ ] Métricas de adoção e conformidade definidas

---

## 10. Padrões de Mercado e Alinhamento com Práticas da Indústria

### 10.1 Princípios de Tagging da Indústria

As melhores práticas de tagging seguem padrões estabelecidos por organizações líderes em API Management:

#### 10.1.1 AWS API Gateway / Azure API Management
- **Foco em Ambiente e Organização**: Tags primárias para ambiente, projeto e centro de custo
- **Evitar Duplicação**: Não duplicar informações já em metadados (nome, descrição, versão)
- **Automação**: Aplicação automática baseada em deployment e configuração

#### 10.1.2 Google Apigee
- **Classificação por Negócio**: Tags para domínio funcional e área de negócio
- **Governança**: Tags de conformidade e status de governança
- **Rastreabilidade**: Tags de projeto e iniciativa para rastreamento

#### 10.1.3 OpenAPI/Swagger Standards
- **Metadados Padrão**: `info.title`, `info.version`, `info.description`, `info.contact` são metadados, não tags
- **Tags no OpenAPI**: Tags no OpenAPI são para agrupamento de endpoints, não para classificação de recursos
- **Extensions**: Use `x-*` extensions para informações customizadas quando apropriado

### 10.2 Alinhamento com Frameworks de Governança

#### 10.2.1 COBIT / ITIL
- **Classificação por Criticidade**: Alinhado com gestão de serviços
- **Rastreabilidade**: Tags de projeto facilitam rastreamento de mudanças
- **Conformidade**: Tags de compliance suportam auditoria

#### 10.2.2 TOGAF
- **Domínios de Negócio**: Tags de domínio alinhadas com arquitetura de negócio
- **Capacidades**: Tags facilitam mapeamento de capacidades de negócio

### 10.3 Lições Aprendidas da Indústria

1. **Menos é Mais**: Tags excessivas reduzem eficácia. Foque no essencial.
2. **Consistência é Crítica**: Nomenclatura padronizada é mais importante que quantidade
3. **Automação Reduz Erros**: Tags manuais são propensas a inconsistências
4. **Metadados Primeiro**: Sempre verifique metadados antes de criar tags
5. **Documentação Contínua**: Taxonomia deve evoluir com a organização

### 10.4 Métricas de Sucesso

Organizações que implementam tagging estruturado reportam:
- **40-60%** de redução no tempo de localização de APIs
- **30-50%** de melhoria na aplicação de políticas
- **25-40%** de aumento na reutilização de APIs
- **50-70%** de melhoria na rastreabilidade de mudanças

---

## 11. Referências e Recursos Adicionais

### 11.1 Documentação Oficial
- Amplify API Management Documentation
- Amplify Engage User Guide
- Axway Amplify Platform Resources

### 11.2 Contatos
- **Suporte Técnico**: [informar contato]
- **Governança de APIs**: [informar contato]
- **Arquiteto de Soluções**: [informar contato]

---

## 12. Glossário

- **Tag**: Etiqueta ou rótulo aplicado a um recurso para categorização
- **Taxonomia**: Sistema de classificação hierárquica
- **Governança**: Processo de controle e supervisão de recursos
- **Conformidade**: Aderência a padrões, políticas e regulamentações
- **Marketplace**: Catálogo de APIs disponíveis para consumo

---

**Versão**: 1.0  
**Última Atualização**: 2024  
**Autor**: [Nome do Autor/Equipe]  
**Aprovação**: [Nome do Aprovador]

---

*Este documento deve ser revisado e atualizado periodicamente para refletir mudanças nas práticas organizacionais e nas capacidades das plataformas.*

