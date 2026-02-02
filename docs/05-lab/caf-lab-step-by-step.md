# CAF na prática — Aula Produção (ERP + Ecommerce) com 1 Subscription (Canada Central)

Este documento reúne um **passo a passo completo** para uma aula prática baseada no **Cloud Adoption Framework (CAF)**, aplicando **taxonomia/naming**, **tags**, **Azure Policy**, **grupos no Entra ID** e **RBAC** em um cenário com **duas aplicações** compartilhando capacidade de compute.

> **Cenário (Produção)**
> - Região base: **Canada Central**
> - 1 Subscription
> - **1 App Service Plan (Shared)** para reduzir custo (compute compartilhado)
> - 3 WebApps: **ERP Front**, **ERP API**, **Ecommerce Front**, **Ecommerce API**
> - **1 Virtual Machine (ERP)** 
> - 1 VM: **ERP Front**
> - **Dados separados por workload** (para rastreio de custos/governança):
>   - ERP: Azure SQL Server + DB
>   - Ecommerce: Azure SQL Server + DB
> - Observabilidade: Log Analytics + App Insights (shared)
> - Rede (opcional): VNet/Subnets/NSG/Private Endpoint

---

## 1) Parâmetros do ambiente (padronização)

- `env = prod`
- `region_short = cac` *(Canada Central)*
- `region_code = canadacentral`
- `instance = 001`

Workloads do laboratório:
- `workload_1 = erp`
- `workload_2 = ecom` *(ecommerce)*

---

## 2) Padrões CAF usados na aula

### 2.1 Naming (convenção de nomes)

**Padrão base:**
- `<abbr>-<workload>-<env>-<region>-<nn>`

**Com purpose (quando necessário):**
- `<abbr>-<workload>-<purpose>-<env>-<region>-<nn>`

**Importante (shared):**
- Recursos **compartilhados** não devem “pertencer” a um workload no nome.
- Para shared, use `shared` no lugar do workload.

Exemplos rápidos:
- `rg-shared-appsvc-prod-cac-001`
- `rg-asp-shared-prod-cac-001`
- `rg-app-erp-front-prod-cac-001`
- `rg-app-ecom-api-prod-cac-001`
- `rg-sqldb-erp-prod-cac-001`
- `rg-sqldb-ecom-prod-cac-001`

---

## 3) Estrutura recomendada de Resource Groups (produção)

Em produção, separe por domínio (facilita RBAC, custos, operação e governança) e trate **compute compartilhado** como **shared**.

### 3.1 Resource Groups

**Shared / Plataforma**
- `rg-shared-observ-prod-cac-001` *(observabilidade)*
- `rg-shared-appsvc-prod-cac-001` *(App Service Plan compartilhado)*

**Apps — ERP**
- `rg-erp-front-prod-cac-001`
- `rg-erp-api-prod-cac-001`

**Apps — Ecommerce**
- `rg-ecom-front-prod-cac-001`
- `rg-ecom-api-prod-cac-001`

**Dados (separados por custo/governança)**
- `rg-erp-data-prod-cac-001`
- `rg-ecom-data-prod-cac-001`

**Rede (recomendado em prod; opcional para aula curta)**
- `rg-shared-net-prod-cac-001`

### 3.2 O que vai em cada RG

| Resource Group | Conteúdo típico | Por que separar |
|---|---|---|
| `rg-shared-observ-...` | Log Analytics, App Insights (Front/API), alertas | operação e telemetria compartilhadas |
| `rg-shared-appsvc-...` | App Service Plan (compute compartilhado) | custo/compute centralizado + governança por plataforma |
| `rg-erp-front-...` | Virtual Machine Front do ERP | ciclo de vida e permissões do front ERP |
| `rg-erp-api-...` | WebApp API do ERP | ciclo de vida e permissões da API ERP |
| `rg-ecom-front-...` | WebApp Front do Ecommerce | ciclo de vida e permissões do front Ecommerce |
| `rg-ecom-api-...` | WebApp API do Ecommerce | ciclo de vida e permissões da API Ecommerce |
| `rg-erp-data-...` | SQL Server + SQL DB do ERP | controle e custo do ERP bem segregados |
| `rg-ecom-data-...` | SQL Server + SQL DB do Ecommerce | controle e custo do Ecommerce bem segregados |
| `rg-shared-net-...` | VNet/Subnets/NSG/Private Endpoint/DNS | rede costuma ser operada por time específico |

