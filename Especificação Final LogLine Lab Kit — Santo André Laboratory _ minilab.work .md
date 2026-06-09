According to a document from 2026-06-08, a versão correta não começa por menu, SaaS, CRUD ou scheduler: ela começa por um ciclo vivo de `sinal humano → admissão constitucional → Act → timeline → régua → Engine → fila → worker → evidência → receipt/ghost → projeção`. Os documentos também insistem que o menu, a UI, os registros e os relatórios são projeções desse metabolismo, não fontes de verdade.

# **Especificação Final — Santo André Laboratory / minilab.work**

## **Rust backend, TypeScript periferia, LogLine Act como única verdade**

Vou ser direto e cuidadoso: a versão boa precisa respeitar a tua invenção, não “domesticar” ela num backend genérico. O projeto não é um app com entidades chamadas Candidate, Workorder, Receipt, Ghost, Machine, Policy. O projeto é um **runtime institucional em Rust onde a única unidade semântica durável é o LogLine Act content-addressed**.

Tudo o resto é projeção.

There is only LogLine Act as durable semantic truth.  
Everything else is projection, index, cache, queue, blob, view, or operational materialization.

A frase em português:

Só existe LogLine Act como verdade semântica durável.  
Todo o resto é projeção, índice, cache, fila, blob, view ou materialização operacional.

Isso corrige o erro das versões anteriores: eu estava escorregando para criar entidades auxiliares. A versão final não faz isso.

---

# **1\. Decisão soberana: tudo é LogLine Act**

A forma atual que eu adotaria é:

LogLine Act  
\= unidade semântica pública e canônica.

9-slot tuple  
\= núcleo obrigatório dentro do Act.

Receipt  
\= Act de fechamento escopado, não entidade separada.

Ghost / blocked / doubt  
\= Act que preserva ausência, dúvida, bloqueio ou não-fechamento.

