# Especificação Operacional — LogLine Lab Kit / Santo André Laboratory / minilab.work

> **Documento de runtime, implementação, pack, app, UX e projection. Não é canon.**

## 1. Status do documento

Este documento é uma **spec operacional consolidada** para implementar um Lab runtime/cockpit sob LogLine. Ele contém:

- **runtime doctrine**: regras derivadas para operar sem criar autoridade semântica paralela;
- **implementation spec**: contratos mínimos de armazenamento, API, componentes e testes;
- **pack spec**: como loadables declaram convenções sem redefinir canon;
- **app spec**: como conectores propõem observações/evidências sem admitir Acts;
- **UX spec**: como `minilab.work` captura, explica e projeta o metabolismo;
- **projection spec**: como perfis de exportação traduzem Acts para formatos externos.

Este documento **não é novo canon**. Ele se subordina, nesta ordem, às seguintes autoridades superiores:

1. LogLine canon existente;
2. 9-slot LogLine Act;
3. hash/canonicalization profile oficial;
4. conformance suite oficial;
5. `engine-main` real;
6. `constitutional-runtime-main` real;
7. verified tests;
8. receipts explícitos;
9. ghosts explícitos para qualquer ausência de prova, dúvida, bloqueio ou não-fechamento.

Se houver tensão entre esta spec e qualquer autoridade superior, esta spec não resolve inventando. A tensão deve ser marcada como **open question**, **ghost**, **implementation risk**, **canon alignment required** ou **conformance required**.

## 2. Escopo

### 2.1 O que este documento governa

Este documento governa:

- a arquitetura operacional do LogLine Lab Kit como installable;
- a separação entre kernel, host, loadables, packs, apps, projection profiles e UI;
- os limites de autoridade de Rust, TypeScript, banco, workers, ruler, projections e API;
- o armazenamento append-only de Acts e a reconstrução de projeções;
- a forma como Santo André pode ser carregado como loadout/pack experimental;
- a forma como `minilab.work` funciona como cockpit/projeção/camada de captura;
- os testes e metas de conformance que impedem falso fechamento.

### 2.2 O que este documento não governa

Este documento não governa:

- a definição canônica do LogLine Act;
- a composição final dos 9 slots;
- o algoritmo autoritativo de canonicalização e hashing;
- a lógica interna do Engine real;
- a lógica interna do Constitutional Runtime real;
- a conformance suite oficial;
- a semântica canônica de Receipt, Ghost, Policy, Machine, Registry ou Workflow além do que o canon já define.

### 2.3 Onde Santo André começa e termina

**Santo André Laboratory** começa como **loadout experimental**: um conjunto de packs, policies derivadas, projeções, experiências e exemplos operacionais carregáveis sobre o Lab Kit.

Santo André termina antes de qualquer afirmação de canon. Ele não redefine Act, hash, receipt, ghost, engine, constitutional admission, registry ou workflow. Quando Santo André precisa de uma convenção própria, ela deve viver em pack/loadout e declarar compatibilidade com o canon vigente.

### 2.4 Onde minilab.work começa e termina

`minilab.work` começa como **cockpit humano**: superfície de captura, navegação, projeção, traceabilidade e experiência.

`minilab.work` termina antes de decidir verdade. Ele não admite Acts, não fecha receipts, não calcula hash autoritativo, não cria estado semântico e não substitui timeline. Toda ação de experiência deve produzir proposta, evidência, observação, blob ref ou evento operacional submetido ao runtime apropriado.

### 2.5 Onde LogLine Lab Kit / Installable começa e termina

O **LogLine Lab Kit / Installable** começa como pacote instalável que fornece host, bridges, armazenamento, API, projeções, runners e mecanismos de loadout para operar sob canon LogLine.

Ele termina antes de virar protocolo. O installable implementa, integra e verifica; não redefine a autoridade canônica que executa.

## 3. Executive thesis

Santo André Laboratory / `minilab.work` é um **Lab runtime / cockpit operacional** construído sob o canon LogLine existente. A única verdade semântica durável é o **LogLine Act content-addressed**. Tudo mais é projeção, índice, cache, fila, blob, view, runtime, pack, app, deployment ou materialização operacional.

