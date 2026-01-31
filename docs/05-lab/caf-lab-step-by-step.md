# CAF na prática — Aula Produção (ERP) com 1 Subscription (Canada Central)

Este documento reúne um **passo a passo completo** para uma aula prática baseada no **Cloud Adoption Framework (CAF)**, aplicando **taxonomia/naming**, **tags**, **Azure Policy**, **grupos no Entra ID** e **RBAC** em um cenário de ERP.

> **Cenário (Produção)**
> - Região base: **Canada Central**
> - 1 Subscription
> - 1 App Service Plan
> - 2 WebApps: **Frontend** e **Backend/API**
> - 1 Azure SQL Database (SQL Server + DB)
> - Observabilidade: Log Analytics + App Insights
> - Rede (opcional): VNet/Subnets/NSG/Private Endpoint

---

## 1) Parâmetros do ambiente (padronização)

- `workload = erp`
- `env = prod`
- `region_short = cac` *(Canada Central)*
- `region_code = canadacentral`
- `instance = 001`

---

## 2) Padrões CAF usados na aula

### 2.1 Naming (convenção de nomes)

**Padrão base:**
- `<abbr>-<workload>-<env>-<region>-<nn>`

**Com purpose (quando necessário):**
- `<abbr>-<workload>-<purpose>-<env>-<region>-<nn>`

**Exemplos rápidos:**
- `rg-erp-front-prod-cac-001`
- `app-erp-front-prod-cac-001`
- `app-erp-api-prod-cac-001`
- `sqldb-erp-prod-cac-001`

---

## 3) Estrutura recomendada de Resource Groups (produção)

Em produção, separe por domínio (facilita RBAC, custos, operação e governança).

### 3.1 Resource Groups

1) **Shared / Observability**
- `rg-erp-shared-prod-cac-001`

2) **Apps**
- `rg-erp-front-prod-cac-001`
- `rg-erp-api-prod-cac-001`

3) **Dados**
- `rg-erp-data-prod-cac-001`

4) **Rede (recomendado em prod; opcional para aula curta)**
- `rg-erp-net-prod-cac-001`

### 3.2 O que vai em cada RG

| Resource Group | Conteúdo típico | Por que separar |
|---|---|---|
| `rg-erp-shared-...` | Log Analytics, App Insights, Key Vault (opcional) | operação e telemetria compartilhadas |
| `rg-erp-front-...` | App Service Plan + WebApp Front | ciclo de vida e permissões do front |
| `rg-erp-api-...` | WebApp API/Backend | ciclo de vida e permissões da API |
| `rg-erp-data-...` | SQL Server + SQL DB + diagnósticos | domínio de dados exige controles mais rígidos |
| `rg-erp-net-...` | VNet/Subnets/NSG/Private Endpoint/DNS | rede costuma ser operada por time específico |

---

## 4) Tags (padrão “mercado” para produção)

**Regra de ouro:** aplique tags **primeiro no RG** e use **Policy** para herdar nos recursos.

### 4.1 Tags obrigatórias (prod)

- `Application=erp`
- `Environment=prod`
- `Owner=platform-team` *(ou `time-erp`)*
- `CostCenter=cc-erp-001` *(ajuste para o seu caso)*
- `ManagedBy=iac` *(recomendado em prod; ou `manual` se for aula via portal)*
- `DataClassification=confidential` *(ou `internal`)*
- `Criticality=high`
- `SupportGroup=cloud-ops`
- `ContactEmail=suporte@empresa.com`

### 4.2 Tags recomendadas (prod)

- `Project=erp-core`
- `BusinessUnit=...`
- `Compliance=lgpd` *(se aplicável)*
- `ServiceTier=standard|premium`
- `Repository=github:org/repo`
- `CreatedOn=YYYY-MM-DD`

### 4.3 Exemplo de tags por RG (modelo)

Use o mesmo “núcleo” e mude `Component`.

**Exemplo para `rg-erp-data-prod-cac-001`:**
```text
Application=erp
Environment=prod
Owner=platform-team
CostCenter=cc-erp-001
ManagedBy=iac
DataClassification=confidential
Criticality=high
Component=data
SupportGroup=cloud-ops
ContactEmail=suporte@empresa.com
Project=erp-core
Compliance=lgpd
ServiceTier=standard
CreatedOn=2026-01-30
```

---

## 5) Criar recursos (com nomes + tags)

> **Recomendação didática:** crie primeiro Shared (logs/insights), depois Apps, depois Dados.

### 5.1 Shared / Observability (`rg-erp-shared-prod-cac-001`)

- Log Analytics Workspace: `log-erp-prod-cac-001`
- App Insights Front: `appi-erp-front-prod-cac-001`
- App Insights API: `appi-erp-api-prod-cac-001`
- *(Opcional)* Key Vault: `kv-erp-prod-cac-001`

**Tags adicionais sugeridas:**
- `Component=shared`

### 5.2 Apps