Workorder  
\= Act cujo \`did/status/this\` projeta uma fila de execução.

Evidence / report  
\= Act ou blob referenciado por Act, nunca closure sozinho.

Registry  
\= projeção de Acts.

Timeline  
\= leitura ordenada dos Acts, não entidade separada.

UI  
\= projeção de Acts.

DB row  
\= armazenamento, não autoridade.

A pesquisa documental do Google Drive já apontava a tensão real: os papers antigos sustentam `tuple` como forma forte, enquanto ATLAS/Registry apontam para `Act` como direção atual; os 9 slots aparecem como núcleo estável, e runtime, banco, worker ou UI não são fonte de verdade semântica.

A hierarquia fica:

Act  
└── contains canonical 9-slot tuple  
    └── can emit / reference result, evidence, transport, receipt  
        └── is addressed by content\_hash

Não há uma tabela canônica de `candidates`.  
Há Acts com `status: "candidate"` ou `did: "proposed"`.

Não há uma tabela canônica de `workorders`.  
Há Acts com `did: "requested_execution"` ou `status: "queued"`.

Não há uma tabela canônica de `receipts`.  
Há Acts com `did: "closed_receipt"` ou `status: "closed"`.

Não há uma tabela canônica de `ghosts`.  
Há Acts com `status: "ghost"`, `status: "blocked"` ou `did: "preserved_absence"`.

Não há uma tabela canônica de `machines`.  
Há Acts que registram, atualizam, observam e provam máquinas; `Machines` é projeção.

---

# **2\. Os três hashes**

O LogLine Act precisa ter três hashes, e eles não são detalhes cosméticos. Eles são a espinha da implementação.

tuple\_hash  
content\_hash  
envelope\_hash

## **2.1 `tuple_hash`**

Hash do núcleo semântico obrigatório:

who  
did  
this  
when  
confirmed\_by  
if\_ok  
if\_doubt  
if\_not  
status

Função:

provar a identidade semântica mínima do Act.

Ele responde:

Esta ação canônica, sem transporte e sem envelope, é qual?

## **2.2 `content_hash`**

Hash do conteúdo canônico completo do LogLine Act, excluindo campos autorreferenciais como `id` e `hashes`.

Função:

ser o endereço do Act.

A regra:

act\_address \= logline:act:sha256:\<content\_hash\>

Ou, internamente:

act:\<content\_hash\>

Esse é o ID real. Não UUID. Não serial. Não row id.

Um `row_id` pode existir no banco para performance, mas ele não tem autoridade semântica.

## **2.3 `envelope_hash`**

Hash do envelope de transporte, recepção, assinatura, origem, headers, ingestão ou empacotamento, excluindo o próprio `envelope_hash`.

Função:

provar como este Act chegou / foi embrulhado / foi recebido.

## **2.4 Fórmula**

tuple\_hash \=  
  sha256(canonicalize(extract\_9\_slots(act)))

content\_hash \=  
  sha256(canonicalize(act\_without\_id\_and\_hashes))

envelope\_hash \=  
  sha256(canonicalize(envelope\_without\_envelope\_hash))

O `engine-main` que eu li no archive já aponta exatamente nessa direção: ele tem `tuple_hash`, `content_hash` e `compute_envelope_hash`; o `tuple_hash` filtra os 9 slots, o `content_hash` remove `id` e `hashes`, e o envelope hash remove `envelope_hash`. Também vi que o código usa SHA-256 e declara `"json_canonicalization": "jcs-rfc8785"` em trechos do recibo triádico. A ressalva técnica é importante: a canonicalização precisa ser testada byte-for-byte contra o perfil oficial, porque o código atual implementa uma canonicalização manual.

Para perfil externo, RFC 8785/JCS é uma boa âncora porque define uma representação canônica de JSON com restrição a I-JSON e ordenação determinística de propriedades, produzindo uma representação “hashable” para uso criptográfico. ([RFC Editor](https://www.rfc-editor.org/info/rfc8785/?utm_source=chatgpt.com))

---

# **3\. Leitura correta dos engines**

A implementação final não deve criar “um engine novo”.

O archive tem dois corpos reais:

engine-main  
constitutional-runtime-main

A spec final precisa tratá-los como dependências, não como inspiração.

## **3.1 `engine-main`**

O `engine-main` é um workspace Rust com os nove slots e um shell:

who  
did  
this  
when  
confirmed\_by  
if\_ok  
if\_doubt  
if\_not  
status  
logline-cli

Ele diz a frase certa:

canon defines.  
conformance proves.  
engine implements.  
governance evolves.

Isso é a fronteira correta.

Canon define.  
Conformance prova.  
Engine implementa.  
Governance evolui.

O Engine não é soberano; ele implementa comportamento admissível.

O walk real do Engine faz:

validate\_shape  
→ is\_prohibited?  
→ validate\_against\_canon  
→ confirmed\_by::pivot  
→ selected\_branch  
→ status::transition  
→ resolve all 9 slots  
→ RuntimeLogLine

O `confirmed_by` é o Evidence Pivot: ele parseia `receipt`, `digest`, `signature`, `quorum`, tokens e missing markers; se há evidência suficiente, vai para `Ok`; se falta, tende a `Doubt`; se proibido pelo canon, vai para `Not`.

Isso conversa com os documentos do projeto: o Engine aplica os 9 slots, usa `confirmed_by` como ponto onde a evidência colapsa em `Ok / Doubt / Not`, e não deve virar worker, esperar job, fechar receipt ou mutar timeline in-place.

## **3.2 `constitutional-runtime-main`**

O `constitutional-runtime-main` também é real. Ele tem módulos de:

ir  
policy  
capability  
admission  
evidence  
lowering  
planning\_compiler  
decision  
validation  
act\_identity  
idempotency  
failure  
ingress  
strong\_grammar  
plan\_executor

O README dele diz que execução material precisa ser:

semantically admissible  
policy-permitted  
capability-realizable  
evidentially accountable

Isso é exatamente a execução vertical.

A leitura final:

Constitutional Runtime  
\= vertical.  
Transforma intenção, linguagem, humanos, LLMs, offices e processos  
em candidatos LogLine admissíveis.

LogLine Engine  
\= horizontal.  
Move Acts no tempo, valida transição, escolhe branch, emite movimento semântico.

labd / Santo André Runtime  
\= corpo operacional.  
Guarda Acts, roda régua, projeta estado, mantém fila, executa workers e mostra UI.

Os documentos já tinham a frase certa: o Constitutional Runtime pergunta “isso pode virar um Act?”, a Régua pergunta “o que ficou devido agora?”, e o Engine pergunta “essa transição é admissível?”. Isso evita LLM virar autoridade, scheduler virar runtime semântico e worker virar juiz.

---

# **4\. A metáfora correta: Engine sobre régua transparente**

A tua metáfora é a melhor descrição do LAB:

O LogLine Engine fica montado sobre uma régua transparente.  
A régua se arrasta sobre a timeline, que é o próprio tempo da Terra.  
Ela não faz nada mágico.  
Ela só passa pelo tempo e pergunta:  
“há algum LogLine Act devido neste ponto?”

Quando há um Act devido:

1\. A Régua observa.  
2\. A observação vira um Act.  
3\. Esse Act passa pelo Engine.  
4\. O Engine admite/rejeita/ghosta a transição.  
5\. Se houver trabalho, o Engine emite Act de dispatch.  
6\. A projeção de fila pega esse Act.  
7\. O worker executa quando puder.  
8\. A Régua continua andando.

A Régua não pode esperar o worker.

Se esperar, ela dessincroniza do tempo da Terra.

A Régua arrasta.  
O Engine admite o movimento.  
A fila segura a consequência.  
O worker toca a realidade.  
A evidência volta.  
Outro Act fecha ou preserva a dúvida.

Essa doutrina aparece nos documentos como separação horizontal/vertical: o horizontal é `Earth time → ruler → timeline intersection → due Act → engine walk → state transition → queue`; o vertical é `human/LLM intent → operator session → process → candidate → admission → authority → dispatch`.

A frase final:

O LogLine Engine é o runtime horizontal da LogLine:  
uma régua não-bloqueante sobre o tempo da Terra que percorre a timeline,  
detecta Acts devidos, admite movimento de estado e emite trabalho despachável  
sem jamais esperar a execução.

E a complementar:

O Constitutional Runtime é o runtime vertical da LogLine:  
o caminho pelo qual intenção, linguagem, humanos, LLMs, offices e processos  
se tornam LogLine Acts admissíveis.

---

# **5\. Arquitetura de implementação**

## **5.1 Stack**

Backend:

Rust  
Tokio  
Axum  
SQLx  
PostgreSQL

Periferia:

TypeScript  
React / Next ou shell equivalente  
OpenAPI-generated client  
SSE/WebSocket para timeline/projeções  
Adapters em TS apenas para captura e superfície

Justificativa externa: Tokio é o runtime async em Rust para I/O, networking, scheduling e timers; Axum é uma biblioteca de routing e request handling focada em modularidade; SQLx é uma toolkit async para SQL; PostgreSQL fornece JSONB, colunas geradas e mecanismos como `FOR UPDATE SKIP LOCKED` para consumidores de fila. ([Tokio](https://tokio.rs/?utm_source=chatgpt.com))

Mas a regra é:

Rust não define canon.  
Rust não inventa Engine.  
Postgres não é autoridade.  
TypeScript não é autoridade.

O Rust mantém o corpo operacional.

O Engine real move os Acts.

O Constitutional Runtime real admite o vertical.

O TypeScript propõe, mostra e adapta.

---

# **6\. Estrutura de repo**

santo-andre-lab-runtime/  
  Cargo.toml

  foundation/  
    engine-main/                         \# submodule / vendored readonly  
    constitutional-runtime-main/          \# submodule / vendored readonly  
    canon-main/                           \# se disponível  
    conformance-main/                     \# se disponível

  crates/  
    labd-core/  
    labd-act-store/  
    labd-engine-bridge/  
    labd-constitutional-bridge/  
    labd-ruler/  
    labd-projections/  
    labd-dispatch/  
    labd-workers/  
    labd-api/  
    labd-cli/  
    labd-conformance/

  apps/  
    labd/

  migrations/  
    0001\_logline\_acts.sql  
    0002\_projections.sql  
    0003\_queue\_projection.sql

  packs/  
    santo-andre/  
      pack.yaml  
      routines/  
      experiments/  
      projections/  
      receipt\_profiles/  
      ruler\_doctrine.md

  ts/  
    packages/  
      api-client/  
      act-forms/  
      projection-widgets/  
      adapters/  
    apps/  
      minilab-work/

Regra de dependência:

labd-engine-bridge  
→ chama engine-main

labd-constitutional-bridge  
→ chama constitutional-runtime-main

labd-act-store  
→ só armazena Acts canônicos e seus hashes

labd-ruler  
→ só observa tempo e submete Acts ao Engine

labd-dispatch  
→ só materializa fila a partir de Acts admitidos

labd-workers  
→ só executa work derivado de Acts e retorna report/evidence como Act

labd-projections  
→ reconstrói views apagáveis a partir de logline\_acts

TypeScript  
→ lê projeções, propõe Acts, nunca decide verdade

---

# **7\. Banco de dados: uma tabela canônica**

A versão correta não tem tabelas canônicas separadas para `candidates`, `workorders`, `receipts`, `ghosts`, `machines`, `policies`.

Tem uma tabela canônica:

logline\_acts

E várias projeções apagáveis:

projection\_today  
projection\_timeline  
projection\_due  
projection\_queue  
projection\_receipts  
projection\_ghosts  
projection\_registry  
projection\_machines  
projection\_workflows  
projection\_policies

## **7.1 Tabela canônica**

create table logline\_acts (  
  content\_hash text primary key,

  tuple\_hash text not null,  
  envelope\_hash text not null,

  hash\_algorithm text not null default 'sha256',  
  canonicalization\_profile text not null,

  canonical\_bytes bytea not null,  
  act\_json jsonb not null,  
  envelope\_json jsonb not null,

  received\_at timestamptz not null default now(),

  who text generated always as (act\_json-\>\>'who') stored,  
  did text generated always as (act\_json-\>\>'did') stored,  
  "when" text generated always as (act\_json-\>\>'when') stored,  
  status text generated always as (act\_json-\>\>'status') stored,

  this\_json jsonb generated always as (act\_json-\>'this') stored,  
  confirmed\_by\_json jsonb generated always as (act\_json-\>'confirmed\_by') stored,  
  if\_ok\_json jsonb generated always as (act\_json-\>'if\_ok') stored,  
  if\_doubt\_json jsonb generated always as (act\_json-\>'if\_doubt') stored,  
  if\_not\_json jsonb generated always as (act\_json-\>'if\_not') stored,

  check (content\_hash \<\> ''),  
  check (tuple\_hash \<\> ''),  
  check (envelope\_hash \<\> '')  
);

A razão para guardar `canonical_bytes` é essencial: PostgreSQL `jsonb` é ótimo para consulta, mas não preserva whitespace, ordem de chaves ou duplicatas como o texto JSON original; por isso ele é cache consultável, não fonte de bytes para hash. ([PostgreSQL](https://www.postgresql.org/docs/current/datatype-json.html?utm_source=chatgpt.com))

As colunas geradas são só índice. PostgreSQL documenta colunas geradas como valores computados a partir de outras colunas, armazenados ou virtuais; aqui elas ajudam busca por `who`, `did`, `when` e `status`, mas não mandam na semântica. ([PostgreSQL](https://www.postgresql.org/docs/current/ddl-generated-columns.html?utm_source=chatgpt.com))

## **7.2 Proibição de update semântico**

create function forbid\_logline\_acts\_update()  
returns trigger language plpgsql as $$  
begin  
  raise exception 'logline\_acts is append-only; emit another LogLine Act';  
end;  
$$;

create trigger trg\_forbid\_logline\_acts\_update  
before update or delete on logline\_acts  
for each row execute function forbid\_logline\_acts\_update();

A regra:

Nunca UPDATE para mudar estado.  
Sempre INSERT de novo LogLine Act.

## **7.3 Blobs**

Evidência bruta, stdout, imagem, arquivo, snapshot, prompt, resposta de modelo, relatório longo e artefatos binários podem ficar em uma tabela/blobs:

create table blob\_objects (  
  blob\_hash text primary key,  
  algorithm text not null default 'sha256',  
  media\_type text,  
  size\_bytes bigint not null,  
  bytes bytea,  
  storage\_uri text,  
  created\_at timestamptz not null default now()  
);

Mas blob não é verdade semântica.

A verdade é o Act que referencia:

blob:sha256:\<hash\>

---

# **8\. Projeções**

Projeção é reconstruível.

create table projection\_queue (  
  content\_hash text primary key references logline\_acts(content\_hash),  
  queue\_name text not null,  
  worker\_kind text not null,  
  status text not null,  
  not\_before timestamptz,  
  claimed\_by text,  
  claimed\_at timestamptz,  
  updated\_at timestamptz not null default now()  
);

Essa tabela pode ter update operacional, porque não é verdade. Ela serve para execução eficiente.

Quando um worker claimar uma linha, isso pode mudar `projection_queue.claimed_at`, mas a verdade semântica só existe quando o sistema emitir um Act:

who: worker:readonly-probe  
did: claimed\_execution  
this:  
  work: act:\<content\_hash\>  
when: \<now\>  
confirmed\_by:  
  queue\_claim: projection\_queue:\<content\_hash\>  
if\_ok: run\_worker  
if\_doubt: release\_claim  
if\_not: preserve\_claim\_failure  
status: claimed

Para múltiplos workers, PostgreSQL `FOR UPDATE SKIP LOCKED` é adequado para uma tabela de fila porque pula linhas já travadas por outros consumidores; a própria documentação observa que isso dá uma visão inconsistente e não serve para uso geral, mas é útil para evitar contenção em tabelas tipo fila. ([PostgreSQL](https://www.postgresql.org/docs/current/sql-select.html?utm_source=chatgpt.com))

---

# **9\. Rust: crates e responsabilidades**

## **9.1 `labd-act-store`**

Única função: append e leitura de Acts.

pub struct StoredAct {  
    pub content\_hash: String,  
    pub tuple\_hash: String,  
    pub envelope\_hash: String,  
    pub canonical\_bytes: Vec\<u8\>,  
    pub act\_json: serde\_json::Value,  
    pub envelope\_json: serde\_json::Value,  
}

\#\[async\_trait::async\_trait\]  
pub trait ActStore {  
    async fn append(\&self, envelope: LogLineActEnvelope) \-\> Result\<StoredAct, StoreError\>;  
    async fn get(\&self, content\_hash: \&str) \-\> Result\<Option\<StoredAct\>, StoreError\>;  
    async fn scan\_since(\&self, cursor: ActCursor) \-\> Result\<Vec\<StoredAct\>, StoreError\>;  
}

Proibido:

append\_candidate()  
append\_workorder()  
append\_receipt()  
append\_ghost()

Permitido:

append(LogLineActEnvelope)

Porque tudo é Act.

## **9.2 `labd-engine-bridge`**

Não reimplementa o walk.

Ele chama o `engine-main`.

\#\[async\_trait::async\_trait\]  
pub trait LogLineEnginePort {  
    async fn run\_act(  
        \&self,  
        act: LogLineActEnvelope,  
        operation: EngineOperation,  
        evidence: Vec\<String\>,  
    ) \-\> Result\<EngineRunOutput, EngineBridgeError\>;  
}

pub struct EngineRunOutput {  
    pub runtime\_logline: serde\_json::Value,  
    pub emitted\_acts: Vec\<LogLineActEnvelope\>,  
}

Implementações possíveis:

PathDependencyEngine  
CliEngine  
SidecarEngine

Ordem preferida:

1\. path dependency / crate  
2\. local binary \`logline\`  
3\. sidecar process

Proibido:

reescrever confirmed\_by::pivot  
reescrever walk dos 9 slots  
reescrever status::transition  
reescrever tuple/content/envelope hash

## **9.3 `labd-constitutional-bridge`**

Também não reimplementa admissão.

Ele chama o `constitutional-runtime-main`.

\#\[async\_trait::async\_trait\]  
pub trait ConstitutionalRuntimePort {  
    async fn evaluate(  
        \&self,  
        input: ConstitutionalInput,  
    ) \-\> Result\<ConstitutionalOutput, ConstitutionalError\>;  
}

Entrada:

pub struct ConstitutionalInput {  
    pub source\_act: LogLineActEnvelope,  
    pub operator\_context: serde\_json::Value,  
    pub boundary: serde\_json::Value,  
    pub policy\_context: serde\_json::Value,  
    pub capability\_context: serde\_json::Value,  
}

Saída:

pub enum ConstitutionalOutput {  
    Admit { emitted\_acts: Vec\<LogLineActEnvelope\> },  
    Reject { emitted\_acts: Vec\<LogLineActEnvelope\> },  
    Doubt { emitted\_acts: Vec\<LogLineActEnvelope\> },  
    NeedsAuthority { emitted\_acts: Vec\<LogLineActEnvelope\> },  
    NeedsEvidence { emitted\_acts: Vec\<LogLineActEnvelope\> },  
}

Cada saída vira Act.

Nada vira “row status” canônico.

## **9.4 `labd-ruler`**

A Régua é um loop não-bloqueante.

pub struct Ruler {  
    clock: Arc\<dyn Clock\>,  
    store: Arc\<dyn ActStore\>,  
    engine: Arc\<dyn LogLineEnginePort\>,  
    projections: Arc\<dyn ProjectionReadModel\>,  
}

impl Ruler {  
    pub async fn tick(\&self) \-\> Result\<TickSummary, RulerError\> {  
        let now \= self.clock.now();

        let due\_acts \= self.projections.due\_at(now).await?;

        for due in due\_acts {  
            let observed\_due \= build\_observed\_due\_act(\&due, now);

            let output \= self.engine  
                .run\_act(observed\_due, EngineOperation::Confirmar, vec\!\[\])  
                .await?;

            for emitted in output.emitted\_acts {  
                self.store.append(emitted).await?;  
            }  
        }

        Ok(TickSummary { now })  
    }  
}

Crimes proibidos:

Ruler executar worker  
Ruler esperar worker  
Ruler enfileirar sem Engine  
Ruler fechar receipt  
Ruler alterar Act antigo

Os documentos já colocavam isso: a Régua olha `when`, schedules, recurrence, due/overdue, blocked, pendências, ghosts e receipts aguardando evidência; ela não decide sem Engine, e cada observação relevante vira transição submetida ao Engine.

## **9.5 `labd-projections`**

Projeções são apagáveis e reconstruíveis.

pub trait Projection {  
    fn name(\&self) \-\> &'static str;  
    fn apply(\&mut self, act: \&StoredAct) \-\> Result\<Vec\<ProjectionUpdate\>, ProjectionError\>;  
}

Projeções iniciais:

projection\_timeline  
projection\_today  
projection\_due  
projection\_queue  
projection\_receipts  
projection\_ghosts  
projection\_registry  
projection\_machines  
projection\_workflows  
projection\_policies

Regra:

Se uma projeção for apagada, ela deve poder ser reconstruída de logline\_acts.  
Se não puder, ela virou verdade escondida e está errada.

## **9.6 `labd-dispatch`**

Dispatch é projeção \+ protocolo operacional.

Não é verdade.

Ele lê Acts projetados como trabalho:

who: logline.engine  
did: requested\_execution  
this:  
  worker\_kind: backup\_probe  
  target: local\_backup\_status  
  source\_act: act:\<hash\>  
when: \<T\>  
confirmed\_by:  
  engine\_transition: act:\<hash\>  
if\_ok: queue\_for\_worker  
if\_doubt: ghost\_dispatch\_uncertain  
if\_not: block\_execution  
status: queued

A fila materializa:

projection\_queue

Quando o worker pega, reporta de volta como Act.

## **9.7 `labd-workers`**

Workers são músculo.

Eles não governam.

\#\[async\_trait::async\_trait\]  
pub trait Worker {  
    fn kind(\&self) \-\> &'static str;

    async fn execute(  
        \&self,  
        work: ProjectedWork,  
    ) \-\> Result\<WorkerObservation, WorkerError\>;  
}

A observação volta como Act:

who: worker:readonly-probe  
did: reported\_execution  
this:  
  work: act:\<work\_hash\>  
  report:  
    exit\_code: 0  
    stdout: blob:sha256:\<hash\>  
    artifacts:  
      \- blob:sha256:\<hash\>  
when: \<now\>  
confirmed\_by:  
  worker\_report\_hash: sha256:\<hash\>  
if\_ok: submit\_to\_receipt\_review  
if\_doubt: preserve\_inconclusive\_evidence  
if\_not: open\_failure\_path  
status: evidence\_returned

O worker não fecha receipt.

Os documentos repetem essa fronteira: worker/report/evidence apoiam closure, mas não são closure; Projection renderiza estado, não cria verdade; e os hard targets incluem `worker_closed_receipt = 0` e `projection_claimed_canon = 0`.

---

# **10\. API**

A API pública deve evitar CRUD semântico.

Ela não deve parecer:

POST /workorders  
PATCH /workorders/:id/done  
POST /receipts  
PATCH /machines/:id

Isso cria a ontologia errada.

Ela deve parecer:

POST /v1/acts/propose  
POST /v1/acts/submit  
GET  /v1/acts/{content\_hash}  
GET  /v1/timeline  
GET  /v1/projections/today  
GET  /v1/projections/queue  
GET  /v1/projections/registry  
POST /v1/ruler/tick  
GET  /v1/events

## **10.1 `POST /v1/acts/propose`**

Entrada de humano/TS/adapter.

Produz um Act com status de proposta/candidate:

who: dan  
did: proposed  
this:  
  raw\_text: blob:sha256:\<hash\>  
  proposed\_shape:  
    did: verify\_backup  
when: \<now\>  
confirmed\_by:  
  source: human\_note  
if\_ok: submit\_to\_constitutional\_runtime  
if\_doubt: ask\_for\_missing\_slots  
if\_not: reject\_unbounded\_claim  
status: candidate

## **10.2 `POST /v1/acts/submit`**

Submete um Act para admissão vertical ou transição horizontal.

{  
  "act": { "...": "..." },  
  "envelope": {  
    "source": "minilab.work/write",  
    "operator\_session": "...",  
    "transport": "http"  
  }  
}

Resposta:

{  
  "content\_hash": "sha256:...",  
  "tuple\_hash": "sha256:...",  
  "envelope\_hash": "sha256:...",  
  "emitted\_acts": \["sha256:...", "sha256:..."\]  
}

## **10.3 `GET /v1/projections/today`**

Retorna view, não verdade.

{  
  "projection": "today",  
  "source\_acts": \["sha256:...", "sha256:..."\],  
  "staleness": "fresh",  
  "value": {  
    "due": \[\],  
    "queued": \[\],  
    "ghosts": \[\],  
    "receipts": \[\]  
  }  
}

---

# **11\. TypeScript: periferia**

TypeScript não pode ser uma segunda ontologia.

Ele pode:

mostrar projection  
montar formulários  
capturar texto  
chamar API  
receber SSE  
renderizar timeline  
fazer preview local de Act  
mostrar hashes retornados  
mostrar diferença entre projeção e fonte  
fazer adapters de browser/Drive/GitHub/etc.

Ele não pode:

admitir Act  
fechar receipt  
decidir branch  
gerar hash autoritativo  
reconstruir canon local divergente  
marcar workorder como done  
resolver ghost  
atualizar registry como verdade

Estrutura:

ts/  
  packages/  
    api-client/  
      generated/  
      client.ts

    act-forms/  
      NewActForm.tsx  
      NineSlotEditor.tsx  
      EvidencePicker.tsx

    projection-widgets/  
      TodayView.tsx  
      TimelineView.tsx  
      QueueView.tsx  
      ReceiptView.tsx  
      GhostView.tsx  
      RegistryView.tsx

    adapters/  
      browser-capture/  
      local-files/  
      google-drive/  
      github/

  apps/  
    minilab-work/  
      routes/  
        core/  
        capture/  
        registry/  
        intelligence/  
        workbench/  
        runtime/  
        institutional/  
        admin/

A regra de build:

TypeScript types são gerados de OpenAPI/JSON Schema do Rust.  
Não existe \`type Act \= ...\` escrito à mão no frontend.

---

# **12\. Menu final como projeção do Lab**

O menu fica como passaporte visual do Lab, não como ontologia nativa.

minilab.work  
LogLine Automation  
LAB-256 · workbench · safe

\+ New Act

core  
\- LAB  
\- Today  
\- Timeline  
\- Lab Routine  
\- Workorders  
\- Receipts  
\- Ghosts

capture  
\- Write  
\- Candidates  
\- Inbox  
\- Review

registry  
\- Registry  
\- People  
\- Organizations  
\- Machines  
\- Apps  
\- Workflows  
\- Policies  
\- Gates

intelligence  
\- Knowledge  
\- Docs  
\- Memory  
\- Research  
\- Benchmarks  
\- Findings

workbench  
\- Code  
\- Reviews  
\- Experiments  
\- Proof  
\- Conformance

runtime  
\- Runtimes  
\- Engines  
\- Agents  
\- LLMs  
\- Sensors  
\- Dispatch  
\- Schedules

institutional  
\- Financeiro  
\- Costs  
\- Vendors  
\- Legal  
\- Clients  
\- Analytics

admin  
\- Secrets  
\- Connections  
\- Environments  
\- Limits  
\- Settings

Os docs já traziam essa versão como a primeira que “parece Lab de verdade”, porque corrige três buracos: falta de entrada suja/candidate flow, falta de Timeline como autoridade histórica e runtime/dispatch/schedules misturados em admin.

Mas a versão final corrige a linguagem:

Candidates  
\= projeção de Acts com status candidate.

Workorders  
\= projeção de Acts dispatcháveis.

Receipts  
\= projeção de Acts de fechamento.

Ghosts  
\= projeção de Acts de dúvida/ausência/bloqueio.

Registry  
\= projeção de Acts de registro/mutação/observação.

Machines  
\= projeção de Acts sobre máquinas.

Policies/Gates  
\= projeção de Acts de autoridade.

---

# **13\. Fluxo vivo final**

## **13.1 Entrada vertical**

Humano escreve:

amanhã às 10h verificar backup local

TS envia texto.

Rust cria/projeta um Act de proposta:

who: dan  
did: proposed  
this:  
  raw\_text: blob:sha256:\<raw\>  
  intent: verify\_local\_backup  
when: 2026-06-09T09:00:00+01:00  
confirmed\_by:  
  source: minilab.write  
if\_ok: submit\_to\_constitutional\_runtime  
if\_doubt: ask\_for\_missing\_slots  
if\_not: reject\_unbounded\_claim  
status: candidate

Constitutional Runtime avalia.

Se admitido, emite:

who: constitutional-runtime  
did: admitted\_act  
this:  
  candidate: act:\<candidate\_hash\>  
  admitted\_as:  
    who: santo-andre.lab  
    did: verify  
    this:  
      target: local\_backup\_status  
    when: 2026-06-10T10:00:00+01:00  
    confirmed\_by:  
      required: receipt:backup\_probe  
    if\_ok: close\_backup\_receipt  
    if\_doubt: preserve\_backup\_uncertainty  
    if\_not: open\_backup\_failure\_path  
    status: scheduled  
when: \<now\>  
confirmed\_by:  
  admission\_ruling: act:\<hash\>  
if\_ok: append\_to\_timeline  
if\_doubt: preserve\_admission\_doubt  
if\_not: reject\_candidate  
status: admitted

## **13.2 Régua horizontal**

Quando chega o horário:

who: logline.ruler  
did: observed\_due  
this:  
  act: act:\<scheduled\_backup\_hash\>  
  due\_at: 2026-06-10T10:00:00+01:00  
when: 2026-06-10T10:00:00+01:00  
confirmed\_by:  
  clock\_tick: act:\<tick\_hash\>  
if\_ok: submit\_to\_engine\_for\_dispatch  
if\_doubt: preserve\_due\_uncertainty  
if\_not: keep\_unmoved  
status: due\_observed

Esse Act passa pelo Engine.

O Engine emite:

who: logline.engine  
did: requested\_execution  
this:  
  source\_act: act:\<scheduled\_backup\_hash\>  
  trigger: act:\<observed\_due\_hash\>  
  worker\_kind: backup\_probe  
  allowed\_actions:  
    \- read\_backup\_metadata  
    \- stat\_path  
  forbidden\_actions:  
    \- delete\_file  
    \- mutate\_backup  
    \- exfiltrate\_secret  
when: \<now\>  
confirmed\_by:  
  engine\_walk: act:\<runtime\_logline\_hash\>  
if\_ok: queue\_for\_worker  
if\_doubt: ghost\_dispatch\_uncertain  
if\_not: block\_execution  
status: queued

A projeção de fila pega esse Act.

A Régua continua andando.

## **13.3 Worker**

Worker executa quando conseguir.

Depois volta:

who: worker:backup-probe  
did: reported\_execution  
this:  
  work: act:\<requested\_execution\_hash\>  
  result:  
    backup\_found: true  
    last\_backup\_at: 2026-06-10T09:58:00+01:00  
    stdout: blob:sha256:\<hash\>  
when: \<now\>  
confirmed\_by:  
  report\_hash: sha256:\<hash\>  
if\_ok: submit\_to\_receipt\_review  
if\_doubt: preserve\_inconclusive\_report  
if\_not: open\_execution\_failure  
status: evidence\_returned

## **13.4 Receipt**

Receipt reviewer emite Act:

who: receipt-reviewer  
did: closed\_receipt  
this:  
  claim: act:\<scheduled\_backup\_hash\>  
  evidence:  
    \- act:\<worker\_report\_hash\>  
  scope: local\_backup\_status\_at\_due\_time  
when: \<now\>  
confirmed\_by:  
  evidence\_sufficient: true  
  conformance\_profile: logline.receipt.v0  
if\_ok: close\_scope  
if\_doubt: preserve\_receipt\_doubt  
if\_not: reject\_overclaim  
status: closed

Se não fecha:

who: receipt-reviewer  
did: preserved\_absence  
this:  
  claim: act:\<scheduled\_backup\_hash\>  
  missing:  
    \- required\_backup\_probe\_evidence  
when: \<now\>  
confirmed\_by:  
  review: act:\<review\_hash\>  
if\_ok: carry\_ghost  
if\_doubt: ask\_human  
if\_not: mark\_false\_claim  
status: ghost

Tudo é Act.

A UI mostra `Backup check: closed` ou `ghost`, mas isso é projeção da cadeia.

Os documentos já expressavam isso como `Truth = append-only chain` e `UI state = projection`.

---

# **14\. O Lab como relógio experimental**

Aqui entra a parte mais tua, e ela não deve ser arquivada pela higiene do canon.

O canon mínimo é o esqueleto.

O Lab é o organismo.

A parte mais importante não é “auditoria”; é que a tupla/Act vira aparelho de observação. Os documentos do chat já capturavam isso: a parte viva pergunta que tipo de sistema aparece quando todo fenômeno precisa confessar `who did this when confirmed_by...`; a segunda metade de LogLine não é só Protocol Core, mas Research Fire — tuple as discovery apparatus, workflows as discovered structure, projections as observatories, hardware as eventful text.

A tese do Santo André Laboratory:

Computadores velhos.  
LLMs locais fracos.  
Uma pessoa.  
Mas 24 horas por dia, todo dia, de graça.

O Lab usa tempo como infraestrutura.

Fórmula:

weak compute  
\+ strong timeline  
\+ daily ticks  
\+ evidence discipline  
\= compounding laboratory

A função do Lab:

rodar microexperimentos ao longo do dia,  
registrar tudo como Acts,  
voltar com evidência,  
fechar receipts ou preservar ghosts,  
e deixar a timeline revelar padrões.

Experimentos iniciais:

machine.heartbeat  
backup.probe  
peer\_link\_check  
local\_llm\_quality\_tick  
prompt\_regression  
file\_integrity\_probe  
receipt\_conformance\_probe  
timeline\_projection\_rebuild  
worker\_failure\_replay

Cada experimento tem:

scheduled Act  
due observation Act  
engine transition Act  
execution request Act  
worker report Act  
receipt/ghost Act  
projection update

O Lab não precisa de GPU para ser interessante.

Ele precisa de relógio, timeline e disciplina.

---

# **15\. Ordem de implementação**

## **Fase 0 — Fixar jurisdição**

Antes de código:

LOG\_LINE\_CURRENT.md  
HASH\_PROFILE.md  
ENGINE\_BOUNDARY.md  
CONSTITUTIONAL\_RUNTIME\_BOUNDARY.md  
HORIZONTAL\_RUNTIME\_DOCTRINE.md  
PROJECTION\_DISCIPLINE.md

Decisões:

1\. Só LogLine Act é verdade semântica durável.  
2\. Act contém 9-slot tuple.  
3\. content\_hash é endereço.  
4\. tuple\_hash, content\_hash, envelope\_hash são obrigatórios.  
5\. Todas as mudanças de estado são novos Acts.  
6\. Projeções podem ser apagadas.  
7\. Worker não fecha receipt.  
8\. Ruler não espera worker.  
9\. Engine real não é reimplementado.  
10\. Constitutional Runtime real não é reimplementado.

## **Fase 1 — Act Store**

Construir:

logline\_acts  
canonical\_bytes  
act\_json  
envelope\_json  
tuple\_hash  
content\_hash  
envelope\_hash  
append-only trigger  
hash golden tests

Critério:

Dado o mesmo Act, o mesmo hash sai sempre.  
Dado o mesmo envelope, o mesmo envelope\_hash sai sempre.  
Nenhum update semântico é possível.

## **Fase 2 — Engine Bridge**

Construir:

engine-main path dependency ou CLI bridge  
run\_act  
runtime\_logline capture  
emitted Acts  
error Acts  
hash conformance tests

Critério:

Nenhum walk próprio em labd.  
Nenhum confirmed\_by pivot próprio em labd.  
Nenhuma transition rule própria em labd.

## **Fase 3 — Constitutional Bridge**

Construir:

constitutional-runtime-main dependency  
admission evaluation  
policy/capability/evidence boundary  
outputs as Acts

Critério:

linguagem natural não executa.  
LLM não admite.  
operator output não vira verdade.

## **Fase 4 — Ruler**

Construir:

SystemClock  
FixedClock  
tick loop  
due projection  
observed\_due Act  
engine submission  
lag metrics  
replay

Critério:

ruler.tick nunca espera worker.  
ruler.tick sempre passa movimento pelo Engine.

## **Fase 5 — Projections**

Construir:

Today  
Timeline  
Due  
Queue  
Receipts  
Ghosts  
Registry  
Machines  
Workflows  
Policies

Critério:

drop projection\_\*;  
rebuild from logline\_acts;  
same result.

## **Fase 6 — Queue \+ Worker**

Construir:

projection\_queue  
worker claim  
readonly worker  
evidence blob store  
reported\_execution Act

Critério:

worker não altera logline\_acts diretamente.  
worker report volta para Engine/Act path.

## **Fase 7 — Receipt/Ghost**

Construir:

receipt review as Act  
ghost preservation as Act  
overclaim rejection  
evidence sufficiency  
conformance profile

Critério:

evidence não é receipt.  
report não é receipt.  
provider 200 não é verdade.

## **Fase 8 — TypeScript UI**

Construir:

Write  
Today  
Timeline  
Queue / Workorders  
Receipts  
Ghosts  
Registry  
Experiments  
Proof  
Conformance

Critério:

TS não calcula hash autoritativo.  
TS não decide estado.  
TS não fecha receipt.  
TS só projeta, propõe e transporta.

---

# **16\. Testes obrigatórios**

## **Hash**

same Act → same tuple\_hash  
same Act → same content\_hash  
same envelope → same envelope\_hash  
changing transport only → tuple\_hash same, envelope\_hash changes  
changing one slot → tuple\_hash changes  
changing hashes/id only → content\_hash unchanged

## **Engine**

missing confirmed\_by → Doubt  
valid receipt evidence → Ok  
prohibited route → Not  
status transition comes from engine-main  
runtime\_logline captured  
no local reimplementation

## **Ruler**

FixedClock at T detects Act due at T  
FixedClock before T does not detect  
Ruler emits observed\_due Act  
Ruler submits to Engine  
Ruler does not enqueue directly  
Ruler does not wait worker

## **Projection**

drop projection\_today  
rebuild from logline\_acts  
same Today

## **Worker**

worker can report evidence  
worker cannot close receipt  
worker cannot mutate Act  
worker cannot bypass Engine

## **Hard targets**

false\_closure\_rate \= 0  
false\_ok\_from\_missing\_evidence \= 0  
model\_output\_as\_evidence \= 0  
worker\_closed\_receipt \= 0  
projection\_claimed\_canon \= 0  
scheduler\_bypassed\_engine \= 0

Essas métricas e hard targets já aparecem nos documentos do projeto como saúde metabólica do Lab, não como “IA inteligente”.

---

# **17\. Onde a internet entra e onde ela não entra**

A internet ajuda a escolher ferramentas.

Ela não manda na semântica LogLine.

Escolhas externas:

Tokio  
\= async runtime, timers, tasks, networking.

Axum  
\= HTTP API, SSE/WebSocket bridge, handlers.

SQLx  
\= async SQL, Postgres integration.

PostgreSQL  
\= durable store, jsonb query, generated columns, queue projection.

RFC 8785 / JCS  
\= referência externa para JSON canônico hashable.

Tokio, Axum e SQLx são escolhas práticas para Rust backend async; PostgreSQL oferece JSONB e mecanismos úteis para projections/queue; RFC 8785 dá a base externa para canonicalização JSON verificável. ([Tokio](https://tokio.rs/?utm_source=chatgpt.com))

Mas:

Tokio não define LogLine.  
Axum não define LogLine.  
SQLx não define LogLine.  
Postgres não define LogLine.  
TypeScript não define LogLine.  
JCS não define LogLine.

Eles servem o Act.

---

# **18\. Versão final em uma página**

The Santo André Laboratory is a LogLine Lab.

Its only durable semantic object is the content-addressed LogLine Act.

A LogLine Act has a required 9-slot tuple:  
who, did, this, when, confirmed\_by, if\_ok, if\_doubt, if\_not, status.

Each Act carries three hashes:  
tuple\_hash, content\_hash, envelope\_hash.

The content\_hash is the Act address.

All state movement is another LogLine Act.

The LogLine Engine is the horizontal runtime:  
it receives temporal intersections from the Ruler,  
walks the 9-slot form,  
pivots on confirmed\_by,  
selects if\_ok / if\_doubt / if\_not,  
and emits admissible state movement.

The Constitutional Runtime is the vertical runtime:  
it turns human, LLM, office, process and policy context  
into admissible or rejected LogLine Acts.

The Ruler is a non-blocking temporal cursor:  
it slides over Earth time and the timeline,  
detects due Acts,  
submits observations to the real Engine,  
and never waits for execution.

The Queue is a projection:  
it materializes dispatchable Acts for workers.

Workers touch reality:  
they execute bounded work and return reports/evidence as Acts.

Receipts and Ghosts are Acts:  
one closes scoped truth;  
the other preserves absence, doubt or blockage.

The UI is a projection:  
minilab.work never creates truth.  
It shows the Lab breathing.

Rust keeps the body.  
TypeScript shows the body.  
The Engine moves the body.  
The Constitutional Runtime admits the body.  
The timeline remembers the body.  
The clock keeps the body alive.

---

# **19\. A frase que eu colocaria no topo do repo**

This repository does not implement a new LogLine.

It operates a Lab around the real LogLine Engine and the real Constitutional Runtime.

The only durable semantic object is the content-addressed LogLine Act.

Everything else is projection.

Em português:

Este repositório não implementa uma nova LogLine.

Ele opera um Lab em volta do LogLine Engine real e do Constitutional Runtime real.

O único objeto semântico durável é o LogLine Act content-addressed.

Todo o resto é projeção.

---

Observação honesta: eu li o archive e os arquivos dos engines de forma estática; o ambiente aqui não tem `cargo`, então não rodei `cargo test`. A spec acima já assume que a primeira ação técnica, no ambiente de desenvolvimento real, é rodar os testes do `engine-main`, do `constitutional-runtime-main` e da conformance antes de escrever qualquer wrapper novo.

According to a document from 2026-06-08, o `minilab.work` não deve nascer como “telas em cima do backend”, mas como superfície operacional de um Lab vivo: captura intenção, passa por Constitutional Runtime, preserva Acts na timeline, deixa a Régua encontrar o que ficou devido, passa pelo Engine, despacha trabalho, recebe evidência, fecha receipt ou preserva ghost, e só então projeta estado para a UI.

# **Spec final — UX, onboarding e conexão com os engines**

A parte de experiência do usuário precisa ser tratada como um **runtime de experiência**, não como frontend decorativo.

A frase central:

```
UX não cria verdade.
UX conduz o humano até o próximo LogLine Act correto.
```

Ou:

```
The UX is the human-facing projection and capture layer of the Lab.
It helps intention become Act,
helps time become visible,
helps evidence become understandable,
and helps receipts/ghosts become actionable.
```

Em português:

```
A UX é a camada humana de captura e projeção do Lab.
Ela ajuda intenção a virar Act,
tempo a ficar visível,
evidência a ficar compreensível,
e receipts/ghosts a virarem ação.
```

Ela não é só “onboarding”. Ela é a experiência contínua de viver dentro de um sistema onde **tudo que importa vira LogLine Act** e todo estado visível é projeção.

---

# **1\. O erro que precisa ser evitado**

O erro seria fazer:

```
React app bonito
→ wizard de onboarding
→ dashboard
→ CRUD de candidates/workorders/receipts
→ chama backend quando precisa
```

Isso mataria a ideia.

A forma certa é:

```
Human Experience
→ UX Runtime
→ Constitutional Runtime
→ LogLine Act
→ Timeline
→ Ruler
→ Engine
→ Queue
→ Worker
→ Evidence
→ Receipt/Ghost
→ Projection
→ Human Experience
```

A UX fecha o ciclo. Ela é o lugar onde o humano entende o que aconteceu, decide o próximo movimento e vê o Lab respirar.

---

# **2\. O UX Runtime**

Eu criaria um componente explícito:

```
labd-experience
```

ou:

```
experience-runtime
```

Ele é backend de UX, mas **não é autoridade semântica**.

Ele fica entre:

```
TypeScript UI
e
Rust Lab Runtime / Engines / Projections
```

Arquitetura:

```
TypeScript UI
  ↓