A regra de produto e arquitetura é: **metabolismo antes de dashboard**.

O sistema não começa por telas, entidades CRUD, tarefas ou agente autônomo. Ele começa por uma cadeia operacional verificável:

```text
human signal → constitutional admission → Act → timeline → ruler → Engine → queue → worker → evidence → receipt/ghost → projection
```

Essa cadeia tem quatro consequências práticas:

1. **Nenhuma UI muda verdade.** A UI captura intenção, mostra projeções e expõe traceabilidade.
2. **Nenhum worker fecha receipt.** Worker executa trabalho bounded e retorna evidence/report como Act ou blob referenciado por Act.
3. **Nenhum scheduler substitui Engine.** O Ruler observa tempo e submete observações ao Engine; ele não espera worker, não fecha receipt e não enfileira sem Engine.
4. **Nenhuma projeção é canon.** Registry, Workorder, Timeline view, Today, export científico e API de consulta são leituras reconstruíveis dos Acts.

A implementação correta preserva dúvida. Se uma operação não tem evidência suficiente, ela não vira “done”; ela vira Ghost, blocked, doubt, needs evidence ou needs authority, conforme admitido pelo canon/runtime real.

## 4. Authority model

A autoridade flui de canon para runtime e de runtime para projeções. Camadas inferiores podem propor, transportar, executar ou renderizar, mas não podem redefinir a camada superior.

| Layer | Authority level | May define | Must not define | Output |
|---|---:|---|---|---|
| LogLine canon | Máxima | Semântica do Act, 9-slot core, categorias canônicas | Convenções locais de pack/app/UI | Canon, normative docs |
| Hash/canonicalization profile | Máxima técnica | Bytes canônicos, tuple/content/envelope hash | Semântica de produto | Hashes verificáveis |
| Conformance suite | Máxima verificadora | Testes de aceitação e rejeição | Features locais | Resultados de conformance |
| Engine real | Autoridade operacional | Transições admitidas pelo Engine, dispatch semântico | Canon novo, UI state | Acts/decisões verificáveis |
| Constitutional Runtime real | Autoridade de admissão | Admit/reject/doubt/needs evidence/needs authority | Canon novo, fechamento por conveniência | Decisão constitucional |
| Lab Kit kernel/host | Implementação subordinada | Storage, bridges, API, projections, runners | Engine ou Constitutional Runtime alternativo como autoridade | Runtime operacional |
| Database | Armazenamento | Persistência append-only e índices operacionais | Verdade semântica | Rows, blobs, projections |
| Ruler | Observador subordinado | Observações de tempo e due Acts propostos | Queue direta, receipt, worker wait | Observation/proposal Acts |
| Worker | Executor subordinado | Evidence/report/output bounded | Receipt fechado, admissão constitucional | Evidence/report Act, blob ref |
| Pack | Convenção carregável | Policies derivadas, views, workflows de pack, fixtures, copy | Canon, hash profile, Engine semantics | Manifest, mappings, tests |
| App | Integração carregável | Tools, auth, resources, act mappers, evidence profiles | Admitted Act, closed receipt | Proposal/observation/evidence/blob/operational output |
| Projection Profile | Interoperabilidade | Export formats and mapping rules | Timeline replacement, truth claim | Export artifact with source hashes |
| UI / minilab.work | Experiência humana | Capture, trace, explain, project | State truth, admission, closure, hashes | Proposal payloads, views, events |
| TypeScript | Transporte/renderização | Forms, clients, rendering, local validation hints | Authoritative hash, admission, closure | Proposed payloads/views |
| Rust backend | Runtime host | Durable store, bridges, verification, projections, dispatch boundaries | Reimplementation of superior authorities | Services and verified artifacts |

## 5. Semantic object model

### 5.1 Act

A **LogLine Act** é a única unidade semântica durável. Um Act é endereçado pelo seu `content_hash`. O endereço real do Act é derivado de conteúdo, não de `row_id`, UUID, sequência ou URL.

### 5.2 9-slot tuple

O 9-slot tuple permanece o núcleo obrigatório do Act. Esta spec não redefine os nomes, tipos ou regras finais dos slots. Quando exemplos usam nomes como `who`, `did`, `this`, `when`, `confirmed_by`, `if_ok`, `if_doubt`, `if_not` e `status`, eles são exemplos derivados e exigem alinhamento com o canon vigente.

