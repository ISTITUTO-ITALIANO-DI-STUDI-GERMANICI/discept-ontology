# DiScEPT Semantic Model Proposal

**Revised draft 0.3.1 | 16 September 2026**

This proposal defines a compact semantic model for multilingual digital scholarly editions. TEI/XML remains authoritative for textual content and documentary structure; RDF/OWL represents works, expressions, explicit translation relations, alignments, scholarly claims and provenance. A separate SHACL shapes graph may later validate whether a dataset satisfies the DiScEPT profile.

The main revision is that translation is now modelled at three connected levels: direct relations between expressions, local translation phenomena, and recurrent translation traits or profiles. Persistent identification, interpretation provenance and run-specific confidence are also treated as first-order design requirements.

### Repository contents

| File | Purpose |
|---|---|
| [`discept.ttl`](discept.ttl) | OWL/RDFS ontology and initial governed SKOS vocabularies |
| [`examples/dante-inferno-i.ttl`](examples/dante-inferno-i.ttl) | Worked example using *Inferno* I.1–3 and Longfellow's translation |
| [`LICENSE.md`](LICENSE.md) | Creative Commons Attribution 4.0 licence |

The Dante example records the translation relation, three verse-level alignments and an illustrative interpretation of a local translation phenomenon. The interpretive attribution is explicitly marked as provisional and provenance-bearing rather than as an automatic OWL inference.

## 1 Purpose and scope

DiScEPT supports the acquisition, alignment, annotation and publication of multilingual scholarly editions. Its semantic model must make editorial data queryable and interoperable without duplicating the whole TEI tree in RDF. It must also preserve the distinction between textual evidence, alignment data, scholarly interpretation and the activities that produced or validated them.

The model is designed to support the following tasks:

- represent works, expressions, translations and the specific sources used by translators;
- align analytical units at token, verse, sentence, paragraph, structural or semantic level;
- record automatic and manual alignment, including models, software, parameters and validation;
- describe local translation phenomena and broader translation traits as provenance-bearing scholarly claims;
- connect textual units to TEI/XML, material witnesses and IIIF resources;
- publish stable, dereferenceable resources as Linked Open Data.

## 2 Architecture and domains of authority

The architecture combines complementary representations with different domains of authority. They are coordinated, but they are not required to duplicate every statement.

| **Layer**        | **Primary responsibility**                                                                  | **Status**                   |
|------------------|---------------------------------------------------------------------------------------------|------------------------------|
| TEI/XML          | Textual content, document structure, editorial markup and the link to facsimiles            | Authoritative representation |
| RDF/OWL          | Entities, translation relations, alignments, interpretations, provenance and external links | Semantic representation      |
| SHACL            | Required properties, cardinalities, value ranges, controlled vocabularies and PID patterns  | Conformance validation       |
| Corpus interface | Concordances, CQL, frequencies, collocations and parallel-corpus queries                    | Optional application layer   |

OWL axioms therefore describe meaning and support inference; they are not used as substitutes for closed-world data validation. A missing required value is reported by SHACL, not inferred to exist by OWL.

## 3 Reused standards

| **Standard**              | **Role in DiScEPT**                                                           | **Core status**         |
|---------------------------|-------------------------------------------------------------------------------|-------------------------|
| LRMoo 1.1.1 and CIDOC CRM | Works, expressions, manifestations, items, creation activities and derivation | Core backbone           |
| W3C Web Annotation        | Annotations, targets and selector-based anchoring                             | Core pattern            |
| PROV-O                    | Agents, activities, generation, derivation, revision and software             | Core provenance         |
| HiCO                      | Interpretation acts, interpretation types and criteria                        | Directly reused         |
| CiTO                      | Evidence and relations of agreement, disagreement or refutation               | Reused with HiCO        |
| SKOS                      | Translation phenomena, traits, methods, levels and validation outcomes        | Controlled vocabularies |
| IIIF                      | Images, canvases and regions connected to transcription                       | Material layer          |
| DC Terms and DCAT         | General metadata and dataset publication                                      | Metadata layer          |
| FaBiO                     | Optional publication-specific typing and bibliographic interoperability       | Extension only          |

