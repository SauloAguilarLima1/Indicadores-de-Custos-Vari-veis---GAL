# Dashboard de Custeio — Grupo Aguilar Lima (GAL)

Contexto e convenções deste projeto para um chat futuro do Claude começar com tudo na mão.

---

## Visão geral

Single-page HTML dashboard de indicadores de custos variáveis dos 4 hotéis do Grupo Aguilar Lima, mais a visão consolidada "Grupo". Sai pelo GitHub Pages do próprio repo.

- **Arquivo único:** `index.html` (HTML + CSS + JS inline, ~1800 linhas)
- **Sem build step.** Edita o HTML, commita, GitHub Pages serve.
- **Charts:** Plotly 2.26.0 via CDN
- **Fonte de dados:** Google Sheets via `gviz/tq` (polling 30s; `SHEET_ID` hardcoded no JS)
- **Tema:** dark default + light toggle, persistido em `localStorage` (`custeio-mode`)

## Workflow obrigatório

- **Push automático.** Após cada alteração no código, o fluxo é: editar → `cp` para `/tmp/gal-repo` (clone temporário) → `git add` → `commit` → `push origin main`. **Não esperar o user pedir "atualize o github"** — é fricção identificada.
- Resumo de fim de turno SEMPRE inclui o SHA do commit.
- Se uma rodada tiver várias alterações relacionadas, agrupa num único commit final.

## Estética — Liquid Glass (Apple)

Referência visual explícita do user: **estilo liquid glass do iOS/macOS.** Aplicar em todo elemento novo:

- Glass panels: `background: var(--glass-bg-strong)` + `backdrop-filter: blur(28px) saturate(180%)`
- Border radius generoso (`--r-xl` = 28px em cards, `--r-md`/`--r-lg` em UI menor)
- Estados ativos usam `color-mix(in srgb, var(--tab-color) X%, transparent)` em camadas + box-shadow multi-layer pra criar **aura/halo**, com centro difuso (efeito "luz por trás de vidro fosco") e bordas mais saturadas.
- Cores acentuadas vêm via CSS variable inline `--tab-color` setada no elemento, **não** hardcoded.

## Layout

- **Header sticky** (`top: 16px`, full-width) — logo, toggle tema, hide-numbers, refresh, indicador de sync
- **Filter bar** abaixo do header — Ano (dropdown) + Período (month picker)
- **Sidebar vertical fixa** à esquerda (`position: fixed`, `left: 16px`, `top: 110px`, `width: 220px`) — lista de unidades com dots brilhantes
- **Container/filter-bar** recuados 256px à esquerda pra abrir espaço pra sidebar
- **Breakpoint 1024px:** abaixo disso, sidebar volta a ser pill horizontal no topo
- **Breakpoint 768px:** grids viram 1-2 colunas, paddings reduzidos

## Unidades (TABS)

Ordem fixa, definida pelo user:

| label | color | dataKey | tipo |
|---|---|---|---|
| **Grupo** | `#6da3d9` | — | `consolidated` |
| **Resende Imperial** | `#5fd6e4` | Resende Imperial | `hotel` |
| **Terra Boa** | `#8db272` | Terra Boa | `hotel` |
| **Pedra Torta** | `#7dc4e0` | Pedra Torta | `hotel` |
| **Vira Canoa** | `#f0b94c` | Vira Canoa | `hotel` |

> O label "Consolidado" foi renomeado pra "Grupo" no mockup do user. `id` interno continua sendo `consolidado` por compatibilidade.

## Features e decisões importantes

### Charts — sem zoom, sem grid
- `dragmode: false` (sem zoom-box por drag)
- `fixedrange: true` em ambos os eixos (sem zoom de jeito nenhum)
- `showgrid: false`, `zeroline: false` (sem fundo quadriculado nem linha do zero)
- `displayModeBar: false`, `scrollZoom: false`, `doubleClick: false` (nenhuma UI de zoom)
- Motivo: user clicava acidentalmente no zoom-box. Não voltar atrás sem pedido explícito.

### Bar chart "Custo / PAX por Hotel"
- `cliponaxis: false` + `margin.t: 56` no override. Sem isso o label "R$ XX,XX" em cima da barra mais alta é cortado.

### Hide numbers (botão olho com risco no header)
- Toggle aplica `body.numbers-hidden` que borra (`filter: blur(...)`) tudo que é numérico:
  - KPIs, custos, YoY badges
  - Labels de período/ano nos pickers
  - Ticks dos eixos, percentuais do pie, valores das barras
  - Tooltips de hover
- **Não** borra: legendas (nomes dos hotéis), labels de categoria
- Slash do ícone (linha diagonal de baixo-esquerda para cima-direita) some quando os números estão escondidos.

### Pickers
- **Ano:** dropdown vertical em ordem crescente (oldest -> newest), mantém badge "parcial"
- **Período:** month picker estilo NPS (intervalo de meses)
- Os dois fecham um ao abrir o outro e fecham com click fora

### Sidebar — item ativo (frosted glass)
- Background: `radial-gradient` com centro quase transparente (3%) e bordas suavemente tingidas (18%) — simula difusão do vidro fosco
- Border + inset shadows: anel de luz vazando pela borda interna
- Box-shadow multi-camada (12/28/56/90px) decrescente — aura externa
- Cor usa `--tab-color` da unidade (cada uma tem a sua)

## Memórias persistidas localmente (Claude do user)

Em chats novos, o Claude do user tem essas memórias locais:
- `github_user`: usuário é `SauloAguilarLima1`
- `project_dashboard_custeio_gal`: aponta pra este repo
- `feedback_auto_push_github`: regra de push automático

## Estrutura de dados (DATA)

```
DATA[year] = {
  hotels: {
    "Resende Imperial": { months, total_costs, cafe, lavanderia, energia, agua, pax, diarias, cost_per_pax, cost_per_day },
    "Terra Boa":        { ... mesmo formato ... },
    "Pedra Torta":      { ... },
    "Vira Canoa":       { ... }
  },
  consolidated: { ... mesmo formato, somado ... }
}
METAS[year][hotel|"_consolidado"] = { cafe: [12], lavanderia: [12] }
```

- `PARTIAL[]` = lista de anos com pelo menos um mês `null` em `total_costs`
- `YEARS[]` = `Object.keys(DATA).sort()` (ascendente)

## Comandos úteis (Bash do Claude)

```bash
# Clone temporário usado no workflow de push:
cd /tmp && git clone https://github.com/SauloAguilarLima1/Indicadores-de-Custos-Vari-veis---GAL.git gal-repo

# Após editar C:\Users\saulo\OneDrive\Desktop\index.html:
cp "/c/Users/saulo/OneDrive/Desktop/index.html" /tmp/gal-repo/index.html
cd /tmp/gal-repo && git add index.html && git commit -m "..." && git push origin main
```

Credenciais HTTPS do GitHub já estão cacheadas no Windows Credential Manager — push funciona sem prompt.
