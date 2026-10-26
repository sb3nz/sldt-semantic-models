# PartRoleCandidateSolution — modelling proposal and review register

**Status:** Discussion draft; not an adopted Catena-X model.  
**Proposed model URN:** `urn:samm:io.catenax.part_role_candidate_solution:1.0.0#PartRoleCandidateSolution`   
**Artifacts:** `PartRoleCandidateSolution.ttl` and `PartRoleCandidateSolution.example.json`.

## 1. Purpose and design boundaries

This aspect describes the **options for fulfilling one PartRole** and any *context-specific technical assessment or selection decision*. The PartRole is the anchor and remains independently identified. A candidate is a local engineering option; a referenced PartType is a distinct twin and does not inherit PartRole asset IDs.

```text
PartRole (root globalAssetId)
  └── candidateSolutions[ ] (0..n)
       ├── candidateSolutionId (mandatory; scoped by PartRole)
       ├── name (mandatory)
       ├── description? / conceptReference? / providerId?
       ├── referencedPartTypeGlobalAssetId? (0..1 in this draft)
       └── contextAssessments[ ]? (0..n)
            ├── engineeringContext
            │    ├── contextId (mandatory)
            │    └── variantId? / derivativeId? / configurationId?
            ├── evaluationStatus (mandatory)
            ├── decisionStatus (mandatory)
            └── assessmentNote? / decisionReference?
```

The model **does not** prescribe the decomposition or usage of roles, requirement and function content, PartInstance assignments, part material/production-site data, engineering approval/release status, or a general product configuration model.

## 2. Inclusion / exclusion decisions in the draft

| Topic | Draft choice | Reason / boundary |
|---|---|---|
| Referenced PartRole identity | **Included:** `ext-ic:globalAssetId` | Identifies the PartRole carrying the aspect; no role owner IDs in the payload. |
| Candidate identity | **Included:** `candidateSolutionId` | A candidate remains identifiable across multiple assessments or selection contexts. |
| Name and description | **Included:** name required, description optional | Lets proposals exist before a PartType is defined. |
| Existing PartType | **Included, optional:** one `referencedPartTypeGlobalAssetId` | Enables navigable links without replacing or sharing PartRole identity. |
| Preliminary design artifact | **Included, optional:** `conceptReference` | External concept model/document stays outside the aspect. |
| Proposal attribution | **Included, optional:** `providerId` (BPNL) | May be different from role owner and PartType manufacturer; not a duplication of `ownerId`. |
| Variant / derivative / configuration | **Included only as assessment context** | One candidate can be evaluated/selected differently in different engineering contexts. |
| Assessment outcome and selection decision | **Included as separate fields** | Suitability and choice are different statements. |
| Optional decision evidence | **Included as a reference**, not embedded evidence | Keeps evidence and approvals in their owning systems. |
| Role decomposition / usage | **Excluded** | `SingleLevelRoleStructureAsSpecified` / `SingleLevelRoleUsageAsSpecified`. |
| Requirements / functions | **Excluded** | Dedicated PartRole reference aspects or external engineering models. |
| `PartInstance` realization / AsBuilt | **Excluded** | Candidate/selection is distinct from manufactured/installed realization. |
| Site, quantity, validity period, sequence index | **Excluded** | No sufficiently general meaning for solution evaluation in this aspect. |
| Release and approval master data | **Excluded** | External governance of the relevant role, type, configuration or decision. |
| Reverse PartType-to-PartRole index | **Excluded** | Can be derived by a service; should not duplicate the authoritative association. |

## 3. Review points to resolve before standardization

### Priority 1 — changes that could reshape the model

- [x] **R1 — Candidate identifier and ownership.** Is `candidateSolutionId` scoped by PartRole sufficient? Who assigns it when several suppliers propose solutions: the PartRole owner or each proposal provider? Prevent collisions and clarify whether there is one authoritative aspect or several independently offered submodels.

  - *Should be sufficient when assigned by the owner of the PartRole. It is not the unique identifier in the dataspace but an individual identifier according to the role owner.*

- [x] **R2 — One referenced PartType or several?** The draft permits *zero or one* PartType per candidate. If alternatives map to different types, represent them as distinct candidates. If a *single* candidate denotes a multi-part solution, should it have a structured PartType-reference collection, or should that be modeled through a dedicated concept/assembly artifact?

  - *Does it necessarily point at a PartType? I guess yes...*
  - *Any multi-part solution should optimally be strucutred as a dedicated assembly artifact with respective varying substructure and meta-information.*

- [x] **R3 — Context identity and applicability.** Is `contextId` a stable reference to an external configuration/derivative/variant object? Are `variantId`, `derivativeId`, and `configurationId` useful secondary keys or redundant? How is release/version of that context resolved? Avoid treating absence of context as universally applicable.

  - *There is no "configuration/derivate/variant" object available yet.*
  - *There is no unambiguous "variantId", "derivateId" or "configurationId" available yet. How does the digitalMasterData specify and can that be reused?*

