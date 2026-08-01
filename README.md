# Kit de Tracking V4

Ferramenta do time de tráfego da V4 para **auditar tracking, montar tracking e ligar a camada de inteligência** (fazer a Meta otimizar por quem compra, não por quem clica) — tudo guiado, sem precisar ser técnico.

Não é um app. É uma pasta que você abre no **Claude Code** (ou Codex) e que te conduz por perguntas.

---

## Como usar (passo a passo)

1. **Baixe o kit:** clone este repositório na sua máquina.
2. **Abra a pasta no Claude Code** (ou Codex).
3. **Digite `/iniciar`** e aperte enter.
4. Responda as perguntas. O kit entende o cenário do cliente e te leva pra etapa certa — te guiando ou fazendo por você quando der.

Pronto. Você não precisa saber de código, nem de n8n, nem de API. O kit pergunta o que precisa e explica cada passo.

---

## O que o kit faz

| Frente | O que é | Estado |
|---|---|---|
| **Camada de inteligência** | Envia qualificado/venda do CRM pra Meta (API de Conversões). A Meta passa a otimizar por comprador. | ✅ pronto (Pulsefy/GHL) |
| **Auditar** | Ver o que está certo/errado no tracking do cliente | 🔜 em construção |
| **Trackear** | Montar ou consertar as tags (GA4, pixel, Clarity) | 🔜 em construção |

Começamos pela **camada de inteligência** de propósito — é a que mais muda resultado. As outras entram conforme o time usar e pedir.

---

## Estrutura da pasta

```
kit-tracking-v4/
├── README.md                     ← você está aqui
├── .claude/skills/
│   ├── iniciar/                  ← a porta de entrada (/iniciar)
│   └── inteligencia-crm/         ← monta a camada de inteligência
└── templates/
    └── pulsefy-ghl/              ← workflow n8n + guia pro Pulsefy/GHL
```

Cada CRM novo vira uma pasta em `templates/`. O desenho é o mesmo; muda só o formato que o CRM manda os dados.

---

## Regras de segurança (o kit sempre respeita)

- **Nunca altera o CRM do cliente.** Só lê e envia pra fora. (Protege contra acidente com dado de cliente.)
- **Telefone e e-mail sempre viram código** antes de sair pra Meta. Nunca em texto puro.
- **Testa antes de ligar.** Nada vai pra produção sem passar pelo teste de evento do Meta.
