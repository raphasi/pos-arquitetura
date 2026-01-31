# 📘 Arquitetura no Azure — Pós-Graduação (CAF na prática)

Bem-vindo(a)! Este repositório reúne **documentação e laboratórios hands-on** para a disciplina de **Arquitetura no Microsoft Azure**, com foco em aplicar o **Cloud Adoption Framework (CAF)** (e boas práticas do mercado) em um cenário realista. 🚀

O laboratório base trabalha um **ERP** em **produção (prod)** com:
- 🌐 **App Service** (Frontend)
- 🔌 **App Service** (Backend/API)
- 🗄️ **Azure SQL Database**
- 📊 Observabilidade (Log Analytics + App Insights)
- 🛡️ Governança com **Tags**, **Azure Policy** e **RBAC**
- 🇨🇦 Região base: **Canada Central**

---

## 🎯 Objetivos da disciplina

Ao final, você será capaz de:

- 🧱 Estruturar ambientes Azure com **Resource Groups** bem definidos  
- 🏷️ Aplicar **padrões de tags** para governança e FinOps  
- 🧭 Padronizar **naming / taxonomia** (convenção de nomes)  
- 🛡️ Implementar **Azure Policy** como guardrails (Audit / Deny / Modify / DINE)  
- 👥 Organizar acesso com **Microsoft Entra ID + RBAC** (least privilege)  
- 📈 Operar o ambiente com **monitoramento e logs** (observabilidade)  

---

## 🧩 Cenário de Laboratório (ERP)

**Ambiente:** `prod`  
**Região:** `Canada Central` (`cac`)  
**Arquitetura:** App Service Plan + WebApps + SQL Database

Exemplos de nomes:
- `rg-erp-front-prod-cac-001`
- `app-erp-front-prod-cac-001`
- `app-erp-api-prod-cac-001`
- `sqldb-erp-prod-cac-001`

---

## 📂 Estrutura do repositório

Sugestão de organização por “pilares” (CAF):

```text
docs/
  01-naming/
  02-tags/
  03-policy/
  04-rbac/
  05-lab/