FaBiO is not used as an alternative bibliographic backbone. It remains aligned with the FRBR model, whereas DiScEPT adopts the current LRMoo model. FaBiO classes may be added as supplementary types when a concrete interoperability use case requires them. Any correspondence with LRMoo must be documented in a separate mapping module. The core ontology must not declare broad equivalence between the two class systems without term-by-term verification.

## 4 Bibliographic model and explicit translation

LRMoo provides the structural backbone. A dsc:Work is realised in one or more dsc:Expression entities. A dsc:Translation is an expression for which the direct source expression is asserted explicitly through dsc:translates. The property is specialised from LRMoo R76 is derivative of and CIDOC CRM P73i is translation of.

> Translation B dsc:translates Expression A

The translation relation is therefore data in the graph, not an inference reconstructed from language, title or membership in the same work. It is not transitive. If C was translated from B and B from A, the graph records both direct relations and preserves the relay chain.

> Translation C dsc:translates Translation B  
> Translation B dsc:translates Expression A

### 4.1 Translation acts and sources

The activity that produced a translation is represented as dsc:TranslationAct, a specialisation of LRMoo F28 Expression Creation and PROV Activity. It creates the translation expression, uses its direct source expression and, when known, identifies the exact source manifestation used by the translator.

> TranslationAct  
> dsc:usesSourceExpression SourceExpression  
> dsc:usesSourceManifestation SourceManifestation  
> dsc:createsTranslation TranslationExpression  
> dsc:translator Translator

This event-based pattern carries date, place and responsibility without turning source and target into permanent qualities of an expression. The same expression may be a target in one direct relation and a source in another.

### 4.2 Translation and interpreting

Translation in the core model denotes textual translation. Oral interpreting may later be introduced as a related linguistic-transformation activity, with links to performance, audio, video and time spans. It must remain distinct from dsc:InterpretiveAssertion, which denotes a scholarly interpretation.

## 5 Textual units and TEI anchoring

The model distinguishes documentary segmentation from analytical segmentation. A dsc:TextSegment corresponds to a TEI element, normally identified by xml:id. A dsc:TextualUnit is the unit used in an alignment or interpretation and may comprise one or more segments. This supports overlapping or discontinuous analytical units without forcing them into the TEI hierarchy.

> TEIDocument  
> └─ TextSegment xml:id="l23"  
> └─ TextualUnit used in an alignment

When an analytical unit is finer than a TEI element, an oa:SpecificResource and selector can provide the anchor. The RDF resource must retain the source TEI document, xml:id and a short anchor quote so that broken links can be detected after editorial changes.

## 6 Alignment cardinality and collective targets

A dsc:Alignment is an addressable entity because it must carry provenance, level, method, validation and possibly computational output. It points to one dsc:AlignmentTarget, which groups all participating dsc:TextualUnit entities and gives them collective semantics.

> Alignment  
> └─ hasAlignmentTarget → AlignmentTarget  
> ├─ hasAlignedUnit → TextualUnit A  
> ├─ hasAlignedUnit → TextualUnit B  
> └─ hasAlignedUnit → TextualUnit C

This pattern replaces the earlier use of oa:Composite. Composite appears only in an informative appendix to the final Web Annotation vocabulary and was removed from the normative vocabulary. DiScEPT therefore defines its own collective target and can provide a JSON-LD mapping when an exchange profile requires one.

The ontology does not contain an OWL minimum-cardinality restriction. The rule that a normal alignment must contain at least two units belongs in SHACL. Alignments may be 1:1, 1:n, n:1 or n:m and may involve more than two languages.

### 6.1 Omission and addition

