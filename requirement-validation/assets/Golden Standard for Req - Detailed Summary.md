# Golden Standard for Requirements Specification (MN) - Chaptered Summary

## Document Overview
This document defines the Nokia Mobile Networks golden standard for writing and maintaining technical requirements in SFS and EFS. Its intent is to enforce clarity, consistency, traceability, and testability across specification, development, and testing work.

Normative baseline:
- The golden standard is the foundation for unit or team internal guidelines.
- Internal guidelines shall be consistent with the golden standard.
- In product requirements, the normative modal verb is shall (not should/may/can for requirement obligation).
- The standard includes good and bad examples to drive compliant authoring behavior.
- It applies to natural language requirements and is also valid for conceptual-model-based requirements.

---

## Chapter 1 - Requirements Types and Requirements Documentation Forms

### Purpose
- Informational chapter to establish common terminology and shared understanding of requirement types and requirement documentation forms.

### Key rules and principles
- In this standard, requirement means requirement statement.
- Requirements shall be associated with a product/program/project/task activity identifier (for example Feature-ID, Pronto-ID, CNI-ID, Jira task).
- Teams shall use terminology consistently (for example distinguish requirement object vs requirement text vs requirement statement).
- The standard covers:
  - Functional requirements
  - Non-functional (quality) requirements
  - Constraints
- Use case in this standard means sequence-of-steps requirement artifacts (broadly including forms like MSC/message flow/CPF), but excludes Nokia user scenario format.

### Important definitions
- Requirement: specification of what should be implemented (behavior, property/attribute, or development constraint).
- Business requirement: high-level business objective and expected business benefit.
- User requirements:
  - User stories
  - Quality requirements (non-functional attributes)
  - Constraints (solution-space limits beyond core functional/quality necessity)
- Functional requirement: behavior under specific conditions.
- Functional subtypes:
  - Basic requirement (expected behavior)
  - SW robustness requirement (behavior when basic path cannot be fulfilled)
- Documentation forms:
  - Natural language
  - Conceptual model
  - Hybrid

### Practical do/don't guidance
- Do classify requirement intent: what actor shall do (functional) vs what capability/attribute actor shall have (non-functional).
- Do ensure traceable linkage to activity ID.
- Do align with architecture-level notion of system as any architecture element.
- Do not treat pseudo code as preferred requirement form; it is allowed but not recommended due to readability, branch-coverage, and standardization weaknesses.
- For complex logic, prefer tables or activity diagrams over pseudo code.

### Example themes
- Functional vs non-functional distinction using basketball-player analogy.
- End-to-end derivation chain: business requirement -> user requirements -> system functional requirements -> lower-level requirements.
- Examples of six non-functional categories (performance, security, reliability, usability, maintainability, portability).
- Constraints examples (for example file size limit, power supply value).

---

## Chapter 2 - Requirements Syntax

### Purpose
- Define mandatory syntax structures for natural-language requirements (single statement, molecular, and use case).

### Key rules and principles
- Functional single statement shall include:
  - Actor (mandatory)
  - Preconditions/trigger (optional when needed)
  - Action (mandatory)
  - Observable result (optional when needed)
- Canonical form uses When/Then and shall.
- While is used for state-driven conditions; When for event-driven (with optional If/In case of).
- Alternative word order is allowed if meaning is preserved.
- For non-functional/constraint requirements, actor attribute syntax shall be used.
- Molecular requirements may replace multiple similar atomic requirements when behavior depends on combinations of common input parameters; table form is expected when complexity increases.
- Use case syntax shall include mandatory sections: Title, Purpose, Actors, Preconditions, Trigger, Main Flow, Postconditions; plus Alternative Flow and Exceptions when applicable; Notes optional.
- Use case step numbering scheme is team-defined but must be consistent within team practice.
- Existing step numbers shall be kept when inserting steps later (to preserve references).

### Important definitions
- Actor: system or system element performing action (in single statements).
- Trigger/precondition: condition stimulus for action.
- Action: what actor shall execute.
- Observable result/objective: externally deducible outcome.
- Use case actors may include people and systems.
- Main flow, alternative flow, postconditions, exceptions explicitly defined.

