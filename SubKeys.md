# []string value compresion format []string

- used to compress subkeys to simplify yaml for LLM operations
- is also a method of compressing complex json
- only need to use values when relevant and exclude when not relevant
- these are examples, but can have any logical value that aligns with intent
  
##

## external_services.service []

```
endpoint:<local-ref> — unique within this service; used for cross-references inside this declaration.
method:<verb> — GET|POST|PUT|PATCH|DELETE for REST; publish|subscribe|receive for events; invoke for RPC; connect|send|receive for realtime.
path:<path> — URL path for REST, topic/queue/subject name for events, method name for RPC, URL path for WebSocket.
protocol:<proto> — rest|graphql|grpc|websocket|sse|webhook|kafka|rabbitmq|sqs|sns|nats|redis_pubsub|mqtt|amqp|ftp|sftp|tcp|udp.
direction:<dir> — request-response|one-way|pub-sub|stream-server|stream-client|stream-bidirectional.
auth:<scheme> — none|bearer|api_key|oauth2|basic|mtls|hmac|aws_sigv4|cookie.
scopes:<csv> — required scopes (semicolon-separated if values contain commas).
request:<schema> — request body or query param schema in key:type,key:type form.
response:<schema> — response body schema.
payload:<schema> — for event endpoints, replaces request/response.
status_codes:<csv> — expected non-success codes the caller must handle (e.g., 400,401,404,429,500).
rate_limit:<rpm> — requests per minute (per-key or global as documented).
idempotent:<bool> — matters for retry logic at the call site.
timeout_ms:<int> — recommended client timeout.
retry_policy:<policy> — none|exponential|fixed|custom.
content_type:<mime> — application/json|application/grpc|text/event-stream|multipart/form-data, etc.
endpoint_url:<url> — concrete URL if known; omit when env-driven only.
endpoint_env:<env-var> — environment variable providing the base URL.
spec_ref:<global-ref> — globally unique reference used from integrations[].endpoint_spec_ref.
version:<semver> — external service contract version.
Event-specific endpoints add: delivery:<at_most_once|at_least_once|exactly_once>, signature:<hmac_sha256|ed25519|none>, dead_letter:<bool>.
File-upload endpoints add: max_size_mb:<int>, allowed_mime:<csv>, chunked:<bool>.
```

## secrets_provider ""

```
provider:<type> — env_var|aws_secrets_manager|gcp_secret_manager|azure_key_vault|hashicorp_vault|doppler|1password|kubernetes_secret|file_mount|keychain_macos|credential_manager_windows|android_keystore|ios_keychain|web_crypto_subtle.
scope:<scope> — build_time|runtime|both.
rotation:<mode> — manual|automatic|scheduled.
region:<region> — for cloud providers that scope secrets regionally.
namespace:<ns> — for K8s, Vault paths, etc.
fallback:<provider> — secondary provider if primary is unavailable.
For mobile apps, the secrets_provider must reflect the device-local keystore for any client-stored credential — provider:ios_keychain or provider:android_keystore — with API keys never shipped in the bundle. The grammar handles this transparently.
For desktop (Electron/Tauri), provider:keychain_macos|credential_manager_windows|secret_service_linux covers the three OS-native stores.
```

## domain-entities-and-relations: []

