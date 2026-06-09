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


## 21. Operational detail annexes

These annexes are part of this implementation spec. They exist to restore operational density without changing the status of the document. They are not canon; they are implementation contracts and review aids subordinate to the authority model above.

### 21.1 Non-negotiable jurisdiction rules

The following rules are deliberately repeated in executable language because they are common failure modes during implementation.

| Rule id | Rule | Enforcement artifact | Failure classification |
|---|---|---|---|
| J-001 | No durable semantic state outside admitted Acts | Act Store tests, projection rebuild tests | `semantic_state_outside_act` |
| J-002 | No admitted Act without Constitutional Runtime decision | API tests, bridge tests | `constitutional_bypass` |
| J-003 | No dispatch queue item without Engine-derived Act | Ruler/dispatch tests | `engine_bypass` |
| J-004 | No worker can close a receipt | Worker host capability tests | `worker_closed_receipt` |
| J-005 | No UI copy may imply closure without receipt Act | UI copy tests | `ui_false_closure` |
| J-006 | No app may emit an admitted Act | App host tests | `app_admitted_act` |
| J-007 | No projection/export may claim canonical truth | Projection golden tests | `projection_claimed_canon` |
| J-008 | No database row id may be used as semantic address | API and schema tests | `row_id_claimed_truth` |
| J-009 | No hash may be calculated authoritatively in TypeScript | client lint/test rule | `client_authoritative_hash` |
| J-010 | Missing evidence must become explicit non-closure | Receipt/Ghost tests | `ghost_suppression` |

A change that violates any jurisdiction rule cannot be treated as a product bug only. It is an authority incident. The fix must include both code correction and a receipt/ghost/conformance artifact, depending on what the real runtime can prove.

### 21.2 Authority decision procedure

When an implementer is unsure whether a component may perform an action, use this procedure:

1. Identify whether the action changes durable semantic meaning.
2. If yes, require an admitted Act and route through the Constitutional Runtime.
3. Identify whether the action dispatches execution.
4. If yes, require Engine-derived dispatch and treat the queue as projection.
5. Identify whether the action closes a scope.
6. If yes, require a scoped Receipt Act; worker output alone is insufficient.
7. Identify whether the action is a read, view, export or cache.
8. If yes, mark it as projection and attach source Act hashes.
9. Identify whether the action is uncertain, missing evidence, blocked or not proven.
10. If yes, preserve the condition as Ghost/doubt/needs evidence/needs authority.

Decision table:

| Question | If true | If false |
|---|---|---|
| Does it create semantic truth? | Must become or reference admitted Act | May remain operational artifact |
| Does it depend on time/due-ness? | Ruler observes, Engine decides | No Ruler authority required |
| Does it schedule work? | Engine-derived dispatch required | It may be UI planning/proposal only |
| Does it assert completion? | Receipt Act required | Use report/evidence/projection language |
| Does it expose external format? | Projection Profile with source hashes | Normal internal projection rules apply |
| Is evidence absent or insufficient? | Preserve Ghost/doubt | Continue normal review path |

### 21.3 Act lifecycle as operational pipeline

The implementation may use operational names for lifecycle handling, but those names are not new canonical entities.

| Operational stage | Meaning | Durable artifact | Allowed storage |
|---|---|---|---|
| `captured_signal` | Human/model/app/sensor input captured | none yet, unless capture itself is admitted | ephemeral event, draft table |
| `proposal_built` | Candidate Act payload assembled | proposal payload, not admitted Act | operational draft, API response |
| `submitted_for_admission` | Proposal sent to Constitutional Runtime | submission envelope, bridge log | bridge table/log |
| `admitted` | Runtime admitted Act | `logline_acts` row with hashes | append-only Act Store |
| `rejected` | Runtime rejected proposal | rejection record or Act if canon requires | bridge log/ghost path |
| `doubt` | Runtime preserved doubt | Ghost/doubt Act if admitted | Act Store |
| `needs_evidence` | Runtime requires evidence | Ghost/needs-evidence Act if admitted | Act Store/projection |
| `engine_evaluated` | Engine processed admitted Acts/observations | Engine output Act(s) | Act Store |
| `dispatchable` | Projection sees work that can be leased | source Acts | queue projection |
| `leased` | Worker has bounded operational lease | none semantic | mutable lease table |
| `reported` | Worker submitted evidence/report | evidence/report Act or proposal | Act Store/blob ref |
| `reviewed` | Closure review happened | receipt or ghost Act | Act Store |
| `projected` | Views/export updated | none semantic | projection tables/cache |

The lifecycle may never be implemented as a mutable semantic state machine where a row moves from `candidate` to `done`. Movement is by appended Acts. Mutable operational rows are leases, cursors or caches only.

### 21.4 Canon alignment register

This document deliberately avoids resolving canon questions. The implementation must maintain an alignment register with at least these entries:

| Register id | Topic | Required proof | Temporary handling |
|---|---|---|---|
| CA-001 | Exact 9-slot names and type constraints | Canon citation + conformance tests | Use examples only as non-authoritative labels |
| CA-002 | Canonicalization profile | Byte-for-byte golden vectors | Reject or ghost unsupported serialization |
| CA-003 | Tuple hash exclusion/inclusion rules | Hash profile tests | Do not infer from UI or DB JSON |
| CA-004 | Content hash self-reference exclusions | Hash profile tests | Fail closed on ambiguity |
| CA-005 | Envelope hash envelope model | Hash profile tests | Store raw envelope and mark risk |
| CA-006 | Receipt Act schema | Canon/runtime fixture | Do not close without fixture-backed path |
| CA-007 | Ghost/doubt statuses | Canon/runtime fixture | Preserve textual ghost with source hashes |
| CA-008 | Engine dispatch contract | `engine-main` tests | No direct dispatch fallback |
| CA-009 | Constitutional admission contract | `constitutional-runtime-main` tests | No local admission fallback |
| CA-010 | Timeline ordering | Canon/conformance suite | Expose ordering profile in API response |

Alignment register entries should be files, not comments in code. Recommended path: `standards/alignment/*.md` or `docs/operations/alignment-register.md`.

### 21.5 Hash handling contract

Hash implementation is authority-sensitive. The implementation must make the following separation explicit:

| Operation | Rust backend | TypeScript | Database | Test requirement |
|---|---|---|---|---|
| Parse submitted Act | allowed | allowed for draft only | stores verified JSON | malformed cases |
| Canonicalize bytes | authoritative only if profile implementation is approved | forbidden as authority | stores canonical bytes | golden vectors |
| Compute tuple hash | authoritative in approved Rust/hash module | display only after server result | stores verified value | tuple vectors |
| Compute content hash | authoritative in approved Rust/hash module | display only after server result | unique verified value | content vectors |
| Compute envelope hash | authoritative in approved Rust/hash module | display only after server result | verified value | envelope vectors |
| Compare hashes | allowed | allowed for UX warning only | allowed constraints | mismatch tests |

Hash failure behavior:

1. If `canonical_bytes` and `content_hash` disagree, reject insert.
2. If existing stored bytes later fail verification, create an incident and ghost; do not mutate the Act row.
3. If TypeScript displays a preview hash, label it `non_authoritative_preview`.
4. If two different byte sequences produce the same semantic proposal before admission, treat as open question unless canon covers equivalence.
5. If canonicalization profile is unavailable, block semantic submission and emit operational error/ghost rather than inventing a local profile.

### 21.6 Act Store operational contract

The Act Store must provide these primitives:

| Primitive | Input | Output | Authority note |
|---|---|---|---|
| `verify_act_bytes` | canonical bytes + expected hashes | verification result | hash profile authority required |
| `insert_admitted_act` | admitted Act envelope from bridge | content hash | append-only only |
| `get_by_content_hash` | content hash | canonical bytes + JSON + envelope | hash verified on read or marked invalid |
| `scan_timeline` | cursor/order profile | ordered Act refs | read model, not projection truth |
| `list_since` | content hash/time cursor | Act refs | for projection rebuild |
| `record_incident` | failure classification | operational record/proposal | not semantic closure |

It must not provide:

- `update_act_status`;
- `delete_act`;
- `mark_done`;
- `upsert_candidate_as_workorder`;
- `set_registry_state`;
- `close_receipt_from_worker_result`.

If operational code asks for these forbidden methods, the design has drifted into an entity backend and must be rejected.

### 21.7 Constitutional Runtime Bridge contract

The bridge to the real Constitutional Runtime is not an optional plugin. It is the admission boundary.

Required request record:

```json
{
  "request_kind": "constitutional_admission",
  "proposal_payload": {},
  "source": "minilab.work/write|app|worker|ruler|api",
  "submitted_at": "RFC3339 timestamp",
  "caller": "operational principal",
  "profile": "declared compatibility profile"
}
```

Required response classification:

| Outcome | Runtime meaning | Local handling |
|---|---|---|
| `admitted` | Proposal became Act under real runtime | verify hashes, append to Act Store |
| `rejected` | Proposal cannot be admitted | return rejection; optionally preserve ghost if runtime/canon says so |
| `doubt` | Runtime preserves uncertainty | append/submit doubt Act as provided |
| `needs_evidence` | Evidence insufficient | produce traceable non-closure path |
| `needs_authority` | Authority missing | block semantic progression and show required authority |
| `runtime_error` | Bridge failed | no local admission fallback; incident/ghost |

Bridge logs are operational artifacts. They help debug admission, but they are not admitted Acts unless the real runtime returns/adopts them as such.

### 21.8 Engine Bridge contract

The Engine Bridge must be narrow and auditable.

Required inputs:

- source Act hashes;
- due observation Act hash when triggered by Ruler;
- declared engine version/profile;
- loadout/pack context if allowed by canon/runtime;
- caller and operational trace id.

Required outputs:

- admitted output Acts or proposal path as returned by the real Engine;
- explicit no-op/doubt/blocked outcome if no dispatch is justified;
- bridge receipt/log with engine version and source hashes;
- ghost if Engine is unavailable and work cannot be evaluated.

Forbidden bridge behavior:

- synthesizing dispatch Acts locally;
- treating a scheduler tick as queue item;
- retrying in a way that creates duplicate semantic Acts without idempotency rules;
- hiding Engine unavailability behind a UI “pending” state without ghost/incident trace.

### 21.9 Ruler contract

The Ruler is an observer of Earth time, not a scheduler with semantic authority.

Ruler invariants:

| Invariant | Meaning | Test |
|---|---|---|
| R-001 | Tick may observe due-ness only | tick output contains observation/proposal, not worker lease | ruler unit test |
| R-002 | Tick must route through Engine | Engine Bridge called before queue projection changes | integration test |
| R-003 | Ruler never waits for worker | no worker future/join dependency in Ruler | static/architecture test |
| R-004 | Ruler never closes receipt | no receipt close capability | capability test |
| R-005 | Ruler emits explicit ghost on ambiguous due state | ambiguous time rules produce non-closure | scenario test |

Ruler input examples:

- current Earth time;
- timezone profile;
- timeline cursor;
- Acts with due semantics as understood by Engine/canon;
- loadout policies that are explicitly subordinate.

Ruler output examples:

- due observation proposal;
- no-due observation;
- ambiguity ghost proposal;
- operational metrics.

### 21.10 Dispatch and queue projection contract

A queue is a projection of dispatchable Acts. It is not the source of dispatch truth.

Allowed queue columns:

- `source_dispatch_act_hash`;
- `work_scope_hash`;
- `projection_version`;
- `lease_owner`;
- `lease_until`;
- `attempt_count`;
- `last_operational_error`;
- `created_from_projection_at`.

Forbidden queue columns as truth:

- `semantic_status`;
- `done`;
- `receipt_closed`;
- `truth_state`;
- `canonical_workorder_id`;
- `admitted_by_worker`.

Operational leases may update because they coordinate execution. A lease timeout is not semantic failure; it may cause a new observation/proposal/ghost path through the runtime.

### 21.11 Worker contract

Workers are bounded executors. They must be easy to kill, retry, sandbox and audit.

Worker input:

- lease id;
- source dispatch Act hash;
- bounded tool/capability set;
- evidence profile requirement;
- time/resource limits;
- blob write policy;
- reporting schema.

Worker output:

- evidence/report proposal;
- blob refs with digests;
- operational logs;
- exit code;
- resource usage;
- failure reason.

Worker must never output:

- admitted Act;
- closed receipt;
- canonical truth update;
- queue mutation not mediated by dispatch host;
- untraceable model claim as evidence.

Worker report contract:

```json
{
  "kind": "worker_report",
  "source_dispatch_act_hash": "sha256:...",
  "worker_id": "local-runner-1",
  "result": "succeeded|failed|partial|blocked",
  "evidence_refs": [
    {
      "blob_hash": "sha256:...",
      "media_type": "application/json",
      "evidence_profile": "profile-id"
    }
  ],
  "proposed_act": {},
  "receipt_closed": false
}
```

The literal `receipt_closed: false` is included in examples to prevent accidental closure semantics. Real schemas may omit it if canon/runtime defines a better negative assertion.

### 21.12 Evidence and blob contract

Evidence is not “whatever the model said.” Evidence must have provenance.

| Evidence kind | Allowed? | Requirements | Notes |
|---|---|---|---|
| File digest | yes | hash, size, media type, capture source | good for local/filesystem apps |
| API response | yes | endpoint, timestamp, auth scope, response digest | store sensitive payloads safely |
| Command output | yes | command, environment, exit code, stdout/stderr digests | stdout alone is not closure |
| Human attestation | yes if canon/runtime admits | signer/identity/scope | may need authority |
| Model answer | not by itself | must be labeled suggestion/report | target `model_output_as_evidence = 0` |
| Screenshot | yes as blob | digest, capture context | projection/UI evidence only if admitted |
| External URL | weak alone | retrieval time, digest if fetched | URL rot must be ghostable |

Blob references should be content-addressed where possible. If external storage cannot guarantee content addressing, the Act must include a digest and retrieval/provenance information.

### 21.13 Receipt review contract

Receipt review is a semantic boundary. It must answer:

1. What scope is being closed?
2. Which Acts created the obligation or dispatch?
3. Which evidence/report Acts support closure?
4. Which authority is allowed to close?
5. What remains outside the closure scope?
6. What ghosts/doubts are preserved?

Receipt review output options:

| Review outcome | Required artifact |
|---|---|
| Close scope | Receipt Act admitted by proper authority |
| Partial close | Scoped Receipt Act + Ghost for remainder |
| Insufficient evidence | Ghost/doubt/needs evidence Act |
| Wrong authority | needs authority outcome/ghost |
| Worker failure | report Act + Ghost/blocking Act |
| Ambiguous scope | no closure; scope clarification proposal |

Copy rule: never show “done” from worker success. Show “evidence received” until a receipt Act exists.

### 21.14 Ghost preservation contract

Ghost is not failure decoration. It is the explicit durable preservation of absence, doubt, blockage or non-closure.

Ghost triggers:

- missing evidence;
- missing authority;
- Engine unavailable;
- Constitutional Runtime unavailable;
- hash profile unavailable;
- projection cannot trace source Acts;
- app output cannot be verified;
- worker timeout with unclear result;
- external system unavailable;
- timeline ordering conflict;
- canon alignment gap.

Ghost minimum fields as implementation expectation:

```json
{
  "kind": "ghost_or_doubt_proposal",
  "source_act_hashes": ["sha256:..."],
  "reason_code": "missing_evidence|needs_authority|engine_unavailable|canon_alignment_required",
  "human_readable_reason": "short explanation",
  "blocked_operation": "receipt_review|dispatch|projection|export|admission",
  "required_next_evidence": [],
  "projection_is_truth": false
}
```

The exact schema requires canon/runtime alignment. Until verified, this remains an implementation expectation and must be marked in the alignment register.

### 21.15 Projection rebuild contract

Every projection must define:

- projection name;
- source Act selection rule;
- ordering rule;
- reduction rule;
- conflict handling;
- ghost handling;
- source hash emission;
- rebuild command;
- golden output fixtures.

Projection rebuild algorithm template:

```text
1. Read projection profile and compatibility declaration.
2. Start from empty projection state.
3. Scan Acts using declared timeline ordering profile.
4. For each Act, decide whether it is relevant.
5. Apply reduction rule without mutating Act Store.
6. Preserve conflicts as projection conflicts or Ghost proposals.
7. Emit projection rows with source_act_hashes.
8. Compare against golden fixtures in tests.
9. Expose API response with projection_is_truth=false.
```

Projection drift rule: if a projection cannot be rebuilt to match current stored projection rows, the stored projection is wrong. The timeline is not wrong merely because projection drift exists.

### 21.16 Registry projection details

Registry is a projection of Acts that appear to register, update, observe or deprecate things according to canon/runtime and loadout conventions.

Registry projection must expose:

- item display key;
- item kind as projection label;
- current projected view;
- source Act hashes;
- unresolved conflicts;
- ghosts affecting the item;
- projection profile version.

Registry projection must not expose:

- canonical entity id not derived from Act content;
- mutable `current_state` as truth;
- hidden merge decisions;
- edits that bypass Act proposal.

### 21.17 Workorder projection details

Workorder is a projection of dispatchable Acts.

A workorder projection row may contain:

- dispatch Act hash;
- source obligation Act hashes;
- projected priority;
- projected due time;
- lease status;
- worker capability requirement;
- evidence profile requirement;
- current operational lease holder;
- last report Act hash;
- receipt/ghost source hash if reviewed.

It must not contain:

- final completion truth;
- canonical task identity independent of Acts;
- worker-created status as semantic state.

### 21.18 Timeline details

Timeline is an ordered reading of Acts. It is not a separate entity.

Timeline API should declare:

- ordering profile;
- cursor type;
- whether ordering is total or partial;
- conflict behavior;
- source store range;
- verification status.

If ordering is uncertain, the API must say so. It must not silently impose a UI order and call it canonical.

### 21.19 Machine and workflow details

Machine and Workflow are especially dangerous because they invite parallel ontology.

Allowed interpretation:

- `Machine` is a projection over Acts that register capabilities, observations, reports and evidence about execution environments.
- `Workflow` is a projection/convention over Acts and policies that describe expected transitions, not a separate truth engine.

Forbidden interpretation:

- `Machine` as canonical mutable resource object with semantic status.
- `Workflow` as hidden state machine that closes or dispatches without Engine/Constitutional Runtime.

### 21.20 Experience Runtime details

The Experience Runtime coordinates human flows without semantic authority.

Allowed responsibilities:

- preserve draft text;
- assemble proposal payloads;
- show missing fields;
- display admission outcome;
- route to trace views;
- create operational notifications;
- subscribe to projection/event updates;
- explain why a closure is not available.

Forbidden responsibilities:

- mark a task done;
- admit a proposal because form validation passed;
- close receipt after optimistic UI success;
- hide ghosts to simplify onboarding;
- call worker directly to bypass dispatch.

### 21.21 minilab.work surface specification

`minilab.work` should expose a small set of surfaces. These are experience names, not canonical objects.

| Surface | Primary question | Backing source | Failure mode to avoid |
|---|---|---|---|
| Today | What is due/observed now? | Ruler/Engine-derived projections | dashboard without trace |
| Write | What signal should enter the metabolism? | proposal builder + Constitutional Runtime | form pretending to create truth |
| Timeline Trace | Why does the system believe/display this? | ordered Acts and source hashes | opaque activity feed |
| Workbench | What bounded work can be executed? | queue projection + worker leases | task manager semantics |
| Receipts | What closed, by which Act, with what scope? | Receipt Acts/projection | “done” labels from reports |
| Ghosts | What is unresolved, absent, blocked or doubtful? | Ghost/doubt Acts/projection | hiding uncertainty |
| Registry | What does the Lab currently project as known? | registry projection | canonical entity browser |
| Runtime | What are Engine/Ruler/Bridge health and incidents? | operational telemetry + Acts where admitted | health page as semantic proof |

### 21.22 UX microcopy contract

The copy system must encode authority.

| Bad copy | Replacement | Reason |
|---|---|---|
| “Task created” | “Proposal submitted” or “Dispatch projected” | task creation implies entity truth |
| “AI decided” | “Model suggested” | model has no authority |
| “Done” | “Receipt closed by Act …” | completion requires receipt |
| “Synced” | “Projection rebuilt” | sync can imply truth transfer |
| “Scheduled” | “Due observation will be submitted to Engine” | scheduler must not bypass Engine |
| “Approved automatically” | “Admitted by Constitutional Runtime” | authority must be named |
| “No issues” | “No ghosts projected from current source Acts” | absence of UI issues is not proof |
| “Saved” | “Draft saved” or “Act admitted” | distinguish draft from semantic truth |

Every user-facing state badge should have a trace affordance. If no trace exists, the badge must not imply semantic state.

### 21.23 Onboarding metabolism script

Onboarding should be a live passage through authority boundaries:

1. User writes a small intention in Write.
2. UI shows it as proposal, not created task.
3. Constitutional Runtime outcome is displayed.
4. If admitted, Timeline Trace shows content hash and tuple/content/envelope hashes.
5. User sees Today as projection, not dashboard.
6. Ruler tick is explained as observation.
7. Engine dispatch is shown as source of work projection.
8. Worker report is shown as evidence, not closure.
9. Receipt review either closes scope or preserves Ghost.
10. User exports or views projection with `projection_is_truth=false`.

If a demo skips one of these boundaries, it must label the skipped part as mocked, ghosted or not yet conformance-proven.

### 21.24 API detailed endpoint contracts

`POST /acts/propose`:

- validates shape only as implementation helper;
- returns proposal id with no semantic authority;
- may include missing fields and warnings;
- must not write `logline_acts` unless proposal capture itself is admitted through proper path.

`POST /acts/submit`:

- calls Constitutional Runtime Bridge;
- returns admitted Act hash only if real runtime admits;
- returns rejection/doubt/needs evidence/needs authority explicitly;
- must include bridge trace id.

`GET /acts/{content_hash}`:

- verifies hash on read or includes verification failure;
- returns canonical bytes or stable representation;
- must not accept row id as substitute.

`GET /timeline`:

- includes ordering profile;
- includes cursor and verification status;
- may include projection hints but must distinguish them from Acts.

`GET /projections/{name}`:

- always returns `projection_is_truth=false`;
- includes projection version;
- includes source Act hashes or marks incomplete projection.

`POST /ruler/tick`:

- records/initiates observation;
- routes through Engine;
- returns no direct queue mutation as semantic result.

`GET /events`:

- streams operational events;
- marks which events correspond to Acts;
- does not turn websocket events into truth.

`POST /experience/{action}`:

- runs UI flow;
- returns proposal/evidence/operational result;
- never claims truth-changing result without Act hash.

### 21.25 Error response contract

Errors must preserve authority boundaries.

```json
{
  "error": {
    "code": "needs_evidence",
    "authority": "constitutional-runtime",
    "semantic_truth_changed": false,
    "source_act_hashes": [],
    "ghost_required": true,
    "message": "Evidence is insufficient for closure."
  }
}
```

Error codes should distinguish:

- malformed request;
- missing authority;
- missing evidence;
- hash mismatch;
- engine unavailable;
- constitutional runtime unavailable;
- projection stale;
- app boundary violation;
- worker boundary violation;
- UI copy/trace violation.

### 21.26 Database tables by jurisdiction

| Table | Mutable? | Semantic authority? | Reason |
|---|---:|---:|---|
| `logline_acts` | no | yes, as storage of admitted Acts | append-only canonical store |
| `act_ingest_attempts` | yes | no | operational bridge/debug state |
| `projection_cursors` | yes | no | rebuild coordination |
| `work_queue_projection` | yes/rebuildable | no | dispatch view + leases |
| `worker_leases` | yes | no | execution coordination |
| `blob_objects` | append preferred | no alone | evidence storage by reference |
| `app_tokens` | yes | no | integration auth |
| `ui_sessions` | yes | no | session state |
| `incidents` | yes/append preferred | no unless admitted | operational governance |
| `pack_registry` | yes | no | loadout configuration |
| `projection_exports` | yes/append preferred | no | interoperability artifacts |

Any migration adding a table must include a jurisdiction comment: why it exists, whether it is rebuildable, whether it is mutable and why it cannot be semantic truth.

### 21.27 Migration review checklist

Before merging a migration:

- Does it create or mutate semantic state outside `logline_acts`?
- Does it include append-only protections where required?
- Does every projection table include source Act hash references?
- Does every mutable table have an operational reason?
- Does any column name imply canonical truth incorrectly?
- Can the projection be rebuilt from Acts?
- Are row ids prevented from leaking as semantic ids?
- Are hash fields constrained and indexed appropriately?
- Is incident behavior defined for mismatch or drift?

