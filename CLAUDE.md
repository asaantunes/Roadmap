# Omnibees Roadmap — Contexto do Projeto

## Infraestrutura
- **App**: ficheiro único `index.html` servido por `server.js` (Node.js) na porta 3000
- **Iniciar servidor**: usar `preview_start "Roadmap"` (MCP Claude Preview)
- **API proxy**: Cloudflare Worker em `https://lucky-star-d742.ana-g-antunes.workers.dev`
- **Fonte de dados**: Aha! REST API via proxy (autenticação Bearer no Worker)
- **Git**: worktree em `.claude/worktrees/hopeful-shaw`, push force para `main`
  ```
  git push origin claude/hopeful-shaw:main --force
  ```
- **Password do site**: `omnibees2024`

## Aha! — Pivot e Campos
- **Pivot ID**: `7612677886557207494`
- **Field refs — auto-resolvidos** via `field_definitions` da resposta da pivot (resistente a rebuilds):
  - Rows: `releases.name`, `releases.start_date`, `releases.end_date`, `releases.parking_lot`
  - Epics: `epics.name`, `epics.progress`, `epics.status`, `epics.manual_risk_comment`
  - Custom fields (por título): `"Prio Produto"`, `"Epic Visibility"`
- **Lookup**: objeto `FD` em `loadData()` com chaves `epicName`, `epicProgress`, `epicStatus`, `epicRisk`, `epicPrio`, `epicVisibility`
- **Hierarquia rows**: segue `child_refs[0]` a 4 níveis (nameRow → startRow → endRow → plRow) — não depende de refs numéricos

## Estrutura de Dados — Épico
```js
{
  name, _epicRef, _releaseRef,
  _priority: Number | null,   // null = sem prioridade definida
  progress: Number | null,    // 0–100
  workflow_status: { name },
  _alert: String | null,
}
```

## Ordenação dos Épicos
- Primário: `_priority` ascendente (null → Infinity, vai para o fim)
- Secundário: `getStatusInfo().group` ascendente (0=live → 9=cancelado)

## STATUS_CONFIG — Ordem do Workflow Aha!
```
group 0 → live
group 1 → released / lançado / lançada
group 2 → pronto para release / ready for release
group 3 → em desenvolvimento / in development
group 4 → pronto para desenvolvimento / ready for development
group 5 → em especificação final
group 6 → em aprovação de escopo
group 7 → em escopo geral
group 8 → backlog de features / backlog
group 9 → cancelada / cancelled / cancelado
default desconhecido → group 9
```

## Layout — Modos de Render (renderAll)
1. `renderHighlights()` — painel TV no topo (fundo escuro, scroll horizontal por meta)
2. `renderTimeline()` — cronograma visual das metas
3. `renderMatrix()` — grelha desktop (CSS Grid, visível > 768px)
4. `renderMobileMatrix()` — layout vertical (visível ≤ 768px)

## Painel de Destaques (TV)
- **Filtro atual**: `isHighlighted(epic)` → `epic._priority != null`
  - Quando houver campo dedicado no Aha!, alterar só esta função
- **Limite**: `HIGHLIGHTS_MAX_PER_SQUAD = 3` (constante no topo do JS)
- **Overflow**: indicador `+N épicos` com classe `.hl-more`
- **Navegação**: clicar no label de squad → `scrollToMatrix(cellKey)`
  → scroll suave até à célula na matrix + flash laranja (animação `cell-flash`)

## Matrix Desktop
- **Colunas**: `180px repeat(N, 420px)`
- **IDs nas células**: `id="cell-${cellKey}"` (para navegação dos highlights)
- **cellKey format**: `` `${year}_${metaNum}_${squad.replace(/[^\w]/g,'_')}` ``
- **Store de épicos**: `window.__cellStore[cellKey]` (usado pelo modal)
- **Cache de features**: `_featCache["epicRef__releaseRef"]`

## Features (lazy load ao clicar ▶)
- `toggleFeatures(btn, epicRef, releaseRef)`
- Filtra `f.release?.reference_num === releaseRef` (só features da meta correta)
- Tooltip no ponto de cor: `title="${workflow_status.name}"`

## Modal de Detalhes
- `openDetails(cellKey)` → mostra todos os épicos incluindo os com nome a terminar em `-`
- Épicos com nome a terminar em `-` estão ocultos na vista resumida (são "épicos de suporte")

## Commits Recentes (main)
- `656bba6` — Highlights: scroll inicial posicionado a mostrar metade do último card passado
- `2980396` — loadData: resolver field refs dinamicamente via field_definitions da pivot
- `c2419ca` — Fix: field refs 89-98 → 93-102 após rebuild da pivot Aha!
- `0466e0d` — Risk badges: suporte para tag 'urgente'