### Practical do/don't guidance
- Do use shall as the only accepted normative verb in single-statement requirements.
- Do not use can to express removal of restrictions; instead freeze old requirement and create new shall-based requirements.
- Do not use may to hide incomplete conditions; split into precise requirements with explicit preconditions.
- Do split optional or alternative logic into explicit flows/guarded branches.
- Do keep use case naming variants acceptable only if section semantics remain equivalent to golden-standard definitions.

### Example themes
- Event-driven vs state-driven phrasing.
- Correct migration from legacy can wording to new shall-based atomic/molecular requirements.
- Good and bad handling of complex preconditions and reallocation behavior.
- Full use case example including alternative flows, exceptions, and timer-based exception handling.

---

## Chapter 3 - Characteristics of a Good Requirements

### Purpose
- Define quality characteristics every requirement shall satisfy: Agreed, Atomic or Molecular, Complete, Consistent, Modifiable, Traceable, Unambiguous, Verifiable.

### Key rules and principles
- Agreed:
  - Requirement approval shall involve relevant stakeholders through official review.
  - Feasibility is part of agreement quality; re-agreement is required when new constraints emerge.
- Atomic or Molecular:
  - Requirement shall be either atomic or molecular.
  - Atomic: one aspect only; no partial compliance ambiguity.
  - Molecular: grouped atomic behaviors, often with table and per-subrequirement identifiers.
  - Conjunction guidance:
    - AND after when is allowed in atomic precondition.
    - OR after when allowed in molecular cases with same action; otherwise prefer tabular/atomic split.
    - Complex AND/OR shall use explicit precedence (parentheses).
    - After shall, OR generally indicates non-atomic design and should be split.
- Complete:
  - Requirement shall contain all required syntax content and required metadata/attributes per local guideline.
  - Completeness includes SW robustness handling and explicit relation between basic and robustness requirements.
  - For logical combinations, all realizable combinations shall be covered (or clearly delegated).
- Consistent:
  - No conflict with same-level/higher-level requirements.
  - Naming shall be consistent across repository; synonyms are forbidden for same object.
  - Name migration from old to new terms shall be documented in controlled way and aligned with local guideline.
- Modifiable:
  - Requirement structure and labeling shall support easy, consistent changes.
  - Avoid duplication; reference source requirement instead of copying formulas/tables.
  - Do not use screenshots for modifiable artifacts like tables/diagrams.
  - Change history shall explicitly state what changed.
- Traceable:
  - Three traceability types:
    - Pre-RS (origin in business/user need)
    - Between requirements (backward and forward)
    - Post-RS (implementation/test linkage)
  - Each requirement shall have unique persistent ID.
  - Forward traceability is mandatory if RMS automation supports it; otherwise recommended.
  - Table rows shall have unique IDs.
  - Use case is one requirement artifact; do not use individual use case steps as traceability targets.
- Unambiguous:
  - Exactly one interpretation.
  - Avoid vague terms, subjective language, undefined abbreviations, compound ambiguity, and negative statements.
  - Optional/parallel/alternative behaviors need explicit guard conditions; diagrams are recommended for complex branching/parallelism.
- Verifiable:
  - Requirement shall permit finite, cost-effective objective verification.
  - Use measurable concrete terms; remove/rewrite non-testable terms.
  - Incompleteness, ambiguity, inconsistency, or infeasibility make requirements unverifiable.

### Important definitions
- Basic requirement and SW robustness requirement relation.
- Requirement-set completeness includes functionality, quality attributes, constraints, interfaces, and response to expected/unexpected inputs.
- Backward/forward traceability semantics.
- Ambiguous-term handling: prohibited unless clearly defined in repository/glossary.

### Practical do/don't guidance
- Do involve PM/spec engineer/developer/tester and relevant stakeholders in agreement.
- Do label atomic elements inside grouped/molecular requirements.
- Do use table format for multi-parameter logic.
- Do define terms/thresholds explicitly or reference authoritative definitions.
- Do not write blanket compliance statements without scope/exclusions.
- Do not create unlabeled table rows or unlabeled grouped requirements.
- Do not duplicate high-level formulas into lower-level requirements.

