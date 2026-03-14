graph TD

%% Node Styles
classDef start_end fill:#f96,stroke:#333,stroke-width:2px;
classDef chemistry_logic fill:#bbf,stroke:#333,stroke-width:2px;
classDef dl_model fill:#9cf,stroke:#333,stroke-width:2px;
classDef validation fill:#dfd,stroke:#333,stroke-width:1px;

Start([Input: SMILES or IUPAC]) --> PreProc[Chemical Pre-processor]

subgraph Translation_Core
    PreProc --> Tokenizer[Specialized SMILES/IUPAC Tokenizer]
    Tokenizer --> NMT_Model{{Transformer-based NMT Model}}
    NMT_Model --> RawOutput[Raw String Output]
end

RawOutput --> ValidityCheck{RDKit Validity Check}

%% Verification Loop
ValidityCheck -- Valid Structure --> CrossCheck[Bi-directional Cross-Check]
ValidityCheck -- Invalid --> Retry[Temperature Sampling / Beam Search]

Retry --> NMT_Model

subgraph Verification_Loop_Agentic
    CrossCheck --> ReTranslate[Translate Result Back to Input Format]
    ReTranslate --> Similarity[Tanimoto / String Similarity Score]
end

Similarity -- "High Confidence (>0.9)" --> Formatter[Output Formatter]
Similarity -- "Low Confidence" --> RuleBased[Fallback: Rule-Based Engine]

RuleBased --> Formatter

Formatter --> Benchmarks[Update Accuracy Metrics]
Formatter --> End([Final Identified Molecule])

%% Apply Styles
class Start,End start_end
class NMT_Model,Tokenizer dl_model
class PreProc,CrossCheck,RuleBased chemistry_logic
class ValidityCheck,Similarity validation
