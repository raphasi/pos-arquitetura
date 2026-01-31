# Padrão de Tags Azure (CAF) — Convenções e Tags mais usadas no mercado

Este documento define um padrão prático de **tags** para recursos no **Microsoft Azure**, alinhado ao espírito do **Cloud Adoption Framework (CAF)** e ao que normalmente é adotado em ambientes corporativos para **governança, FinOps, operação e segurança**.

> **Objetivo:** permitir que qualquer pessoa consiga responder rapidamente:
> - “O que é esse recurso?”
> - “De quem é?”
> - “Qual o ambiente?”
> - “Qual projeto/custo?”
> - “Como operar e auditar?”

---

## 1) Princípios

1. **Tags são metadados de gestão**, não configuração técnica (config técnica vai em IaC/parametrização do serviço).
2. **Padronização > criatividade**: um conjunto pequeno, bem definido e obrigatório funciona melhor que dezenas de tags “livres”.
3. **Tags devem habilitar ações**: custo (chargeback), inventário, compliance, alertas, automação e auditoria.
4. **Aplicar na raiz e herdar** sempre que possível (Subscription/RG → Recursos), para reduzir esforço e evitar inconsistência.
5. **Valores controlados (enums)** para tags críticas (ex.: Environment, DataClassification, Criticality).

---

## 2) Regras de formatação (padrão de mercado)

### 2.1 Convenção para nomes das tags
- **PascalCase** (mais legível em portal e exportações): `CostCenter`, `BusinessUnit`, `DataClassification`
- Evitar espaços e caracteres especiais.
- Evitar variações que geram duplicidade: `costcenter`, `cost_center`, `Cost Centre` etc.

### 2.2 Convenção para valores
- Preferir **minúsculas** e sem acento (facilita filtro e padronização): `dev`, `prod`, `internal`
- Evitar espaços: use hífen `-` quando necessário: `time-erp`, `pos-arq-azure`
- Datas no padrão ISO: `YYYY-MM-DD` (ex.: `2026-06-30`)
- E-mails em caixa baixa: `suporte@empresa.com`

---

## 3) Catálogo de tags recomendado

### 3.1 Tags obrigatórias (Required)
Estas tags devem existir em **todos os Resource Groups** e, idealmente, serem herdadas para os recursos.

| Tag | Para quê serve | Exemplo | Valores recomendados |
|---|---|---|---|
| `Application` | Identifica o sistema/workload | `erp` | livre, mas padronizado |
| `Environment` | Diferenciar ambientes | `dev` | `dev`, `test`, `qa`, `stage`, `prod` |
| `Owner` | Responsável pelo recurso (pessoa/time) | `time-erp` | livre, padronizado |
| `CostCenter` | Chargeback/showback e relatórios | `pos-arq-azure` | livre, padronizado |
| `ManagedBy` | Como o recurso é gerenciado | `iac` | `iac`, `manual`, `vendor` |
| `DataClassification` | Sensibilidade do dado | `internal` | `public`, `internal`, `confidential`, `restricted` |

> **Por que essas 6?** Elas cobrem o “mínimo corporativo” para **inventário + custo + responsabilidade + governança de dados**.

---

### 3.2 Tags fortemente recomendadas (Recommended)
Use quando você quiser elevar maturidade sem “explodir” o padrão.

| Tag | Para quê serve | Exemplo | Sugestão de valores |
|---|---|---|---|
| `Project` | Diferenciar iniciativas dentro do mesmo workload | `erp-pos` | livre |
| `BusinessUnit` | Unidade/área do negócio | `educacao` | livre |
| `Criticality` | Prioridade operacional | `medium` | `low`, `medium`, `high`, `mission` |
| `ServiceTier` | Camada do serviço | `standard` | `dev`, `standard`, `premium` *(ajuste ao seu contexto)* |
| `Repository` | Link/slug do repositório IaC/app | `github:tftc/erp` | livre |
| `WorkItem` | ID de demanda/tarefa | `azdo-1234` | livre |
| `SupportGroup` | Time de suporte responsável | `cloud-ops` | livre |
| `ContactEmail` | Contato para alertas/segurança | `suporte@empresa.com` | e-mail |
| `CreatedBy` | Auditoria de criação | `raphael` | livre |
| `CreatedOn` | Data de criação | `2026-01-30` | `YYYY-MM-DD` |
| `ExpiryDate` | Recurso temporário (labs) | `2026-06-30` | `YYYY-MM-DD` |
| `Compliance` | Indicador de compliance | `none` | `none`, `lgpd`, `pci`, `sox`, `hipaa` *(exemplos)* |

---