```
Entity subkeys:
entity:<Name> — PascalCase entity name.
owned_by:<microservice> — which service owns writes to this entity.
storage_ref:<store> — links to a storage[] entry by store:<ref>.
tenant_scoped:<bool> — whether records carry a tenant_id.
soft_delete:<bool> — deleted_at column vs hard delete.
audited:<bool> — whether changes are logged to an audit table.

Attribute subkeys:
attr:<Entity>.<field>:<type> — uuid|string|int|decimal|float|bool|timestamp|date|json|blob|text|enum:<values>.
nullable:<bool> — defaults to false.
unique:<bool> — DB-level unique constraint.
indexed:<bool> — whether to create an index.
default:<value> — literal default.
validation:<rule> — email|url|regex:<pattern>|range:<min-max>|length:<min-max>|enum:<values>.
pii:<bool> — marks personally identifiable information for compliance handling.
encrypted_at_rest:<bool> — field-level encryption above storage-level.
max_length:<int>, min_length:<int> — string bounds.
format:<format> — email|url|uuid|iso8601|e164|rfc5322.

Relation subkeys:
rel:<Entity>.<kind>:<Other> — kind is belongs_to|has_one|has_many|many_to_many.
cardinality:<1-1|1-n|n-1|n-n> — explicit cardinality.
cascade:<mode> — on delete/update behavior.
foreign_key:<field> — which column holds the reference.
join_table:<name> — for n-n relations.
lazy_load:<bool> — hint to the ORM layer.
```

## integrations: []


```
Core subkeys:
from:<source>, to:<dest> — microservice names or "external".
type:<sync|async> — blocking vs non-blocking.
protocol:<proto> — same vocabulary as endpoints.
direction:<dir> — request-response|fire-and-forget|pub-sub|stream.
category:<cat> — api_communication|data_retrieval_and_aggregation|authentication_and_authorization|realtime_and_streaming_communication|event_driven_interaction|file_upload_download|direct_low_level_communication.

External-linkage subkeys (when to is external):
external_service:<name> — must match an external_services.service[].name.
endpoint_spec_ref:<ref> — must match an endpoint's spec_ref: on that service. This is the byte-identity link that closes the hallucination gap.

Connection subkeys:
endpoint:<url> — concrete or templated URL.
endpoint_env:<env> — env var carrying the URL.
auth:<scheme> — authentication scheme for this integration.

Resilience subkeys:
timeout_ms:<int> — client timeout.
retry_policy:<policy> — none|exponential:<base-ms>:<max-retries>|fixed:<interval-ms>:<max-retries>.
circuit_breaker:<bool> — whether to wrap in a breaker.
rate_limit:<rpm> — outbound rate limit the caller enforces.

Observability subkeys:
idempotency_key:<header> — e.g., Idempotency-Key.
correlation_id:<header> — e.g., X-Correlation-ID.
trace_propagation:<bool> — whether to propagate W3C trace headers.

Event-specific:
event_name:<name>, topic:<name>, delivery:<guarantee>.
```

## storage: []


```
Core subkeys:
store:<ref> — unique reference.
category:<category> — relational|document|key_value|wide_column|graph|vector|multi_model|ledger|time_series|spatial|blockchain|version_control|file_operation|object_storage|cache.
type:<db_type> — postgresql|mysql|mariadb|mssql|oracle|sqlite|mongodb|couchdb|firestore|redis|memcached|dynamodb|cassandra|scylladb|neo4j|pinecone|qdrant|milvus|weaviate|influxdb|timescaledb|qldb|immudb|s3|gcs|azure_blob|gitea|github|gitlab|local_fs|nfs.

Connection subkeys:
endpoint:<url> — connection URL.
endpoint_env:<env> — env var.

Ownership and access subkeys:
owner:<microservice> — sole writer by policy.
readers:<csv> — microservices with read access.
writers:<csv> — microservices with write access (should contain owner).
entities:<csv> — entity names persisted here, linking back to domain-entities-and-relations[].entity:.

Operational subkeys:
tenant_isolation:<mode> — none|row|schema|database|cluster.
encryption_at_rest:<bool>.
backup:<policy> — none|daily|hourly|continuous.
ha:<mode> — none|replica|multi_az|multi_region.
replicas:<int> — replica count.

Content subkey:
purpose:<text> — one-line description.

Model subkey:
model:[ <prefix>: <value>; ... ] — category-specific inner grammar per your existing L1 prompt.
```

## flows: []


