# Camada de Inteligência — Pulsefy → Meta (template n8n)

Guia para o time. Sem app, sem servidor: **um workflow no n8n que você duplica por cliente.**

---

## O que isso faz (em um parágrafo)

Quando um lead avança no funil do Pulsefy — qualificou, comprou, ou foi marcado como sem perfil — um workflow do Pulsefy avisa o n8n. O n8n pega o telefone e o e-mail, **hasheia** (transforma em código irreversível), e manda o evento pro Pixel da Meta pela API de Conversões. A Meta reconhece a pessoa, credita a campanha que trouxe ela, e passa a otimizar por **quem qualifica e compra** — em vez de por quem só clica.

---

## O desenho

```
PULSEFY                         N8N                              META
card muda de etapa  ──webhook──►  Recebe → Configuração          Pixel reconhece
(ex: virou Ganho)                 → Monta evento (hash)  ──CAPI──►  a pessoa e
                                  → Envia pra Meta                otimiza melhor
```

Três coisas para preencher por cliente (só isso): **PIXEL_ID**, **TOKEN do pixel**, e a **URL do webhook** nos workflows do Pulsefy.

---

## O que precisa CAIR no contato do Pulsefy (pré-requisito)

Para a Meta casar a pessoa, o contato precisa ter, no mínimo, o **telefone** — e ele sempre vem (é a chave da conversa do WhatsApp). O resto melhora o resultado:

| Campo | Para quê | Vem de onde |
|---|---|---|
| Telefone | A chave: a Meta casa a pessoa por ele | Sempre (conversa do WhatsApp) |
| E-mail | Melhora o casamento | Se o formulário capturar |
| Valor da venda | Vai no evento de compra | Preenchido quando o card vira Ganho |
| gclid | Só para o Google (fase 2) | Campo oculto do formulário |

---

## PARTE 0 — Preparar a API de Conversões no Meta (uma vez por cliente)

É aqui que o setup começa: sem o token da API de Conversões, o n8n não tem como falar com a Meta. Tudo acontece no **Gerenciador de Eventos** (`business.facebook.com/events_manager`), dentro da conta de negócio (BM) onde vive o pixel do cliente.

**Passo 1 — Ache (ou crie) o pixel do cliente.**
- Gerenciador de Eventos → **Fontes de dados**. Se o pixel do cliente já aparece na lista, selecione ele.
- Se não existe: **Conectar fontes de dados** → **Web** → **Pixel da Meta** → dê um nome → Criar. (A tag do pixel no site é outra coisa — para a inteligência via CRM, o que importa é o pixel existir como fonte de dados.)

**Passo 2 — Anote o ID do pixel.**
Com o pixel selecionado, o **ID** (uma sequência de números) aparece no topo, ao lado do nome. É o **PIXEL_ID** que vai no n8n.

**Passo 3 — Gere o token da API de Conversões.**
- Com o pixel aberto → aba **Configurações**.
- Role até a seção **API de Conversões** → **Configurar manualmente** (ou "Gerar token de acesso", dependendo da versão da tela).
- Clique em **Gerar token de acesso**. A Meta mostra o token **uma vez** — copie na hora. É o **TOKEN** que vai no n8n.
- Esse token é secreto e não expira sozinho. Se vazar, volte aqui e gere outro (o antigo morre).

**Passo 4 — Deixe a aba "Testar eventos" à mão.**
- Ainda no pixel → aba **Testar eventos**. É nela que você vai ver os eventos chegando na hora do teste (Parte C), e é dela que sai o **código de teste** (`TESTxxxx`).

> **Precisa ler anúncios do cliente também?** Isso é outro acesso (system user + token da BM) e **não** é necessário para a inteligência funcionar — o token do pixel acima já basta para enviar qualificado/venda. Leitura de anúncios só entra se um dia você quiser auditar UTM, que não é o escopo deste workflow.

---

## PARTE A — Configurar no Pulsefy (GHL)

Você vai criar **um workflow (automação) para cada sinal**. Cada um dispara quando o card muda e manda um webhook pro n8n com um campo `evento` fixo.

**Passo 1.** Pulsefy → Automação → criar workflow novo.

**Passo 2.** Gatilho: **Opportunity Stage Changed** (mudança de etapa) e filtre pela etapa que representa o sinal — por exemplo, a etapa **Qualificado**.

**Passo 3.** Ação: **Webhook** (método POST) → cole a URL do n8n (você pega na Parte B) → corpo (Custom Data / JSON):

```json
{
  "evento": "qualificado",
  "phone": "{{contact.phone}}",
  "email": "{{contact.email}}",
  "contactId": "{{contact.id}}"
}
```

**Passo 4.** Repita para cada sinal, mudando só o gatilho e o campo `evento`:

| Sinal | Gatilho no Pulsefy | `evento` | Campos extras |
|---|---|---|---|
| Qualificou | Etapa → Qualificado | `qualificado` | — |
| **Comprou** | Etapa → Ganho / Won | `comprou` | `"valor": "{{opportunity.monetary_value}}"`, `"dealId": "{{opportunity.id}}"` |
| Recompra | Etapa → Ganho (recorrente) | `recorrente` | mesmos de comprou |
| Sem perfil | Tag "sem perfil" adicionada | `sem_perfil` | — |
| Curioso | Tag "curioso" adicionada | `curioso` | — |
| Concorrente | Tag "concorrente" adicionada | `concorrente` | — |

> O `evento` é o que o n8n lê para decidir o que mandar pra Meta. Cada workflow do Pulsefy tem um `evento` fixo — é isso que mantém tudo simples.

---

## PARTE B — Configurar no n8n

**Passo 1.** No n8n (`n8n.pulsefy.ia.br`): menu → **Import from File** → escolha `template-inteligencia-meta.json`.

**Passo 2.** Abra o nó **"Configuração do cliente"** e cole os três valores da **Parte 0**:

- **PIXEL_ID** — o ID do pixel (Parte 0, passo 2).
- **TOKEN** — o token da API de Conversões (Parte 0, passo 3).
- **TEST_EVENT_CODE** — o código de teste (Parte 0, passo 4). Deixe enquanto valida; **apague depois**, senão os eventos reais entram como teste e não contam.

**Passo 3.** Clique no nó **"Recebe do Pulsefy"** → copie a **URL de produção** do webhook → é ela que vai nos workflows do Pulsefy (Parte A, passo 3).

**Passo 4.** Salve. **Não ative ainda** — teste primeiro.

---

## PARTE C — Testar ANTES de ligar de verdade

1. Gerenciador de Eventos da Meta → aba **Testar eventos** → copie o **código de teste** → cole no nó Configuração (campo TEST_EVENT_CODE).
2. Ative o workflow no n8n.
3. No Pulsefy, mova **um card de teste** para a etapa Qualificado (ou marque uma tag).
4. Em segundos, o evento aparece em **Testar eventos** na Meta.
5. Confira: o nome do evento está certo? O telefone aparece **hasheado** (código, nunca o número em claro)?
6. Deu certo? **Apague o TEST_EVENT_CODE** do nó Configuração. A partir daí os eventos contam de verdade.

---

## PARTE D — Duplicar para um cliente novo (ex.: Acervo)

1. No n8n → **Duplicate** o workflow → renomeie ("Inteligência — Acervo").
2. Abra o nó **Configuração do cliente** → troque PIXEL_ID e TOKEN pelos da Acervo.
3. No Pulsefy da Acervo → crie os workflows da Parte A apontando para a **nova URL** do webhook (cada workflow duplicado tem a sua).
4. Teste (Parte C) → ligue.

Cada cliente = uma cópia do workflow + a tabelinha de automações no Pulsefy dele. Nada de código novo.

---

## Regras de segurança (não negociáveis)

- **Só leitura + envio.** Nenhum nó deste workflow escreve, edita ou apaga qualquer coisa no Pulsefy. Ele lê o que o webhook mandou e envia pra fora. (Isso protege contra o tipo de acidente que já apagou dados de cliente no passado.)
- **Telefone e e-mail sempre hasheados** (SHA-256) antes de sair. Nunca viajam em claro.
- **O token do pixel é secreto.** Vive só no nó Configuração do n8n de vocês.

---

## O que esse template NÃO faz (para ninguém se frustrar)

- **Não tem painel próprio.** A visibilidade fica no Gerenciador de Eventos da Meta (mostra os eventos chegando, em tempo real) e no próprio Pulsefy (o funil que o time já usa).
- **Os sinais negativos não fazem mágica.** Mandar um evento "Curioso" para a Meta **não** faz ela evitar gente parecida sozinha. O que melhora a Meta é: (1) **otimizar a campanha pelo evento de venda/qualificado**, (2) criar **lookalike de quem comprou**, (3) criar **público de exclusão** com quem você marcou como ruim. Os sinais negativos servem para a exclusão e para o seu relatório — são insumo, não botão mágico.
- **Google é a fase 2.** Precisa do gclid no formulário + uma planilha. É um segundo destino (mais um nó no workflow) que a gente monta depois que o Meta estiver rodando e provando valor.

---

## Como usar a Meta depois que os eventos chegam

- **Otimizar a campanha:** troque o evento de otimização para **Lead** (que agora significa qualificado). Pode fazer desde o dia 1 — abaixo de ~50 eventos/semana roda em aprendizado limitado, e mesmo assim costuma ser melhor que otimizar por curioso.
- **Lookalike:** ao juntar **100 pessoas** (acumula até 180 dias, não é por semana) num público de compradores ou qualificados, crie um lookalike 1–3%. É a semente boa.
- **Exclusão:** público personalizado com quem virou "sem perfil / curioso" → exclua das campanhas de captação.