### 5.3 Hashes obrigatórios

Todo Act armazenado deve possuir:

- `tuple_hash`: hash do núcleo 9-slot canônico;
- `content_hash`: hash do conteúdo canônico completo do Act, excluindo campos autorreferenciais conforme o perfil oficial;
- `envelope_hash`: hash do envelope de transporte/recepção, excluindo o próprio `envelope_hash` conforme o perfil oficial.

Contrato conceitual:

```text
tuple_hash   = sha256(canonicalize(extract_9_slot_tuple(act)))
content_hash = sha256(canonicalize(act_without_self_referential_fields))
envelope_hash = sha256(canonicalize(envelope_without_envelope_hash))
```

A fórmula acima é descritiva. A fórmula executável é a do hash/canonicalization profile oficial e deve ser testada byte-for-byte.

### 5.4 Objetos que não viram entidades canônicas paralelas

| Termo | Forma correta | Proibição |
|---|---|---|
| Candidate | Proposed Act ou payload ainda não admitido | Tabela/entidade canônica `candidates` |
| Workorder | Projeção de Acts dispatcháveis | Entidade canônica de tarefa |
| Receipt | Act de fechamento escopado | Linha mutável `receipt.done = true` como verdade |
| Ghost | Act de ausência, dúvida, bloqueio ou não-fechamento | Erro silencioso ou pendência invisível |
| Registry | Projeção de Acts | Catálogo autoritativo fora da timeline |
| Machine | Acts sobre máquinas + projeção | Entidade canônica separada |
| Workflow | Acts/policies/projeções conforme canon | Orquestrador paralelo com verdade própria |
| UI state | Cache local e affordance visual | Estado semântico |

### 5.5 Append-only state movement

Estado semântico se move por append-only Acts. Um estado novo não sobrescreve o anterior; ele adiciona um Act que referencia, fecha, contradiz, observa, preserva, bloqueia ou substitui o efeito de Acts anteriores conforme regras superiores.

Rows mutáveis podem existir para performance operacional, locks, leases, cursor de projeção e caches. Essas rows devem ser reconstruíveis, descartáveis ou auditáveis contra Acts. Elas nunca são a fonte da verdade semântica.

## 6. Runtime metabolism

### 6.1 Vertical path

```text
human/LLM/app/sensor signal
→ proposed Act
→ Constitutional Runtime
→ admitted | rejected | doubt | needs evidence | needs authority
```

O caminho vertical transforma sinal em proposta e proposta em decisão constitucional. A decisão pode admitir um Act, rejeitar, preservar dúvida ou exigir evidência/autoridade adicional. A UI e apps podem preparar propostas, mas não podem promover proposta a Act admitido.

### 6.2 Horizontal path

```text
Earth time
→ Ruler
→ due observation Act
→ Engine
→ dispatch Act
→ queue projection
```

O Ruler observa tempo terrestre e submete observações ao Engine. Ele não enfileira diretamente. A fila é projeção de Acts dispatcháveis resultantes do Engine.

### 6.3 Execution path

```text
queue projection
→ worker
→ evidence/report Act
→ receipt review
→ closed receipt | ghost
```

O worker recebe trabalho bounded, executa, preserva evidências e retorna report/evidence. O fechamento acontece por Act de receipt após revisão adequada. Se evidência, autoridade ou execução forem insuficientes, o resultado correto é Ghost/doubt/blocked/needs evidence, não fechamento falso.

### 6.4 Projection path

```text
Acts
→ rebuildable projections
→ UI/API/export
```

Projeções são derivadas. Elas devem carregar source Act hashes quando exibem ou exportam afirmações. Se uma projeção não pode apontar para Acts-fonte, ela deve se declarar incompleta ou ghost.

## 7. Component architecture