**App Service Plan**  
Se for compartilhado por front+api, você pode colocá-lo em `rg-erp-front...` (mais simples) ou `rg-erp-shared...` (mais corporativo).  
Sugestão para aula: **colocar no RG do front**.

- RG: `rg-erp-front-prod-cac-001`
- App Service Plan: `asp-erp-prod-cac-001`

**WebApp Front**
- RG: `rg-erp-front-prod-cac-001`
- WebApp: `app-erp-front-prod-cac-001`
- Tags: `Component=front`

**WebApp API**
- RG: `rg-erp-api-prod-cac-001`
- WebApp: `app-erp-api-prod-cac-001`
- Tags: `Component=api`

### 5.3 Dados (`rg-erp-data-prod-cac-001`)

- SQL logical server: `sql-erp-prod-cac-001`
- SQL Database: `sqldb-erp-prod-cac-001`
- Tags: `Component=data` e, se quiser reforçar, `Criticality=high`

---

## 6) Azure Policy (baseline de produção)

> Estratégia recomendada: **Audit** por poucos minutos para demonstrar compliance → em seguida aplicar **Deny/Modify/DINE** conforme o caso.

### 6.1 Policies no escopo Subscription (baseline do ambiente)

1) **Allowed locations**
- **O que faz:** só permite criação em regiões aprovadas
- **Escopo:** Subscription
- **Effect recomendado:** `Deny`
- **Parâmetro:** permitir apenas `canadacentral`

2) **Require tags (tags obrigatórias)**
- **O que faz:** bloqueia criação sem as tags essenciais
- **Escopo:** Subscription
- **Effect recomendado:** `Deny`
- **Tags (exemplo):** `Application`, `Environment`, `Owner`, `CostCenter`, `ManagedBy`, `DataClassification`, `Criticality`

3) **Inherit tags from RG**
- **O que faz:** herda tags do RG para recursos
- **Escopo:** Subscription
- **Effect recomendado:** `Modify`

4) **App Service — HTTPS only**
- **O que faz:** exige HTTPS
- **Escopo:** Subscription *(ou RGs de apps)*
- **Effect recomendado:** `Deny`

5) **Diagnostic settings via DeployIfNotExists (DINE)**
- **O que faz:** configura logs/métricas para Log Analytics automaticamente
- **Escopo:** Subscription
- **Effect recomendado:** `DeployIfNotExists`
- **Destino:** `log-erp-prod-cac-001`

6) **Deny public IP (opcional em prod)**
- **O que faz:** reduz exposição bloqueando Public IP
- **Escopo:** Subscription
- **Effect recomendado:** `Deny` *(ou `Audit` se houver exceções)*

> **Dica de organização:** crie uma **Initiative** chamada `CAF-Prod-Baseline` com os itens 1 a 5 (e 6 opcional).

### 6.2 Policies no escopo RG de Dados (mais rígido)

7) **SQL — Public network access disabled**
- **O que faz:** bloqueia acesso público ao SQL
- **Escopo:** `rg-erp-data-prod-cac-001`
- **Effect recomendado:** `Deny`
- **Pré-requisito:** Private Endpoint + DNS privado *(senão você pode travar a aplicação)*

---

## 7) Grupos no Entra ID (para RBAC)

Em produção, prefira **RBAC por grupos**, não por usuários.

### 7.1 Grupos recomendados

- `grp-erp-prod-admin` *(professor/responsável — poucas pessoas)*
- `grp-erp-prod-dev` *(time app/deploy)*
- `grp-erp-prod-ops` *(operação/monitoramento)*
- `grp-erp-prod-dba` *(dados)*
- `grp-erp-prod-secops` *(auditoria/segurança — leitura)*
- `grp-erp-prod-finops` *(custos/budgets)*

---

## 8) RBAC (roles) — quais usar e onde aplicar

### 8.1 Subscription (mínimo necessário)

| Grupo | Role | Escopo | Observação |
|---|---|---|---|
| `grp-erp-prod-admin` | Owner | Subscription | apenas 1–2 pessoas |
| `grp-erp-prod-finops` | Cost Management Reader | Subscription | custos e budgets globais |
| `grp-erp-prod-secops` | Reader *(ou Monitoring Reader)* | Subscription | visibilidade sem alteração |
| `grp-erp-prod-ops` *(opcional)* | Reader | Subscription | visão geral (se necessário) |

### 8.2 RGs de Apps

**RG Front — `rg-erp-front-prod-cac-001`**

| Grupo | Role | Escopo | Por quê |
|---|---|---|---|
| `grp-erp-prod-dev` | Website Contributor *(ou Contributor)* | RG | deploy e configuração do front |
| `grp-erp-prod-ops` | Reader | RG | troubleshooting básico |
| `grp-erp-prod-secops` | Reader | RG | auditoria/inspeção |
| `grp-erp-prod-finops` | Reader | RG | análise de custo por RG |