### 21.28 Repository ownership expanded

Recommended ownership by path:

| Path | Owner | Review requirement |
|---|---|---|
| `standards/authority.md` | authority maintainers | required for every authority change |
| `standards/hash-profile.md` | hash/conformance maintainers | requires golden vector update |
| `kernel/crates/act-store` | kernel maintainers | requires append-only/hash tests |
| `kernel/crates/engine-bridge` | engine bridge maintainers | requires real engine contract tests |
| `kernel/crates/constitutional-bridge` | admission maintainers | requires runtime contract tests |
| `kernel/crates/ruler` | runtime maintainers | requires no-bypass tests |
| `kernel/crates/worker-host` | execution maintainers | requires no-closure tests |
| `loadables/packs/*` | pack maintainers | requires pack conformance |
| `loadables/apps/*` | app maintainers | requires app authority tests |
| `loadables/projection-profiles/*` | projection maintainers | requires golden exports |
| `ui/minilab-work` | UX maintainers | requires trace/copy tests |

### 21.29 Pack validation rules

Pack loader must reject a pack if:

- manifest has no compatibility declaration;
- manifest declares canon override;
- manifest declares hash profile override;
- manifest adds admission behavior;
- manifest adds receipt closure behavior;
- manifest hides ghosts from views;
- manifest defines projection without source Act hash requirements;
- pack tests are missing for authority-sensitive declarations.

Pack loader may warn if:

- pack uses experimental canon compatibility range;
- pack depends on app not installed;
- pack declares UX copy that needs translation review;
- pack projection has no golden fixtures;
- pack policy convention is ambiguous.

### 21.30 Santo André pack expanded example

Santo André as a pack/loadout may include:

```yaml
id: santo-andre
kind: logline.pack
version: 0.2.0
status: experimental-loadout
compatibility:
  logline_canon: "declared-range-required"
  engine_main: "declared-version-required"
  constitutional_runtime_main: "declared-version-required"
  conformance_suite: "declared-suite-required"
views:
  - today
  - timeline_trace
  - ghosts
  - receipts
copy_rules:
  - no_false_done
  - no_ai_decided
  - receipt_hash_required_for_closure
projections:
  - registry_projection
  - workorder_projection
  - lab_runtime_projection
forbidden:
  defines_canon: false
  overrides_hash_profile: false
  bypasses_engine: false
  bypasses_constitutional_runtime: false
```

This example is intentionally loadout-scoped. It must not appear in root standards as a protocol requirement.

### 21.31 App sandbox rules

Apps need capability boundaries.

| Capability | Default | Reason |
|---|---|---|
| Read external resource | allowed by auth scope | required for observation |
| Write external resource | explicit opt-in | may create real-world effects |
| Emit proposal | allowed | not semantic truth |
| Emit evidence/blob ref | allowed with profile | supports review |
| Emit admitted Act | forbidden | admission boundary |
| Close receipt | forbidden | closure boundary |
| Trigger worker | forbidden unless through host/dispatch | prevents engine bypass |
| Store secrets | allowed in app token store | operational only |
| Compute authoritative hash | forbidden | hash boundary |

Apps that interact with external systems must declare whether they cause external side effects. Side effects must be represented as evidence/report/proposal paths and may require human or constitutional authority.

### 21.32 Local LLM app boundary

A local LLM app is allowed to draft, summarize, classify, suggest and map. It is not allowed to decide truth.

Allowed local LLM outputs:

- draft Act proposal;
- suggested evidence extraction;
- summary with source Act hashes;
- classification suggestion;
- UI explanation;
- risk note;
- ghost suggestion.

Forbidden local LLM outputs:

- admitted Act;
- receipt closure;
- authoritative evidence without source;
- untraceable claim;
- policy override;
- hidden dispatch decision.

All LLM output that informs evidence must be marked as model output and must not count as evidence by itself. This enforces `model_output_as_evidence = 0`.

### 21.33 Projection profile validation rules

A projection profile must fail validation if:

- it omits `projection_is_truth=false`;
- it omits source Act hash requirements;
- it maps external identifiers as canonical LogLine ids;
- it cannot explain lossiness;
- it hides unresolved ghosts;
- it exports closure without Receipt Act source;
- it exports workflow execution without Engine-derived source.

A projection profile should include:

- lossiness declaration;
- target standard version;
- mapping table;
- source hash strategy;
- generated timestamp;
- profile version;
- golden fixture output;
- validation command.

### 21.34 Scientific interoperability mapping notes

Optional standard profiles must be treated as translations:

| Standard | Useful for | Non-truth warning |
|---|---|---|
| PROV | provenance graph | graph nodes are projection of Acts |
| RO-Crate | research object packaging | crate metadata is export view |
| DataCite | dataset citation metadata | DOI metadata is not timeline |
| DCAT | data catalog interoperability | catalog entries are projection rows |
| JATS | article-like scientific publication | article text is not receipt |
| CWL | workflow description/execution interop | workflow run must still trace to Acts |
| FAIR profiles | discoverability/reuse | FAIR metadata is not semantic authority |

Every exported file should contain a header or metadata field equivalent to:

```json
{
  "projection_is_truth": false,
  "source_system": "logline-lab-kit",
  "source_act_hashes": ["sha256:..."],
  "profile_version": "..."
}
```

### 21.35 Test scenarios expanded

Hash tests:

- tuple hash stable under irrelevant JSON key order;
- content hash changes when semantic content changes;
- content hash ignores only approved self-referential fields;
- envelope hash changes when transport envelope changes;
- invalid canonical bytes fail closed;
- TypeScript preview hash cannot be accepted as authoritative.

Constitutional Runtime tests:

- admitted response stores Act;
- rejected response does not store Act as admitted;
- doubt response is visible in Timeline Trace;
- needs evidence blocks receipt closure;
- runtime unavailable produces no local admission;
- bridge logs cannot be used as semantic proof.

Engine tests:

- due observation routes through Engine;
- no dispatch when Engine says no;
- dispatch Act source hashes appear in queue projection;
- Engine unavailable does not create queue item;
- duplicate Engine response is idempotent by content hash or ghosted.

Ruler tests:

- tick does not wait on worker;
- tick does not write queue projection directly;
- timezone ambiguity produces ghost/doubt;
- repeated tick does not create false duplicate work;
- Ruler output is traceable.

Worker tests:

- worker cannot call receipt close API;
- worker output marked evidence/report;
- worker timeout does not become done;
- worker blob refs include digests;
- model-generated report is not counted as evidence alone.

Projection tests:

- rebuild from empty state matches stored projection;
- projection includes source Act hashes;
- projection marks stale/incomplete state;
- projection conflict is not hidden;
- export contains `projection_is_truth=false`.

UI tests:

- no banned copy appears in closure contexts;
- every status badge has trace path;
- Write says proposal before admission;
- Today explains Ruler/Engine source;
- Timeline Trace shows content hash;
- onboarding passes through at least one ghost/receipt explanation.

### 21.36 Release gate checklist

A release cannot proceed unless:

- all hard targets equal zero;
- hash golden vectors pass;
- append-only migration tests pass;
- Engine Bridge tests pass against declared engine version or are ghosted as unsupported;
- Constitutional Runtime Bridge tests pass against declared runtime version or are ghosted as unsupported;
- pack/app/projection manifests validate;
- UI copy tests pass;
- projection rebuild tests pass;
- incident classes are documented;
- compatibility declarations are updated.

If a release ships with a known gap, the gap must be explicit in release notes as ghost/open question/implementation risk/conformance required. It must not be hidden as roadmap language.

### 21.37 Implementation issue templates

Authority-relevant implementation issues should use this template:

```markdown
## Authority impact
- [ ] Canon alignment required
- [ ] Engine behavior affected
- [ ] Constitutional Runtime behavior affected
- [ ] Hash/canonicalization affected
- [ ] Projection only
- [ ] UI only

## Source Acts / test artifacts
- Source Act hashes:
- Conformance tests:
- Ghosts/open questions:

## Forbidden shortcuts checked
- [ ] No UI truth
- [ ] No worker receipt closure
- [ ] No scheduler bypass
- [ ] No app admitted Act
- [ ] No projection truth claim
```

### 21.38 Incident response playbooks

`hash_mismatch`:

1. Stop accepting affected submissions.
2. Preserve raw bytes and verification failure.
3. Do not mutate `logline_acts`.
4. Create incident artifact and ghost if semantic uncertainty remains.
5. Add golden test before reopening.

`engine_bypass`:

1. Disable bypassing path.
2. Identify queue/projection rows created without Engine source.
3. Mark them invalid operationally.
4. Submit ghost/incident path for affected semantic claims.
5. Add no-bypass regression test.

`false_closure`:

1. Find UI/app/worker/projection claim that implied closure.
2. Verify whether Receipt Act exists.
3. If not, retract projection/copy and preserve ghost.
4. Add copy/authority test.
5. Review worker/app capabilities.

`projection_truth_claim`:

1. Remove truth language from projection/export.
2. Add `projection_is_truth=false`.
3. Add source Act hashes.
4. Add golden export test.
5. Review profile manifest.

### 21.39 Documentation quality bar

Future documentation changes must be judged by these criteria:

- Does the first page state authority status?
- Does it distinguish canon from runtime doctrine?
- Does every noun have jurisdiction?
- Does every workflow identify the admission boundary?
- Does every execution path identify Engine and worker boundaries?
- Does every closure path require receipt?
- Does every uncertainty path preserve ghost?
- Does every projection/export deny truth claim?
- Does every example remain minimal but complete?
- Does any paragraph sneak in new ontology?

### 21.40 Extended canonical example with traces

This example is still illustrative and subordinate. It is longer than the short example because implementers need a traceable path.

Scenario: a user wants to run a local analysis on `dataset-x` tomorrow at 09:00.

1. The user writes the sentence in `minilab.work/Write`.
2. TypeScript stores a draft and asks the backend to build a proposal.
3. The proposal response says `semantic_truth_changed=false`.
4. The user submits the proposal.
5. The API calls the Constitutional Runtime Bridge.
6. If the real runtime admits the Act, the backend verifies tuple/content/envelope hashes.
7. The Act Store inserts canonical bytes into `logline_acts` append-only.
8. Timeline Trace displays `content_hash`, `tuple_hash`, `envelope_hash`, source, profile and admission outcome.
9. Today does not show “scheduled task”; it shows projected due information with trace.
10. At 09:00, Ruler observes due-ness.
11. Ruler submits due observation to Engine.
12. Engine returns dispatch decision or non-dispatch/ghost.
13. Dispatch Act, if admitted, appears in timeline.
14. Queue projection rebuilds and shows dispatchable bounded work.
15. Worker host leases the projection row.
16. Worker runs with bounded capabilities.
17. Worker writes blob evidence with digest.
18. Worker submits evidence/report proposal.
19. Constitutional Runtime admission is applied to that evidence/report Act if required.
20. Receipt review evaluates evidence against scope.
21. If sufficient, a scoped Receipt Act closes exactly that scope.
22. If partial, Receipt closes the proven scope and Ghost preserves the remainder.
23. If insufficient, Ghost/needs evidence preserves non-closure.
24. Registry/Today/Receipts/Ghosts projections rebuild.
25. Any PROV/RO-Crate/DataCite export includes source Act hashes and `projection_is_truth=false`.

At no point does UI state, worker success, queue lease or app output become semantic truth by itself.


### 21.41 Phase work breakdown

The phase list in section 17 is the release spine. This annex turns each phase into implementable work.

#### Phase 0 — jurisdiction and authority files

Deliverables:

- `standards/authority.md` with the layer hierarchy;
- `standards/alignment-register.md` with canon/engine/runtime open questions;
- `CODEOWNERS` with authority-sensitive paths;
- `docs/operations/incidents.md` with incident classes;
- root README warning that the repo is not new canon.

Acceptance criteria:

- every authority-sensitive path has an owner;
- every known canon gap has a register entry;
- every implementation package declares whether it is kernel, host, loadable, projection or UI;
- no Santo André convention is placed in root standards as protocol.

Ghosts to create if unresolved:

- missing canon citation for 9-slot profile;
- missing hash profile vectors;
- missing real Engine invocation contract;
- missing real Constitutional Runtime invocation contract.

#### Phase 1 — Act Store and hash profile

Deliverables:

- append-only `logline_acts` migration;
- hash verification crate/module;
- golden vector fixtures;
- read API by `content_hash`;
- incident path for hash mismatch.

Acceptance criteria:

- insert requires verified tuple/content/envelope hashes;
- update/delete on Act rows is blocked;
- `canonical_bytes` are stored and retrievable;
- `act_json` is verified against canonical bytes;
- row id is never returned as semantic id.

Implementation risks:

- accidentally trusting JSONB normalization;
- using a different canonicalization library in tests and runtime without proof;
- allowing TypeScript preview hashes into persistence.

#### Phase 2 — Engine Bridge

Deliverables:

- bridge adapter to real `engine-main`;
- request/response trace format;
- no-direct-dispatch tests;
- Engine unavailable ghost path;
- idempotency behavior for repeated Engine outputs.

Acceptance criteria:

- dispatch projection changes only after Engine-derived source;
- no scheduler/ruler path writes queue directly;
- Engine version/profile is recorded;
- unsupported Engine outcome creates explicit incident/ghost.

Implementation risks:

- mocking Engine in production;
- interpreting Engine logs as admitted Acts;
- creating fallback dispatch logic in Rust host.

#### Phase 3 — Constitutional Runtime Bridge

Deliverables:

- bridge adapter to real `constitutional-runtime-main`;
- admission outcome enum;
- submit API wired to bridge;
- bridge tests for admitted/rejected/doubt/needs evidence/needs authority;
- runtime unavailable incident path.

Acceptance criteria:

- no admitted Act can be inserted without bridge outcome;
- rejected proposal cannot appear as admitted Act;
- doubt and needs-evidence remain visible to UI;
- local validation is only advisory.

Implementation risks:

- treating schema validation as admission;
- using app credentials as semantic authority;
- hiding runtime errors as “pending”.

#### Phase 4 — Ruler

Deliverables:

- time observation loop;
- tick endpoint;
- timezone/clock configuration;
- Ruler-to-Engine integration;
- ambiguity ghost tests.

Acceptance criteria:

- Ruler cannot create queue row without Engine;
- Ruler cannot close receipt;
- Ruler cannot wait for worker;
- repeated ticks are idempotent or explicitly ghosted.

Implementation risks:

- building a scheduler instead of a Ruler;
- treating cron expressions as truth;
- hiding timezone ambiguity.

#### Phase 5 — Projections

Deliverables:

- projection framework;
- timeline, registry, workorder, today, receipt and ghost projections;
- rebuild command;
- projection cursors;
- golden projection fixtures.

Acceptance criteria:

- every projection row has source Act hashes;
- projection API returns `projection_is_truth=false`;
- rebuild from empty state matches golden fixture;
- conflicts and missing sources are visible.

Implementation risks:

- denormalized projection becoming de facto truth;
- UI reading projection row id as canonical id;
- untraceable export rows.

#### Phase 6 — Queue and Worker

Deliverables:

- queue projection table;
- worker lease table;
- worker host capability boundary;
- blob/evidence write path;
- report proposal path.

Acceptance criteria:

- worker has no receipt close capability;
- worker report includes source dispatch hash;
- blob refs include digest and profile;
- worker failure never becomes semantic closure.

Implementation risks:

- letting worker update workorder status;
- treating exit code 0 as receipt;
- using model text as evidence.

#### Phase 7 — Receipt/Ghost

Deliverables:

- receipt review flow;
- closure submission path through proper authority;
- Ghost/needs-evidence/needs-authority paths;
- UI trace surfaces for receipts and ghosts;
- false-closure tests.

Acceptance criteria:

- every closure displayed has Receipt Act hash;
- partial closure preserves remainder;
- insufficient evidence cannot produce green done state;
- Ghosts are queryable and visible.

Implementation risks:

- product pressure to hide ghosts;
- broad receipts closing more than their scope;
- ambiguous evidence accepted as sufficient.

#### Phase 8 — Loadout / Packs

Deliverables:

- pack schema;
- pack loader;
- Santo André pack example;
- pack conformance tests;
- compatibility declaration UI/logging.

Acceptance criteria:

- pack cannot override canon/hash/admission/engine;
- pack projections require source hashes;
- pack copy rules pass UX authority tests;
- pack version compatibility is explicit.

Implementation risks:

- Santo André conventions leaking into kernel;
- pack policies becoming hidden canon;
- pack examples becoming required protocol behavior.

#### Phase 9 — App Host

Deliverables:

- app schema;
- app sandbox/capability model;
- auth/token storage;
- tool/resource/event adapters;
- act mapper/evidence profile validation;
- app authority tests.

Acceptance criteria:

- app cannot emit admitted Act;
- app cannot close receipt;
- app side effects are declared;
- app output is proposal/observation/evidence/blob/operational only.

Implementation risks:

- connector webhooks creating truth directly;
- OAuth identity confused with semantic authority;
- app mapper bypassing Constitutional Runtime.

#### Phase 10 — Projection Profiles

Deliverables:

- projection profile schema;
- profile loader;
- PROV/RO-Crate/DataCite examples if useful;
- golden export tests;
- lossiness declarations.

Acceptance criteria:

- every export includes `projection_is_truth=false`;
- every exported assertion has source Act hashes where possible;
- closure exports require Receipt Act source;
- profile version is recorded.

Implementation risks:

- scientific export perceived as truth replacement;
- lossy mapping hidden;
- external identifiers used as canonical Act ids.

#### Phase 11 — minilab.work Experience

Deliverables:

- Today surface;
- Write surface;
- Timeline Trace;
- Receipts/Ghosts views;
- onboarding metabolism;
- copy test suite.

Acceptance criteria:

- Write shows proposal before admission;
- Today shows Ruler/Engine trace;
- Timeline Trace shows content hashes;
- no banned closure/AI-decision copy;
- every status badge has trace.

Implementation risks:

- dashboard language replacing metabolism;
- feature tour replacing onboarding through authority;
- optimistic UI implying truth.

#### Phase 12 — conformance/release hardening

Deliverables:

- CI gates for hard targets;
- release compatibility report;
- incident playbook tests;
- conformance artifacts;
- release notes with ghosts/open questions.

Acceptance criteria:

- all hard targets are zero;
- unsupported paths are explicit ghosts;
- release cannot ship with silent authority bypass;
- compatibility ranges are accurate.