| Componente | Responsabilidades | Limites |
|---|---|---|
| Rust backend | Host do runtime, Act Store, bridges, projections, dispatch, workers boundary, API server | Não reimplementa Engine/Constitutional Runtime reais |
| TypeScript | UI, clients, rendering, form validation hints, event transport | Não calcula hash autoritativo; não admite; não fecha |
| Act Store | Persistência append-only de canonical bytes, hashes, envelope e índices mínimos | Não interpreta verdade fora do canon/runtime |
| Engine Bridge | Chama `engine-main` real, registra input/output/receipts/ghosts | Não cria engine alternativo |
| Constitutional Runtime Bridge | Chama `constitutional-runtime-main` real para admissão | Não decide por fallback local |
| Ruler | Observa tempo, propõe due observations, submete ao Engine | Não espera worker; não fecha receipt; não enfileira direto |
| Projections | Reconstrói views de Acts, expõe cursors e source hashes | Não substitui timeline |
| Dispatch | Materializa queue projection e leases operacionais | Não cria workorder canônico |
| Workers | Executam tarefas bounded e produzem evidence/report | Não fecham receipt |
| Receipt/Ghost review | Submete closure/ghost Acts via autoridade adequada | Não usa estado de worker como closure automática |
| Experience Runtime | Orquestra fluxos de UI/capture/onboarding/traces | Não muda verdade sem Act |
| API | Expõe propose/submit/timeline/projections/events/experience | Evita CRUD semântico |

## 8. Installable / Pack / App / Projection split

### 8.1 LogLine Lab Installable

O installable é a distribuição executável do Lab Kit: binários, serviços, migrations, tests, bridges, default projections e host para loadables. Ele implementa infraestrutura subordinada ao canon e aos runtimes reais.

### 8.2 Pack

Pack é um pacote declarativo de convenções operacionais carregáveis: policies derivadas, views, prompts/copy, fixtures, projection configs, workflows de pack e testes. Pack evolui mais rápido que canon, mas sempre declara compatibilidade e nunca redefine canon.

### 8.3 App

App é conector para sistemas externos ou capacidades locais. Ele declara ferramentas, recursos, eventos, autenticação, widgets, act mappers e evidence profiles. Output de App é proposal, observation, evidence, blob ref ou operational output; nunca Act admitido nem receipt fechado.

### 8.4 Projection Profile

Projection Profile é um tradutor de timeline/Acts para formato externo. Ele serve interoperabilidade, auditoria e exportação, não verdade.

### 8.5 Loadout

Loadout é uma composição instalável de packs, apps, projection profiles, UI surfaces, configs e policies derivadas para um Lab específico.

### 8.6 Santo André e minilab.work

Santo André é loadable/experimental: um loadout de laboratório, não canon. `minilab.work` é cockpit/projection/capture surface, não protocolo.

## 9. Repository structure

Layout proposto:

```text
/
  AGENTS.md
  README.md
  CODEOWNERS
  standards/
    authority.md
    hash-profile.md              # pointer/compat declaration; not canon replacement
    conformance.md
    projection-profile.schema.json
    pack.schema.json
    app.schema.json
  kernel/
    crates/
      act-store/
      hash-verify/
      engine-bridge/
      constitutional-bridge/
      ruler/
      projections/
      dispatch/
      worker-host/
      api/
    migrations/
    tests/
      conformance/
      golden/
  host/
    labd/
    config/
    loadout-loader/
  loadables/
    packs/
      santo-andre/
      research-clock/
    apps/
      github/
      local-files/
      local-llm/
    projection-profiles/
      prov/
      ro-crate/
      datacite/
  ui/
    minilab-work/
      app/
      components/
      traces/
      copy/
      tests/
  docs/
    operations/
    incidents/
    examples/
```

Jurisdiction notes:

- `standards/` declares compatibility and local contracts; it must not replace canon.
- `kernel/` owns authority-sensitive implementation boundaries and conformance tests.
- `host/` owns installable orchestration and loadout resolution.
- `loadables/` owns packs/apps/projection profiles with lower authority.
- `ui/` owns capture/projection experience only.
- `docs/operations/` owns operational doctrine and incident procedures.

Santo André must not be the repository root identity unless the repository is explicitly scoped as a Santo André loadout. The root identity should be Lab Kit / installable / host.

## 10. Database and storage

### 10.1 Canonical `logline_acts` table

Contract example for PostgreSQL:

```sql
CREATE TABLE logline_acts (
  row_id BIGSERIAL PRIMARY KEY,
  content_hash TEXT NOT NULL UNIQUE,
  tuple_hash TEXT NOT NULL,
  envelope_hash TEXT NOT NULL,
  canonical_bytes BYTEA NOT NULL,
  act_json JSONB NOT NULL,
  envelope_json JSONB NOT NULL,
  received_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  source TEXT,
  conformance_profile TEXT NOT NULL,
  CHECK (content_hash <> ''),
  CHECK (tuple_hash <> ''),
  CHECK (envelope_hash <> '')
);

CREATE INDEX logline_acts_received_at_idx ON logline_acts (received_at);
CREATE INDEX logline_acts_tuple_hash_idx ON logline_acts (tuple_hash);
```

`canonical_bytes` is the byte sequence used for authoritative hash verification. `act_json` exists for query ergonomics and must be derivable from or verified against `canonical_bytes` under the canonicalization profile. If there is a mismatch, the row is invalid and must produce ghost/incident, not silent repair.

### 10.2 Append-only trigger

```sql
CREATE OR REPLACE FUNCTION prevent_logline_act_update_delete()
RETURNS trigger AS $$
BEGIN
  RAISE EXCEPTION 'logline_acts is append-only';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER logline_acts_no_update
BEFORE UPDATE ON logline_acts
FOR EACH ROW EXECUTE FUNCTION prevent_logline_act_update_delete();

CREATE TRIGGER logline_acts_no_delete
BEFORE DELETE ON logline_acts
FOR EACH ROW EXECUTE FUNCTION prevent_logline_act_update_delete();
```

### 10.3 Blob storage

Blobs store large evidence, artifacts, external payloads and exports. A blob is not semantic truth by itself. It becomes relevant only when referenced by an Act with hash, media type, size, provenance and evidence profile.

### 10.4 Rebuildable projections

Projection tables may include:

- `projection_cursors`;
- `workorder_projection`;
- `registry_projection`;
- `timeline_projection`;
- `today_projection`;
- `machine_projection`;
- `receipt_projection`;
- `ghost_projection`.

They must be rebuildable from `logline_acts` and must expose source Act hashes. They may be truncated and rebuilt without semantic loss.

### 10.5 Operational mutable tables

Operational updates are allowed for:

- worker leases;
- retry counters;
- rate-limit buckets;
- API sessions;
- websocket subscriptions;
- projection cursors;
- blob upload multipart state;
- app OAuth tokens;
- UI preferences.

These tables are operational convenience. They are not semantic truth and must not be used as closure, admission, proof or canon.

## 11. API shape

The API avoids semantic CRUD. It exposes proposal, submission, timeline, projections, ruler, events and experience actions.

| Endpoint | Purpose | Authority note |
|---|---|---|
| `POST /acts/propose` | Build/validate proposal payload | Not admitted truth |
| `POST /acts/submit` | Submit to Constitutional Runtime bridge | Admission depends on real runtime |
| `GET /acts/{content_hash}` | Fetch Act by content address | Canonical read if hash verified |
| `GET /timeline` | Ordered Act reading | Read model, not entity store |
| `GET /projections/*` | Rebuildable views | Must return `projection_is_truth=false` |
| `POST /ruler/tick` | Trigger/record ruler observation cycle | Ruler submits through Engine |
| `GET /events` | Operational event stream | Not semantic truth unless backed by Acts |
| `POST /experience/*` | UI action/capture flows | Produces proposal/evidence, not truth |

Projection response contract:

```json
{
  "projection_is_truth": false,
  "projection_name": "today",
  "source_act_hashes": ["sha256:..."],
  "items": []
}
```

Experience action contract:

```json
{
  "action": "write.capture",
  "result_kind": "proposal",
  "proposal_id": "operational-only-id",
  "submitted_act_hash": null,
  "requires_constitutional_admission": true
}
```

## 12. UX / minilab.work

`minilab.work` is a human-facing projection and capture layer. It must make authority visible.

| Surface | Definition | Must show |
|---|---|---|
| Today | Meeting point with the Ruler; due observations and current obligations as projection | Source Acts, due basis, Engine status |
| Write | Mouth of the vertical runtime; signal-to-proposal surface | Proposed tuple/content, admission status, missing evidence |
| Timeline Trace | Trust surface for reading ordered Acts and their consequences | content hashes, receipts, ghosts, projections derived |
| Onboarding | First passage through metabolism, not feature tour | Signal → proposal → admission → trace → projection |

