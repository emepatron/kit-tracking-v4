# Histórico e decisões — de onde veio este kit

Registro honesto de como chegamos aqui. Serve para quem pegar este projeto no futuro entender o **porquê** de ele ser um kit de skills, e não um app.

## A origem

Partiu de uma ideia trazida pelo **Jorge** ao time de GTs (referência de mercado: `trackerads.app`, ~R$297/mês): o lead do WhatsApp entra no CRM, um humano marca o resultado (comprou, sem perfil, curioso), e isso é enviado à Meta pela API de Conversões — para o algoritmo aprender quem realmente compra, em vez de otimizar por quem só clica.

## O caminho que NÃO deu certo (e a lição)

Antes deste kit, foi construído um **app completo** (V4 Tracker): Next.js + Postgres, deploy no VPS, 6 fases, ~1.378 testes, com Auditor de GTM/Meta, Implementador de tags, Ponte de WhatsApp, credenciais na interface, login Google, painéis. Funcionava e chegou a rodar em produção.

**Por que foi descontinuado (01/08/2026):** cresceu muito além da ideia original, sem nenhum GT ter usado de verdade. O dono (Emerson) concluiu que virou complexidade desnecessária para o que, no fundo, é uma camada de integração. O app foi **apagado por completo** (VPS, banco, DNS, repo, credenciais).

**As lições que ficaram:**
1. Validar valor com o usuário real **antes** de expandir escopo. Seis fases sem um GT usando foi o erro-raiz.
2. O dono pedir uma feature não significa que o produto precisa dela. Faltou puxar o freio de escopo mais cedo.
3. "Se o usuário precisa perguntar qual é o valor, o produto falhou."

## A decisão: kit de skills, não app

O insight que resolveu tudo: **a camada de inteligência é encanamento** — e encanamento é o que o n8n faz. A V4 já tem n8n rodando (`n8n.pulsefy.ia.br`) e o time já usa Claude Code. Então, em vez de um app para manter:

- Um **repositório de skills** que o time clona e abre no Claude Code / Codex.
- Porta de entrada `/iniciar` (feita para leigo) que pergunta o cenário e roteia.
- O trabalho vira **workflow no n8n**, duplicável por cliente, com template por CRM.

Vantagens sobre o app: zero manutenção, distribuição por `git clone`, melhora sozinho a cada skill nova (o time puxa no próximo pull), e a "regra" vira guia que conduz o humano em vez de código que precisa rodar 24/7.

## O que este kit faz

- **auditar** — raio-X da página pública (pixel, GA4, GTM, Clarity, WhatsApp) + auditoria por dentro do GTM (regras reais herdadas do app: pixel duplicado considerando gatilhos, GA4 ausente, gatilho duplo em conversão, Conversion Linker, tag base do pixel, Clarity).
- **trackear** — monta/conserta as tags por tipo de funil, com snippets oficiais prontos e o caminho de criar em rascunho via API quando há acesso.
- **inteligencia-crm** — o coração: monta o fluxo CRM → Meta no n8n (template Pulsefy/GHL de 4 nós), guiando ou fazendo por API. Google é fase 2 (gclid do form + planilha).

## Aprendizados técnicos que sobreviveram (validados contra APIs reais)

- **GHL / Pulsefy:** host `services.leadconnectorhq.com`, header `Version: 2021-07-28`, User-Agent não-Python. O `attributionSource` do contato traz `adId`, `adName`, `ctwaClid` (campanhas CTWA de WhatsApp) e as UTMs capturadas por formulário. Confirmado lendo contato real.
- **Casamento de lead:** por identidade (telefone/e-mail). A Meta atribui a campanha sozinha — o app nunca manda "campanha", então é impossível contaminar atribuição.
- **CAPI Meta:** evento com telefone/e-mail **sempre hasheados** (SHA-256, normalizados). `event_id` idempotente (com `dealId` quando há) evita contar a mesma venda duas vezes. Validar sempre com `test_event_code` antes de ligar.
- **Google:** atribui por `gclid` (não por identidade). No fluxo de formulário, o gclid precisa estar num campo oculto. Conversão offline sobe por CSV manual **ou** planilha do Google com importação agendada (automático, sem depender da aprovação da API do Google Ads).
- **Régua de otimização Meta:** otimizar por evento pode desde o dia 1 (aprendizado limitado abaixo de ~50 eventos/semana por conjunto). Lookalike precisa de 100 pessoas acumuladas. Sinais negativos não fazem a Meta "evitar" gente ruim sozinha — servem para público de exclusão e relatório.

## Regras de segurança (herdadas do incidente de junho/2026)

- O fluxo **só lê e envia para fora** — nunca escreve, edita ou apaga no CRM do cliente.
- Telefone e e-mail **sempre hasheados** antes de sair.
- Testar antes de ligar em cliente real.

## Estado em 01/08/2026

- Repo público: `github.com/emepatron/kit-tracking-v4`.
- Skills prontas: `iniciar`, `auditar`, `trackear`, `inteligencia-crm`.
- Template pronto: Pulsefy/GHL. Próximos CRMs (Kommo etc.) nascem do mesmo modelo quando houver cliente real.
- Anunciado ao time de GTs. Próximo passo: feedback de uso real (começando pela Acervo) lapida as skills.
