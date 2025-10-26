Ad Creative

Perfetto ✅
Ora creo per te una documentazione completa in formato Markdown per il workflow n8n “Dashboard Generazione Campagne”.
Questa documentazione è pensata per essere pubblicata su GitHub come README.md del progetto.

---

# 🧠 Dashboard Generazione Campagne (n8n Workflow)

> Sistema automatizzato di generazione campagne pubblicitarie multi-piattaforma (Meta, Google, TikTok) basato su AI.
> Integra modelli Google Gemini, LangChain Agent, e gestione dinamica delle richieste in modalità “Easy” o “Advanced”.

---

## 📋 Indice

1. [Panoramica](#panoramica)
2. [Funzionamento Step-by-Step](#funzionamento-step-by-step)
3. [Nodi Principali](#nodi-principali)
4. [Flusso Dati e Connessioni](#flusso-dati-e-connessioni)
5. [Input JSON e Modalità Operative](#input-json-e-modalità-operative)
6. [Output Strutturato](#output-strutturato)
7. [Esempi di Utilizzo (curl)](#esempi-di-utilizzo-curl)
8. [Credenziali Necessarie](#credenziali-necessarie)
9. [Suggerimenti e Estensioni Future](#suggerimenti-e-estensioni-future)

---

## 🧭 Panoramica

Questo workflow gestisce automaticamente la generazione di campagne pubblicitarie per più piattaforme (Meta Ads, Google Ads, TikTok Ads).
È progettato per integrarsi con dashboard esterne o frontend applicativi, ricevendo input strutturati via Webhook HTTP POST.

Tecnologie chiave:

* 🧩 n8n (orchestrazione)
* 🤖 Google Gemini (PaLM) – generazione testo AI
* 🧱 LangChain Agent – gestione logica, parsing e output strutturato
* 🧮 Structured Output Parser – validazione JSON
* 🔁 Webhook + Switch – routing tra modalità “Easy” e “Advanced”

---

## ⚙️ Funzionamento Step-by-Step

|   #   | Nodo                            | Descrizione                                                                                                                           |
| :---: | :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| 1 | Webhook                     | Riceve una richiesta POST da un’app o form esterno contenente i dati della campagna.                                                |
| 2 | Switch                      | Verifica se la campagna è di tipo "easy" o "advanced", in base a body.campaign_type.                                            |
| 3 | easy mode                   | Prepara un prompt semplificato per la generazione automatica: descrizione prodotto e piattaforme.                                     |
| 4 | advanced mode               | Costruisce un prompt dettagliato con dati su budget, target demografico, obiettivi, tono, colori, CTA ecc.                            |
| 5 | Google Gemini Chat Model    | Fornisce capacità di ragionamento linguistico e generazione AI (LangChain integration).                                               |
| 6 | AI Agent (CampaignForge AI) | Elabora i prompt e genera la strategia pubblicitaria multi-piattaforma. Applica le regole definite per creatività, budget e test A/B. |
| 7 | Structured Output Parser    | Converte la risposta del modello in un JSON coerente e validato.                                                                      |
| 8 | Respond to Webhook          | Restituisce il risultato all’endpoint chiamante (frontend, app o dashboard).                                                          |

---

## 🧩 Nodi Principali

| Nome Nodo                                     | Tipo                                              | Scopo                                                         | Stato           |
| --------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------- | --------------- |
| Webhook                                   | n8n-nodes-base.webhook                          | Punto di ingresso per richieste HTTP POST                     | ✅ Attivo        |
| Switch                                    | n8n-nodes-base.switch                           | Routing tra modalità Easy e Advanced                          | ✅ Attivo        |
| easy mode                                 | n8n-nodes-base.set                              | Costruzione prompt base                                       | ✅ Attivo        |
| advanced mode                             | n8n-nodes-base.set                              | Costruzione prompt avanzato con targeting completo            | ✅ Attivo        |
| Google Gemini Chat Model                  | @n8n/n8n-nodes-langchain.lmChatGoogleGemini     | Modello linguistico AI                                        | ✅ Attivo        |
| AI Agent (CampaignForge AI)               | @n8n/n8n-nodes-langchain.agent                  | Generatore logico di campagne multi-piattaforma               | ✅ Attivo        |
| Structured Output Parser                  | @n8n/n8n-nodes-langchain.outputParserStructured | Parsing JSON con schema di output predefinito                 | ✅ Attivo        |
| Documentazione Ufficiale e Best Practices | @n8n/n8n-nodes-langchain.vectorStoreSupabase    | Archivio vettoriale per consultazione linee guida (opzionale) | ⚙️ Disabilitato |
| Respond to Webhook                        | n8n-nodes-base.respondToWebhook                 | Risposta finale all’utente                                    | ✅ Attivo        |

---

## 🔄 Flusso Dati e Connessioni

graph LR
A[Webhook POST] --> B[Switch campaign_type]
B -->|easy| C[easy mode]
B -->|advanced| D[advanced mode]
C --> E[AI Agent]
D --> E
E --> F[Structured Output Parser]
E --> G[Respond to Webhook]
subgraph AI Layer
  H[Google Gemini Chat Model] --> E
end

---

## 🧠 Input JSON e Modalità Operative

### Modalità Easy

Input minimo:

{
  "campaign_type": "easy",
  "campaign_data": {
    "description": "Promuovere smartwatch fitness con focus su durata batteria",
    "platforms": ["meta", "google", "tiktok"]
  }
}

Prompt generato automaticamente:

URL Prodotto/Servizio (Opzionale): …
Descrizione Campagna: …
piattaforme: meta, google, tiktok

---

### Modalità Advanced

Input completo con targeting, budget, tono e KPI:

{
  "campaign_type": "advanced",
  "campaign_data": {
    "product_url": "https://techfit.it/smartband-pro-x1",
    "demographics": { "age_min": 18, "age_max": 45, "genders": ["all"] },
    "geographic": { "countries": ["IT"], "cities": ["Roma", "Milano"] },
    "objectives": { "primary": "vendite", "kpi": "ROAS > 3.5" },
    "budget": { "total": 1500, "currency": "EUR", "duration": 21 },
    "creative": { "tone_of_voice": "energetico", "cta": "Scopri di più", "message_key": "durata batteria", "color_palette": "nero, blu, argento" },
    "platforms": ["meta", "google", "tiktok"]
  }
}

---

## 📤 Output Strutturato

Il risultato finale (inviato come risposta al webhook) contiene le sezioni:

{
  "creative_strategy": { "total_creatives": 8, ... },
  "meta_ads": { "creative_count": 4, "creatives": [...] },
  "google_ads": { "creative_count": 2, "ad_groups": [...] },
  "tiktok_ads": { "creative_count": 2, "video_concepts": [...] },
  "cross_platform": {
    "total_budget": 1500,
    "budget_distribution": { "meta": 750, "google": 450, "tiktok": 300 },
    "campaign_duration": 21,
    "kpis": ["CTR > 2%", "ROAS > 3.5"],
    "creative_rotation_schedule": "Meta: ogni 4 giorni | Google: simultaneo | TikTok: scaglionato a 7 giorni"
  }
}

---

## 💻 Esempi di Utilizzo (curl)

### Richiesta Easy Mode

curl -X POST https://n8n.tuodominio.it/webhook/3 \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_type": "easy",
    "campaign_data": {
      "description": "Promuovere un nuovo smartwatch fitness",
      "platforms": ["meta", "google", "tiktok"]
    }
  }'

### Richiesta Advanced Mode

curl -X POST https://n8n.tuodominio.it/webhook/3 \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_type": "advanced",
    "campaign_data": {
      "product_url": "https://techfit.it/smartband-pro-x1",
      "budget": { "total": 1500, "currency": "EUR", "duration": 21 },
      "platforms": ["meta", "google", "tiktok"]
    }
  }'

---

## 🔐 Credenziali Necessarie

| Servizio                 | Tipo         | Campo Credenziale            |
| ------------------------ | ------------ | ---------------------------- |
| Google Gemini        | AI Model     | googlePalmApi              |
| OpenAI (opzionale)   | Embeddings   | openAIApiKey               |
| Supabase (opzionale) | Vector Store | supabaseUrl, supabaseKey |

---

## 🚀 Suggerimenti e Estensioni Future

* ✅ Abilitare il nodo “Documentazione Ufficiale e Best Practices” per permettere all’AI di consultare linee guida reali Meta/Google/TikTok.
* 🧩 Aggiungere un Database (PostgreSQL o Notion) per storicizzare le strategie generate.
* 📊 Collegare un nodo “HTTP Request” per inviare automaticamente la strategia a un dashboard frontend.
* 🔁 Integrare un nodo “Cron” per aggiornare periodicamente le campagne attive.
* 📈 Aggiungere un nodo di “Performance Tracking” (API Ads Manager) per ottimizzazione continua.

---

## 📚 Licenza

Distribuito sotto licenza MIT.
Creato per ambienti di automazione n8n AI Campaign Systems.

---
