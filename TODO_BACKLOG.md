# TODO Backlog

This document catalogs the outstanding `TODO` comments in the codebase as of 2026-07-03, grouped by theme.
It exists so that the remaining `TODO`s are visible in one place and can be triaged into issues or fixed deliberately.
Line numbers drift as files change — search for the quoted text rather than relying on the line number.

Intentional placeholders are excluded: `TODO: set your AWS region`-style markers in `docs/` and `examples/` are
instructions for users filling in templates, not tech debt. Likewise, the `TODO (if needed)` markers in the
Gemini providers (`gcp_vertex_gemini`, `google_ai_studio_gemini`) document optional provider API surface that is
deliberately unimplemented until someone needs it.

## Cross-cutting themes (issue-tagged)

These recurring TODOs reference a single tracking issue and span many files. They are best fixed as one sweep each.

| Issue | Theme | Sites |
| ----- | ----- | ----- |
| #3983 | Audit error-handling callsites | ~20 sites across `provider-proxy`, `tensorzero-http`, `mock-provider-api`, and e2e tests |
| #6664 | Make evaluation/feedback lookups order-independent instead of sorting keys or requiring a particular order | `db/evaluation_queries.rs`, `db/postgres/inference_queries.rs` (x2), `endpoints/internal/evaluations/get_human_feedback.rs`, plus 5 e2e test sites |
| #5691 | Align types between ClickHouse and Postgres; database isolation in tests; observability config cleanup | `db/postgres/feedback.rs`, `db/postgres/inference_queries.rs`, `db/model_inferences.rs`, `endpoints/datasets/v1/update_datapoints.rs`, `utils/gateway.rs`, `ui/e2e_tests/observability.inferences.inference_id.spec.ts` |
| #6441 | Implement proper search relevance ordering for Postgres | `db/postgres/dataset_queries.rs`, `db/postgres/inference_queries.rs` (x2), `endpoints/datasets/v1/types.rs`, `tensorzero-types/src/inference_filters.rs` |
| #2642 | Improve SFT optimizer error handling so the failing example index is reported | `gcp_vertex_gemini_sft.rs`, `together_sft.rs`, `openai_sft.rs`, `openai_rft.rs` in `tensorzero-optimizers` |
| #857  | Implement closing the `reqwest` connection pool in the Python client | `tensorzero-python/src/lib.rs` (4 sites) |
| #4259 | Remove the `tokio::spawn` guard in the Python client | `tensorzero-python/src/lib.rs` (2 sites) |
| #5270 | Provider e2e test gaps | `tests/e2e/providers/common.rs` (4 sites) |
| #4041/#4042/#4044 | OpenAI Responses API compatibility gaps (`finish_reason` handling, `web_search_call` content blocks, unknown web-search events) | `clients/openai-node/tests/inference-openai-responses-api.test.ts` |

## In-progress migrations and deprecations

