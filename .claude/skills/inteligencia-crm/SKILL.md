---
name: inteligencia-crm
description: Monta a camada de inteligência (sinais do CRM → Meta via API de Conversões, e depois Google) usando n8n, sem app. Use quando o usuário quiser "ensinar a Meta quem compra", enviar qualificado/venda do CRM pra Meta, otimizar campanha por cliente e não por clique, ou configurar CAPI a partir do CRM. Funciona com qualquer CRM que dispare webhook; traz template pronto por CRM (Pulsefy/GHL primeiro).
---

# Camada de inteligência — CRM → Meta (via n8n)

Você está montando, para um cliente, o fluxo que ensina a Meta quem realmente compra. Sem app: um workflow no n8n que a pessoa duplica por cliente. O usuário pode não ser técnico — **guie passo a passo, uma etapa por vez, e ofereça FAZER por ele quando houver acesso** (n8n via API, teste de evento via curl).

## A ideia, em uma frase (diga assim se a pessoa perguntar)
"Quando um lead qualifica ou compra no CRM, a gente avisa a Meta com o telefone/e-mail dele (em código, nunca em claro). A Meta reconhece a pessoa, credita a campanha que trouxe ela, e passa a caçar mais gente parecida com quem comprou — em vez de com quem só clicou."

## Etapa 1 — Qual CRM?
Pergunte qual CRM o cliente usa. Hoje o template pronto é:
- **Pulsefy / GoHighLevel (GHL)** → use `templates/pulsefy-ghl/` (Pulsefy é GHL white-label; mesmos webhooks e merge fields).

Outros CRMs (Kommo, Piperun, Hubspot, RD, etc.): o desenho é o mesmo (webhook do CRM → n8n → Meta). Se ainda não houver template daquele CRM na pasta `templates/`, avise que ele será criado a partir do modelo Pulsefy, adaptando os campos que o CRM manda — e siga só se a pessoa quiser abrir esse caminho novo.

## Etapa 2 — Leia o guia e o template do CRM
Abra e siga `templates/pulsefy-ghl/guia.md` (ou o do CRM escolhido). Ele tem as 5 partes: Parte 0 (configurar a API de Conversões no Meta), A (workflows no CRM), B (n8n), C (testar), D (duplicar). Conduza a pessoa por elas **em ordem, uma por vez**, confirmando cada uma antes de seguir. O arquivo do workflow é `templates/pulsefy-ghl/inteligencia-meta.json`.

## Etapa 3 — Guiar OU fazer (a diferença)

**Sempre guie.** Além disso, quando a pessoa TIVER o acesso e PEDIR, você pode fazer:

- **Criar o workflow no n8n via API:** se a pessoa te passar a URL do n8n e uma API key (ou disser que há credencial configurada), você pode `POST /api/v1/workflows` com o JSON do template já preenchido (PIXEL_ID/TOKEN nos lugares certos). Confirme os valores com ela antes. Nunca ative o workflow sem o teste da Parte C ter passado.
- **Disparar um evento de TESTE no Meta:** se a pessoa te der pixel ID + token + test_event_code, você pode montar e enviar um evento de teste via curl (telefone fictício, hasheado) pra ela ver aparecer na aba "Testar eventos". É a forma mais rápida de provar que o token está certo antes de mexer no CRM.

Antes de qualquer ação que escreva em algum sistema (n8n, Meta), **confirme com a pessoa** o que você vai fazer.

## Etapa 4 — O gate (não pule)
Só considere "pronto pra produção" depois que um evento de TESTE apareceu na aba "Testar eventos" do Meta com o telefone **hasheado** (código, não o número). Aí sim: apagar o test_event_code e ativar. Sem esse teste, não ligue em cliente real.

## Regras de segurança (repita pra pessoa se ela for mexer sozinha)
- O workflow **só lê e envia pra fora** — nenhum nó escreve/edita/apaga no CRM.
- Telefone e e-mail **sempre hasheados** (SHA-256) antes de sair.
- O token do pixel é **secreto** — vive só no nó Configuração do n8n.
- Com `test_event_code` preenchido, os eventos **não contam** (viram teste). Apague ao ir pra produção.

## Honestidade (não venda mágica)
- **Sinais negativos** (curioso, sem perfil) NÃO fazem a Meta "evitar" gente parecida sozinha. O que melhora a Meta: otimizar pela venda/qualificado, lookalike de comprador, público de exclusão dos ruins. Os negativos servem pra exclusão e relatório.
- **Sem painel próprio:** a visibilidade fica no Gerenciador de Eventos do Meta e no próprio CRM.
- **Google é a fase 2** (precisa de gclid no formulário + planilha) — só depois do Meta rodar.
