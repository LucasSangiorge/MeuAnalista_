# MeuAnalista

Plataforma web que transforma qualquer planilha em análise inteligente. O usuário sobe um arquivo CSV, Excel ou JSON, faz perguntas em português e recebe um relatório completo gerado por IA — com resumo, métricas, insights, gráficos interativos e recomendações.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js 15 (App Router) + TypeScript + Tailwind CSS |
| Backend | FastAPI (Python) + Uvicorn |
| IA | Anthropic Claude API (claude-haiku-4-5-20251001) |
| Pagamentos | Stripe Checkout + Webhooks |
| PDF | @react-pdf/renderer |
| Gráficos | ApexCharts (react-apexcharts) |
| Dados | Pandas |

---

## Estrutura do projeto

```
MeuAnalista/
├── meuanalista-api/       # Backend FastAPI
│   ├── agents/            # Agentes de IA (orquestrador, análise, insights, visualização, QA)
│   ├── routers/           # Endpoints da API
│   ├── services/          # Lógica de negócio (parser, chart builder, cleaner, stripe, store)
│   ├── models/            # Schemas Pydantic
│   ├── core/              # Configurações (config.py, settings)
│   ├── data/              # subscriptions.json (gerado automaticamente)
│   └── main.py
│
└── meuanalista-web/       # Frontend Next.js
    ├── app/               # Páginas (App Router)
    │   ├── analisar/      # App principal de análise
    │   ├── planos/        # Página de preços
    │   ├── faq/           # Perguntas frequentes
    │   ├── sucesso/       # Pós-pagamento
    │   └── cancelado/     # Pagamento cancelado
    ├── components/
    │   ├── app/           # Componentes do app (ReportCard, ChartRenderer, UpgradeModal, etc.)
    │   ├── landing/       # Componentes da landing page
    │   └── pdf/           # Geração de PDF (PdfReport, PdfExportButton)
    └── lib/               # Utilitários (api.ts, userEmail.ts, pdfTier.ts, session.ts)
```

---

## Funcionalidades implementadas

### Análise com IA
- Upload de arquivos CSV, Excel (.xlsx/.xls) e JSON (até 10 MB)
- Perguntas em português sobre os dados
- Arquitetura multi-agente com 6 agentes Claude especializados:
  - **Orquestrador** — define contexto (PME ou pessoal) e coordena os demais
  - **Limpeza** — identifica problemas no dado antes da análise
  - **Análise** — extrai métricas, padrões e anomalias
  - **Insights** — traduz análise técnica em linguagem simples para leigos
  - **Visualização** — sugere gráficos adequados (barra, pizza, cartão KPI)
  - **QA** — valida e enriquece o resultado final
- Histórico de conversa (até 3 turnos anteriores) para perguntas de follow-up
- Limite de 3 análises/mês para usuários free (bypass para usuários pagos)

### Gráficos interativos
- Renderização com ApexCharts: barra, pizza (donut), área, linha, scatter, KPI card
- Free: 1 gráfico por análise + CTA de upgrade discreto
- Analista/Pro: todos os gráficos desbloqueados

### Exportação em PDF
- Gerado no frontend com @react-pdf/renderer (sem dependência de servidor)
- Tema claro, formato A4
- Exporta gráficos como imagem via `ApexCharts.exec('dataURI')`
- Plano Analista: PDF com branding MeuAnalista no rodapé
- Plano Pro: PDF white-label (sem identificação)

### Limpeza de dados
- Remoção automática de linhas/colunas vazias
- Deduplicação
- Normalização de espaços e nomes de colunas
- Free: arquivo com marca d'água no topo
- Pro: arquivo limpo sem marca d'água

### Pagamentos (Stripe)
- Checkout hospedado pelo Stripe (sem dados de cartão no nosso servidor)
- Dois planos: Analista (R$49/mês) e Pro (R$99/mês)
- Webhook para ativar/desativar assinaturas automaticamente
- Portal do cliente para gerenciar/cancelar assinatura
- Identificação de tier por e-mail (armazenado no localStorage)
- Armazenamento de assinaturas em `data/subscriptions.json`