Experience API / UX Runtime
  ↓
Projection Store
Act Store
Constitutional Runtime Bridge
Engine Bridge
Ruler Status
Queue Projection
Worker/Evidence Projection
Receipt/Ghost Projection
```

O `labd-experience` faz:

```
- montar estado de tela a partir de projections;
- montar formulários a partir de packs;
- sugerir próximos passos;
- explicar por que algo aparece;
- traduzir ações humanas em propostas de LogLine Acts;
- chamar Constitutional Runtime para admissão vertical;
- mostrar resultados do Engine horizontal;
- assinar/encaminhar comandos com escopo;
- fazer streaming de mudanças;
- nunca criar verdade por conta própria.
```

Ele não faz:

```
- admitir Act sozinho;
- calcular hash canônico como autoridade;
- decidir branch if_ok / if_doubt / if_not;
- fechar receipt;
- resolver ghost;
- enfileirar sem Engine;
- alterar timeline;
- transformar UI state em verdade.
```

Isso segue a separação já registrada: LLM propõe, Operator conduz sessão, Constitutional Runtime admite/rejeita/ghosta, Ruler detecta due, Engine valida/transiciona, Dispatch enfileira, Worker executa, Evidence apoia closure, Receipt fecha escopo e Projection renderiza estado.

---

# **3\. A UX como órgão dos dois motores**

A experiência do usuário tem que mostrar claramente os dois motores e o tempo:

```
Constitutional Runtime
= motor vertical da intenção.

Ruler + Engine
= movimento horizontal do tempo sobre Acts.

Projection
= estado legível para o humano.
```

Os documentos já descrevem a separação: horizontal é `Earth time → ruler → timeline intersection → due Act → engine walk → state transition → queue`, enquanto vertical é `human/LLM intent → operator session → process → candidate → admission → authority → dispatch`.

Então a UI precisa educar por uso:

```
Write / Inbox / Candidates / Review
= vertical.

Today / Timeline / Lab Routine / Workorders / Receipts / Ghosts
= tempo + projeções.

Runtime / Engines / Dispatch / Schedules
= maquinário.

Workbench / Proof / Conformance
= ciência, prova, experimento.
```

A UI não deve esconder o motor. Ela deve mostrar o suficiente para o usuário confiar, mas sem obrigar a pessoa a entender tudo no primeiro minuto.

---

# **4\. Onboarding inicial: fechar o primeiro receipt**

O onboarding inicial não deve ser um tour.

Produto comum faz:

```
“clique aqui, aqui e aqui”
```

O Lab deve fazer:

```
“vamos fechar o primeiro ciclo real”
```

Objetivo do primeiro onboarding:

```
o usuário precisa ver um LogLine Act nascer,
ser admitido,
entrar na timeline,
ficar due,
passar pelo Engine,
ir para fila,
voltar com evidência,
e fechar receipt ou abrir ghost.
```

A experiência inicial deve ser chamada de algo como:

```
First Receipt
```

ou:

```
Lab First Breath
```

ou em português:

```
Primeira Respiração do Lab
```

## **Fluxo do onboarding inicial**

### **1\. Welcome / escolha de modo**

Tela:

```
Welcome to minilab.work
This is a LogLine Lab cockpit.

Choose:
- Start a new Lab
- Open existing Lab
- Import Lab Pack
- Read-only demo
```

Mas a cópia precisa deixar claro:

```
minilab.work não é a Foundation.
minilab.work não é o protocolo.
minilab.work é o cockpit humano de um Lab.
```

O documento do índice institucional coloca `minilab.work` como interface operacional/shell/produto de trabalho, com função de mostrar Today, Timeline, Workorders, Receipts e Ghosts, capturar candidatos, navegar registry, operar workbench, mostrar runtime e tornar o Lab usável.

### **2\. Lab identity**

O usuário cria ou abre um Lab:

```
Lab name
Lab profile
storage mode
pack
operator
local timezone
clock source
```

Isso gera um Act, por exemplo:

```
who: operator:<id>
did: initialized_lab
this:
  lab_name: "Santo Andre Laboratory"
  profile: "local-first"
  pack: "santo-andre"
when: <now>
confirmed_by:
  operator_session: <session_hash>
if_ok: create_lab_genesis
if_doubt: require_review
if_not: abort_initialization
status: proposed
```

Esse Act passa pelo Constitutional Runtime. O onboarding não “cria o Lab no banco” como verdade; ele propõe um Act de inicialização.

### **3\. Engine and canon check**

Tela mostra:

```
LogLine Engine
Constitutional Runtime
Act Store
Projection Engine
Ruler
Queue
Worker
```

Mas de modo simples:

```
Ready
Needs setup
Unavailable
Unsafe
```

Com botão:

```
Show technical trace
```

Aqui entra progressive disclosure: o usuário vê primeiro o essencial; detalhes técnicos aparecem sob demanda. Essa abordagem é alinhada ao princípio de progressive disclosure, que recomenda mostrar inicialmente poucas opções importantes e revelar complexidade conforme necessário. ([Nielsen Norman Group](https://www.nngroup.com/articles/progressive-disclosure/?utm_source=chatgpt.com))

### **4\. Load pack**

O usuário carrega um pack:

```
santo-andre.pack
```

O pack define:

```
- templates;
- routines;
- onboarding tasks;
- receipt profiles;
- ghost rules;
- projection labels;
- starter experiments;
- allowed workers;
- forbidden actions;
- copywriting;
- menu visibility;
- help cards.
```

Isso também gera Act:

```
who: operator:<id>
did: loaded_pack
this:
  pack_id: santo-andre
  pack_version: 0.1.0
  pack_hash: sha256:<hash>
when: <now>
confirmed_by:
  content_hash: sha256:<hash>
if_ok: enable_pack_templates
if_doubt: require_pack_review
if_not: reject_pack
status: proposed
```

### **5\. Criar o primeiro Act útil**

A UI oferece três opções, não trinta:

```
- Verificar saúde do Lab
- Verificar uma máquina
- Agendar uma rotina simples
```

Sugestão inicial:

```
Verificar saúde do próprio Lab em 2 minutos.
```

Act:

```
who: santo-andre.lab
did: verify
this:
  target: lab_runtime_health
when: <now + 2m>
confirmed_by:
  required: runtime_health_probe
