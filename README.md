# Manual de Funcionamento — Agência Verbo

Site estático (HTML único) com o manual interno da agência: organização das pastas dos
clientes no Google Drive, padrões de nomenclatura de arquivos e funcionamento das
automações do ClickUp.

Identidade visual baseada no projeto **verbo-ai-chat** e no **cronograma-estratégico-2026**
(fundo escuro, verde neon `#CCFF00`, fontes RNS Camelia + Montserrat).

## Estrutura

```
manual-site/
├── index.html              # site completo (CSS + JS inline)
├── fonts/RNSCamelia-*.woff2 # fontes da marca
├── logo_horizontal.png, bg.png, digital_y.png
└── README.md
```

## Ver localmente

Basta abrir o `index.html` no navegador (duplo clique) ou:

```powershell
start index.html
```

## Publicar no GitHub Pages

Pré-requisito: ter o [GitHub CLI (`gh`)](https://cli.github.com/) instalado e logado
(`gh auth login`).

```powershell
# dentro da pasta manual-site/
git init
git add .
git commit -m "Manual de funcionamento da Agência Verbo"

# cria o repositório e dá push (ajuste o nome/conta se quiser)
gh repo create verbo-manual --public --source=. --push

# ativa o GitHub Pages na branch main, pasta raiz
gh api -X POST repos/{owner}/verbo-manual/pages -f "source[branch]=main" -f "source[path]=/"
```

Depois de ~1 min o site fica disponível em:

```
https://<sua-conta>.github.io/verbo-manual/
```

Esse é o link para compartilhar com a equipe.

> Alternativa sem `gh`: criar o repo manualmente em github.com, dar push, e ativar o Pages
> em **Settings › Pages › Branch: main / (root)**.

## Agente de IA de suporte (n8n)

A página tem um botão flutuante **"Tirar dúvida"** que abre um chat. Ele conversa com um
agente de IA hospedado no n8n da Verbo, que conhece todo o conteúdo do manual.

### Como ativar

1. No n8n (`https://automacoes-verbo-n8n.xpddsl.easypanel.host`), vá em **Workflows › Import from File**
   e importe o arquivo **`n8n-manual-suporte.json`** desta pasta.
2. No nó **OpenAI Chat Model**, selecione a **credencial OpenAI** da Verbo (a mesma usada nos
   outros agentes). Modelo padrão: `gpt-4o-mini` (rápido e barato — pode trocar se quiser).
3. **Salve** e **ative** (toggle "Active") o workflow.
4. O webhook fica em: `https://automacoes-verbo-n8n.xpddsl.easypanel.host/webhook/manual-suporte`
   — é exatamente a URL já configurada no `index.html` (constante `SUPPORT_WEBHOOK`). Se mudar o
   path do webhook, atualize essa constante no `index.html`.

### Como funciona

- O site envia `POST { sessionId, chatInput }` para o webhook.
- O **Agente de Suporte** responde usando o manual inteiro embutido no *system prompt* (sem precisar
  de banco vetorial). A **Memória da Sessão** mantém o contexto da conversa por `sessionId`.
- A resposta volta como `{ "output": "..." }` e aparece no chat.

> Para atualizar o que o agente sabe, edite o texto do campo **systemMessage** no nó
> *Agente de Suporte* (no JSON ou direto no n8n) sempre que o conteúdo do manual mudar.

## Atualizar o conteúdo

Edite o `index.html` (todo o conteúdo está em seções `<section>` com ids: `drive`, `pastas`,
`instagram`, `carrossel`, `clickup`, `automacoes`). Depois:

```powershell
git add .
git commit -m "Atualiza manual"
git push
```

O GitHub Pages republica automaticamente.