- **Streaming chunk type migration**: remove imports after migrating OpenAI/OpenRouter streaming chunk types to `tensorzero-types-providers` (`providers/openai/mod.rs`, `providers/openrouter.rs`).
- **`#[arg(long, alias = "run-migrations")]`** in `gateway/src/cli.rs` — deprecated alias to remove after a deprecation window.
- **Config deprecation (#4626)**: finish deprecation in `config/mod.rs`.
- **`datapoint_name` column** should be renamed to `task_name` in a future ClickHouse migration (`migration_0025.rs`, `migration_0026.rs`).
- **Evaluator cutoffs (#6603)**: delete fallback in `evaluations/src/lib.rs` after cutoffs are fully removed from evaluator configs.
- **Evaluation names (#6676)**: stop writing redundant data now that human-readable evaluation names exist (`evaluations/src/lib.rs`, `evaluations/src/evaluators/mod.rs`).
- **`datasets.rs`**: `list_datapoints` to be deprecated in favor of `get_datapoints` (`db/datasets.rs`).
- **`db/clickhouse/mod.rs`**: `TODO: deprecate this` on a legacy method.

## Gateway and core inference

- `endpoints/batch_inference.rs`: add proper error handling to batch responses (#503) and remove unnecessary clones.
- `endpoints/openai_compatible/types/chat_completions.rs`: decide whether to warn when overriding already-set parameters.
- `endpoints/stored_inferences/v1/get_inferences.rs`: consider bounding the number of inferences returned to avoid unbounded queries.
- `endpoints/stored_inferences/render_samples.rs`: make drop-vs-error on failures configurable.
- `endpoints/datasets/v1/update_datapoints.rs`: implement `update_datapoints` on the `DatasetQueries` trait (#5691); refactor file-writing logic so calls are hard to forget; provide object-storage mocks for tests.
- `endpoints/feedback/mod.rs`: rename request/response types to `CreateFeedbackRequest`/`CreateFeedbackResponse` and export.
- `endpoints/internal/inference_count.rs`: distinguish between feedback types.
- `endpoints/internal/model_inferences.rs`: decide on a non-`Stored*` message type for the API.
- `relay.rs`: preserve original chunk as a string (2 sites); decide interaction with `dryrun`.
- `cache.rs`: tool-call access requires re-running `collect_chunks`.
- `model.rs`: error-handling strategy (2 sites); decide how to expose the Responses API for wrapped providers.
- `inference/types/`: improve token estimate; decide whether `tool_config` counts toward advance estimates; reconsider `Serialize` impls on resolved input types (3 sites); add config hash to `StoredSample`.
- `inference/types/batch.rs`: add errors to `ProviderBatchInferenceResponse` (#503).
- `model_table.rs`: investigate `derive(TS)` failure when bounds are added to `BaseModelTable`.
- `tool/storage.rs`: decide the Python interface for `ToolChoice`; destructuring pattern for private fields.
- `function/function_config.rs`: avoid a clone in schema handling.
- `client/client_inference_params.rs`: avoid reconstructing a hashmap.
- `client/mod.rs`: support a dedicated cache Valkey URL for embedded gateways.
- `utils/gateway.rs`: startup SQL test for `write_retention_config` (#5764); handshake with API to validate credentials.
- `config/mod.rs`: document retention config clearly in user-facing docs (#5764).
- `gateway/src/main.rs`: require optimization migrations soon (#6912); decide autopilot config approach.
- `gateway/src/routes/evaluations.rs`: allow evaluators defined under evaluations to be referenced (#6983).
- `gateway/tests/auth.rs`: find a way to list all routes.
- `providers/openai/grader.rs`: re-add `sampling_params`.
- `providers/openai/responses.rs`: decide whether users can customize a name.
- `providers/aws_sagemaker.rs`: add a way to serialize the `WrappedProvider`.
- `rate_limiting/mod.rs`: change Postgres to match interval semantics; enforce that `TicketReceipt` is consumed by `return_tickets`.
- `experimentation/track_and_stop/`: input validation; nonzero default epsilon (#4282).
- `embeddings.rs`: handle embedding-specific responses; error-handling strategy; store request info more appropriately (#399).
- `tensorzero-error/src/lib.rs`: take `sqlx` errors instead of strings.
- `tensorzero-auth/src/key.rs`: internalize `pub` cache API once the cache moves out of the gateway.
- `tensorzero-mcp/src/handler.rs`: refactor client-side tool wrapping.
- `tensorzero-inference-types/src/tool.rs`: add `tool_choice` getter once handling is decided.
- `ts-executor-pool/src/runtime/ops.rs`: make handle opaque on the JavaScript side.
- `autopilot-tools`: move `GetFeedbackByVariantToolParams` to its proper home.
- `tensorzero-python/src/lib.rs`: extend Python `ABC` once PyO3 supports it.

## Database layer

- `db/clickhouse/query_builder/mod.rs`: extract inference filters into their own file; validate ORDER BY with metrics; separate SQL-generation tests from filter tests; push LIMIT/OFFSET down to subqueries (#4608).
- `db/clickhouse/inference_queries.rs`: query both tables when a function name is given (#4181); proper ORDER BY generation with metric joins (#4181).
- `db/clickhouse/dataset_queries.rs`: reorganize module; consider detecting multiple returned datapoints.
- `db/clickhouse/model_inferences.rs`: factor out common query logic.
- `db/clickhouse/feedback.rs`: `max_samples_per_variant` ignored due to pre-aggregated tables.
- `db/postgres/mod.rs`: relax version check for blue/green deployments (#7127); expand-contract for breaking schema changes; customizable Postgres timeout; promote `pgcron_setup.sql` to a migration (#6176).
- `db/postgres/postgres_setup.rs`: quote identifiers so special characters don't break setup (2 sites).
- `db/postgres/episode_queries.rs`: accurate episode count (#6472).
- `db/postgres/feedback.rs`: revisit query performance (#5927, 2 sites).
- `db/postgres/model_inferences.rs`: single pass instead of O(n^2).
- `db/postgres/dataset_queries.rs`: decide empty-map serialization (2 sites).
- `db/postgres/evaluation_queries.rs`: make `evaluator_inference_id` an `Option<Uuid>`.
- `db/postgres/batching.rs`, `db/clickhouse/batching.rs`: retry policy on flush errors.
- `db/batching.rs`: try `F: FnMut(&mut Vec<T>)`.
- `db/batch_inference.rs`: strongly typed responses (2 sites).
- `db/inferences.rs`: move types to `endpoints/stored_inferences/v1/types.rs`.
- `db/stored_datapoint.rs`: new `StoredContentBlockChatOutput` type (#4405); optional fetch of files from object storage.
- `db/stored_input.rs` (`inference/types/stored_input.rs`): optional parameter to fetch files from object storage.
- `db/delegating_connection.rs`: audit database operations when observability is disabled (#6469).
- `db/evaluation_queries.rs`: order-independent lookup (#6664).

## Optimizers and evaluations

- `tensorzero-optimizers/src/endpoints.rs`: revisit `u32::MAX` bound; support referencing a configured optimizer by name (2 sites).
- `tensorzero-optimizers/src/fireworks_sft.rs`: use returned name as the TensorZero model name.
- `tensorzero-optimizers/src/openai.rs`: test that the last message is from the assistant.
- `tensorzero-optimizers/src/gepa/sequential/mod.rs`: random split from `train_examples` if `None` (#4772); semaphore sharing with `run_evaluation_core_streaming` (#4914).
- `evaluations/src/lib.rs`: centralize environment variable reading (#5754).
- `evaluations/src/evaluators/regex_eval.rs`: precompile regexes instead of rebuilding per inference (#6584).

## UI

- `utils/tensorzero/tensorzero.ts`: generate the HTTP client from a schema; support streaming inference (#5394); avoid decoding job handles.
- `utils/tensorzero/base-client.ts`: ensure server errors don't leak sensitive information.
- `utils/tensorzero/errors.ts`: error types copied from `error.rs` — should be generated or shared.
- `utils/resolve.server.ts`: avoid duplicating resolved-file types; resolution to be handled in the gateway (#4674, #4675).
- `utils/jsonschema.ts`: better schema validation (currently only checks for a valid JSON object).
- `utils/clickhouse/common.ts`: remove the type — queries should not run in the UI.
- `components/ui/toast.tsx`: implement success variant styles.
- `components/ui/DeleteButton.tsx`, `AddButton.tsx`: consider unifying with similar components.
- `components/metric/CurationMetricSelector.tsx`: fix `react-hooks/exhaustive-deps` suppressions (2 sites); generalize generic to accept null.
- `components/inference/InferenceDetailContent.tsx`: remove the separate `input` field from `InferenceDetailData` (derive from the inference instead).
- `components/input_output/`: hide system button when no `system` template exists; narrow `system` typing; better fallback handling in content blocks; handle invalid intermediate states (#4903, 2 sites).
- `components/autopilot/EventStream.tsx`: better error handling.
- `components/dataset/*.stories.tsx`: handle very long dataset lists (virtualization).
- `hooks/use-local-storage.ts`: decide whether to expose storage errors.
- `routes/api/curated_inferences/count.route.ts`: fix `react-hooks/exhaustive-deps` suppression.
- `routes/api/tensorzero/inference.utils.tsx`: unsupported in Node bindings.
- `routes/config/route.tsx`: add TOML support.
- `routes/datasets/.../datapointOperations.server.ts`: more type safety; avoid sequential queries.
- `routes/datasets/.../datapointOperations.server.test.ts`: provide a mock client at the `tensorzero-node` level.
- `routes/playground/`: inference through the gateway instead of the Node route; stabilize props instead of custom comparison; incorrect logic flagged in `route.tsx`.
- `routes/optimization/supervised-fine-tuning/route.tsx`: fix optimizer output to match e2e test.
- `routes/evaluations/`: add validation to `LaunchEvaluationModal`; reuse tooltip animations in `EvaluationTable`.

## Tests

- `tests/e2e/providers/common.rs` and friends: expand streaming tests to build up content from chunks (2 sites); re-enable Mistral reasoning tests when upstream is fixed; re-enable GCP Vertex Gemini streaming reasoning (#6680).
- `tests/e2e/db/`: stop assuming pre-existing data (#3903, 2 sites); Postgres parity for config queries (#5691); rename `batch` module once nextest stops filtering on the name (#5862); rate-limit tests should not run in the ClickHouse job (#5744); read for `variant_by_episode`.
- `tests/e2e/raw_response/retries.rs`: redesign the dummy model so responses are provided in the test body (#6796).
- `tests/e2e/human_feedback.rs`: actually write human feedback and verify.
- `tests/e2e/otel_export.rs`: investigate local flakiness.
- `durable-tools/src/tests.rs`: re-enable tests after adding test helpers to `durable` (2 sites).
- `clients/openai-go`: implement `test_async_basic_inference`.
- `clients/openai-node`: bump OpenAI client version for custom tool types (#4648).
- `ui/app/utils/clickhouse/inference.test.ts`: lower timeout once ClickHouse LTS issue is understood.
- `ui/e2e_tests/`: ID roundtripping (#4058, 3 sites); replace fixed waits with proper synchronization; refresh-button timing in playground.

## Miscellaneous

- `tensorzero-overhead/src/disjoint_intervals.rs`: more efficient implementation.
- `examples/autopilot/benchmarks/.../session.py`: tighten session logic.