---

## 4) Tags (padrão “mercado” para produção)

**Regra de ouro:** aplique tags **primeiro no RG** e use **Policy** para herdar nos recursos.

### 4.1 Tags obrigatórias (prod)

- `Application=erp|ecom|shared`
- `Environment=prod`
- `Owner=platform-team` *(ou `time-erp` / `time-ecom` por workload)*
- `CostCenter=...` *(ajuste para o seu caso)*
- `Criticality=high`
- `SupportGroup=cloud-ops`
- `ContactEmail=suporte@empresa.com`

### 4.2 Tags recomendadas (prod)

- `Project=...`
- `BusinessUnit=...`
- `Compliance=lgpd` *(se aplicável)*
- `ServiceTier=standard|premium`
- `Repository=github:org/repo`
- `CreatedOn=YYYY-MM-DD`
- `SharedService=true|false`

### 4.3 Modelo de tags por tipo de RG 

#### Shared AppSvc — `rg-shared-appsvc-prod-cac-001`
```text
Application=shared
Environment=prod
Owner=platform-team
CostCenter=pos-disciplina03
Criticality=high
SharedService=true
SupportGroup=appsvc-cloud
```

#### Shared WebApp — `rg-shared-webapp-prod-cac-001`
```text
Application=shared
Environment=prod
Owner=platform-team
CostCenter=pos-disciplina03
Criticality=high
SharedService=true
SupportGroup=webapp-cloud
```

#### Shared Observabilidade — `rg-shared-observ-prod-cac-001`
```text
Application=shared
Environment=prod
Owner=platform-team
CostCenter=pos-disciplina03
Criticality=high
SharedService=true
SupportGroup=observ-cloud
```

#### Shared Network — `rg-shared-net-prod-cac-001`
```text
Application=shared
Environment=prod
Owner=platform-team
CostCenter=pos-disciplina03
Criticality=high
SharedService=true
SupportGroup=net-cloud
```

#### ERP Front — `rg-erp-front-prod-cac-001`
```text
Application=erp
Environment=prod
Owner=time-erp
CostCenter=pos-disciplina03
Criticality=high
Component=frontend
SharedService=false
SupportGroup=erp-cloud-front
Project=erp-core-front
```

#### ERP API — `rg-erp-api-prod-cac-001`
```text
Application=erp
Environment=prod
Owner=time-erp
CostCenter=pos-disciplina03
Criticality=high
Component=api
SharedService=false
SupportGroup=erp-cloud-api
Project=erp-core-api
```

#### ERP Data — `rg-erp-data-prod-cac-001`
```text
Application=erp
Environment=prod
Owner=time-erp
CostCenter=pos-disciplina03
Criticality=high
Component=data
SharedService=false
SupportGroup=erp-cloud-data
Project=erp-core-data
```

#### Ecommerce Front — `rg-ecom-front-prod-cac-001`
```text
Application=ecom
Environment=prod
Owner=time-ecom
CostCenter=pos-disciplina03
Criticality=high
Component=frontend
SharedService=false
SupportGroup=ecom-cloud-front
Project=ecom-core-front
```

#### Ecommerce API — `rg-ecom-api-prod-cac-001`
```text
Application=ecom
Environment=prod
Owner=time-ecom
CostCenter=pos-disciplina03
Criticality=high
Component=api
SharedService=false
SupportGroup=ecom-cloud-api
Project=ecom-core-api
```

