# MeuAnalista

**Plataforma web que transforma planilhas em análises inteligentes com IA e ajuda pequenas empresas a organizar o caixa.**

O usuário envia um arquivo CSV, Excel ou JSON, faz perguntas em português e recebe um relatório com resumo, métricas, insights, gráficos interativos e recomendações. Para pequenas empresas, o sistema também oferece um módulo de gestão financeira e de estoque.

🔗 **Acesse:** [meuanalista.ia.br](https://meuanalista.ia.br) · Lançado no Product Hunt em maio de 2026

> O código-fonte fica em um repositório privado por conter a integração de pagamentos de um produto em produção. Este repositório apresenta o projeto, a arquitetura e as decisões técnicas.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js (App Router) · TypeScript · Tailwind CSS · PWA |
| Backend | Python · FastAPI · Pydantic · Pandas |
| IA | API do Claude (Anthropic), com arquitetura multiagente |
| Banco de dados | Supabase / PostgreSQL |
| Pagamentos | Stripe Checkout, Webhooks e Portal do Cliente |
| Autenticação | E-mail e senha (bcrypt) e login com Google |
| E-mails | Resend (verificação de conta, recuperação de senha, alertas) |
| Relatórios | PDF (@react-pdf/renderer), Excel e HTML |
| Gráficos | ApexCharts |
| Deploy | Vercel (frontend) · Railway (backend) · GitHub Actions |

---

## Arquitetura multiagente

A análise não é feita por uma única chamada à IA. Um **orquestrador** planeja o trabalho e distribui as etapas entre **agentes especializados**, cada um com seu próprio prompt e sua própria chamada à API:

```
Arquivo + pergunta do usuário
            │
            ▼
   Perfil do arquivo (file_profiler)
            │
            ▼
       Orquestrador ── define o contexto (empresa ou uso pessoal) e o plano
            │
   ┌────────┼──────────┬──────────────┐
   ▼        ▼          ▼              ▼
Limpeza   Análise   Insights    Visualização
   └────────┴──────────┴──────────────┘
            │
            ▼
      QA (validação) ── confere e enriquece o resultado final
            │
            ▼
   Relatório: resumo, métricas, insights e gráficos
```

Tarefas simples usam um modelo mais rápido e barato, e as análises usam um modelo mais capaz, para equilibrar custo e qualidade.

---

## Funcionalidades

### Análise de dados com IA
- Upload de CSV, Excel (.xlsx/.xls) e JSON
- Perguntas em português, com sugestões automáticas de perguntas para cada arquivo
- Histórico de conversa para perguntas de acompanhamento
- Limpeza automática: remove linhas e colunas vazias, duplicatas e normaliza nomes de colunas
- Gráficos interativos: barra, pizza, área, linha, dispersão e cartões de KPI
- Exportação do relatório em PDF, Excel e HTML

### Gestão para pequenas empresas (ERP)
- Cadastro de produtos e serviços, com histórico de preços
- Controle de entradas, saídas, parcelas e despesas variáveis
- Alerta de estoque mínimo e de parcelas a vencer, enviados por e-mail
- Relatórios mensais, ranking de produtos e exportação para Excel
- Acesso de funcionários com login próprio para registrar vendas
- Registro (log) das operações realizadas no sistema
- Análise do negócio com IA a partir dos dados cadastrados

### Contas e segurança
- Cadastro com verificação de e-mail e login com Google
- Recuperação e troca de senha
- Limite de requisições (rate limiting) para proteger os endpoints de autenticação
- Limpeza automática de contas não verificadas

### Pagamentos e planos
- Checkout hospedado pelo Stripe: nenhum dado de cartão passa pelo servidor
- Webhooks do Stripe ativam e cancelam assinaturas automaticamente
- Portal do cliente para gerenciar ou cancelar a assinatura
- Plano gratuito com limites de uso e planos pagos mensais e anuais

### Suporte e administração
- Sistema de chamados: o usuário abre um ticket e acompanha as respostas
- Painel administrativo com estatísticas de uso e relatório de chamados
- Área para consultores com cadastro e captação de clientes

---

## Decisões técnicas

- **Camada de banco de dados com três modos:** o mesmo código funciona com Supabase, PostgreSQL ou SQLite, escolhido pelas variáveis de ambiente. Isso facilita desenvolver e testar localmente sem depender do banco de produção.
- **Webhooks em vez de confiar no redirecionamento:** a assinatura só é ativada quando o Stripe confirma o pagamento pelo webhook, e não quando o usuário volta para a página de sucesso.
- **Modelos de IA por tipo de tarefa:** etapas simples usam um modelo menor para reduzir o custo por análise.
- **Automação com GitHub Actions:** um workflow agendado mantém o banco do plano gratuito do Supabase ativo.

---

## Autor

**Lucas Sangiorge**, estudante de Análise e Desenvolvimento de Sistemas, focado em desenvolvimento backend com Python.

[GitHub](https://github.com/LucasSangiorge) · [Site do projeto](https://meuanalista.ia.br)
