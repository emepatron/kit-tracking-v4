# Snippets das tags (para colar no GTM) + onde achar cada ID

Os códigos abaixo são os oficiais de cada plataforma. Troque **só o ID** (a parte em MAIÚSCULA). O resto é fixo.

---

## GA4 — não precisa de snippet

No GTM, GA4 é uma **tag nativa**: Nova tag → tipo **"Google tag"** (Google Tag) → cole o ID → gatilho **All Pages**. Sem código.

- **Onde achar o ID (`G-XXXXXXX`):** Google Analytics → Admin → Fluxos de dados → clique no fluxo do site → "ID de métricas" no canto.

---

## Pixel da Meta — base (PageView)

Tipo **HTML personalizado**, gatilho **All Pages**:

```html
<script>
!function(f,b,e,v,n,t,s)
{if(f.fbq)return;n=f.fbq=function(){n.callMethod?
n.callMethod.apply(n,arguments):n.queue.push(arguments)};
if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
n.queue=[];t=b.createElement(e);t.async=!0;
t.src=v;s=b.getElementsByTagName(e)[0];
s.parentNode.insertBefore(t,s)}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', 'SEU_PIXEL_ID');
fbq('track', 'PageView');
</script>
```

- **Onde achar o ID do pixel:** Gerenciador de Eventos da Meta → selecione o pixel → o ID (números) aparece no topo.

---

## Pixel da Meta — evento (Lead)

Tipo **HTML personalizado**, gatilho no **momento do lead** (envio do formulário / página de obrigado). A base (acima) precisa existir primeiro.

```html
<script>
fbq('track', 'Lead');
</script>
```

Para compra, troque por `fbq('track', 'Purchase', {value: 0.00, currency: 'BRL'});`.

---

## Microsoft Clarity

Tipo **HTML personalizado**, gatilho **All Pages**:

```html
<script type="text/javascript">
(function(c,l,a,r,i,t,y){
c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
})(window, document, "clarity", "script", "SEU_PROJECT_ID");
</script>
```

- **Onde achar o ID do projeto:** clarity.microsoft.com → seu projeto → Configurações → Overview (é um código curto, tipo `a1b2c3d4e5`).

---

## Conversion Linker (Vinculador de conversões) — não precisa de snippet

Tag **nativa**: Nova tag → tipo **"Vinculador de conversões"** → gatilho **All Pages**. Sem configuração. É o que segura a atribuição do clique do Google nos navegadores modernos.

---

## Google Ads — conversão

Depende da conta. No GTM: Nova tag → tipo **"Acompanhamento de conversões do Google Ads"** (nativo) → cole o **ID de conversão** (`AW-XXXXXXX`) e o rótulo → gatilho no momento da conversão.

- **Onde achar:** Google Ads → Objetivos → Conversões → a ação de conversão → detalhes da tag.