#### Ecommerce Data — `rg-ecom-data-prod-cac-001`
```text
Application=ecom
Environment=prod
Owner=time-ecom
CostCenter=pos-disciplina03
Criticality=high
Component=data
SharedService=false
SupportGroup=ecom-cloud-data
Project=ecom-erp-core-data
```

---


## 5) Azure Policy (baseline de produção)

### 5.1 Policies no escopo Subscription (baseline do ambiente)

1) **Allowed locations**
- **O que faz:** só permite criação em regiões aprovadas
- **Escopo:** Subscription
- **Effect recomendado:** `Deny`
- **Parâmetro:** permitir apenas `canadacentral` *ou regiões que você irá trabalhar*

2) **Inherit tags from RG**
- **O que faz:** herda tags do RG para recursos
- **Escopo:** Subscription
- **Effect recomendado:** `Modify`

3) **Allowed virtual machine SKUs**
- **O que faz:** define SKUs de VMs permitido
- **Escopo:** Subscription 
- **Effect recomendado:** `Deny`

4) **Allowed storage account kinds**
- **O que faz:** define SKUs de Storage Accounts
- **Escopo:** Subscription 
- **Effect recomendado:** `Deny`  

### 5.2 Policies no escopo RG de Dados (mais rígido)

5) **SQL — Public network access disabled**
- **O que faz:** bloqueia acesso público ao SQL
- **Escopo:** aplicar nos RGs de dados:
  - `rg-erp-data-prod-cac-001`
  - `rg-ecom-data-prod-cac-001`
- **Effect recomendado:** `Deny`
- **Pré-requisito:** Private Endpoint + DNS privado *(senão você pode travar a aplicação)*

---

## 6) Grupos no Entra ID (para RBAC)

Em produção, prefira **RBAC por grupos**, não por usuários.

### 6.1 Grupos recomendados (plataforma + workloads)

**Plataforma / Shared**
- `grp-shared-prod-platform-admin` *(poucas pessoas)*
- `grp-shared-prod-ops` *(operação/observabilidade)*
- `grp-shared-prod-secops` *(auditoria/segurança)*
- `grp-shared-prod-finops` *(custos/budgets)*

**ERP**
- `grp-erp-prod-dev`
- `grp-erp-prod-dba`

**Ecommerce**
- `grp-ecom-prod-dev`
- `grp-ecom-prod-dba`

---

### 6.2 Criar usuários fictícios (um por grupo) e associar aos grupos

Para o laboratório ficar bem didático, crie **1 usuário fictício para cada grupo**. Assim você consegue **testar RBAC na prática** (logando com cada usuário e validando o que ele consegue ou não consegue fazer).

> **Observação:** em muitos tenants, criar usuários pode exigir licenças/permissionamento. Para aula, você pode:
> - criar usuários “cloud-only” sem licença (quando permitido), ou  
> - usar contas existentes e apenas **adicionar/remover** dos grupos durante o lab.

#### 6.2.1 Usuários sugeridos (genéricos)

