# CLAUDE.md — drescribia-internato

> Guia para Claude (Claude Code, claude.ai/code) a trabalhar neste repositório.

## O que é este projecto

**Dr. Escriba IA — Edição Internato MGF (v10.0-alpha)**

Aplicação single-file (`index.html`) destinada a **médicos internos da
especialidade de Medicina Geral e Familiar** em Portugal. É um irmão do
projecto principal `drescribia` (que serve especialistas), mas com
funcionalidades específicas do internato:

- **Modo SOAP** (igual ao especialista, partilhado): Urgência, VD, Consulta, Internamento.
- **Modo Internato** (exclusivo desta edição): Logbook OM, Avaliações, Diário, Journal Club.

O toggle no header alterna entre os dois modos. O modo Internato usa a
paleta amber/copper (`#bda060`) para se distinguir visualmente do SOAP
(azul/verde/violeta).

## Relação com o repo `drescribia`

| Repo | Público | Versão | URL |
|---|---|---|---|
| `proteinaludica/drescribia` | Especialistas MGF | v9.0 (estável, só SOAP) | project-0u29a.vercel.app |
| `proteinaludica/drescribia-internato` (este) | Internos MGF | v10.0-alpha (SOAP + Internato) | TBD |

**Os dois projectos partilham o motor SOAP.** Quando houver bug-fix no SOAP
de um, o utilizador (Roberto) decide se porta para o outro. Não há sync
automático — é decisão consciente, projecto a projecto.

## Princípios de arquitectura (não quebrar)

1. **Single-file**: tudo dentro de `index.html`. Sem build, sem split em
   ficheiros separados. Portabilidade máxima (file://, USB, sem servidor).
2. **CSP restritiva**: scripts só de `'self'` + `cdnjs.cloudflare.com`
   (PDF.js, Tesseract OCR). Não introduzir CDNs novos sem actualizar a CSP
   e justificar.
3. **localStorage namespacing**:
   - `dx_*` para SOAP (partilhado com `drescribia`)
   - `dx_int_*` exclusivo do Internato (`dx_int_mode`, `dx_int_state_v1`)
4. **PT-PT clínico**: terminologia de Medicina Geral e Familiar portuguesa.
   Nada de pt-BR ou inglês excepto rubricas oficiais (OM, ICPC-2, CIE-10).
5. **Privacidade**: dados ficam só no browser do médico. Sem telemetria,
   sem backend, sem analytics. PDFs gerados em memória.

## File map (linhas aproximadas)

- **CSS**: linhas 1–530 (variáveis, layout, dark mode, modo Internato a partir de ~451)
- **HTML SOAP** (header, tabs S/O/P/R, painéis): ~530–7000
- **HTML Internato** (toggle, tabs internas, panel-internato): bloco específico ~540–600 (toggle) + painéis dedicados
- **JS** (calculadoras, generate, persist, ICD lookup, PDF.js, Tesseract): final do ficheiro

## Convenções de commits e branches

- **Branches**: `claude/<descrição-curta>` para trabalho do Claude Code
- **Commits**: estilo Conventional (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`)
- **Co-authoring**: commits do Claude usam `Claude <noreply@anthropic.com>` como autor
- **Versão visível**: `<title>` e `#hdr-title` reflectem a versão actual quando há mudança
  significativa (ex: `v10.0-alpha` → `v10.0-beta`)

## Como adicionar campos clínicos novos

Padrão a seguir (do projecto SOAP):

1. **Markup**: campo dentro do painel correspondente (S/O/P ou tab Internato)
2. **Listener**: input dispara `buildSnapshot()` (auto-save) via id que **não**
   começa por `tg-` (excepto chips)
3. **Output**: contributo para `buildSoc()`, `buildO()`, `buildP()` ou
   handler do Internato
4. **PII clearing**: se campo contém PII do doente, adicionar à whitelist
   de `dxClearPII`
5. **ICD lookup**: se campo é diagnóstico, anexar `icdAttach()`

## Calculadoras clínicas (CALCS array)

Novas escalas vão ao `CALCS` array (procurar `var CALCS = [`). Tipos:
- `chips` (default): toggle ON/OFF de pontuação
- `multi`: select 0-N por item (ex.: PHQ-9, GAD-7, ESS)
- `class`: classe funcional (ex.: NYHA, CCS, EHRA, Killip)

Ver convenção em comentário acima do array.

## Pitfalls específicos do Internato

- **Não quebrar SOAP**: trabalho no modo Internato deve estar **isolado** do
  SOAP via CSS `body.dx-mode-internato` e `body:not(.dx-mode-internato)`.
- **Tabs Internato têm IDs próprios**: `tabs-internato`, `panel-internato`,
  classes `tab-int`, `panel-int`. Não reutilizar IDs do SOAP.
- **localStorage do Internato**: usar SEMPRE prefixo `dx_int_*`. Nunca
  escrever em `dx_autosave_v1` (chave do SOAP) a partir do modo Internato.
- **OM rubrics**: ao adicionar avaliações, manter alinhamento com
  rubricas oficiais da Ordem dos Médicos (Estatuto do Internato Médico).

## Sub-agents — limites prudentes

Após o incidente da V9f (sessão rebentou com 23 sub-agents paralelos):

- **Máximo 3 sub-agents simultâneos** numa sessão
- **`/compact`** quando passar 30k tokens
- **Terminar cada sub-agent** com merge ou abandono explícito antes de iniciar próximo
- Se a tarefa pedir 5+ sub-agents, dividir em sessões separadas

## Roadmap (Fase 1 → Fase N)

- **Fase 1** ✅ — Esqueleto: toggle SOAP/Internato + painel-base com 4 tabs (em curso)
- **Fase 2** — Logbook OM com rubricas oficiais (Estatuto IM)
- **Fase 3** — Avaliações estruturadas (orientador, fim de estágio)
- **Fase 4** — Diário com tags e busca
- **Fase 5** — Journal Club com bibliografia integrada
- **Fase N** — Sync cloud (opcional, com encriptação local)

## Não usar tools fora desta whitelist

Tudo o resto está fora do scope deste projecto:
- ❌ Frameworks (React, Vue, etc) — quebra single-file
- ❌ Build tools (webpack, vite) — quebra portabilidade
- ❌ Backend / API — quebra privacidade
- ❌ Analytics / telemetria — quebra privacidade
- ❌ CDN não-allowlisted — quebra CSP

---

**Última actualização**: 2026-05-05 (criação do repo, separação de `drescribia`).
