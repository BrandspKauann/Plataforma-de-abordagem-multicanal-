# Visão Geral do Projeto

Este workflow do **n8n** implementa o **backend de um SaaS de disparo de mensagens personalizadas multicanal (E-mail e WhatsApp)**, com foco em **prospecção B2B outbound altamente personalizada**, utilizando **IA generativa**, **Supabase como base de dados** e **gatilho via API (Webhook)**.

O objetivo é permitir que aplicações externas (frontend, painel admin ou outro serviço) iniciem campanhas passando parâmetros de personalização, enquanto o n8n orquestra:

* Leitura de leads
* Pesquisa de mercado automatizada via IA
* Geração de assunto e corpo de e-mails personalizados
* Disparo de mensagens
* Atualização de status da campanha

---

# Arquitetura Geral

**Fluxo macro:**

1. Aplicação externa envia requisição HTTP
2. n8n recebe via Webhook
3. Parâmetros da campanha são normalizados
4. Leads são buscados no Supabase
5. Para cada lead:

   * Pesquisa de mercado com IA
   * Geração de assunto
   * Geração do corpo do e-mail
   * Envio do e-mail
   * Atualização do status no banco

---

# Tecnologias Utilizadas

## Orquestração

* **n8n** – Automação e backend serverless

## Banco de Dados

* **Supabase** (PostgreSQL)

  * Leitura de leads
  * Atualização de status da campanha

## Inteligência Artificial

* **OpenAI (via LangChain no n8n)**

  * Modelos utilizados:

    * gpt-4.1-mini
    * gpt-5-mini

## Comunicação

* **Webhook HTTP (REST)** – Entrada do SaaS
* **SMTP** – Envio de e-mails

---

# Entrada da API (Webhook)

## Endpoint

```
POST /webhook/plataforma
```

## Estrutura esperada do JSON

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

# Normalização de Dados (Edit Fields)

O node **Edit Fields** padroniza os dados recebidos e cria variáveis internas como:

* nome da campanha
* canais ativos (email / whatsapp)
* nome do usuário
* empresa
* solução
* tom da IA
* nome da tabela no Supabase

Isso desacopla o frontend da lógica interna do workflow.

---

# Leitura de Leads (Supabase)

Node: **Get many rows**

* Operação: `getAll`
* Tabela dinâmica (recebida via Webhook)
* Retorna todos os leads da campanha

Cada lead deve conter ao menos:

* nome
* empresa
* email

---

# Controle de Canal (IF)

Node: **Tem whatsapp também?**

Responsável por decidir o fluxo:

* Apenas e-mail
* E-mail + WhatsApp (estrutura preparada)

Atualmente o fluxo de WhatsApp está previsto, mas o disparo está focado em e-mail.

---

# Loop de Processamento de Leads

Node: **Split In Batches**

* Processa os leads em lotes
* Evita sobrecarga de API e modelos de IA
* Permite escalar campanhas grandes

---

# Pesquisa de Mercado com IA

Node: **Pesquisa de mercado / Pesquisa de mercado.**

## Objetivo

Criar contexto real da empresa-alvo antes da escrita do e-mail.

## A IA analisa:

* Modelo de negócio
* Setor
* Público-alvo
* Notícias recentes
* Possíveis dores e oportunidades

## Restrições

* Resposta curta
* Foco em dados acionáveis
* Limite de caracteres

Essa saída alimenta os próximos agentes de IA.

---

# Geração do Assunto do E-mail

Node: **Assunto**

## Regras principais

* Apenas 1 assunto
* Personalizado com nome da empresa
* Linguagem B2B
* Emoji profissional
* Foco em dor ou oportunidade
* Evita palavras de spam

---

# Geração do Corpo do E-mail

Node: **Corpo do email**

## Estratégia

A IA assume o papel de **copywriter sênior de outbound B2B**, gerando:

* Saudação personalizada
* Conexão com a dor do lead
* Proposta de valor implícita
* CTA de baixo atrito (15 minutos)

A saída é **somente o corpo do e-mail**, pronto para envio.

---

# Envio de E-mail

Node: **Send email1**

* SMTP configurado
* Assunto gerado pela IA
* Corpo totalmente personalizado

---

# Atualização de Status da Campanha

Node: **Update a row (Supabase)**

Após o envio:

* Campo `enviou_email = SIM`
* Evita reenvio duplicado
* Permite métricas e relatórios

---

# Cálculos e Formulação do Problema

## Objetivo

Maximizar **taxa de resposta em outbound B2B** mantendo escala e personalização.

---

## Variáveis

* **N** = número de leads
* **B** = tamanho do batch
* **Cᵢ** = custo por chamada de IA
* **Tᵢ** = tempo médio por lead

---

## Complexidade

* Complexidade temporal: **O(N)**
* Cada lead é processado uma única vez

---

## Custo estimado por campanha

```
Custo total ≈ N × (C_pesquisa + C_assunto + C_corpo)
```

Onde:

* Cada termo representa uma chamada de modelo de linguagem

---

# Pontos Fortes da Arquitetura

* Backend serverless
* Totalmente orientado a eventos
* Fácil integração com qualquer frontend
* IA desacoplada da aplicação
* Escalável por batch
* Multicanal por design

---

# Próximas Evoluções Sugeridas

* Implementar disparo WhatsApp (Z-API, Twilio, Gupshup)
* Rate limit por domínio
* A/B testing de assuntos
* Webhook de callback (status)
* Dashboard de métricas

---

# Conclusão

Este workflow transforma o n8n em um **backend completo de SaaS de outbound com IA**, capaz de competir com plataformas enterprise, mantendo flexibilidade, baixo custo e alto nível de personalização.