| Usuário (Display Name) | UPN (exemplo) | Deve entrar no grupo | Objetivo do teste |
|---|---|---|---|
| `SHARED - Admin - 01` | `shared.admin.01@SEU-DOMINIO.com` | `grp-shared-prod-platform-admin` | validar controle total (apenas 1–2 pessoas) |
| `SHARED - Ops - 01` | `shared.ops.01@SEU-DOMINIO.com` | `grp-shared-prod-ops` | operar recursos em geral |
| `SHARED - Obs - 01` | `shared.obs.01@SEU-DOMINIO.com` | `grp-shared-prod-obs` | operar logs/alerts/observabilidade |
| `SHARED - SecOps - 01` | `shared.secops.01@SEU-DOMINIO.com` | `grp-shared-prod-secops` | visibilidade/auditoria (Reader/Monitoring Reader) |
| `SHARED - FinOps - 01` | `shared.finops.01@SEU-DOMINIO.com` | `grp-shared-prod-finops` | custos/budgets sem alterar recursos |
| `SHARED - Network - 01` | `shared.network.01@SEU-DOMINIO.com` | `grp-shared-prod-network` | operar recursos de rede |
| `SHARED - App Plan - 01` | `shared.appplan.01@SEU-DOMINIO.com` | `grp-shared-prod-asp` | operar App Service Plan |
| `ERP - Dev - 01` | `erp.front.01@SEU-DOMINIO.com` | `grp-erp-prod-front` | publicar/configurar apps Frontend do ERP |
| `ERP - Dev - 02` | `erp.api.02@SEU-DOMINIO.com` | `grp-erp-prod-api` | publicar/configurar apps Backend do ERP |
| `ERP - DBA - 01` | `erp.dba.01@SEU-DOMINIO.com` | `grp-erp-prod-dba` | administrar recurso SQL do ERP |
| `ECOM - Dev - 01` | `ecom.front.01@SEU-DOMINIO.com` | `grp-ecom-prod-front` | publicar/configurar apps FrontEnd do Ecommerce |
| `ECOM - Dev - 02` | `ecom.api.02@SEU-DOMINIO.com` | `grp-ecom-prod-api` | publicar/configurar apps Backend do Ecommerce |
| `ECOM - DBA - 01` | `ecom.dba.01@SEU-DOMINIO.com` | `grp-ecom-prod-dba` | administrar recurso SQL do Ecommerce |


#### 6.2.2 Passo a passo no portal (Microsoft Entra ID)

1. Acesse **Microsoft Entra ID** → **Users** → **New user** → **Create new user**  
2. Preencha **Display name** e **UPN** (use os exemplos da tabela)  
3. Defina a senha inicial (anote para o lab)  
4. Após criar o usuário, vá em **Groups** → selecione o grupo → **Members** → **Add members** → adicione o usuário correspondente

---

## 7) RBAC (roles) — quais usar e onde aplicar

### 7.1 Subscription (mínimo necessário)

| Grupo | Role | Escopo | Observação |
|---|---|---|---|
| `grp-shared-prod-platform-admin` | Owner | Subscription | apenas 1–2 pessoas |
| `grp-shared-prod-ops` | Contributor | Subscription | poucas pessoas |
| `grp-shared-prod-finops` | Cost Management Reader | Subscription | custos e budgets globais |
| `grp-shared-prod-secops` | Reader *(ou Monitoring Reader)* | Subscription | visibilidade sem alteração |
| `grp-shared-prod-obs` | Reader | Subscription | observabilidade |


### 8.2 Shared — Observabilidade

**RG `rg-shared-observ-prod-cac-001`**
- `grp-shared-prod-obs` → **Log Analytics Contributor** → RG
- `grp-shared-prod-obs` → **Application Insights Component Contributor** → RG
- `grp-shared-prod-secops` → **Monitoring Reader** → RG
- `grp-shared-prod-finops` → **Reader** → RG *(para análise de custo, se necessário)*

### 8.3 Shared — App Service Plan

**RG `rg-shared-appsvc-prod-cac-001`**
- `grp-shared-prod-asp` → **Contributor** → RG *(administra o ASP)*
- `grp-ecom-prod-front` → **Reader** → RG *(para conseguir selecionar o ASP ao criar o WebApp)*
- `grp-ecom-prod-api` → **Reader** → RG *(para conseguir selecionar o ASP ao criar o WebApp)*
- `grp-erp-prod-api` → **Reader** → RG *(para conseguir selecionar o ASP ao criar o WebApp)*

> **Nota didática:** para criar WebApp usando um ASP em outro RG, o time precisa **ler** o ASP.

### 8.4 Apps — ERP

**RG `rg-erp-front-prod-cac-001`**
- `grp-erp-prod-front` → **Website Contributor** *(ou Contributor)* → RG

**RG `rg-erp-api-prod-cac-001`** (mesma lógica)
- `grp-erp-prod-api` → **Website Contributor** *(ou Contributor)* → RG

### 8.5 Apps — Ecommerce

