# Padrão de Nomenclatura Azure (CAF) — Taxonomia Única + Códigos de Recursos + Regiões

Este documento define um padrão único de nomes para recursos no **Microsoft Azure**, inspirado nas recomendações do **Cloud Adoption Framework (CAF)**: começar pelo **tipo do recurso (abreviação)** e, em seguida, identificar **workload**, **ambiente**, **região** e **número da instância**.

---

## 1) Objetivo

Padronizar nomes para:

- facilitar governança, operação e suporte;
- reduzir ambiguidade (o nome “explica” o recurso);
- manter consistência em laboratórios e projetos reais;
- apoiar automação e auditoria (IaC, scripts, inventário).

---

## 2) Componentes do nome

| Componente | Descrição | Exemplos |
|---|---|---|
| `abbr` | abreviação do tipo de recurso | `vnet`, `snet`, `nsg`, `app`, `asp`, `sqldb` |
| `workload` | nome do sistema/projeto | `erp`, `navigator`, `sharepoint` |
| `purpose` *(opcional)* | função/segmento do recurso (quando necessário) | `front`, `api`, `data`, `app`, `mgmt`, `shared` |
| `env` | ambiente | `dev`, `test`, `qa`, `stage`, `prod` |
| `region` | **código curto** (ex.: `eus2`) ou **código oficial** (ex.: `eastus2`) | `brs`, `weu`, `eus2` |
| `instance` | número sequencial | `001`, `002` |

> **Regra de consistência da turma:** escolha **um** padrão para `region` e não misture:  
> - **curto** (`eus2`, `brs`, `weu`) **ou**  
> - **oficial** (`eastus2`, `brazilsouth`, `westeurope`).

---

## 3) Formatos do padrão

### 3.1 Padrão base

```
<abbr>-<workload>-<env>-<region>-<instance>
```

Exemplo:

- `vnet-erp-dev-eus2-001`

### 3.2 Padrão com `purpose` (quando fizer sentido)

Use `purpose` para diferenciar recursos do mesmo tipo com funções distintas (subnets/NSGs/private endpoints/apps, etc.):

```
<abbr>-<workload>-<purpose>-<env>-<region>-<instance>
```

Exemplos:

- `snet-erp-data-dev-eus2-001`
- `app-erp-api-dev-eus2-001`

---

## 4) Catálogo de recursos e códigos (svc)

### 4.1 Rede e Conectividade

| Recurso Azure | Código (svc) |
|---|---|
| Virtual Network | vnet |
| Subnet | snet |
| Network Security Group | nsg |
| Route Table | rt |
| Public IP | pip |
| Network Interface | nic |
| Load Balancer | lb |
| Application Gateway | agw |
| Azure Front Door | fd |
| Private DNS Zone | dnsz |
| Private Endpoint | pep |
| VPN Gateway | vpngw |
| ExpressRoute Gateway | ergw |

### 4.2 Compute

| Recurso Azure | Código (svc) |
|---|---|
| Virtual Machine | vm |
| Virtual Machine Scale Set | vmss |
| Azure Virtual Desktop (Host Pool) | avd |

### 4.3 Web / Apps

| Recurso Azure | Código (svc) |
|---|---|
| App Service Plan | asp |
| App Service WebApp (Frontend) | app |
| App Service WebApp (Backend/API) | app *(use `purpose=api` no nome)* |
| Function App | func |
| API Management | apim |

> **Nota:** mantenha o código `app` para Web App e diferencie Front/API usando `purpose`:
> - `app-erp-front-dev-eus2-001`
> - `app-erp-api-dev-eus2-001`

### 4.4 Dados

| Recurso Azure | Código (svc) |
|---|---|
| Azure SQL Logical Server | sql |
| Azure SQL Database | sqldb |
| PostgreSQL (Flexible Server) | pg |
| Cosmos DB Account | cosmos |
| Azure Cache for Redis | redis |

### 4.5 Storage e Integração

| Recurso Azure | Código (svc) |
|---|---|
| Storage Account | st |
| Service Bus Namespace | sb |
| Event Hubs Namespace | eh |

### 4.6 Segurança e Identidade

| Recurso Azure | Código (svc) |
|---|---|
| Key Vault | kv |
| User Assigned Managed Identity | uai |
| WAF Policy | wafp |

### 4.7 Observabilidade e Operações

| Recurso Azure | Código (svc) |
|---|---|
| Log Analytics Workspace | log |
| Application Insights | appi |
| Azure Monitor Workspace | amw |
| Alert Rule | alrt |

---

## 5) Mapa de regiões — abreviação curta

Este “mapa de abreviações” é útil para reduzir o tamanho dos nomes, mantendo legibilidade.

### 5.1 Américas

| Região (nome) | Código oficial | Abreviação curta |
|---|---|---|
| Brazil South | brazilsouth | brs |
| Brazil Southeast *(restrita)* | brazilsoutheast | brse |
| Canada Central | canadacentral | cac |
| Canada East | canadaeast | cae |
| Central US | centralus | cus |
| Chile Central | chilecentral | clc |
| East US | eastus | eus |
| East US 2 | eastus2 | eus2 |
| Mexico Central | mexicocentral | mxc |
| North Central US | northcentralus | ncus |
| South Central US | southcentralus | scus |
| West Central US | westcentralus | wcus |
| West US | westus | wus |
| West US 2 | westus2 | wus2 |
| West US 3 | westus3 | wus3 |

