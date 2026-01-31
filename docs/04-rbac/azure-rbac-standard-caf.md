# CAF na prática — Blueprint de Resource Groups + RBAC (matriz de acesso) para o ERP

Este documento é um **modelo prático** para você usar em aula e/ou entregar aos alunos: define uma estrutura de **Resource Groups (RGs)** e uma **matriz RBAC** (quem acessa o quê e com qual nível) para um workload de ERP rodando em **uma única subscription** no **Microsoft Azure**.

> **Cenário de referência (ERP):**
> - WebApp **Front** (App Service)
> - WebApp **API/Back** (App Service)
> - **Azure SQL Database**
> - Observabilidade (Log Analytics + App Insights)
> - (Opcional) Rede: VNet, subnets, NSG, Private Endpoint

---

## 1) Objetivo didático

Ao final do laboratório, o aluno deve conseguir:

1. **Organizar recursos** por domínio operacional (RGs).
2. Aplicar o princípio de **menor privilégio** (least privilege).
3. Entender na prática **escopo RBAC**: subscription vs RG vs recurso.
4. Produzir um entregável claro: **Matriz RBAC + evidência**.

---

## 2) Princípios e regras (padrão “mercado”)

1. **RBAC por grupos**, não por usuários diretamente.
2. **Escopo mínimo necessário**: preferir RG em vez de subscription.
3. **Separar responsabilidades**:
   - dev/app (aplicação)
   - dados (DBA)
   - operações/monitoramento (ops)
   - segurança (secops)
   - finops (custos)
4. Comece simples e **evolua**: se a turma é iniciante, use menos grupos/roles.
5. Use **Owner** com muito cuidado (em aula, restrinja a 1–2 pessoas).

---

## 3) Estrutura recomendada de Resource Groups (RGs)

> Padrão sugerido de RGs por domínio (com naming do seu padrão):
> - `rg-erp-shared-<env>-<region>-001`
> - `rg-erp-front-<env>-<region>-001`
> - `rg-erp-api-<env>-<region>-001`
> - `rg-erp-data-<env>-<region>-001`
> - `rg-erp-net-<env>-<region>-001` *(opcional)*

### 3.1 O que vai em cada RG

| Resource Group | Conteúdo típico | Por que separar |
|---|---|---|
| `rg-erp-shared-...` | Log Analytics (`log`), App Insights (`appi`), Key Vault (`kv`), Automation (opcional) | observabilidade e componentes compartilhados |
| `rg-erp-front-...` | App Service Plan (`asp`) e WebApp Front (`app`) | ciclo de vida e permissões do front |
| `rg-erp-api-...` | WebApp API (`app` com purpose `api`) | ciclo de vida e permissões da API |
| `rg-erp-data-...` | SQL Server (`sql`), SQL DB (`sqldb`), diagnósticos | controle mais rígido de dados |
| `rg-erp-net-...` | VNet (`vnet`), subnets (`snet`), NSG (`nsg`), Private Endpoints (`pep`), DNS Zones (`dnsz`) | rede costuma ter governança própria |

---

## 4) Modelo de grupos (Entra ID) para RBAC

> Você pode criar grupos reais no **Microsoft Entra ID** ou simular com usuários, mas o ideal é **grupos**.

### 4.1 Grupos sugeridos (mínimo viável)

| Grupo (sugestão) | Quem entra | Função |
|---|---|---|
| `grp-erp-dev` | devs / time app | deploy e operação de apps |
| `grp-erp-dba` | DBA / dados | administrar SQL |
| `grp-erp-ops` | operação/monitoramento | ler logs, criar alertas, troubleshooting |
| `grp-erp-secops` | segurança | revisar postura, logs, segredos |
| `grp-erp-finops` | finops/gestão | custos, budgets e relatórios |

> Se quiser simplificar para iniciantes: use apenas `grp-erp-dev`, `grp-erp-ops`, `grp-erp-admin`.

---

## 5) Matriz RBAC (recomendação)

### 5.1 Papéis (roles) comuns em ambientes Azure

> Os nomes podem variar um pouco conforme o catálogo do tenant, mas o conceito é estável.

- **Reader**: leitura total no escopo.
- **Contributor**: cria/altera recursos (não gerencia acesso).
- **User Access Administrator**: gerencia RBAC (atribuições) no escopo.
- **Website Contributor**: focado em App Service (em vez de Contributor genérico).
- **SQL DB Contributor / SQL Server Contributor**: focado em SQL (quando disponível).
- **Monitoring Reader / Log Analytics Reader**: leitura de monitoramento.
- **Log Analytics Contributor**: gerencia workspace (consultas/config).
- **Application Insights Component Contributor**: gerencia App Insights.
- **Key Vault Secrets User**: ler segredos (para apps via Managed Identity).
- **Key Vault Administrator**: administra Key Vault (evitar em excesso).
- **Cost Management Reader / Contributor**: custos e budgets.

> Em aula: prefira **Reader** para segurança/finops e evite dar **Contributor** onde não precisa.

---

### 5.2 Matriz RBAC por Resource Group (recomendada)

#### A) `rg-erp-front-<env>-<region>-001`

