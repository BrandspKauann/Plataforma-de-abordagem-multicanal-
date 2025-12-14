# 🚀 Backend SaaS Multicanal com IA (n8n)

> **Disparo inteligente de mensagens personalizadas (E-mail & WhatsApp)** usando automação, IA e banco de dados em tempo real.

---

## 🧭 Visão Geral

Este workflow do **n8n** implementa o **backend de um SaaS de outbound multicanal**, focado em **prospecção B2B altamente personalizada**.

Ele permite que qualquer aplicação externa (frontend, painel admin ou outro serviço) **inicie campanhas via API**, enquanto o n8n orquestra todo o processamento:

* 📥 Recebimento da campanha
* 🧠 Pesquisa de mercado com IA
* ✍️ Escrita automática de e-mails personalizados
* 📤 Envio de mensagens
* 📊 Atualização de status e métricas

---

## 🏗️ Arquitetura Geral

```
[ Frontend / App ]
        |
        v
[ Webhook (API) ]
        |
        v
[ Normalização de Dados ]
        |
        v
[ Supabase (Leads) ]
        |
        v
[ Loop de Leads ]
        |
        v
[ IA: Pesquisa → Assunto → Corpo ]
        |
        v
[ Envio de Mensagem ]
        |
        v
[ Atualização de Status ]
```

---

## 🧰 Tecnologias Utilizadas

| Camada         | Tecnologia                | Função                         |
| -------------- | ------------------------- | ------------------------------ |
| Orquestração   | **n8n**                   | Backend serverless e automação |
| Banco de Dados | **Supabase (PostgreSQL)** | Leads e status de campanha     |
| IA             | **OpenAI (LangChain)**    | Pesquisa + Copywriting         |
| Comunicação    | **Webhook REST**          | Entrada da campanha            |
| Envio          | **SMTP**                  | Disparo de e-mails             |

**Modelos de IA:**

* gpt-4.1-mini
* gpt-5-mini

---

## 🔌 API – Entrada do Sistema

### Endpoint

```
POST /webhook/plataforma
```

### Payload esperado

```json
{
  "campaign": {
    "name": "Campanha Teste",
    "tableName": "campaign_leads_vitoria"
  },
  "email": {
    "enabled": true,
    "data": {
      "seuNome": "Kauann",
      "nomeEmpresa": "Brandsp",
      "solucaoEmpresa": "Automação com IA",
      "tomIA": "profissional"
    }
  },
  "whatsapp": {
    "enabled": false
  }
}
```

---

## 🧩 Normalização de Dados

**Node:** `Edit Fields`

Transforma o payload da API em variáveis internas padronizadas:

* 🏷️ Nome da campanha
* 📧 Canal e-mail ativo
* 💬 Canal WhatsApp ativo
* 👤 Nome do usuário
* 🏢 Empresa
* 🎯 Solução
* 🧠 Tom da IA
* 🗃️ Tabela de leads (Supabase)

> Isso desacopla completamente o frontend da lógica interna do workflow.

---

## 🗃️ Leitura de Leads (Supabase)

**Node:** `Get many rows`

| Configuração | Valor              |
| ------------ | ------------------ |
| Operação     | getAll             |
| Tabela       | Dinâmica (via API) |

Cada lead deve conter:

* `nome`
* `empresa`
* `email`

---

## 🔀 Controle de Canal

**Node:** `Tem whatsapp também?`

Decide o fluxo da campanha:

* ✅ Apenas e-mail
* 🔄 E-mail + WhatsApp (estrutura pronta)

> O projeto já nasce **multicanal por design**.

---

## 🔁 Loop de Processamento

**Node:** `Split In Batches`

* Processa leads em lotes
* Evita sobrecarga de API
* Permite escalar campanhas grandes

---

## 🧠 Pesquisa de Mercado com IA

**Nodes:** `Pesquisa de mercado`

A IA analisa automaticamente:

* Modelo de negócio
* Setor e mercado
* Público-alvo
* Notícias recentes
* Dores e oportunidades

📏 **Restrições:**

* Resposta curta
* Conteúdo acionável
* Limite de caracteres

---

## 📨 Geração do Assunto

**Node:** `Assunto`

Regras:

* 1 único assunto
* Personalizado com nome da empresa
* Linguagem B2B
* Emoji profissional
* Foco em dor ou oportunidade

---

## ✍️ Geração do Corpo do E-mail

**Node:** `Corpo do email`

A IA atua como **copywriter sênior de outbound B2B**, gerando:

* Saudação personalizada
* Conexão direta com a dor
* Proposta de valor implícita
* CTA de baixo atrito (15 min)

📤 Saída: **somente o corpo do e-mail**

---

## 📤 Envio de Mensagens

**Node:** `Send email`

* SMTP configurado
* Assunto e corpo gerados por IA
* Pronto para escala

---

## 📊 Atualização de Status

**Node:** `Update a row`

Após o envio:

* `enviou_email = SIM`
* Evita duplicidade
* Base para métricas

---

## 📐 Formulação do Problema

### 🎯 Objetivo

Maximizar taxa de resposta em outbound B2B mantendo escala e personalização.

### 🔢 Variáveis

* **N** = número de leads
* **B** = tamanho do batch
* **Cᵢ** = custo por chamada de IA
* **Tᵢ** = tempo médio por lead

### ⏱️ Complexidade

* Temporal: **O(N)**

### 💰 Custo estimado

```
Custo ≈ N × (C_pesquisa + C_assunto + C_corpo)
```

---

## 🌟 Pontos Fortes

* Backend serverless
* Arquitetura orientada a eventos
* Fácil integração com qualquer frontend
* IA desacoplada
* Escalável por batch
* Multicanal nativo

---

## 🛣️ Próximas Evoluções

* WhatsApp (Z-API, Twilio, Gupshup)
* A/B testing de assuntos
* Rate limit inteligente
* Webhook de callback
* Dashboard de métricas

---

## ✅ Conclusão

Este workflow transforma o **n8n em um backend completo de SaaS de outbound com IA**, combinando **escala, personalização e baixo custo**, pronto para produção e crescimento.