```
flow:<ref> — unique reference.
trigger:<trigger> — user-action|scheduled|event|webhook|api-call|system-startup|timer.
actor:<actor> — authenticated-user|anonymous-user|admin|service-account|external-system.
initiator:<source> — microservice or external.
terminator:<dest> — microservice or external, or "none" for one-way.
sync:<bool> — whether the caller blocks until completion.
sla_ms:<int> — total end-to-end SLA.
step:<description> — repeatable; one per step.
implements_acceptance:<csv> — indexes of acceptance entries this flow satisfies (0-indexed into validation.acceptance[]).
implements_workflow:<csv> — workflow names + step indexes this flow realizes (e.g., save-recipe.2,save-recipe.3).
```

## constraints: []


```
constraint:<ref> — unique reference.
kind:<security|compliance|nfr> — mechanical classification (removes L1's judgment call per the earlier conversation).
scope:<scope> — where it applies.
severity:<must|should|may> — RFC 2119 semantics.
measurable:<bool> — whether this has a verifiable metric.
metric:<name> — e.g., page_render_p95_ms, bundle_size_kb, contrast_ratio.
threshold:<value> — target value (e.g., 1500, 250, 4.5).
applies_to:<csv> — microservices, modules, or "all".
standard:<standard-ref> — WCAG_2_1_AA|GDPR|HIPAA|SOC2|PCI_DSS|OWASP_ASVS_L2|NIST_800_53 — or empty for NFRs.
body:<verbatim-text> — the original constraint string, preserved for user display.
```

## design: []


