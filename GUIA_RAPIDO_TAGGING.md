# Guia Rápido de Tagging
## Referência Rápida para Amplify API Management e Amplify Engage

---

## 📝 Formato de Tags

> **IMPORTANTE**: Baseado na implementação do APIM-CLI, tags são um `Map<String, String[]>` (objeto com chaves e arrays de valores).

### YAML (Recomendado)
```yaml
tags:
  environment:
    - production
  criticality:
    - critical
  project:
    - api-modernizacao
  business-unit:
    - ti
```

### JSON
```json
"tags": {
  "environment": ["production"],
  "criticality": ["critical"],
  "project": ["api-modernizacao"],
  "business-unit": ["ti"]
}
```

**Nota**: Este é o formato nativo do APIM-CLI (`TagMap`). O formato objeto (chave: array) é sempre usado internamente.

---

## 📋 Tags Essenciais (Obrigatórias)

> **⚠️ IMPORTANTE**: Não duplique informações que já estão nos metadados (nome, versão, descrição, contato). Use tags apenas para informações complementares.

| Categoria | Valores Sugeridos | Exemplo | Nota |
|-----------|-------------------|---------|------|
| **Environment** | `dev`, `test`, `stage`, `prod` | `environment:prod` | **Obrigatório** |
| **Criticality** | `critical`, `high`, `medium`, `low` | `criticality:critical` | **Obrigatório** |
| **Project** | `api-modernizacao`, `digitalizacao` | `project:api-modernizacao` | Altamente recomendado |
| **Business Unit** | `financeiro`, `vendas`, `ti` | `business-unit:financeiro` | Altamente recomendado |
| **Compliance** | `compliant`, `non-compliant`, `pending-review` | `compliance:compliant` | Altamente recomendado |

---

## 🏷️ Tags Opcionais (Use conforme necessidade)

| Categoria | Valores Sugeridos | Exemplo | Quando Usar |
|-----------|-------------------|---------|-------------|
| **API Type** | `rest`, `soap`, `graphql` | `api-type:rest` | Se não está claro nos metadados |
| **Domain** | `payment`, `order`, `customer` | `domain:payment` | Para agrupamento por área de negócio |
| **Owner** | `team-integration`, `team-backend` | `owner:team-integration` | Se precisa agrupar além do contato |
| **Security Level** | `public`, `internal`, `confidential` | `security-level:confidential` | Complementa security schemes |
| **SLA Tier** | `gold`, `silver`, `bronze` | `sla-tier:gold` | Para classificação de SLA |
| **Region** | `us-east`, `sa-east`, `eu-west` | `region:sa-east` | Para APIs multi-região |

## 🎯 Tags para Assets no Amplify Engage

> **IMPORTANTE**: No Amplify Engage, a hierarquia é: **Produtos → Assets → APIs/Serviços**
> - **Assets** são agrupamentos de serviços/APIs relacionados
> - **Produtos** são agrupamentos de assets

### Hierarquia
```
Produto: E-Commerce Platform
  ├── Asset: Payment Services
  │     ├── API: Payment Processing API
  │     ├── API: Refund API
  │     └── API: Payment Status API
  └── Asset: Order Management
        ├── API: Order Creation API
        └── API: Order Status API
```

| Categoria | Valores Sugeridos | Exemplo | Quando Usar |
|-----------|-------------------|---------|-------------|
| **Product** | `ecommerce-platform`, `marketplace`, `payment-gateway`, `analytics-platform` | `product:ecommerce-platform` | **Obrigatório** - Nível mais alto |
| **Asset** | `payment-services`, `order-management`, `inventory-services`, `customer-services` | `asset:payment-services` | **Recomendado** - Agrupa APIs relacionadas |
| **Asset Owner** | `payment-team`, `order-team`, `inventory-team`, `integration-team` | `asset-owner:payment-team` | Equipe responsável pelo asset |
| **Asset Status** | `draft`, `review`, `approved`, `published`, `archived`, `deprecated` | `asset-status:published` | Status do agrupamento de APIs |

