# Azure Policy (CAF) — Políticas essenciais para ambientes em geral

Este documento descreve um baseline prático de **Azure Policy** alinhado ao espírito do **Cloud Adoption Framework (CAF)**, com foco em políticas mais usadas no mercado para **governança, segurança, observabilidade e controle de custos**.

> **Objetivo:** criar *guardrails* automáticos para padronizar configurações, reduzir riscos e manter compliance de forma contínua.

---

## 1) Objetivo

Criar um baseline de governança para:

- padronizar configurações e reduzir *drift*;
- diminuir risco (segurança e exposição indevida);
- aumentar rastreabilidade (tags, logging);
- controlar custos (SKUs, regiões permitidas, recursos proibidos).

Em termos simples: **Azure Policy é o “guardrail automático”**.

---

## 2) Princípios de implementação (padrão de mercado)

1. **Comece em `Audit`** (descoberta) → depois evolua para **`Deny`/`Modify`** (enforcement).
2. Padronize via **Initiatives** (pacotes): *Tagging*, *Security Baseline*, *Observability*.
3. Aplique no **nível mais alto que fizer sentido** (subscription) e use exceções controladas (ex.: RG de laboratório).
4. Evite `Deny` agressivo no começo (pode quebrar deploy). Prefira `Audit` e `DeployIfNotExists` para aprender primeiro.

---

## 3) Effects mais usados (e quando usar)

- **`Deny`**: bloqueia criação/alteração fora do padrão (alto impacto; use quando tiver certeza).
- **`Audit`** / **`AuditIfNotExists`**: não bloqueia; registra não conformidade (ideal para iniciar).
- **`Modify`**: corrige automaticamente (muito útil para tags; às vezes para configurações).
- **`DeployIfNotExists` (DINE)**: implanta automaticamente recurso/config necessária (ex.: *diagnostic settings*).

> Na prática, o caminho de maturidade mais comum é: **Audit → Modify/DINE → Deny**.

---

## 4) Níveis de aplicação (escopo) para uma aula com 1 subscription

- **Subscription**: baseline geral (regras do “ambiente todo”).
- **Resource Group**: regras específicas por domínio (ex.: RG de dados mais rígido).
- **Resource**: normalmente só para exceções; pouco usado em laboratório.

---

## 5) Catálogo de políticas principais (baseline)

A seguir, políticas agrupadas por tema. Em cada item: **o que faz / importância / onde aplicar / effect recomendado**.

---

### 5.1 Governança e Padronização

#### Policy: Exigir tags obrigatórias
- **O que faz:** exige tags como `Environment`, `CostCenter`, `Owner`, `Application`.
- **Importância:** sem tags você perde rastreabilidade e controle de custos (FinOps).
- **Onde aplicar:** **Subscription** (com exceção controlada, se necessário).
- **Effect recomendado:** começar em **Audit** e evoluir para **Deny**; para herança usar **Modify**.

#### Policy: Herdar tags do Resource Group para recursos
- **O que faz:** copia tags do RG para recursos criados sem tags (onde suportado).
- **Importância:** reduz esforço e inconsistência.
- **Onde aplicar:** **Subscription** (ou em RGs padrão).
- **Effect recomendado:** **Modify**.

#### Policy: Restringir regiões permitidas (*Allowed locations*)
- **O que faz:** permite criar recursos apenas em regiões aprovadas (ex.: `brazilsouth`, `eastus2`).
- **Importância:** compliance, residência de dados, latência, custo e padronização.
- **Onde aplicar:** **Subscription**.
- **Effect recomendado:** **Audit** inicialmente; depois **Deny**.

---

### 5.2 Segurança (baseline)

#### Policy: Exigir HTTPS/TLS em serviços públicos
- **O que faz:** força que endpoints usem HTTPS (ex.: App Service).
- **Importância:** reduz risco de tráfego em texto claro.
- **Onde aplicar:** **Subscription** (ou RG de apps).
- **Effect recomendado:** **Audit** → **Deny** (dependendo do impacto).

#### Policy: Bloquear IP público em NIC/VM (quando aplicável)
- **O que faz:** impede que VMs tenham exposição direta à internet.
- **Importância:** reduz superfície de ataque.
- **Onde aplicar:** **Subscription** (baseline corporativo) ou **RG** (produção/mais rígido).
- **Effect recomendado:** **Audit** em labs; **Deny** em ambientes maduros.

#### Policy: Exigir *secure transfer* / criptografia no Storage
- **O que faz:** garante “Secure transfer required” e/ou restringe configurações inseguras.
- **Importância:** evita tráfego não criptografado e configurações frágeis.
- **Onde aplicar:** **Subscription**.
- **Effect recomendado:** **Deny** (ou **Modify** quando suportado).

#### Policy: Bloquear acesso público ao SQL (*Public network access*)
- **O que faz:** força SQL a não aceitar acesso público (somente privado).
- **Importância:** reduz exposição do banco.
- **Onde aplicar:** **RG de dados** (ou subscription se for padrão da empresa).
- **Effect recomendado:** **Audit** no começo; **Deny** somente se o lab já usar rede privada.

---