```
Visual common factors (token entries) subkeys:
token:<name> — CSS custom property name (--color-brand-primary) or platform-equivalent key. Name is the materialization key; the pipeline uses it verbatim.
kind:<kind> — enumerated type: color|color_overlay|typography_family|typography_size|typography_weight|typography_leading|typography_tracking|spacing|sizing|grid_column|grid_gutter|container_width|radius|shadow|border_width|motion_duration|motion_easing|motion_distance|breakpoint|z_index|opacity|data_viz_sequence|data_viz_categorical|data_viz_diverging|data_viz_gridline|density_scale|touch_target|focus_ring_width|focus_ring_offset.
value:<literal> — raw value preserved byte-for-byte (e.g., #2E7D5B, 16px, Inter, cubic-bezier(0.4, 0.0, 0.2, 1), 200ms).
scope:<scope> — global|theme:<theme-ref>|brand:<brand-ref>|density:<density-ref> — tokens can be scoped to named themes, brand variants, or density modes.
mode:<mode> — light|dark|high_contrast|reduced_motion|any. Controls which color scheme or motion preference this token applies under. any means unconditional.
consumes:<csv> — asset references this token depends on (font assets for typography tokens, image assets for background tokens).
derives_from:<token-ref> — for tokens computed from other tokens (overlay opacity applied to a base color, spacing multiples of a base unit). Materializes as color-mix() or calc() in CSS, or equivalent in platform theme systems.
a11y_role:<role> — focus_ring|error_state|success_state|warning_state|info_state|disabled_state|<empty>. Ties a token to its accessibility semantic so the pipeline can verify contrast ratios against the intended usage.

Interaction/UX patterns (pattern entries) subkeys (Track B — interaction/UX patterns):
pattern:<ref> — unique reference for this pattern selection.
kind:<kind> — enumerated pattern dimension (full list below).
applies_to:<target> — global|theme:<ref>|screen:<ref>|component:<ref>|route:<path>|viewport:<breakpoint-ref>.
value:<option> — enumerated choice from the catalog for this pattern kind.
mode:<mode> — conditional application: light|dark|high_contrast|reduced_motion|touch|mouse|keyboard|any.
consumes:<csv> — asset refs the pattern references.
implements_a11y:<csv> — references to a11y:<ref> entries this pattern helps satisfy. Closes the loop between chosen patterns and accessibility requirements.

Pattern kinds and their enumerated option vocabularies:Layout patterns:
kind:layout_form → single_page|wizard|inline_edit|modal|drawer|split_view
kind:layout_list → table|card_grid|feed|kanban|timeline|tree
kind:layout_detail → master_detail|full_page|overlay|side_panel|stacked
kind:layout_dashboard → fixed_grid|masonry|tabs|accordion|single_column
kind:layout_landing → hero_with_features|hero_with_cta|split_hero|video_hero|minimal
kind:fluid_strategy → percentages_with_clamp|fixed_breakpoints|container_queries|hybrid
Navigation patterns:
kind:nav_primary → top_bar|sidebar|bottom_nav|hamburger|tab_bar|floating_menu
kind:nav_secondary → tabs|breadcrumbs|stepper|segmented_control|none
kind:nav_mobile_adaptation → collapse_to_hamburger|bottom_nav|stacked|drawer
kind:breadcrumb_collapse → middle_ellipsis|trailing_only|show_all|hidden_on_mobile
Interaction patterns:
kind:confirmation → optimistic|confirm_dialog|undo_toast|inline_confirm|none
kind:form_validation → on_blur|on_submit|on_change|debounced|server_only
kind:error_recovery → inline_retry|toast_with_retry|full_page_error|silent_background_retry
kind:tooltip_trigger → hover|click|focus|long_press|hover_and_focus
kind:selection_model → single|multi|range|checkbox_column|row_click
State patterns:
kind:empty_state → illustration_with_cta|text_only|skeleton_forever|minimal_icon
kind:loading_state → skeleton|spinner|progress_bar|shimmer|deferred_until_slow
kind:error_state → full_page|inline|toast|banner|modal
kind:success_feedback → toast|inline_check|banner|full_page|silent
Information hierarchy patterns:
kind:visual_hierarchy → size_driven|color_driven|weight_driven|position_driven|mixed
kind:content_density → compact|regular|spacious|custom
kind:reading_flow → top_to_bottom|z_pattern|f_pattern|gutenberg|radial
kind:emphasis_strategy → single_focal_point|progressive_disclosure|grouped_sections|narrative
kind:whitespace_strategy → generous|balanced|dense|variable
Motion patterns:
kind:motion_choreography → staggered|synchronized|cascading|none
kind:page_transition → fade|slide|none|shared_element|morph
kind:reduced_motion_behavior → disable_all|cross_fade_only|reduce_distance|respect_preference
Feedback patterns:
kind:notification_batching → immediate|batched_per_minute|digest|user_configured
kind:toast_stacking → stack_bottom_right|stack_top_right|queue|replace_previous
kind:progress_feedback → determinate_preferred|indeterminate_fallback|always_indeterminate|contextual
kind:tooltip_delay → instant|short_250ms|medium_500ms|long_1000ms
Form patterns:
kind:label_placement → top|left|floating|inline|hidden_with_aria
kind:required_field_indicator → asterisk|optional_label|both|none
kind:input_help_placement → below|right|tooltip|on_focus
kind:password_reveal → always|on_hover|never|user_toggle
Data patterns:
kind:table_density → comfortable|compact|dense|responsive
kind:table_zebra → enabled|disabled|on_hover_only
kind:sort_indicator → arrows|carets|text_label|none
kind:pagination_style → numbered|prev_next|infinite_scroll|load_more|virtual_scroll
Brand patterns:
kind:voice_tone → formal|conversational|playful|technical|warm|authoritative|<custom>
kind:imagery_treatment → photographic|illustrated|geometric|minimal|mixed|none
kind:iconography_style → filled|outlined|duotone|rounded|sharp|custom_keyline
Theming patterns:
kind:theme_mode_strategy → system_preference_only|user_toggle_only|both_with_system_default|both_with_user_default
kind:brand_variant_strategy → single|multi_brand_runtime|multi_brand_build_time
kind:density_strategy → single|user_selectable|viewport_adaptive|role_based
Performance patterns:
kind:image_loading → eager_above_fold|lazy_always|native_lazy|intersection_observer|blurhash_placeholder
kind:font_loading → swap|optional|block|fallback|preload_critical
kind:asset_bundling → inline_critical|split_by_route|split_by_component|single_bundle
Accessibility patterns (many of these are also a11y requirements — see Track D):
kind:focus_visible_strategy → always_visible|keyboard_only|always_visible_high_contrast|custom
kind:skip_links → enabled|enabled_on_focus|disabled
kind:complex_widget_focus → roving_tabindex|aria_activedescendant|native|hybrid
kind:live_region_strategy → polite_for_status|assertive_for_errors|both|none

File-specific compositions (composition entries) subkeys 
composition:<ref> — unique reference.
scope:<scope> — screen:<ref>|component:<ref>|route:<path>|layout:<ref>. Identifies which source file will own the materialized styles.
description:<text> — the file-specific styling concern. Must reference tokens via var(--name) for common design factors. Raw literals are forbidden (they'd belong in a token).
consumes:<csv> — asset refs this composition references.
references_tokens:<csv> — token names the composition depends on. Lets the pipeline verify every referenced token exists.
references_patterns:<csv> — pattern refs this composition implements file-specific details for.
viewport:<breakpoint-ref> — composition applies at a specific breakpoint range.
mode:<mode> — applies under specific color/motion/input modes.

Accessibility requirements (a11y entries):
a11y:<ref> — unique reference.
kind:<kind> — contrast|focus|keyboard|screen_reader|motion|cognitive|target_size|language|structure|timing.
scope:<scope> — global|screen:<ref>|component:<ref>|pattern:<ref>.
standard:<standard> — WCAG_2_1|WCAG_2_2|WCAG_3_0|Section_508|EN_301_549|ADA.
level:<level> — A|AA|AAA.
criterion:<criterion-ref> — WCAG success criterion ID (e.g., 1.4.3, 2.4.7, 4.1.3).
requirement:<text> — verbatim requirement body.
measurable:<bool> — whether there's a verifiable metric.
metric:<name> — contrast_ratio_normal|contrast_ratio_large|contrast_ratio_error|touch_target_px|focus_indicator_width_px|motion_threshold_dpi_per_s.
threshold:<value> — target value (4.5, 3, 44, 2, etc.).
verified_by:<pattern-ref or composition-ref> — which pattern or composition implements this a11y requirement. Mechanical traceability for coverage validation.

Kind-specific coverage::
contrast — covers wcag.color_contrast_ratios schema factors. Multiple a11y entries, one per color semantic pairing (text on bg, text on surface, error text on bg, etc.).
focus — covers focus_management_and_order schema factors. Entries for initial focus rules, modal focus trap rules, restore focus rules, complex widget focus model.
keyboard — covers keyboard_only_navigation schema factors. Entries for tab order, skip links, roving tabindex patterns.
screen_reader — covers screen_reader_testing_guidelines schema factors. Entries for assistive tech compatibility, live regions, ARIA roles.
motion — covers prefers_reduced_motion behavior. Entry for reduced-motion strategy.
target_size — covers touch_targets.min_target_px. Entry asserting the minimum target size.
cognitive — covers reading level, timing, plain language, error recovery.
language — covers lang attribute requirements, translation, directionality.
structure — covers heading hierarchy, landmarks, semantic markup.
timing — covers session timeouts, auto-refresh, pause/stop/hide controls.

Component API surface (component entries):
component:<ref> — unique reference (e.g., button, input_text, table_data, toast, tooltip).
kind:<kind> — button|input|select|checkbox|radio|switch|slider|textarea|table|list|card|tabs|breadcrumbs|pagination|modal|drawer|popover|tooltip|toast|banner|alert|progress_linear|progress_circular|spinner|skeleton|avatar|badge|chip|tag|icon|icon_button|menu|dropdown|date_picker|time_picker|color_picker|file_upload|lux_meter|rating|stepper|segmented_control|accordion|tree|kanban_card|timeline_entry|data_viz_chart|<custom>.
variants:<csv> — e.g., primary,secondary,tertiary,ghost,danger for buttons.
sizes:<csv> — e.g., xs,sm,md,lg,xl.
states:<csv> — e.g., default,hover,focus,pressed,disabled,loading,error,success.
slots:<csv> — named insertion points (e.g., leading_icon,trailing_icon,content,actions).
aria_role:<role> — ARIA role(s) the component exposes.
css_vars_prefix:<prefix> — token prefix for component-scoped customization (from schema's api_surface.css_vars_prefix).
theming_hooks:<csv> — from schema's api_surface.theming_hooks.
implements_patterns:<csv> — pattern refs this component realizes.
implements_a11y:<csv> — a11y refs this component satisfies.
consumes:<csv> — asset refs (icon assets, illustration assets).

Brand positioning (brand entries):
brand:<ref> — unique reference. Single brand:default for most apps; multi-brand apps declare one per variant.
core_idea:<text> — the single positioning idea the brand stands for.
taglines:<csv> — approved taglines, pipe-escaped if they contain commas.
voice_tone:<text> — narrative description or reference to a voice_tone pattern value.
imagery_themes:<csv> — e.g., nature,home_cooking,rustic,modern.
imagery_treatments:<csv> — e.g., warm_tone,high_contrast,desaturated,duotone.
iconography_style:<pattern-value> — must match an iconography_style pattern value.
alt_text_pattern:<text> — default alt text generation pattern for brand imagery.
clear_space_rule:<text> — minimum clear space around logo/marks.
min_size_px_by_context:<csv> — context-to-size map (e.g., favicon:16,header:32,footer:24).
consumes:<csv> — logo/mark/monogram asset refs. Links brand entries to the asset declarations that provide the visual marks.
```