Omission and addition require a separate pattern because an absence is not a textual segment. In draft 0.3.1 they are represented as interpretive assertions about the relevant translation and one or more attested textual units; the model does not create fictitious empty segments. Before the first SHACL profile is fixed, a pilot dataset must determine whether an explicit AlignmentGap entity is needed for navigation and visualisation.

## 7 Translation phenomena, traits and profiles

DiScEPT distinguishes the translation relation from the scholarly description of what a translation does. The semantic model makes three analytical levels explicit.

| **Level**                    | **Example**                                                       | **Representation**                                                  |
|------------------------------|-------------------------------------------------------------------|---------------------------------------------------------------------|
| Translation relation         | The Italian expression is a translation of the German expression  | dsc:Translation and dsc:translates                                  |
| Local phenomenon             | This aligned passage is an explicitation                          | InterpretiveAssertion with dsc:translationPhenomenon                |
| Translation trait or profile | This translation shows a recurrent tendency towards explicitation | InterpretiveAssertion about a Translation with dsc:translationTrait |

Local phenomena and profile-level traits are SKOS concepts rather than OWL classes. A quantitative query may derive counts and distributions from local assertions. The scholarly conclusion that a pattern constitutes a translation trait remains an explicit InterpretiveAssertion with its own provenance. OWL must not infer that conclusion automatically from an arbitrary frequency threshold.

Draft 0.3.1 includes the initial vocabularies directly in `discept.ttl`. The first local translation phenomena are explicitation, implicitation, omission, addition, condensation, expansion, modulation, transposition and reordering. Separate SKOS schemes are used for correspondence types, alignment levels, alignment methods, certainty levels and validation outcomes, so that analytically distinct categories are not collapsed into a single list.

> Local assertions  
> └─ quantitative aggregation  
> └─ InterpretiveAssertion about Translation  
> ├─ translationTrait → tendency towards explicitation  
> └─ prov:wasDerivedFrom → local assertions or statistical result

## 8 Interpretation with HiCO

The content of a scholarly claim and the activity that produced it are separate entities. dsc:InterpretiveAssertion is an OA Annotation and PROV Entity. It is generated by hico:InterpretationAct, which records the responsible agent, type of interpretation, criterion, evidence, date and relations to other interpretations.

> InterpretiveAssertion  
> ├─ interprets → Alignment TextualUnit Expression or Translation  
> ├─ translationPhenomenon or translationTrait → SKOS Concept  
> ├─ certainty → SKOS Concept  
> └─ prov:wasGeneratedBy → hico:InterpretationAct  
> ├─ hico:hasInterpretationType  
> ├─ hico:hasInterpretationCriterion  
> ├─ cito:citesAsEvidence  
> └─ cito:agreesWith disagreesWith or refutes

HiCO is reused directly; DiScEPT does not coin a parallel InterpretationActivity class. Translation-phenomenon attribution and translation-trait attribution are initial HiCO interpretation types. Semantic analysis and quantitative aggregation are initial interpretation criteria. Their vocabularies remain extensible through SKOS.

## 9 Provenance, confidence and validation

An alignment score belongs to the generation performed by a specific run, not to the abstract correspondence independently of its production. dsc:AlignmentGeneration therefore specialises prov:Generation and carries dsc:confidence. The qualified generation links the alignment to dsc:AlignmentActivity.

> Alignment  
> ├─ prov:wasGeneratedBy → AlignmentActivity  
> └─ prov:qualifiedGeneration → AlignmentGeneration  
> ├─ prov:activity → AlignmentActivity  
> └─ confidence → decimal

The alignment model is a versioned prov:Entity. The software that executes it is a prov:SoftwareAgent. This distinction allows the graph to record, for example, a LaBSE model version, an ONNX runtime version, the algorithm, the threshold and the date of the run without conflating them.

Validation is a separate activity. Each dsc:ValidationActivity produces a dsc:ValidationResult with a SKOS outcome. The full history is retained; an application may materialise a current status for convenience, but that value must not replace the provenance-bearing validation results.