### Exemplo: API em Asset e Produto
```yaml
# API: Payment Processing API
# Pertence ao Asset: Payment Services
# Pertence ao Produto: E-Commerce Platform

tags:
  product:
    - ecommerce-platform
  asset:
    - payment-services
  domain:
    - payment
  api-type:
    - rest
  environment:
    - production
  criticality:
    - critical
```

### Exemplo: Múltiplas APIs no Mesmo Asset
```yaml
# Todas as APIs de Payment Services compartilham:
tags:
  product:
    - ecommerce-platform
  asset:
    - payment-services
  domain:
    - payment

# APIs individuais podem ter tags adicionais específicas
```

---

## ❌ Tags a EVITAR (já são metadados)

- `version` - Use `info.version` dos metadados
- `name` ou `title` - Use `info.title` dos metadados  
- `description` - Use `info.description` dos metadados
- `contact` ou `owner` - Use `info.contact.*` dos metadados

---

## 🔒 Tags de Segurança e Governança

| Categoria | Valores Sugeridos | Exemplo |
|-----------|-------------------|---------|
| **Security Level** | `public`, `internal`, `confidential`, `restricted` | `security-level:confidential` |
| **Access Policy** | `open`, `authenticated`, `authorized`, `restricted` | `access-policy:authenticated` |
| **Compliance** | `lgpd`, `gdpr`, `pci-dss`, `sox` | `regulation:pci-dss` |

---

## 📊 Tags para Monitoramento

| Categoria | Valores Sugeridos | Exemplo |
|-----------|-------------------|---------|
| **SLA Tier** | `gold`, `silver`, `bronze` | `sla-tier:gold` |
| **Monitoring** | `enabled`, `disabled` | `monitoring:enabled` |
| **Alerting** | `enabled`, `disabled` | `alerting:enabled` |

---

## 🎯 Exemplos Práticos

### API de Produção Crítica
```
# Obrigatórias
environment:prod
criticality:critical

# Altamente Recomendadas
project:api-modernizacao
business-unit:financeiro
compliance:compliant

# Opcionais
api-type:rest
domain:payment
security-level:confidential
sla-tier:gold
region:sa-east
regulation:pci-dss
```
**Nota**: Versão está em `info.version` dos metadados, não precisa ser tag.

### API de Desenvolvimento
```
# Obrigatórias
environment:dev
criticality:medium

# Altamente Recomendadas
project:api-modernizacao
business-unit:ti
compliance:pending-review

# Opcionais
api-type:rest
domain:notification
```

### API Legacy (Deprecada)
```
# Obrigatórias
environment:stage
criticality:low

# Altamente Recomendadas
project:api-modernizacao
compliance:non-compliant

# Opcionais
api-type:soap
domain:order
version:legacy  # Exceção: classificação adicional
status:deprecated
```

---

## ✅ Checklist de Tagging

Ao criar ou atualizar uma API, verifique:

- [ ] **Tags obrigatórias aplicadas** (Environment, Criticality)
- [ ] **Tags altamente recomendadas aplicadas** (Project, Business Unit, Compliance)
- [ ] **Tags opcionais aplicadas** (conforme necessidade organizacional)
- [ ] **NÃO duplicar metadados** (versão, nome, descrição, contato já estão nos metadados)
- [ ] **Tags consistentes** com a taxonomia definida
- [ ] **Metadados preenchidos** corretamente (nome, versão, descrição, contato)

---

## 🔄 Processo de Aplicação

1. **Identificar** o tipo de recurso (API, Policy, Gateway)
2. **Selecionar** tags obrigatórias baseadas no contexto
3. **Adicionar** tags recomendadas relevantes
4. **Validar** consistência com taxonomia
5. **Documentar** tags aplicadas

---

## 📈 Benefícios

- ✅ **Organização**: Localização rápida de recursos
- ✅ **Governança**: Aplicação consistente de políticas
- ✅ **Segurança**: Identificação de recursos críticos
- ✅ **Conformidade**: Rastreabilidade e auditoria
- ✅ **Análise**: Relatórios e dashboards baseados em tags

---

**💡 Dica**: Mantenha um glossário de tags atualizado e compartilhe com todas as equipes para garantir consistência.

---

*Para informações detalhadas, consulte o documento completo: `MELHORES_PRATICAS_TAGGING.md`*