Implementation risks:

- treating failing conformance as warning;
- shipping with mocked Engine/Runtime as if real;
- untracked authority-relevant changes.

### 21.42 Boundary anti-pattern catalog

The implementation must reject these anti-patterns early.

| Anti-pattern | Why it is wrong | Correct shape |
|---|---|---|
| `tasks` table as source of work truth | creates CRUD task manager | workorder projection from dispatch Acts |
| `status = done` from worker | false closure | worker report + receipt review |
| cron writes queue | scheduler bypasses Engine | Ruler observation → Engine → dispatch projection |
| UI optimistic admission | UI claims truth | proposal pending admission |
| app webhook inserts Act | app gains authority | webhook creates proposal/observation |
| projection export without hashes | untraceable interoperability | include source Act hashes |
| local hash library in frontend | client becomes authority | server returns verified hashes |
| registry edit form updates row | registry becomes entity DB | form proposes Act |
| pack defines 9 slots | pack redefines canon | pack declares compatibility only |
| model answer stored as evidence | model replaces proof | model output labeled suggestion/report |

### 21.43 Language and naming guidance

Names carry authority. Prefer names that reveal projection status.

Allowed names:

- `workorder_projection`;
- `registry_projection`;
- `receipt_projection`;
- `ghost_projection`;
- `proposal_drafts`;
- `worker_leases`;
- `projection_cursors`;
- `app_observations`;
- `blob_refs`;
- `experience_events`.

Names to avoid unless explicitly scoped:

- `tasks`;
- `entities`;
- `truth_state`;
- `canonical_registry_items`;
- `machine_status` as semantic status;
- `workflow_state` as independent state;
- `ai_decisions`;
- `done_items`;
- `approved_by_ui`.

When a convenient product name is needed, pair it with a jurisdiction note in code comments and docs.

### 21.44 Minimal code comments required for authority-sensitive modules

Each authority-sensitive module should start with a comment like:

```text
Jurisdiction: projection only.
This module derives a view from admitted Acts. It must not admit Acts,
close receipts, compute authoritative hashes, or call workers directly.
If this module cannot trace a row to source Act hashes, it must mark the
projection incomplete or create an explicit ghost path.
```

Recommended jurisdiction labels:

- `canon-pointer-only`;
- `hash-authoritative`;
- `admission-bridge`;
- `engine-bridge`;
- `projection-only`;
- `worker-boundary`;
- `experience-only`;
- `app-connector`;
- `pack-loadable`;
- `operational-storage`.

### 21.45 CI checks

Recommended CI jobs:

| Job | Command shape | Blocks release? |
|---|---|---:|
| hash golden vectors | `cargo test -p hash-verify` | yes |
| append-only migration | `cargo test -p act-store append_only` | yes |
| bridge contracts | `cargo test -p engine-bridge -p constitutional-bridge` | yes |
| ruler no-bypass | `cargo test -p ruler no_bypass` | yes |
| worker capabilities | `cargo test -p worker-host no_receipt_close` | yes |
| projection rebuild | `cargo test -p projections rebuild_golden` | yes |
| pack schema | `labctl validate packs` | yes |
| app schema | `labctl validate apps` | yes |
| projection profile exports | `labctl export --golden` | yes |
| UI copy/trace | `pnpm test:authority-copy` | yes |
| docs jurisdiction | `scripts/check-doc-authority` | yes for authority docs |

Until these commands exist, their absence should be tracked as implementation risk, not silently ignored.

### 21.46 Compatibility declaration format

Each release should publish a compatibility declaration:

```yaml
release: 0.1.0
kind: logline-lab-kit-release
compatibility:
  logline_canon: "declared-range"
  hash_profile: "declared-profile"
  conformance_suite: "declared-suite"
  engine_main: "declared-version-or-commit"
  constitutional_runtime_main: "declared-version-or-commit"
components:
  kernel: "0.1.0"
  host: "0.1.0"
  ui_minilab_work: "0.1.0"
loadouts:
  santo_andre: "0.2.0"
known_ghosts:
  - id: CA-001
    reason: "Exact 9-slot profile pending canon citation"
hard_targets:
  false_closure_rate: 0
  worker_closed_receipt: 0
  app_admitted_act: 0
  projection_claimed_canon: 0
```

This declaration is not marketing. It is a release artifact for governance and conformance.

### 21.47 Minimal local development modes

Development mode may use mocks only if the UI and logs make authority status impossible to confuse.

| Mode | Allowed | Required label |
|---|---|---|
| `mock-ui-only` | static screens, no truth | `no runtime authority` |
| `mock-engine` | development dispatch fixtures | `engine not real; no conformance` |
| `mock-constitutional` | form flow testing | `admission not real` |
| `projection-fixture` | projection UI tests | `fixture projection` |
| `full-runtime` | real bridges and conformance | `runtime authority enabled` |

A demo using mocks must never say “admitted,” “closed,” or “done” without adding “mock” or “fixture” to the visible state.

### 21.48 Security and secrets boundary

Security implementation must not become semantic authority.

- Authentication identifies callers; it does not admit Acts.
- Authorization permits operations; it does not close receipts.
- OAuth scopes permit app access; they do not create evidence sufficiency.
- Secret storage protects credentials; it does not validate truth.
- Audit logs help governance; they are not semantic timeline unless admitted as Acts.

Authority-sensitive security events may produce Act proposals or incidents, but the security subsystem must not bypass the same admission rules as other components.

### 21.49 Observability boundary

Metrics and logs are operational observations.

Allowed metrics:

- admission latency;
- Engine bridge latency;
- projection rebuild lag;
- worker lease counts;
- ghost counts by reason;
- hash mismatch incidents;
- app boundary violations;
- UI trace coverage.

Forbidden interpretation:

- metric `success_count` as receipt closure;
- log line as admitted Act;
- dashboard green state as proof;
- absence of error logs as evidence.

Observability should help find ghosts; it should not hide them.

### 21.50 Preservation note

This expanded version intentionally keeps the consolidated structure while restoring operational density. The goal is not to preserve every repeated paragraph from the source. The goal is to preserve the invention, jurisdiction boundaries, implementation guidance, failure modes, tests, contracts and governance surface that were lost when the prior rewrite became too short.

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