Copywriting rules:

- Say “proposal submitted”, not “task created”, unless an admitted Act proves dispatchability.
- Say “worker reported evidence”, not “AI completed”.
- Say “receipt closed by Act `<hash>`”, not “done”, unless a scoped receipt Act exists.
- Say “needs evidence”, “blocked”, “doubt” or “ghost preserved” instead of implying closure.
- Say “projection updated”, not “truth changed”.
- Say “model suggested” or “model drafted”, not “AI decided”.

## 13. Packs

### 13.1 Pack manifest

```yaml
id: santo-andre
kind: logline.pack
version: 0.1.0
compatibility:
  logline_canon: ">=x.y <x.z"
  hash_profile: "official-profile-id"
  conformance_suite: "suite-id"
declares:
  policies: []
  projections: []
  copy_rules: []
  workflow_conventions: []
  fixtures: []
  tests: []
forbidden:
  defines_canon: false
  overrides_hash_profile: false
  admits_acts: false
  closes_receipts: false
```

### 13.2 Packs may declare

- local policy conventions subordinate to canon;
- UI copy and terminology;
- projection definitions;
- fixtures and examples;
- workflow conventions expressed as Acts/projections;
- app dependencies;
- conformance tests for the pack.

### 13.3 Packs may never do

- redefine Act or 9-slot tuple;
- change hash/canonicalization behavior;
- bypass Engine or Constitutional Runtime;
- declare projections as truth;
- close receipt by pack rule alone;
- hide ghosts.

### 13.4 Examples

`Santo André pack` may define laboratory-specific copy, views, policies and fixtures. It remains experimental/loadable.

`research-clock pack` may define due-observation conventions for research cadence. It must route time observations through Ruler → Engine.

Pack evolution is allowed to be faster than canon evolution because packs have lower authority.

## 14. Apps

### 14.1 Connection app manifest

```yaml
id: github
kind: logline.app
version: 0.1.0
auth:
  type: oauth2
tools:
  - id: list_pull_requests
resources:
  - id: repository
events:
  - id: pull_request_opened
widgets:
  - id: pull_request_card
act_mappers:
  - id: pr_event_to_observation_proposal
evidence_profiles:
  - id: github_api_response
outputs:
  allowed:
    - proposal
    - observation
    - evidence
    - blob_ref
    - operational_output
  forbidden:
    - admitted_act
    - closed_receipt
```

### 14.2 Allowed and forbidden outputs

Allowed:

- proposal payload;
- observation payload;
- evidence Act proposal;
- blob ref with digest and provenance;
- operational output for UI or worker continuation.

Forbidden:

- admitted Act;
- closed receipt;
- authoritative hash;
- queue item not derived from Engine output;
- semantic mutation of Registry/Workorder/Machine state.

GitHub, local files and local LLM integrations are examples only. Each app must prove boundary behavior through app authority tests.

## 15. Projection Profiles

### 15.1 Projection profile manifest

```yaml
id: prov
kind: logline.projection_profile
version: 0.1.0
format: PROV
projection_is_truth: false
requires:
  source_act_hashes: true
  generated_at: true
  profile_version: true
mappings:
  - from: logline.act
    to: prov.entity_or_activity
```

### 15.2 Scientific exports

Scientific exports are interoperability artifacts. They help external tools understand provenance, datasets, workflows or publications. They do not replace timeline and do not become semantic authority.

Optional standard profiles may include PROV, RO-Crate, DataCite, DCAT, JATS, CWL and FAIR-oriented exports. Every exported assertion must include source Act hashes where possible and must set `projection_is_truth=false`.

## 16. Test and conformance plan