### 5.3 Observabilidade (monitoramento e logs)

#### Policy: Exigir *Diagnostic Settings* (enviar logs/métricas)
- **O que faz:** garante que recursos enviem logs e métricas para Log Analytics/Storage/Event Hub.
- **Importância:** sem logs você não opera e não investiga incidentes.
- **Onde aplicar:** **Subscription**.
- **Effect recomendado:** **DeployIfNotExists**.

#### Policy: Padronizar o destino de logs (Workspace alvo)
- **O que faz:** define (via parâmetros em initiative) o workspace “alvo” para todos os diagnósticos.
- **Importância:** padroniza observabilidade e facilita correlação.
- **Onde aplicar:** **Subscription**.
- **Effect recomendado:** usar via **Initiative** com DINE.

---

### 5.4 Custos e FinOps

#### Policy: Restringir SKUs/tiers
- **O que faz:** limita SKUs caros (App Service Plan, SQL tiers premium) a cenários/ambientes aprovados.
- **Importância:** evita “custo surpresa”.
- **Onde aplicar:** **Subscription** ou **RG** (ex.: prod permite tiers maiores).
- **Effect recomendado:** **Audit** → **Deny** (com parâmetros).

#### Policy: Bloquear tipos de recurso não permitidos
- **O que faz:** impede criação de recursos fora do catálogo (ex.: Kubernetes/Databricks) no lab.
- **Importância:** controle e simplicidade; reduz custo e bagunça.
- **Onde aplicar:** **Subscription** do lab.
- **Effect recomendado:** **Deny**.

---

### 5.5 Rede e Arquitetura (para evoluir o lab)

#### Policy: Exigir NSG em subnets
- **O que faz:** garante (ou audita) que toda subnet tenha NSG associado.
- **Importância:** controle de tráfego e segmentação.
- **Onde aplicar:** **Subscription** ou RG de rede.
- **Effect recomendado:** **AuditIfNotExists** → **Deny** (se fizer sentido).

#### Policy: Restringir criação de Public IP
- **O que faz:** bloqueia (ou audita) recursos do tipo Public IP.
- **Importância:** reduz exposição; incentiva serviços gerenciados.
- **Onde aplicar:** **Subscription** (ou apenas RG “prod-like”).
- **Effect recomendado:** **Audit** em labs; **Deny** em empresas.

---

## 6) “Pacotes” (Initiatives) recomendados

### Initiative 1 — Tagging Baseline
Inclui:
- RequireTags (**Audit/Deny**)
- InheritTagsFromRG (**Modify**)
- AllowedValues para tags críticas (opcional)

**Escopo:** Subscription  
**Meta:** padronizar inventário e custo.

### Initiative 2 — Security Baseline (mínimo)
Inclui:
- HTTPS only (**Audit/Deny**)
- Storage secure transfer (**Deny/Modify**)
- No public IP (**Audit**)
- SQL public network disabled (**Audit**)

**Escopo:** Subscription (ou RG por domínio)  
**Meta:** reduzir exposição.

### Initiative 3 — Observability Baseline
Inclui:
- Diagnostic settings via **DeployIfNotExists** (App Service, SQL, Storage, etc.)
- parâmetros apontando para 1 Log Analytics Workspace

**Escopo:** Subscription  
**Meta:** garantir logs/monitoramento.

---

## 7) Roteiro “mão na massa” (sugestão de aula)

1. Criar RGs do workload (ex.: ERP).
2. Aplicar **Tagging Baseline** em `Audit`.
3. Rodar compliance e ver recursos não conformes.
4. Ajustar tags e evoluir para `Deny` (quando estiver estável).
5. Ativar **Observability Baseline** com DINE (logs para Log Analytics).
6. Discutir trade-offs:
   - por que `Deny` pode quebrar pipeline;
   - por que `Audit` não resolve tudo;
   - quando `Modify` é melhor que `Deny`.

---

## 8) Exemplos (escopo e resultado esperado)

### Exemplo A — *Allowed locations* (Subscription)
- **Escopo:** subscription
- **Resultado:** só permite `brazilsouth` e `eastus2` (ou as regiões definidas).

### Exemplo B — Tags obrigatórias + herança (Subscription)
- **Require:** `Environment`, `CostCenter`, `Owner`, `Application`
- **Inherit:** copiar tags do RG para recursos (Modify).

### Exemplo C — Diagnostics (Subscription)
- **DINE** para enviar logs de:
  - App Service (front e api)
  - Azure SQL
  - Storage
- Destino: workspace central (ex.: `log-erp-dev-eus2-001`)

---

## 9) Checklist de conformidade (para avaliação)

- [ ] Existe pelo menos 1 initiative de **Tagging** atribuída na subscription.
- [ ] Todas as tags obrigatórias aparecem nos RGs.
- [ ] Existe baseline de **Allowed locations** (audit ou deny).
- [ ] Existe pelo menos 1 política de segurança aplicada (HTTPS/secure transfer/no public ip).
- [ ] Diagnósticos estão configurados via DINE (ou documentada estratégia alternativa).
- [ ] Exceções (se existirem) estão documentadas e justificadas.

