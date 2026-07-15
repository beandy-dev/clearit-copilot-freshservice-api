# ClearIT Copilot — Widget FreshService

Custom App instalável na sidebar de tickets do FreshService. Conecta o analista L1 ao backend de IA do ClearIT Copilot diretamente no contexto do chamado, sem trocar de ferramenta.

**Squad Sherlock (B5) — Pulse Mais 2026**

---

## O que é

Widget que renderiza na **sidebar direita** de qualquer ticket no FreshService. Ao abrir um chamado, o Copilot aparece automaticamente com a descrição pré-preenchida. O analista clica "Buscar Diagnóstico" e recebe sugestão de resolução com fonte citada.

---

## Funcionalidades

- **Pré-preenchimento automático** — Captura subject e description do ticket aberto via FreshService SDK
- **Diagnóstico com IA** — Chama o backend (busca semântica + Gemini) e exibe resultado na sidebar
- **Busca na web** — Botão para pesquisar documentação externa quando confiança é baixa
- **Análise de compatibilidade** — Compara diagnóstico interno com fontes externas
- **Chat follow-up** — Perguntas adicionais sem sair do ticket
- **Feedback estruturado** — Útil / Parcial (com edição) / Não ajudou → enviado ao backend
- **Retry automático** — Lida com cold start do Render (2 tentativas)
- **Configurável** — URL do backend via parâmetros de instalação (iparams)

---

## Stack

| Componente | Tecnologia |
|------------|-----------|
| Plataforma | Freshworks Platform 3.0 |
| FDK | 10.x (Freshworks Developer Kit) |
| Node (build) | 24.11.x |
| Frontend | HTML + CSS + Vanilla JS |
| SDK | FreshService App Client (`fresh_client.js`) |
| Backend | Externo (Render) — configurável via iparams |

---

## Pré-requisitos para Desenvolvimento

```bash
# Instalar Node 24.11 (via nvm)
nvm install 24.11
nvm use 24.11

# Instalar FDK 10
npm install https://cdn.freshdev.io/fdk/latest-v24.tgz -g
```

> O FDK 10 requer **exatamente** Node 24.11.x. Versões superiores (24.15+) não são aceitas.

---

## Comandos

```bash
# Validar estrutura e manifesto
fdk -u validate --app-dir .

# Empacotar para upload (gera dist/*.zip)
fdk -u pack --app-dir . --skip-coverage

# Rodar localmente para teste
fdk -u run --app-dir .
```

---

## Deploy no FreshService

1. Execute `fdk pack --skip-coverage`
2. Acesse **Admin → Apps → Custom Apps** no FreshService
3. Clique **New App → Custom App**
4. Upload o `.zip` gerado em `dist/`
5. Preencha o nome do app e salve
6. Na instalação, configure a **URL do Backend**
7. Instale e teste num ticket

---

## Configuração (Parâmetros de Instalação)

| Parâmetro | Tipo | Obrigatório | Default | Descrição |
|-----------|------|-------------|---------|-----------|
| `backend_url` | text | Sim | `https://clearit-copilot.onrender.com` | URL do backend de IA |

O administrador configura durante a instalação do app. Pode apontar para qualquer instância do backend (produção, staging, localhost via ngrok).

> **Nota:** O backend em `clearit-copilot.onrender.com` está temporariamente disponível para testes e avaliação da banca do programa Pulse Mais 2026. Enquanto estiver ativo, o widget funciona sem configuração adicional. Quando for desativado, será necessário realizar o deploy de uma instância própria do backend (ver [clearit-copilot-backend](https://github.com/beandy-dev/clearit-copilot-backend)) e atualizar a `backend_url` com a nova URL.

---

## Estrutura

```
freshservice-app/
├── manifest.json          # Platform 3.0, modules.service_ticket, engines
├── config/
│   └── iparams.json       # Parâmetros de instalação (backend_url)
├── app/
│   ├── index.html         # Widget completo (diagnóstico + chat + feedback)
│   └── icon.svg           # Ícone do app na sidebar
└── README.md
```

---

## Manifesto (manifest.json)

```json
{
  "platform-version": "3.0",
  "modules": {
    "service_ticket": {
      "location": {
        "ticket_sidebar": {
          "url": "index.html",
          "icon": "icon.svg"
        }
      }
    }
  },
  "engines": {
    "node": "24.11.1",
    "fdk": "10.1.7"
  }
}
```

**Importante:** Platform 3.0 não usa campo `product`. A associação ao FreshService é feita via `modules.service_ticket`.

---

## Posições Disponíveis

O widget pode ser instalado em diferentes locais do FreshService:

| Location | Onde aparece |
|----------|-------------|
| `ticket_sidebar` | Sidebar direita do ticket (atual) |
| `full_page_app` | Página inteira dedicada |
| `cti_global_sidebar` | Sidebar global (qualquer tela) |
| `change_sidebar` | Sidebar de Change Requests |
| `asset_sidebar` | Sidebar de Assets |

Para múltiplas posições, adicione no `modules.service_ticket.location`.

---

## Troubleshooting

| Problema | Causa | Solução |
|----------|-------|---------|
| "Platform version below 3.0" | Manifesto antigo | Usar platform-version "3.0" sem campo "product" |
| "Node.js runtime 18 and above" | engines.node formato errado | Usar versão completa: "24.11.1" |
| "Node.js 24 required" | FDK antigo | Instalar FDK 10.x com Node 24.11 |
| Zip rejeitado | Empacotamento manual | Sempre usar `fdk pack` (adiciona .report.json e digest.md5) |
| Feedback não envia | Render dormindo (cold start) | Retry automático implementado (2 tentativas) |
| Mixed Content | Backend em HTTP | Backend deve ser HTTPS (Render já é) |

---

## Fluxo do Usuário

```
Analista abre ticket no FreshService
        ↓
Widget aparece na sidebar (descrição pré-preenchida)
        ↓
Clica "Buscar Diagnóstico"
        ↓
Backend processa (LGPD → Embedding → Busca → Gemini)
        ↓
Exibe: Diagnóstico + Ações + Fonte + Confiança
        ↓
Analista pode: Regerar | Buscar na web | Chat | Feedback
```

---

## Repositórios Relacionados

| Repo | Descrição |
|------|-----------|
| [sherlock-clearit-copilot](https://github.com/beandy-dev/sherlock-clearit-copilot) | POC completa (frontend + backend + docs) |
| [clearit-copilot-backend](https://github.com/beandy-dev/clearit-copilot-backend) | Backend em produção (Render) |

📖 **Manual do Sistema:** [https://clearit-copilot.onrender.com/manual.html](https://clearit-copilot.onrender.com/manual.html)

---

## Equipe

| Membro | Frente |
|--------|--------|
| Beatriz Andrade Lourenço | Tecnologia e Produto |
| [Davi da Paz Mota](https://github.com/davidapaz05) | Tecnologia e Produto |
| [Maria Eduarda Ferreira Santos](https://github.com/dudazfd) | Tecnologia e Produto |
| [Maria Eloisa Gomes da Conceição](https://github.com/mariaeloisa69) | Negócios e Estratégia |
| [Phelipe Alexandre de Almeida](https://github.com/phelipe134) | Tecnologia e Produto |

---

