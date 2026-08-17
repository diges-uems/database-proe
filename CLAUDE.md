# database-proe

App single-file React (CDN Babel/Tailwind, sem build) pra PROE/UEMS acompanhar
matrículas de cursos. `index.html` é a aplicação inteira. Backend é Google
Apps Script (`apps-script/Code.gs`, local/gitignored — nunca sobe pro GitHub)
que lê/escreve a planilha "Base de Dados".

## Leitura via Supabase (espelho)

App lê de um espelho Supabase (projeto `HubDIges`, `rzsknikdafffwcupxxjv`,
tabela `public.proe_cursos`) em vez da planilha direto — muito mais rápido.
Escrita continua só via Apps Script (planilha é fonte da verdade), que
sincroniza pro Supabase depois de cada save/import/delete via
`syncRowsToSupabase()`/`deleteRowFromSupabase_()` em `Code.gs`.

RLS: `anon`/`authenticated` só têm select. Só `service_role` (key fica em
`PropertiesService.getScriptProperties()` do Apps Script, nunca no client)
pode escrever. Tabela é solta no `public` do projeto HubDiges (não é schema
isolado nem projeto novo) — ver aviso equivalente em
`C:\Users\bruno.lopes\Projetos\hub-diges\CLAUDE.md`.

`apps-script/Code.gs` **não é deployado automaticamente** — precisa colar
manual no editor do Apps Script depois de qualquer mudança.

## 2026-08-17 — Performance da sincronização + hint de dado parcial

- **Gargalo achado**: `fetchAPI` (index.html) esperava resposta do Apps
  Script (`projPromise`, endpoint `/exec`) antes de renderizar QUALQUER
  troca de filtro/página, mesmo já lendo os registros do Supabase — cold
  start do Apps Script anulava o ganho de velocidade. Fix: projeções agora
  cacheadas em `projecoesRef` (useRef), buscadas só 1x por sessão, só
  invalidadas pelo botão "Sincronizar".
- Limpou texto de debug ("BUGOU TUDO SÓ CONSIGO ARRUMAR DEPOIS DA 18 😢")
  que ficou no título e no header depois de edições diretas no GitHub web
  editor durante um susto de bug ao vivo.
- Adicionado hint (ⓘ com tooltip) nos cards "Taxa de Ocupação" e "Evasão
  Média Global" quando o ano de referência ainda está em andamento
  (`emAndamento()`) — mesmo padrão do "A apurar" já usado por linha,
  avisando que os números de 2026 só fecham no fim do ano letivo.
- Achados do hook de design (`overused-font` Inter, `layout-transition`,
  `dark-glow`, `gray-on-color` L551/L757) são pré-existentes, não tocados
  nessas mudanças — não mexidos.