### Example themes
- Thermal-threshold multi-branch behavior as atomic set or molecular table.
- 3GPP compliance with explicit exclusions and may->shall adoption scope.
- RTWP reporting/clearance use cases (good consistency vs inconsistent naming/fault semantics).
- Alarm filtering with properly labeled table-driven rules and cross-references.

---

## Chapter 4 - Writing Style

### Purpose
- Define language and presentation rules that improve readability, precision, and maintainability.

### Key rules and principles
- Requirements shall be written in US English with proper grammar/spelling/punctuation.
- Active voice is recommended, especially after then; passive in precondition can be acceptable.
- Sentences shall be short, direct, and clear.
- Avoid jargon unless defined; define specialized terms in glossary.
- Avoid negative requirements; prefer positive explicit behavior.
- Use official names for messages/procedures/parameters (3GPP/interface/NIDD), not ad hoc variants.
- Maintain grammatical parallelism in lists/steps.
- Excluding formulas, mathematical and programming operators should be avoided in textual conditions; use words instead.
- If formulas are used, use one agreed formatting style per team guideline.
- Highlight references and visual annotations according to local guideline; lifecycle of visual tags should be controlled and obsolete tags removed.
- For complex condition/behavior matrices, use tabular representation with row-level IDs and no ambiguous empty cells.
- Use inclusive language only.

### Important definitions
- Distinction between acceptable requirement text highlighting (team-guided) and clutter-inducing long-lived markup.
- Diff visibility in reviews is part of quality communication.

### Practical do/don't guidance
- Do show changes clearly in review submissions.
- Do use examples for complicated requirements.
- Do split oversized requirement objects when readability suffers.
- Do not keep outdated temporary tags forever.
- Do not rely on crossed-out/underlined inline churn as permanent documentation method.
- Do not encode large logic in dense prose when table form is clearer.

### Example themes
- Active vs passive requirement rewrite.
- Good vs bad use case main-flow sentence structure.
- Good tabular requirements with complete IDs and no empty-condition ambiguity.
- Inclusive terminology replacement in legacy text.

---

## Chapter 5 - Notes Usage

### Purpose
- Define what notes are, what they are for, and what they are not.

### Key rules and principles
- A note is supplementary information; it is not a requirement.
- Notes shall be associated with the relevant requirement(s).
- Valid note content includes:
  - References for definitions not in glossary
  - Contextual/background information
  - Rationale when no dedicated rationale attribute exists
  - Clarifying support information that aids understanding
- Teams may choose note storage mode:
  - Dedicated attribute
  - Same requirement object under requirement text
- If multiple notes apply, they shall be labeled for referenceability.
- Notes shall be kept up to date whenever related requirement changes.

### Important definitions
- Requirement vs note separation is strict.
- Notes can include references and business context but cannot replace required normative behavior statements.

### Practical do/don't guidance
- Do put notes at bottom of requirement object when inline-object option is used.
- Do not put requirements in notes.
- Do not use notes to declare unsupported functionality; supported behavior belongs in requirement text.
- Do not use notes to mark non-implementation; unimplemented requirements should be deleted/marked deleted per process.

### Example themes
- Notes as contextual supplement, parameter-definition pointer, and rationale carrier.
- Bad notes showing not implemented / unsupported / hidden requirement misuse.

---

## Chapter 6 - Statement of Compliance within the Requirement

### Purpose
- Define how to document compliance against external standards/protocols/recommendations in a precise, auditable way.

### Key rules and principles
- Specification engineers shall document SoC unambiguously for full or partial compliance.
- SoC shall specify:
  - External document name and version
  - External element identifier (message IE, recommendation ID, requirement ID)
  - Full vs partial compliance
  - Exact compliant subset for partial cases
  - System behavior for unsupported elements/values
  - Software release validity
- Recommended representation: single-statement requirement with a compliance table.

### Important definitions
- Statement of Compliance (SoC): explicit confirmation of consistency with an external item, fully or partially.

### Practical do/don't guidance
- Do document unsupported-element handling explicitly (for example reject and fault reporting).
- Do not make broad compliance claims without element-level granularity and release scope.

### Example themes
- Partial compliance table for message handling, with mixed compliant/non-compliant elements and defined rejection behavior.