## assets: []


```
Core subkeys:
asset:<ref> — unique reference.
path:<path> — source path within the repo or bundle.
kind:<kind> — svg|png|jpg|jpeg|webp|avif|ico|icon|image|logo|font|video|audio|document|template|data|lottie|json|xml.
mime:<mime> — actual MIME type.
serves:<csv> — microservices that reference this asset.

Placement subkeys:
placement:<placement> — global.header|global.footer|global.font-face|global.meta|global.sidebar|page:<ref>.<section>|component:<ref>|route:<path>|screen:<ref> (screen for mobile/desktop).

Usage subkeys (composed as semicolon-separated inner grammar, matching your existing usage: pattern):
render:<mode> — inline-svg|background-image|img-tag|favicon|font-face|video-element|audio-element|icon-sprite|native-image-view (iOS/Android)|svg-drawable|resource-bundle.
font-family:<name>, font-weight:<range>, font-style:<style> — for font assets.
alt:<text> — accessibility alt text.
lazy:<bool> — defer loading.
preload:<bool> — emit <link rel="preload"> or platform equivalent.
breakpoint:<min-max> — responsive breakpoint range.
aspect_ratio:<w:h>, object_fit:<mode>.

Distribution subkeys:
source:<source> — bundled (ships with app), cdn (served from CDN), external (third-party host).
cdn_url:<url> — when source is cdn.
bundled_at:<time> — build-time or runtime download.
variants:<csv> — variant references (e.g., logo-dark,logo-light,logo-2x).
dimensions:<wxh> — source dimensions.
responsive:<csv> — breakpoints for which variants exist.

Legal subkey:
license:<license-ref> — MIT|Apache-2.0|CC-BY-4.0|proprietary|commercial:<vendor>.

Accessibility subkey:
alt:<text> — top-level alt for simple cases (duplicates usage.alt for convenience).

Descriptive subkey:
purpose:<text> — free-form description for elicitation UI.
```