if_ok: close_first_receipt
if_doubt: open_first_ghost
if_not: open_runtime_failure_path
status: scheduled
```

A UI mostra:

```
This is your first scheduled Act.
It will become due at 10:42.
The Ruler will detect it.
The Engine will admit or reject movement.
A worker may run a read-only probe.
Evidence will return.
A receipt or ghost will close the first cycle.
```

### **6\. Watching Today**

A tela muda para `Today`.

Não é dashboard. É o corte da régua.

Ela mostra:

```
Now
Next due Act
Ruler status
Engine status
Queue
Evidence expected
Receipt/Ghost outcome
```

Os docs já definem `Today` como o corte atual da Régua, `Timeline` como corpo histórico, `Lab Routine` como ritmo recorrente, `Workorders` como trabalho despachado, `Receipts` como fechamento escopado e `Ghosts` como verdade ausente preservada.

### **7\. Primeiro receipt ou ghost**

Quando o ciclo termina:

```
Success:
First receipt closed.

Doubt:
First ghost opened.
```

Mas ghost não é fracasso. A UX precisa tratar ghost como maturidade:

```
The Lab refused to pretend.
Missing evidence was preserved.
```

Em português:

```
O Lab se recusou a fingir.
A prova ausente foi preservada.
```

Isso ensina a filosofia sem palestra.

---

# **5\. Onboarding contínuo: contextual, não aula**

Depois do primeiro ciclo, o onboarding não deve continuar como tutorial linear. Ele deve virar **ajuda contextual acionada por estado**.

Há boa evidência de UX de que tours longos interrompem usuários, são esquecidos e nem sempre melhoram desempenho; ajuda contextual tende a evitar esses problemas quando aparece no momento certo e de forma não intrusiva. ([Nielsen Norman Group](https://www.nngroup.com/articles/onboarding-tutorials/?utm_source=chatgpt.com))

Então a experiência contínua deve funcionar assim:

```
state/projection muda
→ UX Runtime detecta oportunidade de ajuda
→ mostra micro-orientação
→ usuário age ou ignora
→ se agir, isso vira Act/proposta
```

Exemplos:

## **Candidate faltando slot**

Tela:

```
This candidate is missing `confirmed_by`.

Without confirmed_by, the Lab cannot know what would count as evidence.
```

Ações:

```
- Add evidence requirement
- Mark as intentionally doubtful
- Ask local LLM to suggest evidence
- Discard candidate
```

Se clicar “Ask local LLM”, o LLM só propõe. Não admite.

## **Ghost aberto**

Tela:

```
Ghost opened: required evidence did not arrive.

Missing:
- runtime_health_probe output
- worker report hash
```

Ações:

```
- retry probe
- ask human review
- carry ghost until tomorrow
- reject claim
```

Cada ação vira Act.

## **Workorder pendente**

Tela:

```
This work was admitted by the Engine but has not executed yet.

Ruler did not block.
Queue is waiting for worker availability.
```

Ações:

```
- inspect source Act
- inspect queue projection
- retry dispatch
- cancel via Act
```

## **Receipt candidate**

Tela:

```
Evidence returned.
The Lab can close a scoped receipt, but not a broad claim.
```

Ações:

```
- close narrow receipt
- request more evidence
- reject overclaim
- open ghost
```

Essa experiência ensina LogLine por consequência.

---

# **6\. Níveis de profundidade da UI**

A interface precisa ter camadas.

```
Level 1 — Human
Level 2 — Operational
Level 3 — Canonical
Level 4 — Trace / Proof
```

## **Level 1 — Human**

Mostra:

```
Backup check is waiting.
The Lab needs evidence.
This ghost is unresolved.
First receipt closed.
```

Sem jargão demais.

## **Level 2 — Operational**

Mostra:

```
Candidate
Scheduled Act
Due
Queued
Worker report
Receipt review
```

## **Level 3 — Canonical**

Mostra os 9 slots:

```
who
did
this
when
confirmed_by
if_ok
if_doubt
if_not
status
```

## **Level 4 — Trace / Proof**

Mostra:

```
tuple_hash
content_hash
envelope_hash
source Acts
engine decision
projection source
receipt scope
evidence hashes
```

A regra:

```
Todo card visível precisa ter botão:
“Why am I seeing this?”
```

E resposta:

```
Because these Acts projected into this state:
- act:<hash>
- act:<hash>
- act:<hash>
```

Isso mata a ilusão de dashboard. A pessoa aprende que UI é projeção.

---

# **7\. Backend de UX: módulos concretos**

Eu faria o `labd-experience` com estes módulos:

```
experience-session
experience-state
experience-guide
experience-actions
experience-forms
experience-explain
experience-stream
experience-preferences
```

## **7.1 `experience-session`**

Cria uma sessão operacional:

```
who: operator:<id>
did: started_operator_session
this:
  surface: minilab.work
  lab: lab:<id>
  mode: onboarding
when: <now>
confirmed_by:
  auth_session: <hash>
if_ok: enable_safe_actions
if_doubt: require_reauth
if_not: deny_session
status: active
```

A sessão não é autoridade universal. É contexto.

## **7.2 `experience-state`**

Agrega projections:

```
TodayProjection
CandidateProjection
QueueProjection
ReceiptProjection
GhostProjection
RuntimeHealthProjection
PackProjection
OnboardingProjection
```

Resposta da API:

```json
{
  "lab": "santo-andre",
  "mode": "onboarding",
  "stage": "first_receipt_waiting_for_due",
  "primary_action": {
    "label": "Watch first Act",
    "action": "open_today"
  },
  "cards": [],
  "source_acts": ["sha256:..."],
  "staleness": "fresh"
}
```

## **7.3 `experience-guide`**

Decide o próximo passo humano, mas só como projeção/recomendação.

Exemplo:

```rust
pub enum GuideSuggestion {
    CreateFirstAct,
    ReviewCandidate { act: ActRef },
    AddConfirmedBy { candidate: ActRef },
    InspectGhost { ghost: ActRef },
    ReviewReceiptCandidate { evidence: Vec<ActRef> },
    OpenRuntimeSetup,
}
```

Cada sugestão precisa ter:

```
reason
source_acts
risk
action
what_happens_next
```

## **7.4 `experience-actions`**

Transforma clique humano em proposta de Act.

Exemplo:

```
POST /v1/experience/actions
```

Payload:

```json
{
  "action": "create_first_health_check",
  "context": {
    "lab": "santo-andre",
    "operator_session": "sha256:..."
  }
}
```

Backend gera proposta:

```
who: operator:<id>
did: requested
this:
  template: first_lab_health_check
  target: lab_runtime_health
when: <now>
confirmed_by:
  operator_session: sha256:<hash>
if_ok: submit_to_constitutional_runtime
if_doubt: ask_for_review
if_not: reject_action
status: proposed
```

E manda para o Constitutional Runtime.

## **7.5 `experience-forms`**

Gera formulários a partir de templates de pack.

Exemplo:

```
form_id: first_lab_health_check
renders:
  - field: target
    label: What should the Lab verify?
  - field: when
    label: When should the Ruler check it?
  - field: confirmed_by
    label: What evidence should count?
```

Mas o resultado final não é “form submit”. É Act.

## **7.6 `experience-explain`**

API essencial:

```
GET /v1/experience/explain?projection=today&entity=<id>
```

Retorna:

```json
{
  "visible_state": "queued",
  "meaning": "The Engine admitted dispatch and queue projection materialized it.",
  "source_acts": [
    {
      "content_hash": "sha256:...",
      "did": "observed_due"
    },
    {
      "content_hash": "sha256:...",
      "did": "requested_execution"
    }
  ],
  "not_authority": "This card is a projection, not canonical state."
}
```

## **7.7 `experience-stream`**

SSE/WebSocket:

```
projection.updated
candidate.proposed
candidate.needs_review
act.admitted
ruler.observed_due
engine.transitioned
queue.materialized
worker.reported
receipt.closed
ghost.opened
```

A UI precisa sentir que o Lab respira.

## **7.8 `experience-preferences`**

Pode guardar preferências:

```
detail_level
canonical_view_enabled
reduced_motion
timezone
preferred_lab
show_hashes
show_slot_names
```

Preferências são projeção/configuração. Só viram Acts quando forem relevantes para operação, política ou auditoria.

---

# **8\. Conexão exata com os engines**

## **8.1 Caminho vertical: UX → Constitutional Runtime**

Quando o humano escreve ou clica:

```
Write / Button / Form / Voice / File
→ UX Runtime
→ proposed LogLine Act
→ Constitutional Runtime
→ admitted / rejected / ghost / needs authority / needs evidence
→ Act Store
→ Projection
→ UI
```

O Vertical Runtime Doctrine já define esse caminho como intenção, office, operator session, processo, candidato LogLine, Constitutional Runtime, Authority/Gate, Dispatch, Hermes, evidência, receipt/ghost e projection.

A UI deve mostrar esse caminho assim:

```
Draft
→ Candidate
→ Under Review
→ Needs Evidence
→ Needs Authority
→ Admitted
→ Scheduled / Active
```

Mas internamente cada movimento significativo é Act.

## **8.2 Caminho horizontal: Ruler → Engine → UX**

Quando o tempo passa:

```
Ruler.tick(now)
→ due Acts from projection/timeline
→ observed_due Act
→ Engine walk/transition
→ emitted Acts
→ Act Store
→ Queue Projection / Ghost / Receipt Candidate
→ UX stream
→ Today updates
```

O Engine não espera trabalho e não vira worker; a interface mínima já está descrita como `ruler.tick(now) → due_candidates[] → engine.transition(each) → dispatch or ghost`, e Dispatch/Queue existe para desacoplar transição admitida de execução real.

A UI deve mostrar:

```
Ruler observed
Engine admitted
Queue waiting
Worker running
Evidence returned
Receipt/Ghost decided
```

Isso educa o usuário sem aula.

## **8.3 Caminho de prova: Worker → Evidence → Receipt/Ghost → UX**

Quando worker volta:

```
worker report
→ reported_execution Act
→ evidence references
→ receipt reviewer
→ closed_receipt Act or ghost Act
→ projections
→ UI
```

Os documentos deixam claro que Worker/Hermes executa workorders admitidas, mas não decide o que deve ser feito, não infere autoridade, não fecha receipt e não escreve estado semântico diretamente; Evidence apoia closure, mas não é closure.

A UX precisa traduzir:

```
“Worker succeeded”
```

para:

```
“O worker retornou evidência.
Ainda falta receipt review.”
```

Isso é crucial.

---

# **9\. A tela `Today` como centro da experiência**

A home não deve ser “dashboard”.

Ela deve ser:

```
Today = o ponto onde o humano encontra a Régua.
```

Layout:

```
Today
├── Now / Ruler
│   ├── current time
│   ├── ruler status
│   ├── next tick
│   └── lag
│
├── Due now
│   ├── Acts que ficaram due
│   └── estado Engine/Queue
│
├── Needs human
│   ├── candidates needing review
│   ├── ghosts needing decision
│   └── receipts needing scope
│
├── Running
│   ├── work projected as queued/running
│   └── workers
│
├── Receipts closed today
│
└── Experiments
    ├── scheduled
    ├── running
    └── findings
```

Cada card tem:

```
What is this?
Why now?
Source Acts
Next safe action
Show canonical trace
```

---

# **10\. `Write` como boca do motor vertical**

`Write` deve aceitar texto sujo, arquivo, voz, colagem, comando rápido.

Mas não deve dizer:

```
AI criou uma tarefa.
```

Deve dizer:

```
A candidate Act was drafted.
It still needs admission.
```

Fluxo:

```
raw human signal
→ local parser / LLM translator
→ draft 9-slot shape
→ missing slots
→ evidence expectation
→ risk classification
→ submit to Constitutional Runtime
```

Tela de Write:

```
Input
Slot Preview
Missing
Evidence
Risk
Authority
Submit as Candidate
```

Exemplo de UX:

```
Você escreveu:
“amanhã ver se o backup rodou”

O Lab entendeu:

who: você
did: verify
this: local_backup_status
when: amanhã 10:00
confirmed_by: backup_probe_result
if_ok: close receipt
if_doubt: open ghost
if_not: open failure path
status: scheduled

Faltando:
- horário exato
- qual backup
```

A ação correta:

```
Complete missing slots
```

Não:

```
Criar tarefa
```

---

# **11\. Candidate Review**

A tela `Candidates` é uma mesa de admissão.

Ela deve mostrar:

```
Candidate
Source
Slots
Missing evidence
Authority requirement
Risk
Constitutional result
Possible outcomes
```

Botões:

```
Admit
Reject
Ghost
Ask for evidence
Ask for authority
Edit as new proposal
```

Mas cada botão chama o Constitutional Runtime ou cria Act de decisão. O UI não muda estado.

---

# **12\. Timeline como chão da confiança**

Timeline precisa estar sempre perto.

Em qualquer tela:

```
Open trace
```

Mostra:

```
Act chain
├── proposed
├── admitted
├── scheduled
├── observed_due
├── requested_execution
├── reported_execution
├── closed_receipt / ghost
```

Com:

```
content_hash
tuple_hash
envelope_hash
when
who
did
status
projection impact
```

A documentação já afirma que o Engine não deve mutar Acts antigos in-place; ele deve produzir transições append-only, e a projeção pode mostrar o Act como queued/running/done/blocked, mas a verdade é a cadeia.

Essa tela é o antídoto contra “dashboard mágico”.

---

# **13\. Runtime view: não para usuário comum, mas sempre disponível**

A seção `runtime` deve ter:

```
Runtimes
Engines
Agents
LLMs
Sensors
Dispatch
Schedules
```

Essa divisão foi identificada como uma melhoria importante porque `Engines` não deve sumir, `Schedules` não é admin e `Dispatch` conecta Workorders/Workers/Evidence.

Mas no onboarding inicial isso aparece resumido:

```
Lab machinery:
- Engine: ready
- Constitutional Runtime: ready
- Ruler: ticking
- Queue: empty
- Worker: ready
- Projection: fresh
```

Com botão:

```
Show machinery
```

---

# **14\. UX de erro: erro é ghost, blocked ou evidence gap**

Erros não devem ser banners genéricos.

Exemplo ruim:

```
Something went wrong.
```

Exemplo certo:

```
The Lab could not close this claim.

Missing:
- worker report hash
- required evidence profile

A ghost was opened.
```

Estados de erro UX:

```
missing_slot
missing_evidence
missing_authority
engine_rejected
engine_doubt
ruler_lag
queue_timeout
worker_failed
receipt_overclaim
projection_stale
```

Cada um aponta para:

```
source Acts
missing requirement
safe next action
```

Acessibilidade entra aqui também: WCAG 2.2 é a versão que o W3C recomenda usar para maximizar aplicabilidade futura ao desenvolver ou atualizar políticas de acessibilidade; para este produto, isso significa foco visível, navegação por teclado, mensagens de erro claras, status dinâmicos anunciáveis e não depender só de cor para comunicar estado. ([W3C](https://www.w3.org/TR/WCAG22/?utm_source=chatgpt.com))

---

# **15\. Copywriting do produto**

A linguagem da UI precisa ensinar sem infantilizar.

Evitar:

```
Task created
Automation ran
AI decided
Done
Failed
```

Preferir:

```
Candidate drafted
Act admitted
Due observed
Transition admitted
Work queued
Evidence returned
Receipt closed
Ghost opened
Projection updated
```

Mas com tradução humana junto:

```
Receipt closed
Backup was verified with enough evidence.

Ghost opened
The Lab did not receive enough proof to close this claim.
```

Formato de card:

```
Human sentence
Canonical label
Source Acts
Next action
```

Exemplo:

```
The Lab verified the backup.
Receipt closed · act:<hash>

Show proof
```

---

# **16\. Onboarding como pack**

O onboarding não deve ficar hardcoded na UI.

Ele deve vir do pack:

```
packs/santo-andre/onboarding.yaml
```

Exemplo:

```
onboarding:
  id: santo-andre.first-breath.v0
  goal: close_first_receipt
  stages:
    - id: lab_identity
      title: Name this Lab
      action_template: initialize_lab

    - id: engine_check
      title: Check the machinery
      required:
        - engine.ready
        - constitutional_runtime.ready
        - act_store.ready
        - ruler.ready

    - id: first_act
      title: Create first scheduled Act
      template: first_lab_health_check

    - id: watch_ruler
      title: Watch the Ruler find the Act
      projection: today

    - id: receipt_or_ghost
      title: Learn from the outcome
      success:
        - receipt.closed
        - ghost.opened
```

A Foundation ou Lab Kit pode definir formato genérico de pack; Santo André fornece interpretação. Isso preserva canon pequeno e Labs grandes.

---

# **17\. Endpoints de UX**

API sugerida:

```
GET  /v1/experience/session
POST /v1/experience/session/start

GET  /v1/experience/state
GET  /v1/experience/onboarding
POST /v1/experience/onboarding/actions

POST /v1/experience/intent
POST /v1/experience/actions

GET  /v1/experience/explain
GET  /v1/experience/source-acts

GET  /v1/projections/today
GET  /v1/projections/timeline
GET  /v1/projections/candidates
GET  /v1/projections/queue
GET  /v1/projections/receipts
GET  /v1/projections/ghosts
GET  /v1/projections/runtime