### 5.2 Europa

| Região (nome) | Código oficial | Abreviação curta |
|---|---|---|
| Austria East | austriaeast | ate |
| Belgium Central | belgiumcentral | bec |
| Denmark East | denmarkeast | dne |
| France Central | francecentral | frc |
| France South *(restrita)* | francesouth | frs |
| Germany North *(restrita)* | germanynorth | gen |
| Germany West Central | germanywestcentral | gwc |
| Italy North | italynorth | itn |
| North Europe | northeurope | neu |
| Norway East | norwayeast | noe |
| Norway West *(restrita)* | norwaywest | now |
| Poland Central | polandcentral | plc |
| Spain Central | spaincentral | spc |
| Sweden Central | swedencentral | swc |
| Switzerland North | switzerlandnorth | chn |
| Switzerland West *(restrita)* | switzerlandwest | chw |
| UK South | uksouth | uks |
| UK West | ukwest | ukw |
| West Europe | westeurope | weu |

### 5.3 Ásia-Pacífico

| Região (nome) | Código oficial | Abreviação curta |
|---|---|---|
| Australia Central *(restrita)* | australiacentral | auc |
| Australia Central 2 *(restrita)* | australiacentral2 | auc2 |
| Australia East | australiaeast | aue |
| Australia Southeast | australiasoutheast | ause |
| Central India | centralindia | cin |
| East Asia | eastasia | eas |
| Indonesia Central | indonesiacentral | idc |
| Japan East | japaneast | jpe |
| Japan West | japanwest | jpw |
| Korea Central | koreacentral | krc |
| Korea South | koreasouth | krs |
| Malaysia West | malaysiawest | myw |
| New Zealand North | newzealandnorth | nzn |
| South India | southindia | sin |
| Southeast Asia | southeastasia | seas |
| West India | westindia | win |

### 5.4 Oriente Médio e África

| Região (nome) | Código oficial | Abreviação curta |
|---|---|---|
| Israel Central | israelcentral | ilc |
| Qatar Central | qatarcentral | qac |
| South Africa North | southafricanorth | zan |
| South Africa West *(restrita)* | southafricawest | zaw |
| UAE Central *(restrita)* | uaecentral | aec |
| UAE North | uaenorth | aen |

---

## 6) Regras e exceções importantes

### 6.1 Quando usar `purpose`
Use `purpose` somente quando ele agrega clareza e evita confusão:

- `snet-...-app-...` vs `snet-...-data-...`
- `nsg-...-app-...` vs `nsg-...-data-...`
- `app-...-front-...` vs `app-...-api-...`
- `pep-...-sql-...` vs `pep-...-st-...`

### 6.2 Recursos com restrições de nome (ex.: Storage Account)
Alguns serviços não aceitam hífen e/ou exigem nome globalmente único.  
Nesses casos, mantenha o padrão **como referência** na documentação e utilize uma **variante compacta** para a criação no Azure.

**Variante compacta sugerida:**

```
<abbr><workload><purpose?><env><region><instance>
```

Exemplo (Storage Account):

- Padrão no documento: `st-erp-dev-eus2-001`
- Nome compacto para criar: `sterpdeveus2001`

> Dica: use sempre minúsculas e remova separadores.

---

## 7) Exemplos finais (cenário ERP)

Assumindo `workload=erp`, `env=dev`, `region=eus2`, `instance=001`.

### 7.1 Rede
- VNet: `vnet-erp-dev-eus2-001`
- Subnet App: `snet-erp-app-dev-eus2-001`
- Subnet Data: `snet-erp-data-dev-eus2-001`
- NSG App: `nsg-erp-app-dev-eus2-001`
- NSG Data: `nsg-erp-data-dev-eus2-001`
- Private Endpoint SQL: `pep-erp-sql-dev-eus2-001`
- Private DNS Zone: `dnsz-erp-dev-eus2-001`

### 7.2 Web / Apps
- App Service Plan: `asp-erp-dev-eus2-001`
- WebApp Front: `app-erp-front-dev-eus2-001`
- WebApp API: `app-erp-api-dev-eus2-001`
- Application Insights Front: `appi-erp-front-dev-eus2-001`
- Application Insights API: `appi-erp-api-dev-eus2-001`
- Log Analytics Workspace: `log-erp-dev-eus2-001`

### 7.3 Dados
- SQL Server (logical): `sql-erp-dev-eus2-001`
- SQL Database: `sqldb-erp-dev-eus2-001`

### 7.4 Storage (exceção compacta)
- Padrão no documento: `st-erp-dev-eus2-001`
- Nome compacto para criar: `sterpdeveus2001`

---

## 8) Exercício rápido (para aula)
Dado o workload `erp`, ambiente `test`, região `brs` e instância `002`, nomeie:

1. Uma VNet
2. Duas subnets (app e data)
3. Dois NSGs (app e data)
4. Um App Service Plan
5. Dois WebApps (front e api)
6. Um SQL Server e um SQL Database
7. Um Log Analytics e dois App Insights (front e api)

> **Gabarito esperado:** siga exatamente as regras das seções 3 e 6.