- [x] **R4 — Two independent status dimensions or one?** The draft splits the original `solutionStatus="under-evaluation"` into `evaluationStatus` (`notEvaluated`, `underEvaluation`, `suitable`, `unsuitable`) and `decisionStatus` (`open`, `notSelected`, `selected`, `confirmed`, `withdrawn`). Is this distinction needed for interoperable use? Agree on meanings and permissible combinations.

  - *Enumerations must be checked/conformed.*
  - *Both seems procedurally useful.*

- [x] **R5 — Meaning of confirmation.** Does `confirmed` mean an engineering selection decision, supplier acceptance, a binding PartType assignment, or an approval/release event? In this draft it means **stronger confirmation of a role-to-solution choice, not approval/release**. `hasConfirmedPartType` can be derived only if a PartType reference is present.

  - *"selected" could mean: This is a candidate solution that was evaluated and is further considered a  and has not been disregarded. Conversely, everything that was "notSelected" is not further elaborated in product development. However, a selected candidate does not secessarily have to reference a PartType yet.*
  - *"confirmed" could mean: This candidate is a confirmed engineering design that is realized as an identifiably PartType. It needs the reference to this PartType but makes no statement about its further release or approval.*
  - *but what about concept design approvals/releases in between "selected" and "confirmed"? Maybe a shared aspect is necessary here. Same discussion/idea is ongoing in the DEMD Aspect Model.*

- [ ] **R6 — Selection multiplicity.** Can multiple solutions have `selected`/`confirmed` for the same role and context, e.g. alternatives, planned second-source suppliers, or joint solutions? If this is prohibited, the constraint must be made explicit; SAMM field cardinality alone will not ensure it.

### Priority 2 — detail and governance choices

- [ ] **R7 — Optional proposals without assessments.** This draft allows candidates with no `contextAssessments`. Is that useful for early conceptual design, or should a default status always be required?
- [ ] **R8 — Proposal provider.** Keep `providerId` as BPNL of the proposer, not manufacturer or owner. Could data-offer metadata already provide trustworthy attribution? If several contributors edit one candidate, do we need proposal provenance beyond a single provider?
- [ ] **R9 — Concept links and evidence.** Are `conceptReference` and `decisionReference` enough, or is a typed reference needed for interoperable tool-to-tool navigation? Should a `confirmed` decision require `decisionReference`?
- [ ] **R10 — Status evolution/history.** The model describes current assessments per context, not a decision history. Do consumers need historical selections, timestamps or audit trails? If yes, consider a separate decision/event model rather than expanding the first Industry Core model.
- [ ] **R11 — Reverse navigation / search.** Starting from a PartType, finding PartRoles that reference it requires a cross-role query/index or optional reverse view. Do not place `ownerRoleId` among PartType `specificAssetIds` merely to support this search.
- [ ] **R12 — SAMM dependencies and field naming.** Decide final SAMM version and shared namespaces; verify `ext-ic:globalAssetId`, `ext-uuid:UuidV4Trait`, and `ext-number:BpnlTrait` in the target Semantic Hub. Align with naming conventions (`globalAssetId`, `candidateSolutionId`, etc.).

## 4. Suggested business rules (not all enforceable using SAMM alone)

1. For any PartRole aspect instance, `candidateSolutionId` **MUST** be unique among its candidates.
2. A `referencedPartTypeGlobalAssetId`, when provided, **MUST** resolve to a `PartType` twin and **MUST NOT** be interpreted as the identity of the PartRole.
3. In the current-state representation, the pair `(candidateSolutionId, engineeringContext.contextId)` **SHOULD** appear no more than once. If multiple authoritative assessments are permitted, add assessment ownership/identity first.
4. `evaluationStatus` refers to a technical evaluation; `decisionStatus` refers to selection/confirmation. These are not synonyms for PartType approval/release.
5. A `confirmed` solution without a PartType reference **MUST NOT** be interpreted as a confirmed PartType assignment; it can at most denote a confirmed conceptual solution.
6. `contextId`, optional variant/derivative/configuration identifiers, and their namespaces **MUST** be resolvable in the relevant engineering collaboration context before using them for automated applicability filtering.
7. No assumption is made that each role can have only one candidate, that a PartType can serve only one role, or that selected candidates can always be materialised as PartInstances.

## 5. Illustrative scenarios covered

**Early concept:** Candidate CAND-0003 has an ID, name, description and no PartType reference or assessment.  
**Variant-sensitive choice:** CAND-0001 is technically suitable and selected in Variant A, but unsuitable and not selected in Variant B. The candidate retains one ID.  
**PartType-backed option:** CAND-0002 refers to a `PartType` globalAssetId; a context-specific assessment may mark it suitable and confirmed.  
**Reused PartType:** A different PartRole can independently reference the same PartType through a candidate; this does not change the PartType's specific asset IDs.

## 6. Verification performed and limitations

- The Turtle file was parsed as RDF syntax using RDFLib, and the example JSON was parsed successfully.
- **No ESMF SAMM CLI/Model Editor semantic validation or JSON-Schema generation was performed.** Those checks, external shared-model resolution, generated payload shape, and model governance review are still required.
- `1.0.0` and all status values are draft choices, not published or ratified Catena-X declarations.