### UX / Produto
- Onboarding com seleção de perfil: consumidor ou consultor
- Página `/para-consultores` para o perfil profissional
- Sugestões automáticas de perguntas ao subir o arquivo
- UpgradeModal com fluxo de 2 passos: e-mail → escolha de plano → Stripe
- Tela de sucesso pós-pagamento com checklist de benefícios e passo a passo
- Página `/planos` com comparação completa dos 3 tiers
- Página `/faq` com 8 perguntas em linguagem leiga
- Footer com links para FAQ, planos e e-mail de suporte

---

## Planos e preços

| | Grátis | Analista R$49/mês | Pro R$99/mês |
|---|---|---|---|
| Análises | 3/mês | Ilimitadas | Ilimitadas |
| Gráficos | 1 por análise | Todos | Todos |
| Exportar PDF | — | Com logo MeuAnalista | White-label |
| Dados corrigidos | — | — | ✓ |

---

## Rodando localmente

### Pré-requisitos
- Python 3.11+
- Node.js 18+
- Conta Anthropic com créditos
- Conta Stripe (modo test)

### Backend

```bash
cd meuanalista-api
python -m venv .venv
.venv\Scripts\activate        # Windows
source .venv/bin/activate     # Mac/Linux
pip install -r requirements.txt
```

Crie o arquivo `.env` na pasta `meuanalista-api/`:

```env
ANTHROPIC_API_KEY=sk-ant-...
CLAUDE_MODEL=claude-haiku-4-5-20251001
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_MEDIO=price_...
STRIPE_PRICE_PREMIUM=price_...
FRONTEND_URL=http://localhost:3000
ALLOWED_ORIGINS=http://localhost:3000
```

```bash
python -m uvicorn main:app --reload
# API disponível em http://localhost:8000
```

Para receber webhooks do Stripe localmente:

```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

### Frontend

```bash
cd meuanalista-web
npm install
```

Crie o arquivo `.env.local` na pasta `meuanalista-web/`:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

```bash
npm run dev
# App disponível em http://localhost:3000
```

---

## Variáveis de ambiente

### Backend (`meuanalista-api/.env`)

| Variável | Descrição |
|---|---|
| `ANTHROPIC_API_KEY` | Chave da API Anthropic |
| `CLAUDE_MODEL` | Modelo Claude a usar (ex: `claude-haiku-4-5-20251001`) |
| `STRIPE_SECRET_KEY` | Chave secreta do Stripe |
| `STRIPE_WEBHOOK_SECRET` | Secret do webhook Stripe (gerado pelo `stripe listen`) |
| `STRIPE_PRICE_MEDIO` | ID do preço do plano Analista no Stripe |
| `STRIPE_PRICE_PREMIUM` | ID do preço do plano Pro no Stripe |
| `FRONTEND_URL` | URL do frontend (para redirect pós-pagamento) |
| `ALLOWED_ORIGINS` | Origins permitidas no CORS (separadas por vírgula) |

### Frontend (`meuanalista-web/.env.local`)

| Variável | Descrição |
|---|---|
| `NEXT_PUBLIC_API_URL` | URL base da API (ex: `http://localhost:8000`) |

---

## Endpoints principais da API

| Método | Rota | Descrição |
|---|---|---|
| POST | `/upload` | Faz upload do arquivo e retorna `file_id` |
| POST | `/analyze` | Executa análise completa com os agentes |
| POST | `/suggest` | Sugere perguntas para o arquivo carregado |
| GET | `/stripe/tier` | Retorna o tier do usuário por e-mail |
| POST | `/stripe/create-checkout` | Cria sessão de checkout no Stripe |
| POST | `/stripe/portal` | Abre portal do cliente Stripe |
| POST | `/stripe/webhook` | Recebe eventos do Stripe |
| POST | `/clean` | Retorna arquivo limpo para download |
| POST | `/clean/preview` | Lista problemas encontrados no arquivo |

---

## Próximos passos

- [ ] Deploy do backend (Railway)
- [ ] Deploy do frontend (Vercel)
- [ ] Ativar modo live no Stripe
- [ ] Apontar domínio `meuanalista.com.br`
- [ ] Trocar modelo para `claude-sonnet-4-6` em produção
- [ ] Renomear planos para nomes mais intuitivos para leigos

---

## Contato

Suporte: suporte@meuanalista.com.br