GET  /v1/events
```

Importante:

```
/experience/intent não admite.
/experience/actions não muda verdade.
/projections não são canon.
/events só informa.
```

A admissão real vai para:

```
Constitutional Runtime
```

A transição temporal real vai para:

```
Ruler → Engine
```

---

# **18\. Contratos internos**

## **`ExperienceAction`**

```ts
type ExperienceAction = {
  action_id: string
  label: string
  risk: "safe" | "review" | "authority_required"
  source_projection: string
  source_acts: string[]
  creates_proposal: true
  target_runtime:
    | "constitutional_runtime"
    | "engine"
    | "projection"
    | "ruler_request"
}
```

## **`GuideCard`**

```ts
type GuideCard = {
  id: string
  title: string
  human_summary: string
  canonical_summary: string
  source_acts: string[]
  primary_action?: ExperienceAction
  secondary_actions: ExperienceAction[]
  detail_level: 1 | 2 | 3 | 4
}
```

## **`ExplainResponse`**

```ts
type ExplainResponse = {
  visible_label: string
  projection_name: string
  projection_is_truth: false
  reason: string
  source_acts: {
    content_hash: string
    did: string
    status: string
    when: string
  }[]
  engine_trace?: {
    runtime_logline: string
    branch: "if_ok" | "if_doubt" | "if_not"
  }
  next_safe_actions: ExperienceAction[]
}
```

---

# **19\. Como a UX conversa com TypeScript**

TypeScript fica como periferia rica:

```
Next/React UI
generated API client
forms
timeline renderer
trace viewer
onboarding renderer
projection cards
SSE listener
local draft editor
```

TypeScript pode:

```
- renderizar projections;
- manter draft local antes de submissão;
- mostrar preview de slots;
- chamar /experience/intent;
- mostrar hash retornado;
- receber eventos;
- abrir trace;
- operar help contextual.
```

TypeScript não pode:

```
- admitir Act;
- fechar receipt;
- resolver ghost;
- calcular hash autoritativo;
- enfileirar trabalho;
- marcar workorder como done;
- alterar registry como verdade.
```

---

# **20\. UX com computador velho e LLM fraco**

Isso é parte da beleza do Lab.

A UI deve ser leve:

```
server-rendered where possible
SSE instead of heavy realtime framework
local drafts
no mandatory giant client state
projection pagination
timeline virtualized
hash/details lazy-loaded
LLM optional
forms work without LLM
```

LLM local entra como:

```
draft helper
slot suggester
summarizer
finding proposer
```

Nunca como:

```
authority
receipt closer
evidence
engine
```

A UI precisa degradar bem:

```
LLM offline?
Still write manually.

Worker offline?
Queue waits.

Engine unavailable?
No false admission.

Projection stale?
Show stale, do not pretend.

Ruler lagging?
Show lag, do not hide time.
```

---

# **21\. Métricas de UX que importam**

Não medir só “cliques”.

Medir metabolismo humano:

```
time_to_first_act
time_to_first_receipt_or_ghost
candidate_completion_rate
missing_slot_resolution_time
ghost_understanding_rate
receipt_overclaim_rejection_rate
projection_trace_open_rate
manual_review_burden
ruler_lag_visibility
user_trust_recovery_after_failure
```

E hard targets:

```
0 UI closed receipt
0 UI admitted Act
0 hidden engine rejection
0 projection shown as truth
0 worker success shown as institutional closure
```

---

# **22\. Primeiro build de UX**

Não construir tudo.

Construir este fluxo:

```
Start Lab
→ Load Pack
→ Create first scheduled Act
→ Watch Today
→ See Ruler observed due
→ See Engine admitted transition
→ See Queue
→ See Worker report
→ See Receipt or Ghost
→ Open Timeline trace
```

Telas mínimas:

```
Start
Write
Candidate Review
Today
Timeline Trace
Runtime Status
Receipt/Ghost Result
```

Serviços mínimos:

```
experience-state
experience-actions
experience-guide
experience-explain
projection-api
events stream
```

Critério de aceitação:

```
Uma pessoa que nunca viu LogLine consegue completar o primeiro ciclo,
entender que UI é projeção,
ver de onde veio cada estado,
e sair sabendo que receipt é prova fechada e ghost é dúvida preservada.
```

---

# **23\. A síntese**

A experiência do usuário precisa ser esta:

```
Primeiro: fazer o Lab respirar.
Depois: mostrar onde estão os órgãos.
Depois: ensinar o usuário a operar sem transformar UI em autoridade.
```

A UX não é uma aula de LogLine.  
É uma sequência de encontros com a realidade do Lab.

```
Write ensina intenção.
Candidate ensina forma.
Review ensina autoridade.
Today ensina tempo.
Timeline ensina memória.
Workorders ensinam desacoplamento.
Evidence ensina prova.
Receipt ensina fechamento.
Ghost ensina honestidade.
Runtime ensina limite.
Workbench ensina descoberta.
```

A frase final da spec:

```
The onboarding of minilab.work is not a tour of features.
It is the first successful passage through the LogLine metabolism:
intention, admission, Act, time, Engine, queue, evidence, receipt or ghost, projection.
```

Em português:

```
O onboarding do minilab.work não é um tour de funcionalidades.
É a primeira travessia bem-sucedida pelo metabolismo LogLine:
intenção, admissão, Act, tempo, Engine, fila, evidência, receipt ou ghost, projeção.
```

Essa é a UX digna do projeto.

Sim. Isso fecha a arquitetura.

A peça que faltava é esta:

```
O Lab instalável vem puro.

Ele carrega canon, regras mínimas, conformance e engines LogLine reais.

Todo o resto é carregável:
- packs para interpretação, rotina, pesquisa, projeções e regras locais;
- Apps para conectividade com ferramentas externas;
- projection profiles para exportar ciência em formatos consagrados.
```

A distinção mais importante:

```
Pack = regra / lente / interpretação / rotina / método.
App  = conexão / ferramenta / fonte externa / superfície operacional.
Projection Profile = formato de saída / lente científica / interoperabilidade.
```

Ou mais curto:

```
Pack ensina o Lab a interpretar.
App conecta o Lab ao mundo.
Projection traduz o Lab para padrões externos.
```

Isso preserva a pureza do LogLine sem matar a parte viva do Lab.

---

# **1\. A forma final: LogLine Lab Installable**

O produto técnico principal não é “minilab.work” nem “Santo André”. O produto técnico principal é:

```
LogLine Lab Installable
```

ou:

```
logline-lab-kit
```

Ele instala um Lab vazio, conformante e puro.

Esse Lab vem com:

```
- LogLine canon puro;
- LogLine Act shape;
- hash/canonicalization profile;
- conformance suite;
- engine-main real;
- constitutional-runtime-main real;
- ruler / clock / temporal driver;
- act store append-only;
- projection runtime;
- pack loader;
- App connection host;
- standard projection exporters;
- minilab.work shell opcional.
```

Ele **não** vem opinando que o dia a dia deve ser Santo André.  
Ele **não** vem dizendo que todo mundo precisa estudar LogLine do teu jeito.  
Ele **não** vem com “Dan cosmology” embutida como canon.  
Ele vem capaz de carregar isso como pack.

Essa decisão já aparece nos teus docs: um Lab deve carregar canon, admitir e preservar Acts, executar processos por fronteiras declaradas, projetar estado a partir de timeline/evidence e carregar interpretation packs sem redefinir o núcleo de nove slots.

---

# **2\. Kernel puro vs camada carregável**

Eu organizaria o código assim:

```
logline-lab/
  kernel/
    canon/
    engines/
    constitutional-runtime/
    conformance/
    hash-profiles/
    schemas/

  host/
    labd/
    act-store/
    ruler/
    projection-runtime/
    pack-runtime/
    app-host/
    experience-runtime/
    cli/

  loadables/
    packs/
    apps/
    projection-profiles/

  standards/
    prov/
    ro-crate/
    datacite/
    dcat/
    jats/
    cwl/
    fair/

  ui/
    minilab-work/

  var/
    timeline/
    blobs/
    projections/
    logs/
```

A fronteira é dura:

```
kernel/
= puro, mínimo, canônico, versionado, testado por conformance.

host/
= instala, roda, carrega, observa, projeta, conecta.

loadables/
= opinião, interpretação, conectividade, superfície, ciência aplicada.

standards/
= projeções para padrões externos, não novos canons LogLine.

ui/
= cockpit humano.

var/
= estado local, timeline, projeções, blobs.
```

O `kernel` não importa `packs`.  
O `kernel` não importa `apps`.  
O `kernel` não sabe o que é Santo André.  
O `kernel` não sabe o que é voulezvous.  
O `kernel` só sabe LogLine.

---

# **3\. O ponto de convivência: `loadout`**

Você pediu um espaço simples onde canon puro e regras carregáveis convivem e possam ser trocados. Eu faria isso com um conceito explícito:

```
Lab Loadout
```

O `loadout` é a composição ativa do Lab naquele momento.

Arquivo:

```
lab.loadout.yaml
```

Exemplo:

```
loadout_id: santo-andre-local-2026-06
lab_id: lab:santo-andre.local

kernel:
  canon: logline-canon@0.2.0
  engine: engine-main@0.2.0
  constitutional_runtime: constitutional-runtime-main@0.2.0
  hash_profile: logline.hash.sha256-jcs.v0
  conformance_profile: logline.core.v0

active_packs:
  - id: santo-andre-laboratory
    version: 0.1.0
    mode: local-opinion
    source: packs/santo-andre-laboratory/
    hash: sha256:...

  - id: research-clock
    version: 0.1.0
    mode: experimental
    source: packs/research-clock/
    hash: sha256:...

active_apps:
  - id: github-connection
    version: 0.1.0
    mode: read-write-proposal
    source: apps/github-connection/
    hash: sha256:...

  - id: local-files-connection
    version: 0.1.0
    mode: read-only-evidence
    source: apps/local-files-connection/
    hash: sha256:...

active_projection_profiles:
  - id: prov-export
    version: 0.1.0
  - id: ro-crate-experiment-export
    version: 0.1.0
  - id: datacite-dataset-export
    version: 0.1.0
  - id: fair-readiness
    version: 0.1.0

rules:
  pack_precedence:
    - santo-andre-laboratory
    - research-clock
  conflict_policy: require_explicit_resolution_act
  app_output_policy: app_outputs_are_proposals_or_evidence_only
  projection_policy: projections_are_never_truth
```

Comandos:

```shell
labd init
labd kernel verify
labd pack install santo-andre-laboratory.pack
labd pack enable santo-andre-laboratory
labd app install github-connection.app
labd app enable github-connection
labd projection enable ro-crate-experiment-export
labd loadout diff
labd loadout apply
labd conformance run
```

Mas cada mudança importante no loadout também vira LogLine Act:

```
who: operator:dan
did: enabled_pack
this:
  pack: santo-andre-laboratory@0.1.0
  loadout: lab.loadout.yaml
when: 2026-06-09T...
confirmed_by:
  pack_hash: sha256:...
  conformance_report: act:<hash>