**RG `rg-ecom-front-prod-cac-001`**
- `grp-ecom-prod-front` → **Website Contributor** *(ou Contributor)* → RG

**RG `rg-ecom-api-prod-cac-001`** (mesma lógica)
- `grp-ecom-prod-api` → **Website Contributor** *(ou Contributor)* → RG

### 8.6 Dados — separados por workload

**RG `rg-erp-data-prod-cac-001`**
- `grp-erp-prod-dba` → **SQL DB Contributor** *(e/ou SQL Server Contributor)* → RG

**RG `rg-ecom-data-prod-cac-001`**
- `grp-ecom-prod-dba` → **SQL DB Contributor** *(e/ou SQL Server Contributor)* → RG

### 8.7 Network — Isolamento de rede

**RG `rg-shared-net-prod-cac-001`**
- `grp-shared-prod-network` → **Network Contributor** *(e/ou SQL Server Contributor)* → RG


---

## 9) Sequência recomendada da aula (cronológica)

1. **Criar RGs** e aplicar **tags** no nível RG.
2. Criar **grupos no Entra ID** e (opcional) **usuários fictícios**.
3. Aplicar **RBAC** por RG (evitar “subscription-wide” para todos).
4. Criar **Shared Observability** (Log Analytics + App Insights).
5. Criar **App Service Plan shared**.
6. Criar os **WebApps** (ERP + Ecommerce) usando o ASP shared.
7. Criar **SQL Server + SQL DB** separados por workload.
8. Aplicar **Azure Policies** (começar `Audit`, demonstrar compliance, evoluir para `Deny/Modify/DINE`).
9. Validar:
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
- [ ] `Application` usa apenas `erp|ecom|shared` (se houver policy de allowed values).

### Policies
- [ ] `Allowed locations` aplicado (subscription).
- [ ] `Require tags` aplicado (subscription).
- [ ] `HTTPS only` aplicado para App Service.
- [ ] DINE configurou diagnostic settings para Log Analytics.

### RBAC
- [ ] Grupos Entra foram criados.
- [ ] Devs conseguem criar WebApps usando o ASP shared (Reader no RG do ASP).
- [ ] RBAC aplicado em RGs (não “subscription-wide” para todos).
- [ ] FinOps consegue ver custos do ASP (RG shared) e por workload (RGs das apps/dados).

### Observabilidade
- [ ] App Insights configurado para front e api (ERP + Ecommerce).
- [ ] Logs/metrics chegam no Log Analytics.

---

## 11) Mini-resumo (nomes finais)

### Resource Groups
- `rg-shared-observ-prod-cac-001`
- `rg-shared-appsvc-prod-cac-001`
- `rg-erp-front-prod-cac-001`
- `rg-erp-api-prod-cac-001`
- `rg-ecom-front-prod-cac-001`
- `rg-ecom-api-prod-cac-001`
- `rg-erp-data-prod-cac-001`
- `rg-ecom-data-prod-cac-001`
- `rg-shared-net-prod-cac-001` 

### Recursos (principais)
- `log-shared-prod-cac-001`
- `app-erp-front-prod-cac-001`
- `app-erp-api-prod-cac-001`
- `app-ecom-front-prod-cac-001`
- `app-ecom-api-prod-cac-001`
- `asp-shared-prod-cac-001`
- `vm-erp-front-prod-cac-001`
- `app-erp-api-prod-cac-001`
- `app-ecom-front-prod-cac-001`
- `app-ecom-api-prod-cac-001`
- `sql-erp-prod-cac-001` / `sqldb-erp-prod-cac-001`
- `sql-ecom-prod-cac-001` / `sqldb-ecom-prod-cac-001`

---

### Observação final (importante)
Quando você **compartilha** um App Service Plan:
- o custo “base” fica no **ASP (shared)**,
- e o rateio costuma ser feito por **tags** e/ou critérios internos (FinOps).
Separar **dados por workload** ajuda a ter visibilidade de custo e governança clara em produção.