---

## Chapter 7 - Document Sources, Document Owner and Change History

### Purpose
- Provide source references, ownership, deployment policy, and controlled evolution history of the golden standard itself.

### Key rules and principles
- Bibliography identifies formal source basis (IEEE, ISO/IEC, RE literature, inclusivity references, etc.).
- Document owner is explicitly named.
- Deployment rule:
  - New requirements after official approval/training shall follow the golden standard.
  - Legacy requirements shall be aligned when updated by feature/CNI/pronto.
  - Remaining cleanup strategy is team-decided.
- Change history shall reflect all chapter/subchapter changes with detail on what changed, how, and by whom.

### Important definitions
- Change history scope applies to all golden-standard chapters.
- Version records include review/approval milestones and references to Confluence chapter versions (including version 1.1 baseline content publication).

### Practical do/don't guidance
- Do keep cross-chapter change log complete and specific.
- Do align rollout practice with training/approval gates.
- Do not leave legacy correction policy undefined at team level.

### Example themes
- Version progression from draft through approved versions and subsequent updates.
- Governance-oriented metadata (owner, deployment notes, history granularity).

---

## Chapter 8 - Open Points

### Purpose
- Capture unresolved areas not yet fully covered by approved version 1.1, with intent for later golden-standard updates.

### Key rules and principles
- Chapter records explicit backlog topics from review rounds to drive next-version standardization.

### Important definitions
- Open points include unresolved specification-method questions and governance/process extensions.

### Practical do/don't guidance
- Do treat listed items as pending policy work, not current mandatory rules.
- Do not assume these topics are fully standardized in v1.1.

### Example themes
- How to specify algorithms and user stories.
- Message Definition Table rules.
- Machine-processable manual IDs for traceability automation.
- Potential terminology change discussion around verifiable vs reproducibly testable.
- Additional use case template variant for multi-flow needs.
- Rules for pseudo code usage.
- Further traceability refinements and possible document-structure/template standardization.

---

## Consolidated Checklist for Requirement Writers

- Confirm requirement type first: functional, non-functional, or constraint.
- Use shall for normative requirement obligation in single statements.
- Write syntax-complete requirement:
  - Functional: actor + action, plus precondition/trigger and observable result when needed.
  - Non-functional/constraint: actor + required attribute/characteristic (+ condition when needed).
  - Use case: include all mandatory sections.
- Ensure requirement is agreed by relevant stakeholders through official review.
- Ensure atomicity:
  - One behavior per atomic requirement.
  - If molecular, assign identifiers to sub-requirements/rows.
- Ensure completeness:
  - All needed metadata/attributes present per local guideline.
  - All relevant input combinations covered or explicitly delegated.
  - SW robustness behavior defined and linked to basic behavior.
- Ensure consistency:
  - No conflicts with peer/higher-level requirements.
  - Use repository-consistent naming; avoid synonyms for same object.
- Ensure modifiability:
  - Avoid duplication; reference source requirements.
  - Keep editable source for tables/diagrams (not screenshots).
  - Record exact changes in change history.
- Ensure traceability:
  - Unique persistent ID for each requirement/sub-requirement.
  - Maintain backward and forward traceability where applicable.
  - Tag features/rationale/links as required by RMS and local process.
- Ensure unambiguity:
  - Remove vague/subjective terms unless formally defined.
  - Define abbreviations/acronyms in glossary or explicit note.
  - Use explicit guards for optional/alternative/parallel flows.
- Ensure verifiability:
  - Use measurable terms and objective acceptance basis.
  - Reference authoritative definitions/threshold requirements.
- Follow writing style rules:
  - US English, concise grammar, active voice preferred.
  - Official parameter/message/procedure names only.
  - Prefer tables for complex multi-parameter behavior.
  - Use inclusive language.
- Use notes correctly:
  - Supplementary/context/rationale only.
  - Never hide requirements or implementation status in notes.
- For external standards compliance:
  - State document/version, exact element scope, full vs partial compliance, unsupported handling, and release validity.
- During reviews:
  - Provide clear diff of what was added/removed/changed.
  - Remove obsolete temporary visual tags in subsequent versions.