| Grupo | Role | Escopo recomendado | Por quê |
|---|---|---|---|
| `grp-erp-dev` | Website Contributor *(ou Contributor)* | RG front | deploy e config do front |
| `grp-erp-ops` | Reader | RG front | troubleshooting básico |
| `grp-erp-secops` | Reader | RG front | auditoria/inspeção |
| `grp-erp-finops` | Reader | RG front | análise de custo por RG |

#### B) `rg-erp-api-<env>-<region>-001`

| Grupo | Role | Escopo recomendado | Por quê |
|---|---|---|---|
| `grp-erp-dev` | Website Contributor *(ou Contributor)* | RG api | deploy e config da API |
| `grp-erp-ops` | Reader | RG api | troubleshooting básico |
| `grp-erp-secops` | Reader | RG api | auditoria/inspeção |
| `grp-erp-finops` | Reader | RG api | análise de custo por RG |

#### C) `rg-erp-data-<env>-<region>-001`

| Grupo | Role | Escopo recomendado | Por quê |
|---|---|---|---|
| `grp-erp-dba` | SQL DB Contributor *(e/ou SQL Server Contributor)* | RG data | administrar SQL (recurso) |
| `grp-erp-dev` | Reader *(ou SQL DB Contributor opcional)* | RG data | app normalmente não precisa administrar banco |
| `grp-erp-ops` | Reader | RG data | investigar incidentes |
| `grp-erp-secops` | Reader | RG data | auditoria |
| `grp-erp-finops` | Reader | RG data | custo do banco |

> **Nota didática:** RBAC controla o **recurso** (management plane), não necessariamente permissões **dentro do SQL** (data plane).

#### D) `rg-erp-shared-<env>-<region>-001`

| Grupo | Role | Escopo recomendado | Por quê |
|---|---|---|---|
| `grp-erp-ops` | Log Analytics Contributor | RG shared | operar logs/workspace |
| `grp-erp-ops` | Application Insights Component Contributor | RG shared | operar App Insights |
| `grp-erp-secops` | Monitoring Reader | RG shared | auditoria/visibilidade |
| `grp-erp-dev` | Reader | RG shared | leitura de telemetria |
| `grp-erp-finops` | Reader | RG shared | custo shared |

**Para Key Vault (se usar):**
- `grp-erp-dev`: **Key Vault Secrets User** — escopo do Key Vault (para o app ler segredos)
- `grp-erp-secops`: **Key Vault Administrator** — escopo do Key Vault (apenas se for administrar)
- `grp-erp-ops`: Reader *(ou Contributor, se operar o cofre)*

#### E) `rg-erp-net-<env>-<region>-001` *(opcional)*

| Grupo | Role | Escopo recomendado | Por quê |
|---|---|---|---|
| `grp-erp-ops` | Network Contributor | RG net | operar rede (quando a aula tiver rede) |
| `grp-erp-secops` | Reader | RG net | auditoria |
| `grp-erp-dev` | Reader | RG net | entendimento/consulta |

---

## 6) No nível Subscription (mínimo)

**Evite** atribuir `Contributor/Owner` para todo mundo na subscription.

| Grupo | Role | Escopo | Observação |
|---|---|---|---|
| `grp-erp-admin` *(professor/responsável)* | Owner | Subscription | apenas 1–2 pessoas |
| `grp-erp-ops` | Reader *(opcional)* | Subscription | se precisar visão ampla |
| `grp-erp-finops` | Cost Management Reader | Subscription | budgets e custos globais |
| `grp-erp-secops` | Reader/Monitoring Reader | Subscription | visibilidade |

---

## 7) Passo a passo (hands-on)

### 7.1 Preparação
1. Criar os RGs do ERP.
2. Criar os grupos no Entra ID (ou definir usuários representando cada grupo).
3. Garantir que apenas o professor tenha Owner na subscription.

### 7.2 Atribuir RBAC
1. Em cada RG: **Access control (IAM)** → **Add role assignment**
2. Atribuir os roles da matriz (seção 5.2).

### 7.3 Validar (evidência)
1. Usuário “Dev” consegue publicar/configurar App Service, mas não altera RBAC.
2. Usuário “DBA” administra SQL (recurso) e vê configurações.
3. Usuário “FinOps” vê custos/budgets sem editar recursos.
4. Usuário “SecOps” vê logs/monitoramento sem alterar infra.

---

## 8) Entregáveis do aluno

1. **Matriz RBAC final** (grupo → role → escopo).
2. Evidência de atribuições (prints ou export).
3. Comentário curto:
   - Por que não dar Owner/Contributor na subscription?
   - Por que separar dados em RG próprio?

---

## 9) Exemplo completo (ERP — `prod`, `cac`)

**RGs:**
- `rg-erp-shared-prod-cac-001`
- `rg-erp-front-prod-cac-001`
- `rg-erp-api-prod-cac-001`
- `rg-erp-data-prod-cac-001`
- `rg-erp-net-prod-cac-001` *(opcional)*

**Resumo de atribuições:**
- `grp-erp-dev` → Website Contributor → `rg-erp-front-prod-cac-001`
- `grp-erp-dev` → Website Contributor → `rg-erp-api-prod-cac-001`
- `grp-erp-dba` → SQL DB Contributor → `rg-erp-data-prod-cac-001`
- `grp-erp-ops` → Log Analytics Contributor → `rg-erp-shared-prod-cac-001`
- `grp-erp-finops` → Cost Management Reader → subscription