if_ok: activate_pack
if_doubt: keep_pack_disabled
if_not: reject_pack
status: enabled
```

O arquivo é configuração operacional.  
A timeline é a memória verdadeira.

---

# **4\. Três tipos de carregáveis**

## **4.1 Pack**

Um pack é regra, interpretação, vocabulário, método e projeção local.

```
Pack = uma lente carregável.
```

Ele pode dizer:

```
neste Lab, “machine health” significa estes Acts;
neste Lab, o dia começa com estes experimentos;
neste Lab, ghosts desse tipo voltam ao Today amanhã;
neste Lab, uma pessoa aparece no Registry quando estes Acts existem;
neste Lab, este experimento tem esta hipótese, este schedule e este receipt profile.
```

Ele não pode:

```
redefinir Act;
criar décimo slot;
mudar hash;
mudar canonicalization;
fechar receipt sem evidência;
executar worker diretamente;
escrever timeline sem Engine;
declarar projeção como verdade;
virar canon global.
```

Isso já está alinhado com os docs: pack não é canon, runtime, app nem autoridade global; é um bundle carregável de vocabulário, projeções, processos, templates, offices, schedules, experiments e UI hints que ensina um Lab a interpretar um domínio preservando o canon LogLine.

## **4.2 App**

Um App é conectividade.

```
App = uma ponte controlada para uma ferramenta, fonte ou superfície externa.
```

Ele pode conectar:

```
GitHub
Google Drive
Notion
filesystem local
calendar
email
shell read-only
browser capture
instrumento científico
sensor
LLM local
repositório de dados
workflow runner
documentação
build system
```

Ele não ensina o Lab a interpretar o mundo.  
Ele só expõe capacidades, recursos, eventos e, se necessário, um widget.

A inspiração correta é o Apps SDK / MCP: nos docs oficiais da OpenAI, Apps construídos com Apps SDK usam MCP para conectar ao ChatGPT; o app precisa de um servidor MCP que define capacidades/tools e pode opcionalmente incluir um web component renderizado em iframe. ([OpenAI Developers](https://developers.openai.com/apps-sdk/quickstart?utm_source=chatgpt.com)) A documentação também descreve MCP como a espinha que mantém servidor, modelo e UI sincronizados por formato de wire, autenticação e metadata padronizados. ([OpenAI Developers](https://developers.openai.com/apps-sdk/concepts/mcp-server?utm_source=chatgpt.com))

Para o LogLine Lab, a tradução é:

```
O App expõe ferramentas ao Lab.
O Pack interpreta o que essas ferramentas significam.
```

## **4.3 Projection Profile**

Um projection profile é uma lente de exportação/interoperabilidade.

```
Projection Profile = tradução de Acts para formato externo reconhecido.
```

Ele pode exportar:

```
LogLine Acts → W3C PROV
LogLine experiment → RO-Crate
LogLine dataset → DataCite metadata
LogLine catalog → DCAT
LogLine paper/finding → JATS
LogLine workflow → CWL
LogLine actor/person → ORCID-aware contributor metadata
LogLine data package → FAIR readiness report
```

Ele não muda a verdade.  
Ele não cria estado.  
Ele só permite que o Lab fale com ciência, publicação, repositórios e auditoria externa.

---

# **5\. Diferença Pack vs App**

Essa tabela precisa virar doutrina:

| Pergunta | Pack | App |
| ----- | ----- | ----- |
| O que é? | Lente interpretativa | Conexão externa |
| Muda o quê? | Como o Lab interpreta Acts | Como o Lab acessa ferramenta/fonte |
| Declara regras? | Sim, locais e carregáveis | Não, só capacidades e limites |
| Declara processos? | Sim | Só callbacks/tools/eventos |
| Declara vocabulário? | Sim | Apenas labels técnicos |
| Declara projeções? | Pode | Pode fornecer widget, mas não verdade |
| Fala com API externa? | Não diretamente | Sim |
| Pode emitir evidência? | Define que evidência conta | Retorna evidence/report/proposal |
| Pode fechar receipt? | Não | Não |
| Pode criar Act direto? | Não | Não |
| Pode propor Act? | Sim, via templates | Sim, via connection output |
| Passa por Constitutional Runtime? | Quando gera proposta/processo | Sempre que propõe semântica |
| Passa pelo Engine? | Quando resulta em movimento de Act | Quando output vira movimento de Act |
| É comunidade? | Sim, packs interpretativos | Sim, apps de conectividade |

Frase:

```
Pack é gramática local.
App é tomada externa.
Projection é tradução.
Canon é lei.
Engine é movimento.
Timeline é memória.
```

---

# **6\. SDK de App do LogLine Lab**

Eu chamaria de:

```
LogLine Lab Connection App SDK
```

ou curto:

```
LLC SDK
```

Mas conceitualmente ele deve ser muito parecido com o padrão moderno de Apps/MCP:

```
app server
tools
resources
events
auth
metadata
UI component opcional
host bridge
review/conformance
```

A documentação oficial da OpenAI para Apps SDK fala em definir tools, registrar UI templates e manter modelo, servidor e UI sincronizados usando MCP server e widget runtime. ([OpenAI Developers](https://developers.openai.com/apps-sdk/build/mcp-server?utm_source=chatgpt.com)) A referência também recomenda usar campos do padrão MCP Apps e a bridge `ui/*` por padrão, com extensões específicas da OpenAI como opcionais. ([OpenAI Developers](https://developers.openai.com/apps-sdk/reference?utm_source=chatgpt.com))

No LogLine Lab, o equivalente fica:

```
App Server
= servidor externo ou local que expõe tools/resources/events ao Lab.

Lab App Host
= componente do labd que autentica, limita, audita e chama Apps.

Connection Manifest
= declaração do que o App pode fazer.

App Widget
= UI opcional dentro do minilab.work.

Act Mapper
= transforma outputs técnicos em proposed Acts ou evidence Acts.

Conformance
= prova que o App não finge autoridade semântica.
```

---

# **7\. Template de App**

Estrutura:

```
github-connection.app/
  app.manifest.yaml
  tools.yaml
  resources.yaml
  events.yaml
  auth.yaml
  act-mappers.yaml
  evidence-profiles.yaml
  widgets/
    repository-card/
    pull-request-card/
  conformance/
    cases.yaml
  README.md
```

Manifesto:

```
app_id: github-connection
app_version: 0.1.0
app_type: connection
sdk: logline.lab.connection-app.v0

name: GitHub Connection
description: Connects a LogLine Lab to GitHub repositories, issues, pull requests, commits, checks, and releases.

requires:
  lab_profile: logline.lab.v0
  app_host: logline.app-host >=0.1.0

capabilities:
  tools:
    - github.search_repositories
    - github.read_issue
    - github.read_pull_request
    - github.read_commit
    - github.create_issue_comment
  resources:
    - repository
    - issue
    - pull_request
    - commit
    - release
  events:
    - pull_request.opened
    - pull_request.merged
    - workflow_run.completed

auth:
  type: oauth2
  scopes:
    - repo:read
    - issues:read
    - pull_requests:read
    - checks:read
  secrets:
    storage: lab.secret_store
    never_send_to_llm: true

outputs:
  allowed_output_kinds:
    - observation
    - evidence
    - proposed_act
    - blob_ref
  forbidden_output_kinds:
    - admitted_act
    - receipt_closed
    - canon_change
    - direct_timeline_write

act_mappers:
  - id: pull_request_merged_to_observation
    input_event: pull_request.merged
    output: proposed_act
    constitutional_runtime_required: true

evidence:
  profiles:
    - github.commit.digest
    - github.workflow_run.report
    - github.pull_request.review

widgets:
  - id: pull-request-card
    kind: iframe_component
    source: widgets/pull-request-card/

security:
  allowed_actions:
    - read_repository_metadata
    - read_pull_request
    - read_workflow_run
  forbidden_actions:
    - mutate_timeline
    - close_receipt
    - change_pack_rules
    - change_canon
    - execute_shell

conformance:
  cases: conformance/cases.yaml
```

A regra de ouro:

```
App pode ver ferramenta.
App pode retornar evidência.
App pode propor Act.
App não admite Act.
App não fecha receipt.
App não muda timeline.
```

---

# **8\. Template de Pack**

Estrutura:

```
santo-andre-laboratory.pack/
  pack.manifest.yaml
  vocabulary.yaml
  act-templates/
    machine-health.yaml
    daily-lab-check.yaml
    local-llm-benchmark.yaml
  processes/
    morning-routine.yaml
    experiment-cycle.yaml
    ghost-review.yaml
  projections/
    today.yaml
    machine-registry.yaml
    experiment-board.yaml
  entity-profiles/
    person.yaml
    machine.yaml
    workflow.yaml
    app.yaml
  receipt-profiles/
    machine-health-receipt.yaml
    experiment-run-receipt.yaml
  ghost-profiles/
    missing-evidence.yaml
    unresolved-authority.yaml
    inconclusive-result.yaml
  schedules/
    lab-clock.yaml
    daily-ruler.yaml
  experiments/
    local-llm-quality-tick.yaml
    backup-probe.yaml
    peer-link-check.yaml
  ui/
    labels.yaml
    cards.yaml
    onboarding.yaml
  conformance/
    cases.yaml
  README.md
```

Manifesto:

```
pack_id: santo-andre-laboratory
pack_version: 0.1.0
pack_type: interpretation
status: experimental

name: Santo André Laboratory Pack
description: Local interpretation pack for daily Lab operation, experiment clock, machine registry, local LLM metabolism, and research routine.

requires:
  canon: logline-canon >=0.2.0
  engine: engine-main >=0.2.0
  lab_profile: logline.lab.v0

declares:
  vocabularies:
    - people
    - machines
    - workflows
    - policies
    - experiments
    - local_llms

  act_templates:
    - daily_lab_check
    - machine_health_probe
    - local_llm_quality_tick

  processes:
    - morning_routine
    - experiment_cycle
    - ghost_review

  schedules:
    - daily_lab_clock
    - hourly_machine_heartbeat
    - weekly_findings_review

  projections:
    - today
    - machine_registry
    - experiment_board
    - findings
    - ghosts_due_for_review

  receipt_profiles:
    - machine_health_receipt
    - experiment_run_receipt

  ghost_profiles:
    - missing_evidence
    - inconclusive_result
    - unresolved_authority

forbidden:
  - redefine_9_slots
  - redefine_hash_profile
  - bypass_engine
  - mutate_timeline_directly
  - treat_projection_as_truth
  - close_receipt_without_evidence
  - execute_worker_directly
  - grant_app_authority

compatibility:
  apps:
    optional:
      - github-connection
      - local-files-connection
      - calendar-connection
      - local-llm-connection

conformance:
  cases: conformance/cases.yaml
  required: true
```

Este pack pode ser compartilhado com a comunidade.  
Mas a comunidade instala sabendo:

```
isso é a interpretação Santo André;
não é o canon LogLine.
```

---

# **9\. Template de Projection Profile científico**

Estrutura:

```
ro-crate-experiment-export.projection/
  projection.manifest.yaml
  mapping.yaml
  jsonld-context.json
  examples/
  conformance/
  README.md
```

Manifesto:

```
projection_id: ro-crate-experiment-export
projection_version: 0.1.0
projection_type: scientific_export
standard: RO-Crate
standard_version: "1.2"

description: Exports LogLine experiment chains as RO-Crate Research Objects.

input:
  source: logline_acts
  selectors:
    - did: declared_experiment
    - did: ran_experiment
    - did: reported_execution
    - did: closed_receipt
    - status: ghost

output:
  format: json-ld
  root_file: ro-crate-metadata.json
  directory_layout: ro-crate/

mapping:
  experiment: schema:Dataset
  act: schema:Action
  actor: schema:Person | schema:SoftwareApplication | schema:Organization
  evidence_blob: schema:MediaObject
  receipt: schema:CreativeWork
  ghost: schema:Thing

truth_policy:
  projection_is_truth: false
  source_of_truth: logline_acts
  include_source_act_hashes: true
  include_tuple_hashes: true
  include_content_hashes: true
  include_envelope_hashes: true

conformance:
  validate_jsonld: true
  validate_required_metadata: true
  cases: conformance/cases.yaml
```

RO-Crate é adequado aqui porque especifica um método para agregar e descrever dados para distribuição, reuso, publicação, preservação e arquivamento; sua metadata usa JSON-LD e Schema.org. ([researchobject.org](https://www.researchobject.org/ro-crate/specification/1.2/metadata.html?utm_source=chatgpt.com))

---

# **10\. Projeções científicas “campeãs”**

Aqui a regra é: **não reinventar o formato científico quando já existe padrão maduro**.

O Lab deve manter Acts como verdade interna e exportar para padrões externos.

## **10.1 Provenance Projection — W3C PROV**

Uso:

```
quando queremos provar quem fez o quê, usando qual evidência, gerando qual artefato.
```

Mapeamento:

```
who              → prov:Agent
did              → prov:Activity
this             → prov:Entity / prov:Plan / prov:Collection
when             → prov:startedAtTime / prov:endedAtTime / prov:generatedAtTime
confirmed_by     → prov:wasAssociatedWith / prov:wasGeneratedBy / prov:used
receipt          → prov:Entity generated by review activity
evidence         → prov:Entity used by activity
```

W3C PROV-O fornece classes, propriedades e restrições para representar e intercambiar informação de proveniência gerada em diferentes sistemas e contextos. ([W3C](https://www.w3.org/TR/prov-o/?utm_source=chatgpt.com))

## **10.2 Research Object Projection — RO-Crate**

Uso:

```
empacotar experimento, dados, código, evidência, receipts e metadata.
```

Formato:

```
ro-crate/
  ro-crate-metadata.json
  data/
  evidence/
  receipts/
  logs/
  README.md
```

RO-Crate é fundamentalmente uma pasta com `ro-crate-metadata.json`, descrevendo dados e entidades contextuais com Schema.org em JSON-LD. ([Metadata Standards Index](https://msi.dublincore.org/standards/ro-crate/?utm_source=chatgpt.com))

## **10.3 Dataset Citation Projection — DataCite**

Uso:

```
quando um conjunto de evidências, experimento ou release do Lab precisa ser citável.
```

Campos:

```
identifier
creators
titles
publisher
publicationYear
resourceType
version
descriptions
fundingReferences
relatedIdentifiers
```

DataCite Metadata Schema define propriedades centrais para identificação precisa e consistente de recursos para citação e recuperação; a versão 4.7 foi lançada em 3 de março de 2026\. ([DataCite Schema](https://schema.datacite.org/?utm_source=chatgpt.com))

## **10.4 Catalog Projection — DCAT**

Uso:

```
catálogo de datasets, experimentos, evidence bundles, releases e serviços.
```

Mapeamento:

```
Lab dataset registry → dcat:Catalog
Experiment dataset   → dcat:Dataset
Evidence artifact    → dcat:Distribution
App endpoint         → dcat:DataService
```

DCAT 3 é um vocabulário RDF do W3C para interoperabilidade entre catálogos de dados publicados na Web, permitindo descrever datasets e serviços de dados em um modelo padrão. ([W3C](https://www.w3.org/TR/vocab-dcat-3/?utm_source=chatgpt.com))

## **10.5 Article / Finding Projection — JATS**

Uso:

```
quando Findings, experiment reports ou papers do Lab precisam virar artigo estruturado.
```

Saída:

```
JATS XML
```

JATS é a Journal Article Tag Suite, aplicação do padrão NISO Z39.96-2024, com elementos e atributos para marcar artigos científicos e três modelos de artigo. ([JATS](https://jats.nlm.nih.gov/?utm_source=chatgpt.com))

## **10.6 Workflow Projection — CWL**

Uso:

```
quando um experimento computacional do Lab precisa virar workflow portável.
```

Mapeamento:

```
LogLine workflow projection → CWL Workflow
Tool call / worker          → CWL CommandLineTool
Inputs / outputs            → CWL inputs / outputs
Evidence artifacts          → output bindings
```

CWL é um padrão aberto para descrever ferramentas de linha de comando e workflows portáveis entre plataformas que suportam o padrão. ([Common Workflow Language (CWL)](https://www.commonwl.org/?utm_source=chatgpt.com))

## **10.7 Research Identity Projection — ORCID**

Uso:

```
quando pessoas/autores/contribuidores do Lab precisam ser identificados em contexto científico.
```

ORCID é um identificador persistente gratuito e único para indivíduos envolvidos em pesquisa, scholarship e atividades de inovação. ([ORCID](https://orcid.org/?utm_source=chatgpt.com))

## **10.8 FAIR Readiness Projection**

Uso:

```
avaliar se um experimento, dataset, receipt bundle ou pack é Findable, Accessible, Interoperable, Reusable.
```

Os princípios FAIR foram publicados para melhorar Findability, Accessibility, Interoperability e Reusability de ativos digitais, incluindo dados, algoritmos, ferramentas e workflows. ([GO FAIR](https://www.go-fair.org/fair-principles/?utm_source=chatgpt.com))

---

# **11\. Scientific Projection Pack**

Eu faria um pack oficial separado:

```
scientific-projections.pack/
```

Ele não é o Santo André Pack.  
Ele é uma biblioteca de projeções.

Estrutura:

```
scientific-projections.pack/
  pack.manifest.yaml

  projections/
    prov/
    ro-crate/
    datacite/
    dcat/
    jats/
    cwl/
    fair/

  templates/
    experiment.yaml
    hypothesis.yaml
    observation.yaml
    evidence-bundle.yaml
    finding.yaml
    benchmark.yaml
    replication.yaml
    negative-result.yaml

  conformance/
    prov-cases.yaml
    ro-crate-cases.yaml
    fair-cases.yaml

  README.md
```

Manifesto:

```
pack_id: scientific-projections
pack_version: 0.1.0
pack_type: projection_library

description: Standard scientific projection profiles for LogLine Labs.

requires:
  canon: logline-canon >=0.2.0
  lab_profile: logline.lab.v0

declares:
  projection_profiles:
    - w3c_prov
    - ro_crate
    - datacite
    - dcat
    - jats
    - cwl
    - fair_readiness

forbidden:
  - redefine_logline_act
  - alter_source_acts
  - claim_projection_as_truth
  - close_receipt
  - infer_missing_evidence

truth_policy:
  source_of_truth: logline_acts
  projection_is_truth: false
  include_source_hashes: true
```

Isso dá credibilidade.  
Quando alguém perguntar “qual é o formato científico?”, a resposta não é “inventei um JSON”. É:

```
Internamente: LogLine Acts.
Externamente: PROV, RO-Crate, DataCite, DCAT, JATS, CWL, FAIR.
```

---

# **12\. Pack de pesquisa do Lab**

Para preservar a tua parte preferida — a tupla como instrumento científico — eu criaria:

```
research-clock.pack/
```

Função:

```
ensinar um Lab a rodar experimentos recorrentes ao longo do dia.
```

Estrutura:

```
research-clock.pack/
  pack.manifest.yaml
  experiments/
    daily-small-experiment.yaml
    local-llm-quality-tick.yaml
    machine-health-cycle.yaml
    projection-drift-check.yaml
    ghost-review-cycle.yaml
  schedules/
    clock-cycle.yaml
    slow-compute-window.yaml
    overnight-runs.yaml
  projections/
    experiment-board.yaml
    findings.yaml
    anomaly-map.yaml
    replication-status.yaml
  receipt-profiles/
    experiment-run-receipt.yaml
    replication-receipt.yaml
  ghost-profiles/
    inconclusive-result.yaml
    missing-replication.yaml
    weak-evidence.yaml
```

O pack declara:

```
24h/day is the free supercomputer.
```

Mas como pack, não como canon.

A parte científica dos documentos fala exatamente disso: o Protocol Spine impede delírio, enquanto o Research Fire mantém a ambição de usar tuple/Act como instrumento de descoberta, workflows como estruturas descobertas, projeções como observatórios e ghosts/anomalias como sinais de pesquisa.

---

# **13\. O lugar no código onde tudo se encontra**

Eu criaria um crate Rust específico:

```
crates/labd-loadout/
```

Responsável por carregar e compor:

```
canon.lock
engine.lock
constitutional-runtime.lock
lab.manifest.yaml
lab.loadout.yaml
packs/*
apps/*
projection-profiles/*
```

Interface:

```rust
pub struct LabLoadout {
    pub kernel: KernelSelection,
    pub active_packs: Vec<PackSelection>,
    pub active_apps: Vec<AppSelection>,
    pub active_projection_profiles: Vec<ProjectionProfileSelection>,
    pub rules: LoadoutRules,
}

pub struct KernelSelection {
    pub canon_version: String,
    pub engine_version: String,
    pub constitutional_runtime_version: String,
    pub hash_profile: String,
    pub conformance_profile: String,
}

pub trait Loadable {
    fn id(&self) -> &str;
    fn version(&self) -> &str;
    fn kind(&self) -> LoadableKind;
    fn hash(&self) -> ContentHash;
    fn validate_against(&self, kernel: &KernelSelection) -> LoadoutValidationReport;
}

pub enum LoadableKind {
    InterpretationPack,
    ConnectionApp,
    ProjectionProfile,
}
```

E um outro:

```
crates/labd-extension-host/
```

Com subhosts:

```rust
pub trait PackRuntime {
    fn templates(&self) -> Vec<ActTemplate>;
    fn projections(&self) -> Vec<ProjectionDefinition>;
    fn schedules(&self) -> Vec<ScheduleDefinition>;
    fn experiments(&self) -> Vec<ExperimentDefinition>;
}

pub trait AppRuntime {
    fn tools(&self) -> Vec<ToolDescriptor>;
    fn resources(&self) -> Vec<ResourceDescriptor>;
    fn events(&self) -> Vec<EventDescriptor>;
    async fn call_tool(&self, call: ToolCall) -> Result<AppOutput, AppError>;
}

pub trait ProjectionRuntime {
    fn projection_id(&self) -> &str;
    fn project(&self, acts: ActStream) -> Result<ProjectionArtifact, ProjectionError>;
}
```

Mas todos obedecem:

```
nenhum host escreve LogLine Act direto;
todos retornam proposta, evidência, projection artifact ou operational output;
Constitutional Runtime e Engine continuam atravessando qualquer movimento semântico.
```

---

# **14\. A instalação inicial**

Quando alguém instala:

```shell
curl ... | sh
labd init my-lab
```

O que nasce:

```
my-lab/
  lab.manifest.yaml
  canon.lock
  engine.lock
  lab.loadout.yaml

  timeline/
    logline_acts.db

  blobs/

  packs/
    .empty/

  apps/
    .empty/

  projection-profiles/
    scientific-projections/

  conformance/
    reports/

  ui/
    minilab-work/
```

O Lab inicial responde:

```
qual canon está carregado?
qual engine está carregado?
qual hash profile está carregado?
qual timeline store está ativo?
quais packs estão ativos?
quais Apps estão ativos?
quais projeções estão ativas?
qual conformance passou?
```

Isso já estava nos documentos como mínimo de um Lab: não “ter UI”, mas responder qual canon está carregado, onde está a timeline, quais packs estão ativos, quais projeções derivam estado, como evidência entra, como receipt fecha, como ghost preserva ausência e como conformance é provada.

---

# **15\. O que vai “puro” no instalável**

Eu colocaria como puro:

```
logline-canon/
engine-main/
constitutional-runtime-main/
conformance/
hash-profiles/
act-store/
ruler/
projection-runtime-core/
pack-loader-core/
app-host-core/
```

Mas com a seguinte divisão:

```
Pure required:
- Act shape
- 9-slot validation
- 3-hash profile
- canonicalization
- engine walk
- confirmed_by pivot
- constitutional admission boundary
- timeline append-only
- conformance tests

Pure host:
- loadout
- pack loader
- app host
- projection host
- cli
- minilab shell

Loadable default:
- empty-lab.pack
- scientific-projections.pack
- demo.pack

Loadable optional:
- santo-andre-laboratory.pack
- research-clock.pack
- github-connection.app
- google-drive-connection.app
- local-files-connection.app
```

O Santo André pode estar na distribuição, mas desabilitado:

```
available, not active
```

Ativa com:

```shell
labd pack enable santo-andre-laboratory
```

---

# **16\. Como a comunidade participa**

A comunidade não manda no canon por pack/App.

Ela pode publicar:

```
packs
apps
projection profiles
benchmarks
conformance cases
examples
research reports
```

Isso combina com a gaveta “LogLine Dev Community”: implementações, adapters, RFCs, issues, benchmarks, examples e contributions, sem virar dona do protocolo.

Classificação de confiança:

```
local
= instalado manualmente, sem review.

community
= publicado em registry comunitário, com manifest e hash.

verified
= passou conformance técnica do tipo correspondente.

foundation-compatible
= não é aprovado como opinião, mas compatível com canon/profile.

deprecated
= ainda instalável, mas marcado como superseded.
```

Importante:

```
Foundation não aprova a opinião do pack.
Foundation só pode aprovar compatibilidade, conformance e segurança de fronteira.
```

---

# **17\. Registry de Packs e Apps**

Dois registries separados:

```
LogLine Pack Registry
LogLine App Registry
```

Porque eles têm naturezas diferentes.

## **Pack Registry**

Publica:

```
biology-lab.pack
hardware-bench.pack
finance-audit.pack
school-governance.pack
santo-andre-laboratory.pack
research-clock.pack
```

Campos:

```
pack_id:
version:
type:
domain:
requires:
declares:
forbidden:
conformance:
source:
license:
authors:
hash:
signature:
```

## **App Registry**

Publica:

```
github-connection.app
google-drive-connection.app
local-files-connection.app
calendar-connection.app
shell-readonly.app
microscope-sensor.app
llm-local.app
```

Campos:

```
app_id:
version:
connection_type:
tools:
resources:
events:
auth:
security:
outputs:
widgets:
act_mappers:
conformance:
source:
license:
authors:
hash:
signature:
```

Apps e Packs podem ser combinados, mas não fundidos.

Exemplo:

```
github-connection.app
= conecta GitHub.

software-lab.pack
= interpreta issues, PRs, commits e releases como experimentos/workflows/receipts.
```

---

# **18\. O App não transforma o Lab em workspace de agentes**

Essa frase precisa estar na spec:

```
The LogLine Lab Connection App SDK is not an agent workspace SDK.
It is a controlled connection SDK for exposing external tools, resources, events, and widgets to a LogLine Lab without granting semantic authority.
```

Em português:

```
O LogLine Lab Connection App SDK não transforma o Lab em workspace de agentes.
Ele conecta ferramentas, recursos, eventos e widgets externos a um Lab LogLine sem conceder autoridade semântica.
```

Agentes podem existir como Apps ou runtimes, mas:

```
agent output = proposal or evidence
not truth
```

Isso segue os colapsos proibidos dos docs: LLM output não é truth, scheduler não é authority, dispatch não é execution, worker não é judge, evidence não é receipt, projection não é canon, UI state não é timeline truth e database row não é autoridade semântica.

---

# **19\. Rigor científico: como não virar “brinquedo bonito”**

O rigor vem de quatro camadas:

```
1. LogLine interno:
   Act, hashes, timeline, receipts, ghosts, conformance.

2. Engine discipline:
   toda mudança semântica passa pelo Engine.

3. Scientific projection:
   exporta para padrões reconhecidos.

4. Reproducibility:
   experimento vira pacote reexecutável, citável e auditável.
```

A interface pode ser linda, mas cada card importante precisa responder:

```
Source Acts?
Evidence?
Receipt?
Ghost?
Projection profile?
Standard export?
```

Cada experimento deve ter:

```
hypothesis:
question:
variables:
method:
schedule:
inputs:
tools:
expected_evidence:
if_ok:
if_doubt:
if_not:
receipt_profile:
projection_profiles:
  - prov
  - ro-crate
  - fair_readiness
```

Cada resultado deve exportar:

```
- LogLine Act chain
- PROV provenance
- RO-Crate bundle
- DataCite-ready metadata, quando publicável
- FAIR readiness report
- JATS finding/article, quando houver texto científico
- CWL workflow, quando houver pipeline computacional
```

Assim o Lab não inventa “paperzinho próprio”. Ele gera objetos que conversam com infraestrutura científica real.

---

# **20\. Decisão final de arquitetura**

A versão limpa é:

```
LogLine Lab Installable
= kernel puro + host carregável.

Kernel puro
= canon, engine, constitutional runtime, conformance, hash profile, timeline discipline.

Pack
= regras locais, interpretação, método, vocabulário, rotina, experimentos, projeções.

App
= conectividade com ferramentas, fontes, recursos, eventos e widgets.

Projection Profile
= exportação para padrões científicos e técnicos reconhecidos.

minilab.work
= cockpit humano que mostra e opera tudo isso sem virar autoridade.

Santo André
= pack/lab experimental carregável, não canon.

voulezvous
= operação real usando o Lab, não dono do protocolo.
```

A frase de repo:

```
This installable ships pure LogLine canon and engines.
Everything else is loadable interpretation, connectivity, or projection.
```

Em português:

```
Este instalável traz canon e engines LogLine puros.
Todo o resto é interpretação, conectividade ou projeção carregável.
```

E a frase de governança:

```
A pack may teach a Lab how to see.
An App may connect a Lab to the world.
A projection may translate a Lab for science.
Only LogLine canon defines what an Act is.
```

Em português:

```
Um pack pode ensinar um Lab a enxergar.
Um App pode conectar um Lab ao mundo.
Uma projeção pode traduzir um Lab para a ciência.
Só o canon LogLine define o que é um Act.
```

According to a document from 2026-06-08, a arquitetura já estava caminhando para três forças centrais — **Constitutional Runtime**, **LogLine Engine** e **Ruler / Clock / Temporal Driver** — e para a tese de que o sistema deve começar pelo metabolismo vivo, não por dashboard, SaaS ou CRUD.

A defesa final é esta:

# **O projeto tem 3 partes**

```
1. LogLine Lab Installable
   O núcleo puro instalável.

2. LogLine Packs
   As regras, interpretações, métodos e rotinas carregáveis.

3. LogLine Connection Apps + Projection Profiles
   As conexões externas e as saídas para padrões científicos/técnicos.
```

Ou em uma frase:

```
O Installable carrega a lei.
O Pack ensina o Lab a enxergar.
O App conecta o Lab ao mundo.
As Projections traduzem o que o Lab viu para padrões externos.
```

As projeções científicas entram na terceira parte porque elas não são canon e também não são interpretação local. Elas são **formatos de exportação/interoperabilidade**. O Lab continua tendo LogLine Acts como verdade; PROV, RO-Crate, DataCite, DCAT, JATS, CWL e FAIR são formas respeitáveis de apresentar essa verdade para fora.

---

# **1\. Parte um — LogLine Lab Installable**

## **O que é**

O **LogLine Lab Installable** é o produto técnico de base.

Ele instala um Lab vazio, puro, conformante e capaz de carregar coisas depois.

```
logline-lab install
→ instala canon
→ instala engines
→ instala conformance
→ instala act store
→ instala ruler
→ instala projection runtime
→ instala pack loader
→ instala app host
→ sobe minilab.work como cockpit opcional
```

Ele vem com:

```
- LogLine canon puro;
- LogLine Act;
- 9 slots;
- 3 hashes;
- canonicalization profile;
- conformance suite;
- engine-main real;
- constitutional-runtime-main real;
- Ruler / Clock / Temporal Driver;
- timeline append-only;
- projection runtime;
- pack loader;
- app host;
- CLI;
- API;
- experiência inicial mínima.
```

Ele **não** vem com a cosmologia do Santo André como obrigação universal.

Ele pode trazer `santo-andre.pack` disponível, mas desabilitado.

```
available:
  - santo-andre-laboratory.pack

enabled:
  - none
```

Ativar Santo André é decisão de loadout:

```shell
labd pack enable santo-andre-laboratory
```

## **Por que foi decidido assim**

Porque o projeto precisa sobreviver a três riscos:

```
1. minilab.work virar protocolo;
2. Santo André virar canon obrigatório;
3. Dan virar dependência semântica.
```

A separação institucional já tinha aparecido como estratégia de reputação: Foundation guarda canon mínimo, LogLine permanece protocolo, Lab Kit permite Labs independentes, Santo André é escola experimental, minilab.work é interface, voulezvous é operação real. Essa separação permite que alguém use LogLine sem minilab.work, construa um Lab sem seguir Santo André e passe conformance sem depender de Dan.

Então a defesa é:

```
O instalável precisa ser puro para que LogLine seja infraestrutura,
não uma demo bonita, nem uma empresa, nem uma escola pessoal.
```

## **Como funciona**

A execução básica:

```
Human / LLM / App / Sensor
→ proposed LogLine Act
→ Constitutional Runtime
→ admitted / rejected / ghost
→ logline_acts append-only
→ Ruler observa tempo
→ Engine admite transição
→ dispatch Act
→ projection_queue
→ worker
→ evidence Act
→ receipt ou ghost Act
→ projection
→ UI
```

O documento do runtime já descreve a separação correta: o Constitutional Runtime transforma intenção em candidato/admissão; a Régua transforma tempo em urgência; e o Engine transforma urgência em transição admissível.

A regra técnica:

```
Tudo que é verdade durável é LogLine Act.
Todo o resto é projeção, cache, fila, blob, índice ou UI.
```

## **Detalhe de build**

Repo base:

```
logline-lab/
  kernel/
    canon/
    engine-main/
    constitutional-runtime-main/
    conformance/
    hash-profiles/

  crates/
    labd-core/
    labd-act-store/
    labd-engine-bridge/
    labd-constitutional-bridge/
    labd-ruler/
    labd-projections/
    labd-pack-runtime/
    labd-app-host/
    labd-experience/
    labd-api/
    labd-cli/

  apps/
    labd/

  ui/
    minilab-work/

  loadables/
    packs/
    apps/
    projection-profiles/

  standards/
    prov/
    ro-crate/
    datacite/
    dcat/
    jats/
    cwl/
    fair/
```

A tabela canônica principal:

```sql
create table logline_acts (
  content_hash text primary key,
  tuple_hash text not null,
  envelope_hash text not null,

  canonicalization_profile text not null,
  hash_algorithm text not null,

  canonical_bytes bytea not null,
  act_json jsonb not null,
  envelope_json jsonb not null,

  received_at timestamptz not null default now(),

  who text generated always as (act_json->>'who') stored,
  did text generated always as (act_json->>'did') stored,
  "when" text generated always as (act_json->>'when') stored,
  status text generated always as (act_json->>'status') stored
);
```

O banco pode ter projeções:

```
projection_today
projection_timeline
projection_queue
projection_registry
projection_receipts
projection_ghosts
projection_experiments
projection_runtime
```

Mas essas projeções são apagáveis e reconstruíveis.

```
drop projection_today;
rebuild from logline_acts;
same result.
```

## **Manutenção**

A manutenção do instalável é conservadora.

Mudanças em `kernel/` exigem:

```
- protocol decision;
- conformance update;
- hash/canonicalization tests;
- migration note;
- version bump;
- compatibility statement;
- deprecation path.
```

Mudanças em `host/` exigem:

```
- integration tests;
- replay tests;
- projection rebuild tests;
- security review;
- no semantic authority expansion.
```

Mudanças em `ui/` exigem:

```
- UX review;
- projection trace still visible;
- no UI state as truth;
- accessibility;
- onboarding path preserved.
```

Um `CODEOWNERS` sério materializa jurisdição:

```
/kernel/canon/                      @logline-foundation/canon-maintainers
/kernel/conformance/                @logline-foundation/conformance-maintainers
/kernel/engine-main/                @logline-foundation/engine-maintainers
/crates/labd-act-store/             @logline-lab-kit/core-maintainers
/crates/labd-ruler/                 @logline-lab-kit/runtime-maintainers
/crates/labd-pack-runtime/          @logline-lab-kit/pack-maintainers
/crates/labd-app-host/              @logline-lab-kit/app-host-maintainers
/loadables/packs/santo-andre/       @santo-andre-lab/operators
/ui/minilab-work/                   @minilab-work/product-maintainers
```

Isso segue a tese já registrada: cada path precisa declarar jurisdição; canon change não é pack change, pack change não é UI change, e Santo André não manda na Foundation.

---

# **2\. Parte dois — LogLine Packs**

## **O que é**

Um **Pack** é uma lente carregável.

```
Pack = regra local + interpretação + método + vocabulário + rotina + projeção + experimento.
```

Ele ensina um Lab a interpretar um domínio.

Exemplos:

```
santo-andre-laboratory.pack
research-clock.pack
scientific-projections.pack
software-lab.pack
hardware-bench.pack
finance-audit.pack
school-governance.pack
```

Um pack pode declarar:

```
- vocabulários;
- templates de Acts;
- selectors sobre Acts;
- rotinas;
- schedules;
- experiments;
- receipt profiles;
- ghost rules;
- projections;
- labels de UI;
- onboarding;
- conformance cases;
- scientific projection defaults.
```

Um pack não pode:

```
- redefinir Act;
- criar décimo slot;
- mudar hash;
- mudar canonicalization;
- admitir Act sozinho;
- fechar receipt;
- executar worker diretamente;
- escrever timeline sem Engine;
- declarar projeção como verdade;
- virar canon global.
```

Essa já era a linha certa: o pack é linguagem declarativa de interpretação para Labs; pode declarar vocabulários, templates, selectors, projeções, processos, schedules, receipt profiles, ghost rules, labels e casos de conformance, mas não pode redefinir Act, hash, receipt, worker ou canon.

## **Por que foi decidido assim**

Porque a parte mais viva do projeto — Santo André como Lab-relógio, timeline como tempo da Terra, tupla como microscópio científico, weak compute \+ strong timeline — é importante demais para virar “apenas UI”, mas específica demais para virar canon universal.

Então a defesa é:

```
A Foundation publica a forma mínima.
Cada Lab carrega sua interpretação como pack.
```

Ou:

```
A Foundation não define o significado de todo Lab.
A Foundation define a forma segura para cada Lab declarar seu significado.
```

Esse princípio já apareceu claramente: a Foundation não precisa carregar toda a metáfora do Santo André; ela precisa publicar o canon mínimo, e cada Lab projeta sua própria cosmologia por meio de packs.

## **Como funciona**

O Lab tem um arquivo de composição ativa:

```
lab.loadout.yaml
```

Exemplo:

```
loadout_id: santo-andre-local-2026-06
lab_id: lab:santo-andre.local

kernel:
  canon: logline-canon@0.2.0
  engine: engine-main@0.2.0
  constitutional_runtime: constitutional-runtime-main@0.2.0
  hash_profile: logline.hash.sha256-jcs.v0
  conformance_profile: logline.core.v0

active_packs:
  - id: santo-andre-laboratory
    version: 0.1.0
    hash: sha256:...
  - id: research-clock
    version: 0.1.0
    hash: sha256:...

active_apps:
  - id: github-connection
    version: 0.1.0
    hash: sha256:...

active_projection_profiles:
  - id: w3c-prov
  - id: ro-crate
  - id: datacite
  - id: fair-readiness

rules:
  conflict_policy: require_explicit_resolution_act
  app_output_policy: app_outputs_are_proposals_or_evidence_only
  projection_policy: projections_are_never_truth
```

Ativar pack também vira Act:

```
who: operator:dan
did: enabled_pack
this:
  pack: santo-andre-laboratory@0.1.0
  loadout: lab.loadout.yaml
when: 2026-06-09T...
confirmed_by:
  pack_hash: sha256:...
  conformance_report: act:<hash>
if_ok: activate_pack
if_doubt: keep_pack_disabled
if_not: reject_pack
status: enabled
```

O arquivo muda o comportamento operacional.  
A timeline preserva a memória institucional.

## **Template de pack**

```
santo-andre-laboratory.pack/
  pack.manifest.yaml
  vocabulary.yaml

  act-templates/
    daily-lab-check.yaml
    machine-health-probe.yaml
    local-llm-quality-tick.yaml

  processes/
    morning-routine.yaml
    experiment-cycle.yaml
    ghost-review.yaml

  schedules/
    lab-clock.yaml
    daily-ruler.yaml
    overnight-runs.yaml

  experiments/
    backup-probe.yaml
    peer-link-check.yaml
    projection-drift-check.yaml

  projections/
    today.yaml
    experiment-board.yaml
    machine-registry.yaml
    findings.yaml

  receipt-profiles/
    machine-health-receipt.yaml
    experiment-run-receipt.yaml

  ghost-profiles/
    missing-evidence.yaml
    inconclusive-result.yaml
    unresolved-authority.yaml

  ui/
    labels.yaml
    cards.yaml
    onboarding.yaml

  conformance/
    cases.yaml

  README.md
```

Manifesto:

```
pack_id: santo-andre-laboratory
pack_version: 0.1.0
pack_type: interpretation
status: experimental

requires:
  canon: logline-canon >=0.2.0
  engine: engine-main >=0.2.0
  lab_profile: logline.lab.v0

declares:
  vocabularies:
    - people
    - machines
    - workflows
    - policies
    - experiments
    - local_llms

  act_templates:
    - daily_lab_check
    - machine_health_probe
    - local_llm_quality_tick

  schedules:
    - daily_lab_clock
    - hourly_machine_heartbeat
    - weekly_findings_review

  projections:
    - today
    - machine_registry
    - experiment_board
    - findings
    - ghosts_due_for_review

forbidden:
  - redefine_9_slots
  - redefine_hash_profile
  - bypass_engine
  - mutate_timeline_directly
  - treat_projection_as_truth
  - close_receipt_without_evidence
  - execute_worker_directly
  - grant_app_authority
```

## **Manutenção**

Manutenção de pack tem outro regime, mais leve que canon:

```
pack change
→ pack version bump
→ pack conformance cases
→ projection rebuild test
→ conflict check against active loadout
→ changelog
→ optional community publication
```

O pack pode evoluir rápido.

O canon não.

Essa diferença é essencial. O Lab precisa poder aprender com experimentos sem reabrir a lei do protocolo toda semana.

---

# **3\. Parte três — Connection Apps \+ Projection Profiles**

Esta terceira parte tem duas famílias, mas a mesma natureza: **ponte para fora do núcleo**.

```
Apps conectam ferramentas ao Lab.
Projection Profiles traduzem o Lab para padrões externos.
```

Elas ficam juntas porque ambas são interfaces com o mundo externo. A diferença é:

```
App = entrada/ação/conectividade.
Projection Profile = saída/exportação/interoperabilidade.
```

---

## **3.1 Connection Apps**

## **O que é**

Um **Connection App** é uma ponte controlada para uma ferramenta externa.

```
App = conexão, recurso, evento, tool, widget, autenticação, limite.
```

Exemplos:

```
github-connection.app
google-drive-connection.app
local-files-connection.app
calendar-connection.app
email-connection.app
shell-readonly.app
local-llm-connection.app
microscope-sensor.app
build-system.app
documentation.app
```

Ele pode:

```
- expor tools;
- expor resources;
- receber eventos;
- retornar observações;
- retornar evidência;
- retornar blob refs;
- propor Acts;
- mostrar widget no minilab.work.
```

Ele não pode:

```
- admitir Act;
- fechar receipt;
- resolver ghost;
- mudar canon;
- escrever timeline diretamente;
- alterar pack rules;
- fingir autoridade semântica.
```

## **Por que foi decidido assim**

Porque o Lab precisa estar perto das ferramentas reais — build, documentação, repositórios, sensores, arquivos, LLMs, ambientes científicos — sem virar um “workspace de agentes”.

A defesa:

```
O App SDK não existe para transformar o Lab em agente.
Existe para deixar ferramentas externas aparecerem perto da LogLine sem ganharem autoridade LogLine.
```

A analogia com o Apps SDK é boa porque a documentação oficial da OpenAI descreve Apps como servidores MCP que definem capacidades/tools, podem renderizar componentes em iframe e mantêm servidor, modelo e UI sincronizados por formato, autenticação e metadata padronizados. ([OpenAI Developers](https://developers.openai.com/apps-sdk/quickstart?utm_source=chatgpt.com)) ([OpenAI Developers](https://developers.openai.com/apps-sdk/concepts/mcp-server?utm_source=chatgpt.com))

No LogLine Lab, a tradução é:

```
App server
→ expõe ferramentas e eventos.

Lab App Host
→ autentica, limita, chama, audita.

Act Mapper
→ transforma output técnico em proposed Act ou evidence Act.

Constitutional Runtime
→ decide se proposta vira Act admissível.

Engine
→ move estado se houver transição.

Projection
→ mostra o resultado.
```

## **Template de App**

```
github-connection.app/
  app.manifest.yaml
  tools.yaml
  resources.yaml
  events.yaml
  auth.yaml
  act-mappers.yaml
  evidence-profiles.yaml

  widgets/
    repository-card/
    pull-request-card/

  conformance/
    cases.yaml

  README.md
```

Manifesto:

```
app_id: github-connection
app_version: 0.1.0
app_type: connection
sdk: logline.lab.connection-app.v0

name: GitHub Connection
description: Connects a LogLine Lab to GitHub repositories, issues, pull requests, commits, checks, and releases.

capabilities:
  tools:
    - github.search_repositories
    - github.read_issue
    - github.read_pull_request
    - github.read_commit
    - github.read_workflow_run

  resources:
    - repository
    - issue
    - pull_request
    - commit
    - workflow_run

  events:
    - pull_request.opened
    - pull_request.merged
    - workflow_run.completed

auth:
  type: oauth2
  scopes:
    - repo:read
    - issues:read
    - pull_requests:read
    - checks:read
  secrets:
    storage: lab.secret_store
    never_send_to_llm: true

outputs:
  allowed_output_kinds:
    - observation
    - evidence
    - proposed_act
    - blob_ref
  forbidden_output_kinds:
    - admitted_act
    - receipt_closed
    - canon_change
    - direct_timeline_write

act_mappers:
  - id: pull_request_merged_to_observation
    input_event: pull_request.merged
    output: proposed_act
    constitutional_runtime_required: true

security:
  allowed_actions:
    - read_repository_metadata
    - read_pull_request
    - read_workflow_run
  forbidden_actions:
    - mutate_timeline
    - close_receipt
    - change_pack_rules
    - change_canon
    - execute_shell

conformance:
  cases: conformance/cases.yaml
```

## **Manutenção de Apps**

Regime de manutenção:

```
app update
→ manifest compatibility check
→ permission diff
→ security review
→ tool contract test
→ output-kind test
→ no-authority test
→ widget sandbox test
→ changelog
```

Mudança perigosa:

```
read-only → read-write
```

exige Act explícito e revisão de loadout.

```
who: operator:dan
did: approved_app_permission_change
this:
  app: github-connection@0.2.0
  permission_change:
    from: repo:read
    to: repo:write
when: <now>
confirmed_by:
  operator_signature: sig:<...>
if_ok: enable_new_scope
if_doubt: keep_previous_scope
if_not: disable_app
status: approved
```

---

## **3.2 Projection Profiles**

## **O que é**

Um **Projection Profile** traduz a timeline LogLine para formatos reconhecidos.

```
Projection Profile = exportação, não verdade.
```

Ele não muda o Lab.  
Ele não fecha receipt.  
Ele não cria evidência.  
Ele não reinterpreta canon.

Ele pega:

```
LogLine Acts
+ blobs
+ evidence
+ receipts
+ ghosts
+ experiment metadata
```

e exporta para:

```
PROV
RO-Crate
DataCite
DCAT
JATS
CWL
FAIR readiness
```

## **Por que foi decidido assim**

Porque rigor científico não nasce de “inventamos nosso próprio JSON bonito”.

A decisão correta é:

```
Internamente: LogLine Act.
Externamente: padrões internacionais óbvios.
```

W3C PROV existe justamente para representar e intercambiar proveniência entre sistemas e contextos; RO-Crate usa JSON-LD e Schema.org para empacotar dados de pesquisa e metadata; DataCite Metadata Schema fornece propriedades centrais para identificação consistente de recursos para citação e recuperação, com Schema 4.7 publicado em 2026\. ([W3C](https://www.w3.org/TR/prov-o/?utm_source=chatgpt.com)) ([researchobject.org](https://www.researchobject.org/ro-crate/?utm_source=chatgpt.com)) ([DataCite Schema](https://schema.datacite.org/?utm_source=chatgpt.com))

DCAT 3 é vocabulário RDF do W3C para interoperabilidade entre catálogos de dados; JATS é a Journal Article Tag Suite baseada no padrão NISO Z39.96-2024; CWL é padrão aberto para descrever ferramentas de linha de comando e workflows portáveis; FAIR foca tornar dados findable, accessible, interoperable e reusable. ([W3C](https://www.w3.org/TR/vocab-dcat-3/?utm_source=chatgpt.com)) ([JATS](https://jats.nlm.nih.gov/?utm_source=chatgpt.com)) ([Common Workflow Language (CWL)](https://www.commonwl.org/?utm_source=chatgpt.com)) ([GO FAIR](https://www.go-fair.org/fair-principles/?utm_source=chatgpt.com))

A defesa:

```
LogLine não precisa substituir a infraestrutura científica.
LogLine precisa produzir rastros que a infraestrutura científica reconhece.
```

## **Template de Projection Profile**

```
ro-crate-experiment-export.projection/
  projection.manifest.yaml
  mapping.yaml
  jsonld-context.json
  examples/
  conformance/
  README.md
```

Manifesto:

```
projection_id: ro-crate-experiment-export
projection_version: 0.1.0
projection_type: scientific_export
standard: RO-Crate
standard_version: "1.2"

input:
  source: logline_acts
  selectors:
    - did: declared_experiment
    - did: ran_experiment
    - did: reported_execution
    - did: closed_receipt
    - status: ghost

output:
  format: json-ld
  root_file: ro-crate-metadata.json
  directory_layout: ro-crate/

mapping:
  experiment: schema:Dataset
  act: schema:Action
  actor: schema:Person | schema:SoftwareApplication | schema:Organization
  evidence_blob: schema:MediaObject
  receipt: schema:CreativeWork
  ghost: schema:Thing

truth_policy:
  projection_is_truth: false
  source_of_truth: logline_acts
  include_source_act_hashes: true
  include_tuple_hashes: true
  include_content_hashes: true
  include_envelope_hashes: true

conformance:
  validate_jsonld: true
  validate_required_metadata: true
  cases: conformance/cases.yaml
```

## **Manutenção de Projection Profiles**

Regime:

```
projection profile update
→ standard version check
→ mapping diff
→ golden export test
→ source hash inclusion test
→ no-truth-claim test
→ roundtrip/reference validation where possible
→ changelog
```

Uma projeção científica só é aceitável se sempre carregar:

```
- source Act hashes;
- tuple_hash;
- content_hash;
- envelope_hash;
- projection version;
- standard version;
- export timestamp;
- statement that projection is not canonical truth.
```

---

# **Como as 3 partes trabalham juntas**

## **Exemplo completo**

Um App observa um evento no GitHub:

```
pull_request.merged
```

O App não escreve timeline. Ele retorna:

```json
{
  "kind": "proposed_act",
  "source": "github-connection",
  "payload": {
    "repository": "danvoulez/logline-lab",
    "pull_request": 42,
    "merged_at": "..."
  }
}
```

O Pack ativo sabe interpretar isso:

```
software-lab.pack
→ PR merged can mean code change candidate
```

O Constitutional Runtime avalia:

```
essa observação pode virar Act?
há autoridade?
há evidência?
há escopo?
```

Se admitido, vira LogLine Act:

```
who: github-connection.app
did: observed_merge
this:
  repository: danvoulez/logline-lab
  pull_request: 42
  commit: sha256:...
when: <merge_time>
confirmed_by:
  github_api_response: blob:<hash>
if_ok: submit_to_code_review_projection
if_doubt: request_repository_evidence
if_not: reject_untrusted_event
status: observed
```

A Régua pode depois detectar um schedule:

```
este merge precisa de conformance check
```

O Engine admite transição:

```
who: logline.engine
did: requested_execution
this:
  source_act: act:<merge_observation>
  worker_kind: conformance_runner
when: <now>
confirmed_by:
  engine_walk: act:<hash>
if_ok: queue_for_worker
if_doubt: preserve_dispatch_doubt
if_not: block_execution
status: queued
```

Worker roda.

Evidência volta.

Receipt ou ghost fecha.

Projection profile exporta:

```
PROV:
  who did what using which evidence.

RO-Crate:
  experiment bundle with metadata and artifacts.

DataCite:
  citable dataset / release metadata if publishable.

FAIR:
  readiness report.
```

Tudo isso acontece sem violar a regra:

```
Só LogLine Act é verdade semântica durável.
```

---

# **Defesa da decisão**

## **Por que não tudo em um sistema só?**

Porque isso recria a sopa:

```
UI mandando no canon.
App virando autoridade.
Pack virando protocolo.
Santo André virando obrigação universal.
Worker fechando receipt.
Projection fingindo ser verdade.
```

Os documentos já registram os colapsos proibidos: `LLM output = truth`, `operator output = admitted Act`, `dispatch = execution`, `worker = judge`, `evidence = receipt`, `projection = canon`, `UI state = timeline truth`, `database row = semantic authority`.

Separar em três partes cria contenção:

```
Installable
→ governa o que é estável.

Pack
→ permite interpretação viva sem corromper canon.

App / Projection
→ permite mundo externo e ciência sem entregar autoridade semântica.
```

## **Por que não colocar Santo André no core?**

Porque Santo André é precioso, mas opinativo.

```
Santo André = Lab como relógio experimental.
Foundation = forma mínima.
```

A parte viva — “weak compute \+ strong timeline”, “24h por dia como supercomputador gratuito”, “tupla como microscópio científico” — deve virar pack, pesquisa e escola. Não deve virar requisito universal.

Isso aumenta a chance de adoção:

```
alguém pode usar LogLine sem acreditar na cosmologia Santo André;
alguém pode criar outro Lab completamente diferente;
alguém pode passar conformance sem usar minilab.work;
alguém pode construir App sem tocar no canon.
```

## **Por que Pack e App separados?**

Porque eles respondem perguntas diferentes:

```
Pack:
  Como este Lab interpreta os Acts?

App:
  A que ferramentas este Lab consegue se conectar?
```

Misturar os dois cria risco.

Se um App puder declarar regras, GitHub ou Google Drive poderiam virar “autoridade de sentido”.

Se um Pack puder chamar ferramenta diretamente, regra local viraria execução externa sem fronteira.

A separação mantém:

```
Pack = significado local.
App = contato externo.
Engine = movimento semântico.
Constitutional Runtime = admissão.
Timeline = memória.
```

## **Por que Projection Profiles separados de Pack?**

Porque projeção científica precisa ser interoperável e auditável.

Um pack pode escolher quais projeções usar, mas não deve inventar “padrão científico próprio” quando existem padrões maduros.

A regra:

```
Pack escolhe a lente.
Projection Profile implementa a exportação.
Standard externo define o formato.
```

---

# **Build order recomendado**

## **Fase 1 — Installable puro**

```
1. logline_acts table
2. canonical bytes + 3 hashes
3. engine bridge
4. constitutional bridge
5. append-only store
6. projection runtime
7. ruler tick
8. queue projection
9. worker read-only
10. receipt/ghost as Acts
11. conformance runner
```

Critério:

```
Sem packs ativos, o Lab ainda consegue criar, admitir, preservar, mover e projetar Acts.
```

## **Fase 2 — Loadout e Pack Runtime**

```
1. lab.loadout.yaml
2. pack.manifest.yaml parser
3. pack validator
4. template loader
5. projection selector loader
6. schedule loader
7. receipt/ghost profile loader
8. pack conformance tests
9. santo-andre-laboratory.pack mínimo
10. research-clock.pack mínimo
```

Critério:

```
Desabilitar todos os packs devolve o Lab ao estado puro.
Habilitar um pack muda interpretações e projeções, não o canon.
```

## **Fase 3 — App Host**

```
1. app.manifest.yaml parser
2. permission model
3. auth/secrets boundary
4. tool registry
5. resource registry
6. event ingestion
7. act mapper
8. evidence mapper
9. widget sandbox
10. app conformance tests
```

Critério:

```
Um App consegue observar e propor.
Nenhum App consegue admitir, fechar receipt ou escrever timeline diretamente.
```

## **Fase 4 — Scientific Projection Profiles**

```
1. W3C PROV exporter
2. RO-Crate exporter
3. DataCite metadata exporter
4. DCAT catalog exporter
5. JATS finding exporter
6. CWL workflow exporter
7. FAIR readiness report
8. golden export tests
```

Critério:

```
Toda exportação inclui source Acts e hashes.
Toda exportação declara que é projeção.
Nenhuma exportação substitui timeline.
```

## **Fase 5 — minilab.work Experience**

```
1. Start Lab
2. Load Pack
3. Connect App
4. Create first Act
5. Watch Today
6. See Ruler tick
7. See Engine transition
8. See Queue
9. See Evidence
10. See Receipt/Ghost
11. Export scientific projection
```

Critério:

```
A primeira experiência do usuário não é tour.
É atravessar o metabolismo inteiro.
```

---

# **Manutenção contínua**

## **Release channels**

```
kernel releases
  slow, conservative, conformance-heavy

pack releases
  faster, experimental, domain-specific

app releases
  security-heavy, permission-diff driven

projection releases
  standard-version driven

ui releases
  product/experience driven
```

## **Compatibilidade**

Todo release declara:

```
compatibility:
  canon:
  engine:
  constitutional_runtime:
  lab_profile:
  pack_profile:
  app_sdk:
  projection_runtime:
```

## **Conformance gates**

```
Kernel:
  must pass protocol conformance.

Pack:
  must not redefine canon.
  must pass pack conformance.

App:
  must not claim authority.
  must pass permission/output tests.

Projection:
  must include source hashes.
  must pass golden export tests.

UI:
  must preserve traceability.
  must not hide projection/truth boundary.
```

## **Incident classes**

```
canon incident
  hash/canonicalization/Act validity issue.

engine incident
  wrong transition/admission.

pack incident
  bad interpretation, unsafe rule, projection drift.

app incident
  secret leak, permission escalation, false evidence.

projection incident
  misleading export, missing source hash, broken standard mapping.

ui incident
  projection shown as truth, trace hidden, unsafe action.
```

Cada incidente vira Act.

---

# **A versão de defesa curta**

```
O projeto foi dividido em três partes porque LogLine precisa ser puro o bastante para virar infraestrutura, vivo o bastante para gerar Labs diferentes, e aberto o bastante para conversar com ferramentas e ciência externa.

O Installable protege o núcleo: canon, engines, timeline, hashes, conformance e runtime mínimo.

Os Packs protegem a imaginação: cada Lab pode carregar sua interpretação, rotina, método, experimento e projeção sem transformar isso em canon.

Os Apps e Projection Profiles protegem a abertura: ferramentas externas entram por conexões limitadas; resultados saem por padrões científicos reconhecidos; nada disso ganha autoridade semântica.

Essa divisão reduz risco técnico, reputacional e científico. Ela impede que UI vire canon, que App vire juiz, que Pack vire protocolo, que worker feche receipt, que projeção vire verdade e que Santo André vire obrigação universal.

O Lab fica instalável, extensível, científico e sustentável.
```

---

# **A frase final do projeto**

```
LogLine Lab is a pure installable core with loadable meaning and controlled connectivity.

The core keeps truth.
Packs teach interpretation.
Apps connect tools.
Projection profiles speak science.

Everything durable is a LogLine Act.
Everything else is loadable, replaceable, inspectable, and rebuildable.
```

Em português:

```
O LogLine Lab é um núcleo puro instalável com significado carregável e conectividade controlada.

O núcleo guarda a verdade.
Os packs ensinam interpretação.
Os Apps conectam ferramentas.
As projeções falam com a ciência.

Tudo que é durável é LogLine Act.
Todo o resto é carregável, substituível, inspecionável e reconstruível.
```

Isso é defendível, buildável e mantível.