## 10 Material witnesses and IIIF

The physical witness is represented directly as LRMoo F5 Item rather than as a new DiScEPT class. A dsc:DigitalSurrogate derives from that item. A dsc:ImageRegion specialises oa:SpecificResource and uses a selector or IIIF region to identify the relevant portion. Text segments or units may then be linked to the region.

> LRMoo F5 Item  
> └─ DigitalSurrogate or IIIF resource  
> └─ ImageRegion  
> └─ TextSegment or TextualUnit

## 11 Persistent identifier policy

A PID policy must be agreed before production data are published. The ontology can require resources to be identified by IRIs, but it cannot decide institutional ownership, namespace governance or persistence commitments.

| **Resource level**                   | **Recommended identifier**                                  | **Revision policy**                                                                    |
|--------------------------------------|-------------------------------------------------------------|----------------------------------------------------------------------------------------|
| Edition or dataset release           | DOI or another repository PID                               | New PID for a substantial published release, linked to earlier versions                |
| Work Expression Translation Agent    | Stable dereferenceable HTTPS IRI                            | Identity remains stable while metadata may be revised                                  |
| TEI-derived segment                  | Deterministic IRI based on expression identifier and xml:id | Preserve xml:id; issue a new version when the identified content changes substantially |
| Alignment Assertion ValidationResult | Opaque UUID or ULID within the DiScEPT namespace            | New version for a substantive change; link with prov:wasRevisionOf                     |

A provisional pattern is:

> {base}/edition/blixen-01  
> {base}/expression/blixen-01-en  
> {base}/segment/blixen-01-en/p23  
> {base}/alignment/550e8400-e29b-41d4-a716-446655440000  
> {base}/assertion/01J...  
> {base}/release/2026-09

The base namespace must be controlled by IISG or protected by a durable redirect service. Content hashes should not be used as the identity of mutable scholarly entities: correcting the members of an alignment must create a traceable revision, not silently replace its identity. A canonical IRI may resolve to the current version, while version-specific IRIs preserve citation and provenance.

The PID policy must decide:

> • the institutional base namespace and the service responsible for dereferencing it;
>
> • which resources receive public IRIs and which remain internal nodes;
>
> • the deterministic rule for TEI-derived identifiers;
>
> • the generator for graph-native identifiers;
>
> • the distinction between stable identity, version and published release;
>
> • redirect, tombstone and deprecation behaviour when resources move or are withdrawn.

## 12 Ontology and SHACL validation

DiScEPT has one semantic model expressed through coordinated artefacts. discept_ontology.ttl defines the meaning of classes and properties. discept_shapes.ttl will define the conditions that a dataset must satisfy. The second file is not another ontology.

| **Question**                         | **OWL and RDFS**                                                  | **SHACL**                                         |
|--------------------------------------|-------------------------------------------------------------------|---------------------------------------------------|
| What is an Alignment                 | Class meaning and relation to OA Annotation                       | Not defined here                                  |
| What can an alignment target contain | Range semantics                                                   | Required class and minimum number of units        |
| What does translates mean            | Explicit non-transitive relation aligned with LRMoo and CIDOC CRM | Required source count for a Translation           |
| Where can confidence occur           | Domain is AlignmentGeneration                                     | Decimal between 0 and 1                           |
| Which vocabulary is permitted        | Range is skos:Concept                                             | Value must belong to the designated ConceptScheme |
| Which resources require PIDs         | IRI-based resource semantics                                      | Node-kind and pattern checks                      |

The shapes graph should be drafted only after the ontology has been tested on representative data. Otherwise early validation rules risk encoding assumptions that fail for relay translation, n:m alignment, omission, discontinuous units or competing interpretations.

## 13 Interoperability with corpus systems

EPTIC and NoSketch Engine are useful reference cases for parallel and multimodal corpus management, but they are not ontological backbones for DiScEPT. Their value lies in testing sentence-level alignment, written and spoken modalities, synchronisation with media and parallel-corpus queries.

