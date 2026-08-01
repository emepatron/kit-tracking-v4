---
name: auditar
description: Audita o tracking de um cliente — o que está certo e o que está quebrado. Escaneia a página pública (pixel, GA4, GTM, Clarity, WhatsApp) e, se houver acesso ao Gerenciador de Tags, audita por dentro (pixel duplicado, GA4 faltando, gatilho duplo, Conversion Linker, tag base do pixel). Use quando o usuário quiser conferir, revisar ou diagnosticar o tracking de um site ou landing page.
---

# Auditar o tracking de um cliente

Você audita o tracking de um cliente e entrega um diagnóstico honesto: o que está **em ordem**, o que é **achado** (problema), e o que **não deu pra conferir**. Público pode ser leigo — explique cada achado em uma frase de negócio. **Precisão acima de cobertura: só afirme o que tem evidência; o que não deu pra ler, diga que não deu — nunca invente falha.**

## Etapa 1 — Raio-X da página (sempre dá pra fazer, sem acesso a nada)

Peça a URL da página (LP ou site). Faça o **fetch da página pública** e leia o HTML procurando:

- **GTM** — `GTM-XXXXXXX` (script do googletagmanager)
- **GA4** — `G-XXXXXXX` (gtag/google-analytics)
- **Pixel da Meta** — `fbq(` / `connect.facebook.net` / o ID numérico do pixel; conte QUANTOS pixels distintos
- **Google Ads** — `AW-XXXXXXX` / gtag de conversão
- **Microsoft Clarity** — `clarity.ms` / `clarity(`
- **Links de WhatsApp** — `wa.me`, `api.whatsapp.com`, `whatsapp://` — liste TODOS (botão do topo, flutuante, rodapé), cada um: tem identificação de origem ou não?

Compare com o **ideal pelo tipo de funil**:
- **LP de captação / site / e-commerce:** deve ter GA4 e Clarity (comportamento do público é essencial — sem Clarity, decisão de CRO no escuro). E-commerce cobra também evento de compra; LP cobra tag de conversão.
- **Campanha direto pro WhatsApp:** foco no link/atribuição, não cobra tag de compra.

**Invariante da honestidade:** se a página TEM GTM mas você não achou (ex.) o Clarity no HTML, NÃO diga "não tem Clarity" — diga "não foi possível confirmar pelo HTML; pode estar dentro do GTM. Confirme por dentro (Etapa 2)". A ausência no HTML com GTM presente não é prova de ausência.

## Etapa 2 — Auditoria por dentro do Gerenciador de Tags (se houver acesso)

Se a pessoa puder **colar a exportação do container** (GTM → Admin → Exportar container → JSON) ou passar a lista de tags, audite com estas regras. Cada achado = fato + evidência + o que fazer.

- **GA4 ausente** (`crítico`): container sem nenhuma tag GA4 → a página está invisível no Analytics. O que fazer: criar a tag GA4 (skill `trackear`).
- **Pixel duplicado** (`atenção`): duas tags com o MESMO ID de pixel que disparam **no mesmo gatilho** → evento dobrado. Se disparam em **gatilhos diferentes** (ex.: uma em All Pages, outra no envio do form), é o desenho legítimo — **não é problema**, calе. Sempre olhe os gatilhos antes de acusar.
- **Gatilho duplo em tag de conversão** (`atenção`): uma tag de conversão (Lead/Purchase) com DOIS gatilhos que podem ser o mesmo momento (ex.: "Custom Event - form submit" + "Submit Nativo do formulário") → risco de contar a conversão duas vezes. O que fazer: no Preview do GTM, envie o form UMA vez e conte os disparos; se disparar 2×, remova um gatilho.
- **Conversion Linker ausente** (`atenção`): há tag de conversão do Google mas NÃO há a tag "Vinculador de conversões" (Conversion Linker) em All Pages → navegadores com proteção derrubam a atribuição do clique. O que fazer: criar o Conversion Linker em All Pages.
- **Pixel sem tag base** (`atenção`): há tag de EVENTO do pixel (Lead/Purchase) mas nenhuma tag base (PageView/init em All Pages) → o evento dispara no vazio. O que fazer: criar a base do pixel em All Pages.
- **Clarity ausente no container** (`atenção`, vira `crítico` em LP/ecom/site): nenhuma tag do Clarity. Ressalva: se o Clarity estiver direto no código da página (fora do GTM), o raio-X da Etapa 1 confirma — nesse caso ignore.

## Etapa 3 — Auditoria da mídia (Meta), se a pessoa tiver acesso aos anúncios

Se houver acesso à conta de anúncios (o operador consegue ver os anúncios ativos):
- **Anúncio ativo sem UTM** (`crítico`): campanha rodando sem parâmetros de URL → verba anônima, nada de origem no analytics. O que fazer: adicionar UTMs no anúncio.
- **Pixel sem disparo recente** (`atenção`): pixel configurado mas sem disparar há dias → a superfície pode estar sem a tag ou sem tráfego.

(A leitura automática de anúncios exige token de sistema da BM — se não houver, oriente a conferir na mão no Gerenciador de Anúncios.)

## Entrega — o laudo

Organize o resultado em três blocos, sempre:
1. **✅ Em ordem** — o que rodou e passou (dá segurança e vira prova de trabalho pro cliente).
2. **⚠️ Achados** — os problemas, cada um com evidência e o que fazer.
3. **⬜ Não deu pra conferir** — o que faltou acesso, dito com honestidade.

Se houver achado que a skill `trackear` resolve (GA4/Clarity/pixel/linker ausentes), ofereça seguir pra ela.
