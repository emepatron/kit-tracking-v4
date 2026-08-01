---
name: trackear
description: Monta ou conserta o tracking de um cliente — cria as tags certas no Gerenciador de Tags (GA4, Pixel da Meta, Clarity, Conversion Linker) pelo tipo de funil, do zero ou corrigindo o que falta. Use quando o usuário quiser instalar tracking, colocar pixel/GA4/Clarity, montar o GTM de um cliente novo, ou aplicar as correções que uma auditoria apontou.
---

# Trackear — montar ou consertar o tracking

Você ajuda a colocar o tracking no lugar. Público pode ser leigo — conduza passo a passo, uma coisa por vez, e **nunca invente um ID**: sem o identificador da ferramenta (o `G-...` do GA4, o número do pixel, o ID do Clarity), a tag não é criada. Os snippets prontos estão em `templates/gtm/snippets.md`.

## Etapa 1 — Entender o cenário

Pergunte:
1. **É do zero (não tem GTM) ou é consertar/completar** (já tem GTM)?
2. **Que tipo de funil?** LP de captação · e-commerce · site institucional · campanha direto pro WhatsApp. (Define o que é ideal ter.)

## Etapa 2 — Se for do zero: o container primeiro

O Gerenciador de Tags (GTM) é a "central" onde todas as tags moram. Se o cliente não tem:
1. Oriente criar o container em `tagmanager.google.com` → Criar conta → dar o nome do site → tipo **Web**.
2. O GTM gera **dois trechos de código** (um pro `<head>`, um pro `<body>`). Alguém com acesso ao site precisa colar os dois no site (WordPress, tema, etc.). **Este passo é do site do cliente — o kit não mexe no código do site de ninguém.**
3. Feito isso, siga pra Etapa 3.

## Etapa 3 — As tags certas por tipo de funil

Monte a lista do que aquele funil precisa (o "ideal"):

| Tag | LP captação | E-commerce | Site | WhatsApp direto |
|---|---|---|---|---|
| **GA4** (base) | ✅ | ✅ | ✅ | opcional |
| **Clarity** (comportamento) | ✅ | ✅ | ✅ | — |
| **Pixel Meta** (base PageView) | ✅ | ✅ | se anuncia | ✅ |
| **Pixel Meta** (evento: Lead/Purchase) | ✅ Lead | ✅ Purchase | — | Lead |
| **Conversion Linker** (se usa Google Ads) | ✅ | ✅ | se anuncia | ✅ |
| **Google Ads** (conversão) | se usa | se usa | — | se usa |

Para cada tag que falta, pegue o **ID com a pessoa** (ela busca na ferramenta — veja onde em `snippets.md`) e monte a tag.

## Etapa 4 — Criar as tags (guiar ou fazer)

**Guiar (sempre funciona):** com o ID em mãos, oriente criar a tag no GTM:
- **GA4:** GTM → Nova tag → tipo **"Google tag"** (nativa) → cole o `G-XXXXXXX` → gatilho **All Pages**.
- **Pixel base:** tipo **HTML personalizado** → cole o snippet de base do `snippets.md` com o ID → gatilho **All Pages**.
- **Pixel evento (Lead/Purchase):** HTML personalizado → snippet de evento → gatilho do momento certo (envio do form, página de obrigado, etc.).
- **Clarity:** HTML personalizado → snippet do Clarity com o ID do projeto → gatilho **All Pages**.
- **Conversion Linker:** tipo **"Vinculador de conversões"** (nativo) → gatilho **All Pages**.

**Fazer (se a pessoa tiver acesso via API):** se ela conectar o GTM (conta de serviço do Google com permissão de Edição no container) e pedir, você pode criar as tags **em um workspace de RASCUNHO** via API do Tag Manager. Regras inegociáveis:
- **Nunca publique.** Só crie o rascunho. Quem publica é a pessoa, depois de conferir no **Preview**.
- **Nunca apague ou edite** tag existente — só adicione.
- Antes de criar, confira se a tag já existe no container (não duplique).

## Etapa 5 — Sempre: Preview antes de publicar

Oriente: GTM → **Visualizar (Preview)** → abrir o site → confirmar que as tags disparam certo → só então **Publicar**. Nada vai ao ar sem esse conferido humano.

## Depois

Tracking no lugar? Se o objetivo é a mídia otimizar por quem compra, ofereça a skill **`inteligencia-crm`** (a camada que ensina a Meta quem é cliente de verdade).