### 3.3 Tags opcionais (Optional / Avançadas)
Boas para ambientes maduros e automação.

| Tag | Para quê serve | Exemplo |
|---|---|---|
| `Backup` | Política de backup esperada | `enabled` |
| `RPO` | Objetivo de perda de dados | `15m` |
| `RTO` | Objetivo de recuperação | `1h` |
| `DR` | Estratégia de DR | `zone`, `region`, `none` |
| `PatchWindow` | Janela de atualização | `sun-02h` |
| `SecurityOwner` | Responsável por segurança | `secops` |

---

## 4) Onde aplicar (prática recomendada)

### 4.1 Resource Groups (ponto “de verdade”)
- Resource Group é o **melhor lugar** para garantir consistência, porque:
  - é onde normalmente se aplica RBAC por domínio;
  - consolida custo por workload/ambiente;
  - facilita herança de tags via políticas.

**Regra:** todo RG deve ter no mínimo as tags Required.

### 4.2 Recursos
- Recursos devem herdar tags do RG sempre que possível.
- Quando um recurso precisa diferir do RG (ex.: banco compartilhado), **documente a exceção**.

---

## 5) Padrão de valores (enums) sugeridos

### 5.1 `Environment`
`dev | test | qa | stage | prod`

### 5.2 `DataClassification`
`public | internal | confidential | restricted`

### 5.3 `Criticality`
`low | medium | high | mission`

### 5.4 `ManagedBy`
`iac | manual | vendor`

---

## 6) Governança (como garantir que todo mundo obedeça)

### 6.1 Modelo prático (o que mais se usa no mercado)
1. **Exigir tags obrigatórias** na criação (deny se não tiver).
2. **Herdar tags do RG para recursos** (append/modify quando possível).
3. **Auditar tags** (audit) em recursos “legados” e corrigir por backlog.

> Implementação típica: **Azure Policy** (Initiatives) aplicadas na Subscription/RG.

### 6.2 Pacotes de políticas recomendados (para a aula)
- **RequireTags**: exige `Application`, `Environment`, `Owner`, `CostCenter`, `ManagedBy`, `DataClassification`
- **InheritFromRG**: aplica tags do RG nos recursos (quando suportado)
- **AllowedValues**: força valores de `Environment`, `DataClassification`, `Criticality`

---

## 7) Exemplos prontos (cenário ERP)

Assumindo workload `erp`, ambiente `dev`, região `eus2`.

### 7.1 Exemplo de tags em um Resource Group `rg-erp-dev-eus2-001`
```text
Application=erp
Environment=dev
Owner=time-erp
CostCenter=pos-arq-azure
ManagedBy=iac
DataClassification=internal
Project=erp-pos
Criticality=medium
SupportGroup=cloud-ops
ContactEmail=suporte@empresa.com
Repository=github:tftc/erp
CreatedBy=raphael
CreatedOn=2026-01-30
ExpiryDate=2026-06-30
```

### 7.2 Exemplo de tags para recursos do ERP (herdadas do RG)
- `app-erp-front-dev-eus2-001`
- `app-erp-api-dev-eus2-001`
- `sqldb-erp-dev-eus2-001`

> **Boa prática:** manter as mesmas tags do RG e só adicionar tags específicas quando necessário (ex.: `Criticality=high` no banco, se ele for o “ponto crítico”).

---

## 8) Checklist de conformidade (para avaliar alunos)

- [ ] Todo **Resource Group** possui todas as tags **Required**.
- [ ] Pelo menos **3 tags Recommended** foram usadas com coerência.
- [ ] Valores seguem o padrão (lowercase, sem espaços, datas ISO).
- [ ] Não existem variações duplicadas (`CostCenter` vs `costcenter`).
- [ ] Foi definido (no documento do aluno) como a governança será aplicada (Policy / processo).
- [ ] Exceções foram documentadas (quando recurso difere do RG).

---

## 9) Sugestão de exercício (mão na massa)

1. Defina as tags (Required + Recommended) do RG do ERP.
2. Aplique as tags no RG e valide que os recursos herdam (ou documente a estratégia).
3. Crie uma política simples:
   - exigir `Environment` e `CostCenter`  
4. Exporte um relatório de custo por `CostCenter` (conceito) e explique o resultado.

---

## 10) Dicas rápidas (as que mais evitam problemas)

- **Poucas tags, bem aplicadas**, valem mais do que muitas tags inconsistentes.
- Defina um “dono” de tag governance (mesmo em aula: um aluno/time).
- Se o recurso é temporário, use `ExpiryDate` — isso muda o jogo em labs.
- Sempre defina `ManagedBy` para separar o que é IaC do que foi criado manualmente.