**RG API — `rg-erp-api-prod-cac-001`** (mesma lógica)

| Grupo | Role | Escopo | Por quê |
|---|---|---|---|
| `grp-erp-prod-dev` | Website Contributor *(ou Contributor)* | RG | deploy e configuração da API |
| `grp-erp-prod-ops` | Reader | RG | troubleshooting |
| `grp-erp-prod-secops` | Reader | RG | auditoria |
| `grp-erp-prod-finops` | Reader | RG | custos |

> **Nota:** Website Contributor atua no App Service, mas **não garante** permissões de rede. Para VNet Integration, normalmente são necessárias permissões adicionais na VNet/Subnet.

### 8.3 RG Data — `rg-erp-data-prod-cac-001`

| Grupo | Role | Escopo | Por quê |
|---|---|---|---|
| `grp-erp-prod-dba` | SQL DB Contributor *(e/ou SQL Server Contributor)* | RG | administrar SQL (recurso) |
| `grp-erp-prod-dev` | Reader | RG | dev normalmente não administra banco no Azure |
| `grp-erp-prod-ops` | Reader | RG | incidentes/investigação |
| `grp-erp-prod-secops` | Reader | RG | auditoria |
| `grp-erp-prod-finops` | Reader | RG | custos do banco |

> Gancho didático: **RBAC (management plane)** ≠ permissões **dentro do SQL (data plane)**.

### 8.4 RG Shared — `rg-erp-shared-prod-cac-001`

| Grupo | Role | Escopo | Por quê |
|---|---|---|---|
| `grp-erp-prod-ops` | Log Analytics Contributor | RG | operar logs/workspace |
| `grp-erp-prod-ops` | Application Insights Component Contributor | RG | operar App Insights |
| `grp-erp-prod-secops` | Monitoring Reader | RG | visibilidade/auditoria |
| `grp-erp-prod-dev` | Reader | RG | leitura de telemetria |
| `grp-erp-prod-finops` | Reader | RG | custo shared |

### 8.5 RG Net (se usar rede) — `rg-erp-net-prod-cac-001`

| Grupo | Role | Escopo | Por quê |
|---|---|---|---|
| `grp-erp-prod-ops` | Network Contributor | RG | operar rede |
| `grp-erp-prod-dev` | Reader | RG | visibilidade/consulta |
| `grp-erp-prod-secops` | Reader | RG | auditoria |

---

## 9) Sequência recomendada da aula (cronológica)

1. **Criar RGs** e aplicar **tags** no nível RG.
2. Criar **grupos no Entra ID** e aplicar **RBAC** por RG.
3. Criar **Shared/Observability** (Log Analytics + App Insights).
4. Criar **App Service Plan + WebApps** (front + api).
5. Criar **SQL Server + SQL Database**.
6. Aplicar **Azure Policies** (começar `Audit`, demonstrar compliance, evoluir para `Deny/Modify/DINE`).
7. Validar:
   - tags herdadas,
   - compliance de policies,
   - permissões por grupos,
   - logs chegando no workspace.

---

## 10) Checklist de validação (para alunos)

### Organização
- [ ] Todos os RGs foram criados conforme padrão.
- [ ] Todos os recursos seguem naming.

### Tags
- [ ] RGs têm as tags obrigatórias.
- [ ] Recursos herdaram tags via Policy (Modify) ou foram aplicadas manualmente.

### Policies
- [ ] `Allowed locations` aplicado (subscription).
- [ ] `Require tags` aplicado (subscription).
- [ ] `HTTPS only` aplicado para App Service.
- [ ] DINE configurou diagnostic settings para Log Analytics.

### RBAC
- [ ] Grupos Entra foram criados.
- [ ] RBAC aplicado em RGs (não “subscription-wide” para todos).
- [ ] Dev não tem Owner na subscription.
- [ ] FinOps consegue ver custos, mas não altera recursos.

### Observabilidade
- [ ] App Insights configurado para front e api.
- [ ] Logs/metrics chegam no Log Analytics.

---

## 11) Mini-resumo (nomes finais)

### Resource Groups
- `rg-erp-shared-prod-cac-001`
- `rg-erp-front-prod-cac-001`
- `rg-erp-api-prod-cac-001`
- `rg-erp-data-prod-cac-001`
- `rg-erp-net-prod-cac-001` *(opcional)*

### Recursos
- `log-erp-prod-cac-001`
- `appi-erp-front-prod-cac-001`
- `appi-erp-api-prod-cac-001`
- `asp-erp-prod-cac-001`
- `app-erp-front-prod-cac-001`
- `app-erp-api-prod-cac-001`
- `sql-erp-prod-cac-001`
- `sqldb-erp-prod-cac-001`

---

### Observação final (importante)
Em produção, **Policy + RBAC + Tags** viram “governança contínua”. O ganho didático é mostrar que CAF não é só teoria: ele vira **estrutura**, **processo**, **padrão** e **controle automatizado**.