> TEI XML  
> ├─ RDF knowledge graph → SPARQL LOD and semantic analysis  
> └─ corpus export → NoSketch Engine or another corpus interface

A corpus export should reuse stable identifiers for expressions and aligned units. The same textual resources can then support both semantic queries and linguistic queries without making the corpus system part of the ontology.

## 14 Competency questions

> 1\. Which expressions realise the same work?
>
> 2\. Which translations exist for a work, and what are their direct source expressions?
>
> 3\. Who performed a translation act, when and where?
>
> 4\. Which expression and manifestation were actually used as the source of a translation?
>
> 5\. Does an edition define one, several or no base expressions?
>
> 6\. Which TEI segments constitute a given analytical textual unit?
>
> 7\. Which textual units participate in an alignment?
>
> 8\. What is the cardinality of an alignment, and which expressions are represented?
>
> 9\. At what level and by what method was the alignment produced?
>
> 10\. Which activity, model and software generated an alignment?
>
> 11\. What run-specific confidence value was produced?
>
> 12\. Who validated an alignment or interpretation, and what was the outcome?
>
> 13\. Which local translation phenomena have been asserted for a passage?
>
> 14\. Which evidence, criterion and degree of certainty support an interpretation?
>
> 15\. Which recurrent translation traits have been asserted for a translation?
>
> 16\. From which local observations or quantitative results was a translation-profile claim derived?
>
> 17\. Which TEI document, element and xml:id correspond to an RDF textual resource?
>
> 18\. Which text segment corresponds to a IIIF canvas or region and to which material witness?
>
> 19\. Can the alignments be exported for parallel-corpus querying?
>
> 20\. Which PID and version identify the cited edition, expression, segment, alignment or assertion?

## 15 Repository and planned artefacts

The current repository is deliberately compact: this README contains the conceptual proposal, `discept.ttl` contains both the ontology and the initial SKOS vocabularies, `LICENSE.md` states the licence, and `examples/dante-inferno-i.ttl` provides the first worked dataset.

When the model has been tested on further cases, the repository may add a separate SHACL shapes graph, larger vocabulary modules and TEI–RDF, FaBiO or corpus-export mappings. These components should be introduced only when they contain operational material and should be versioned independently while declaring which ontology version they conform to.

## 16 Open design decisions and next test

The following decisions remain open and should be resolved through a pilot dataset:

> • the institutional PID namespace, identifier minting service and versioning policy;
>
> • the final representation of omission and addition, including whether an explicit AlignmentGap is needed;
>
> • the representation of ordered and discontinuous textual units;
>
> • the policy for one, several or no base expressions in an edition;
>
> • the governance process for translation phenomena, traits, criteria and validation outcomes;
>
> • the exact FaBiO mappings required by concrete interoperability scenarios.

The next test dataset should contain one source expression, a direct translation, a relay translation, 1:1 and 1:n alignments, an omission, an automatic alignment with confidence, a human validation, two competing local interpretations and one translation-trait claim derived from several observations. Only after this test should the first SHACL profile be frozen.

## 17 Reference specifications

| [<u>LRMoo version 1.1.1</u>](https://cidoc-crm.org/lrmoo/ModelVersion/version-1.1.1) | [<u>CIDOC CRM</u>](https://www.cidoc-crm.org/)                                  |
|--------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| [<u>W3C Web Annotation Data Model</u>](https://www.w3.org/TR/annotation-model/)      | [<u>W3C Web Annotation Vocabulary</u>](https://www.w3.org/TR/annotation-vocab/) |
| [<u>PROV-O</u>](https://www.w3.org/TR/prov-o/)                                       | [<u>SHACL</u>](https://www.w3.org/TR/shacl/)                                    |
| [<u>Historical Context Ontology</u>](http://purl.org/emmedi/hico)                    | [<u>FaBiO</u>](https://www.sparontologies.net/ontologies/fabio)                 |