| Test area | Acceptance criteria |
|---|---|
| Hash tests | tuple/content/envelope hashes match official profile byte-for-byte |
| Engine bridge tests | all dispatch decisions come from `engine-main` real or are ghosts |
| Constitutional Runtime bridge tests | admission/rejection/doubt outcomes come from real runtime |
| Ruler tests | Ruler submits observations to Engine and never queues directly |
| Projection rebuild tests | projections rebuild from Acts with identical output |
| Worker boundary tests | workers produce evidence/report only and cannot close receipts |
| Receipt/Ghost tests | missing evidence creates ghost/doubt/needs evidence, not closure |
| Pack conformance tests | packs cannot override canon/hash/engine/admission |
| App authority tests | apps cannot emit admitted Acts or closed receipts |
| Projection golden export tests | exports contain source hashes and `projection_is_truth=false` |
| UI traceability tests | UI displays source hashes/status and never claims truth from local state |

Hard targets:

```text
false_closure_rate = 0
false_ok_from_missing_evidence = 0
model_output_as_evidence = 0
worker_closed_receipt = 0
projection_claimed_canon = 0
scheduler_bypassed_engine = 0
ui_state_claimed_truth = 0
app_admitted_act = 0
```

## 17. Implementation phases

| Phase | Scope | Exit criteria |
|---:|---|---|
| 0 | Jurisdiction and authority files | `standards/authority.md`, CODEOWNERS and compatibility declarations exist |
| 1 | Act Store and hash profile | append-only store verifies tuple/content/envelope hashes |
| 2 | Engine Bridge | calls real Engine and ghosts unsupported paths |
| 3 | Constitutional Runtime Bridge | calls real constitutional runtime for submit/admission |
| 4 | Ruler | time observations route through Engine only |
| 5 | Projections | timeline/registry/workorder/today projections rebuild from Acts |
| 6 | Queue and Worker | queue is projection; workers return evidence/report only |
| 7 | Receipt/Ghost | closure and non-closure are explicit Acts |
| 8 | Loadout / Packs | pack loader and pack conformance tests pass |
| 9 | App Host | app manifests, auth, mappers and authority tests pass |
| 10 | Projection Profiles | golden exports with source hashes and non-truth flag pass |
| 11 | minilab.work Experience | Today, Write, Timeline Trace and onboarding obey UX authority rules |
| 12 | Conformance/release hardening | hard targets are enforced in CI/release gates |

## 18. Maintenance and governance

### 18.1 Release channels

| Channel | Owns | Compatibility declaration |
|---|---|---|
| kernel | Act Store, bridges, ruler, projections, dispatch, worker boundary | canon/hash/conformance/engine/runtime versions |
| host | installable packaging and loadout resolution | kernel and loadable compatibility |
| pack | local conventions and fixtures | canon/kernel/conformance range |
| app | connectors and mappers | host/app schema/evidence profile |
| projection | export formats and mappings | source Act hash requirements/profile version |
| UI | capture/projection experience | API schema and authority-copy rules |

### 18.2 Incident classes

- `hash_mismatch`: canonical bytes/json/hash disagreement;
- `engine_bypass`: dispatch/queue happened without Engine Act;
- `constitutional_bypass`: admitted truth without Constitutional Runtime decision;
- `false_closure`: UI/app/worker/projection implied closure without receipt Act;
- `ghost_suppression`: absence/doubt/block not preserved;
- `projection_truth_claim`: projection or export claimed canonical authority;
- `app_authority_violation`: app emitted admitted Act or closed receipt;
- `ui_authority_violation`: UI local state claimed semantic truth.

### 18.3 CODEOWNERS/jurisdiction model

Authority-sensitive areas require review from maintainers responsible for canon alignment/conformance:

```text
/standards/                 @authority-maintainers
/kernel/crates/act-store/   @kernel-maintainers @authority-maintainers
/kernel/crates/*-bridge/    @kernel-maintainers @authority-maintainers
/kernel/crates/ruler/       @kernel-maintainers
/loadables/packs/           @pack-maintainers
/loadables/apps/            @app-maintainers
/loadables/projection-profiles/ @projection-maintainers
/ui/minilab-work/           @ui-maintainers
```

Rule: every authority-relevant change must produce an Act or conformance artifact. If the system cannot yet produce an Act for its own change, the release must include an explicit conformance artifact and a ghost marking the bootstrap gap.

## 19. De-duplication and rewrite rules

These rules govern future edits to this document and derived docs:

- Merge repeated explanations into the first authoritative section that owns them.
- Keep strong phrases only once; this document keeps “metabolismo antes de dashboard” as doctrine.
- Turn poetic metaphors into short doctrine boxes or remove them.
- Convert long examples into one canonical end-to-end example.
- Convert repeated prohibitions into compact boundary tables.
- Convert aspirations into acceptance criteria.
- Convert vague nouns into files, tests, manifests, APIs or ghosts.
- Remove any paragraph that sounds like canon expansion unless explicitly classified as derived runtime doctrine or pack convention.

### Canonical end-to-end example

```text
1. Human writes: “run local analysis on dataset X tomorrow at 09:00.”
2. Write surface builds a proposed Act payload.
3. Constitutional Runtime admits, rejects, doubts or requests evidence/authority.
4. If admitted, Act appears in timeline by content_hash.
5. At due time, Ruler emits due observation proposal to Engine.
6. Engine produces dispatch Act if appropriate.
7. Queue projection shows dispatchable work.
8. Worker executes bounded command and returns evidence/report Act proposal plus blob refs.
9. Review submits closure; if evidence is sufficient, receipt Act closes scope.
10. If evidence is insufficient, Ghost/doubt/needs evidence Act preserves non-closure.
11. Today, Timeline Trace, Registry and exports update as projections with source hashes.
```

## 20. Output format for derived documents

Derived documents must:

- use clear headings;
- state status and authority at the top;
- include tables where they reduce ambiguity;
- include code/schema examples only where they define contracts;
- keep examples minimal but complete;
- avoid marketing tone;
- avoid generic startup/product language;
- avoid “AI workspace” framing;
- avoid hidden new ontology.

## Change Log From Source Material

### What was preserved

- The central thesis: Santo André Laboratory / `minilab.work` is a Lab runtime/cockpit under existing LogLine canon.
- The claim that durable semantic truth is only the content-addressed LogLine Act.
- The phrase “metabolismo antes de dashboard”.
- The operational chain: `human signal → constitutional admission → Act → timeline → ruler → Engine → queue → worker → evidence → receipt/ghost → projection`.
- The boundaries: UI does not decide state; TypeScript proposes/transports/renders; Worker produces evidence/report only; Ruler observes time and routes through Engine; projections are not truth.
- The need for explicit receipts and ghosts.

### What was demoted from possible canon to runtime/pack/projection/implementation

- Santo André became a loadout/pack convention, not a canonical identity.
- `minilab.work` became cockpit/projection/capture surface, not protocol.
- Candidate, Workorder, Registry, Machine, Workflow, Today and UI state became Acts/projections/operational views, not canonical entities.
- Pack declarations became compatibility-scoped loadables, not semantic authorities.
- App outputs became proposals/observations/evidence/blob refs/operational outputs, not admitted Acts.
- Scientific exports became projection profiles for interoperability, not truth.

### What was removed as repetition

- Repeated statements that “everything else is projection” were consolidated into the thesis, authority model and boundary tables.
- Repeated warnings against CRUD/SaaS/dashboard framing were consolidated into scope, executive thesis and API/UX sections.
- Repeated worker/ruler/UI prohibitions were consolidated into component and test tables.
- Long rhetorical passages were converted into contracts, manifests, database schemas, endpoint shapes and acceptance criteria.

### What remains unresolved/ghost

- Exact canonical names, types and validation rules of the 9-slot tuple require canon alignment.
- Exact hash/canonicalization byte profile requires conformance verification.
- Exact `engine-main` and `constitutional-runtime-main` invocation contracts require implementation discovery and bridge tests.
- Exact Receipt and Ghost schemas require canon/runtime alignment.
- Exact projection ordering semantics for Timeline require conformance or canon reference.
- Bootstrap self-governance remains ghost until authority-relevant changes can themselves produce Acts.

### What must be verified against canon, engine, conformance or tests

- `tuple_hash`, `content_hash` and `envelope_hash` calculations.
- Append-only Act Store behavior and canonical bytes/json consistency.
- Constitutional admission outcomes.
- Engine dispatch outcomes.
- Ruler non-bypass behavior.
- Worker inability to close receipts.
- Projection rebuild determinism and source-hash traceability.
- Pack/app/projection profile boundary enforcement.
- UI copy and traceability rules.
- All hard targets listed in the conformance plan.
