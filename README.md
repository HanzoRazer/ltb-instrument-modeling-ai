# ltb-instrument-modeling-ai

A standalone repository that houses ML-powered second-pass correction for instrument geometry. The repository contains two tightly coupled components: InstrumentBodyGenerator (IBG), a second-pass geometry corrector that refines first-pass extraction output; and the ML training layer that teaches IBG how to correct geometry more accurately by learning from a corpus of authentic instrument measurements.

The components are co-located because they co-evolve: the ML layer's output format and IBG's consumption of it are coupled, and changes to one typically require corresponding changes to the other.

This repository is part of the Luthiers Toolbox ecosystem (`ltb-*` prefix convention). It sits downstream of vectorizer pipelines and upstream of design tools that consume completed body geometry.

## Core components

### InstrumentBodyGenerator (IBG)

IBG takes first-pass body geometry output from upstream consumers and applies a second-pass correction informed by ML recognition of instrument type, shape, and brand.

IBG does **not**:
- generate geometry from landmark points,
- synthesize geometry from sparse inputs, or
- replace extraction logic.

It refines geometry that already exists.

Typical flow:
1. An upstream consumer sends a geometry payload and context.
2. The ML layer identifies instrument type/shape/brand.
3. IBG applies corrections (proportions, curves, known relationships).
4. IBG returns corrected geometry.

### ML training layer

The training layer includes three scaffolded components extracted during the IBG + ML Repo Extraction sprint:

- **TrainingDataCollector**  
  Collects training examples from the 275-plan instrument library as plans are processed and measured through the plan-ingestion workflow.

- **GeometryCoachV2**  
  Core recognition/coaching logic. Produces classification output (instrument type, shape family, brand) and correction guidance consumed by IBG.

- **FeedbackSystem**  
  Collects in-pipeline feedback data. It is currently running but underutilized until the full training loop is wired.

## Training data pipeline

A planned pipeline processes plans from the 275-plan library into normalized training examples consumed by GeometryCoachV2.

Inputs: PDFs, DXFs, and image formats.  
Outputs: landmark measurements, dimensional data, and normalized shape characteristics for the training corpus.

Pipeline implementation is planned for a future iteration; it is not fully built in this repository yet.

## Upstream consumers of IBG

- **Body Outline Editor** (currently connected)
- **Blueprint vectorizer v3.6** (connection to be verified)
- **Photo vectorizer blueprint path** (connection to be implemented in the PhotoVectorizerV2 blueprint extraction path)
- **Future consumers** that produce first-pass body geometry and require authenticity-informed refinement

## Scope boundary

This repository handles **instrument body geometry correction** and its supporting ML layer.

Out of scope:
- other instrument components (rosettes, headstocks, inlays, soundholes, marquetry),
- first-pass extraction from source assets,
- outline generation from sparse landmarks,
- inverse design from acoustic/ergonomic targets.

## Architectural relationships

- **Analogous to `calculators/plate_design/inverse_solver.py`**  
  IBG is treated as foundational library code consumed by imports, not by direct router/API registration.

- **Distinct from deprecated `sg.coach`**  
  Naming similarity is coincidental. This repository coaches geometry correction logic, not human practice.

- **Dependency direction**  
  This repository has no Ross-authored repo dependency. Consumers import from this repository.

## Evolution considerations

- Consumer integrations are expected to grow.
- Correction quality improves as training data from the 275-plan corpus accumulates.
- ML infrastructure may later generalize to other components if second-consumer evidence supports it.
- IBG and vectorizer evolution are intentionally decoupled through first-pass/second-pass separation.

## Component classification

- **IBG files:** Library/foundational.
- **TrainingDataCollector:** Scaffolded.
- **GeometryCoachV2:** Scaffolded.
- **FeedbackSystem:** Scaffolded, running but underutilized until downstream training consumption is wired.

## License and governance

Licensing and commercial packaging posture are part of broader strategy decisions. This repository follows `ltb-*` governance conventions and is maintained by Ross Echols.
