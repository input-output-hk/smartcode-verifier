# Smart Code Verifier

## Important Notice

**This repository is for the open-source version of the tool.**
Active developments are in a private repository: https://github.com/input-output-hk/sc-fvt 

Please **contact us** to get access to the private repository if you want to contribute.
Open an issue in this repository and we'll contact you back asap!

## Introduction
Smart Code Verifier aims at solving the difficulty developers face when trying to exhaustively verify their smart contract implementations. In the current state-of-the-art for Cardano's smart contracts, two main approaches exist: the use of interactive theorem provers like Coq or Agda to formally verify the intended behavior of smart contracts, and the use of testing techniques like property-based testing.

Existing formal verification tools for Cardano smart contracts are predominantly based on interactive theorem provers. While these tools can ensure security, correctness, and reliability, they typically require expert knowledge of formal methods. Their expressive power makes them valuable for proving complex and intricate properties. However, the steep learning curve, limited support for automated reasoning, and inability to produce counterexamples for falsified theorems make them almost inaccessible to non-experts. Additionally, most of these approaches require developers to express their smart contracts in a formal language, potentially creating gaps between the source code and the formal model's semantics. These gaps often appear when abstractions are introduced to facilitate reasoning.

As a result, most developers and auditors prefer to use powerful testing techniques like property-based testing, e.g., QuickCheck-Dynamic for Haskell or the Fuzz library for Aiken. However, these testing techniques require significant development effort and do not guarantee exhaustiveness. Specifically, the effort needed to implement test scenarios and attacks can grow exponentially with the number and complexity of test objectives. Moreover, the generated scenarios or attacks may still be insufficient to cover all edge cases, leaving bugs and security vulnerabilities undetected.

Our proposed solution is an Automated Verification Tool, developed in Lean4, that combines symbolic model-checking and theorem proving techniques to hide the complexity of formal verification of smart contract implementations. Our goal is to deliver a push-button verification approach where users simply annotate their smart contract code with a set of properties that must be satisfied.

To achieve this, we will provide a universal annotation language with a well-defined syntax and semantics to ensure consistency across the various Cardano smart contract ecosystems (e.g., Plinth, Aiken, Plu-ts, Opshin). Dedicated transpilers are also expected to be developed to automatically translate the annotated smart contract code into Lean4, thus eliminating the need for users to learn Lean4 themselves. This approach streamlines the verification process and avoids discrepancies between the source code and its formal representation.

To minimize user intervention and eliminate the need for manually writing proofs, the verification tool will rely on a set of normalization and simplification techniques, along with SMT solving, to automatically prove/refute as many proof obligations as possible. Proof artifacts will be automatically reconstructed using SMT results and the applied rewriting rules. The tool will also allow users to inspect any unsolved subgoals, enabling them to identify missing auxiliary lemmas. 

The expressive power of the annotation language will enable users to describe both global properties related to the blockchain state, and transaction-related or temporal properties. This capability will allow the formal verification of individual smart contracts, but also of entire decentralized application (DApp) protocols composed of multiple interconnected minting and validator scripts. Hence, users will be able to assess the correctness and security of their DApp protocol as a whole.

Consequently, users will NOT be required to model the blockchain state, blockchain events, or even assumptions guaranteed by the Cardano ledger rules, as these will be considered implicitly in the backend Lean4 formalization. 

The tool’s support for counterexample generation across multiple blockchain events for falsified properties will help users identify potential issues and vulnerabilities within their DApp protocol.

Our approach stands out by seamlessly integrating formal verification into the traditional smart contract development workflow via an easy-to-use CLI tool. The tool's architecture consists of several key components, as illustrated in the diagram below:

* Universal Annotation language: A formal annotation language that, in addition to the usual logical operators, will provide a set of builtin primitives (e.g., transaction-related helper functions, blockchain state observers, temporal operators) to express properties characterizing the expected behaviors of smart contracts or showing absence of security vulnerabilities/deadlock state (see an example in the VSCode screenshot).
* Smart contract language to Lean4 transpilers (C1): Each dedicated transpiler is expected to provide several essential capabilities to facilitate user experience. It will automatically translate annotated smart contract code into Lean4 while ensuring that a predefined set of coding rules are met. Moreover, any Lean4 type-checking errors are expected to be pushed back to the source code level for debugging purposes. A transpiler may also offer the option to automatically generate a set of proof obligations, either to address common security vulnerabilities (e.g., large datum attacks, large value attacks, etc) or to establish low-level correctness.
* Plinth and Plutus Ledger Api Lean4 formalization (C2): This Lean4 formalization serves multiple crucial purposes, namely: it ensures the correctness and absence of security vulnerabilities in all exposed Plinth and Plutus Ledger Api libraries; it provides the fundamental components for developing the Plinth-to-Lean4 transpiler, including the built-in primitives for the universal annotation language and the automatic generation of proof obligations; it introduces the low-level functionalities required to formalize the EUTxO blockchain state; and it may also be used to directly specify and prove Plinth-based smart contracts in Lean4.
* EUTxO Blockchain state Lean4 formalization (C2): Lean4 formalization characterizing the blockchain state and events necessary for smart contact verification. In essence, the EUTxO Ledger state will be modeled as a reactive system, following a state-machine framework. The formalization will account for the assumptions guaranteed by the Cardano ledger rules to accurately define well-formed and well-balanced script contexts, immutability of the blockchain state and absence of double-spending (i.e., a UTXO cannot be spent more than once). 
* CEK Machine Lean4 formalization (C2):Lean4 formalization of the CEK machine to evaluate Plutus Core terms. This formalization is necessary for proving smart contracts code at the UPLC level. Note that for this use case, the properties to be verified must still be expressed using the formal annotation language.
* Lean4 Optimizer (C3): A preprocessor that will apply a set of normalization and simplification rules aiming at reducing model complexity and generating formulas optimized for efficient SMT solving. This optimization phase is essential to enhance the performance of the verification process by allowing the SMT solver to automatically prove or refute as many proof obligations as possible without requiring any user intervention. By simplifying the model representation, the preprocessor will play an essential role in minimizing manual effort while maximizing the tool’s verification capabilities on significant DApp protocols.
* Lean4-to-SMT Translator (C4): Translator to efficiently encode a Lean4 specification into SMTLib2 format while accurately handling higher-order logic to first-order logic transformations as well as induction proof schemas for recursive functions. 
* SMT solvers (C5): Backend solvers invoked to automatically verify smart contract properties, generate proof artifacts or counterexample traces. 
* Interactive proof mode (C6): An interactive mode that will be made available on demand, allowing users to inspect any unsolved subgoals during the verification process. This mode will provide advanced users with the flexibility to explore the proof state, enabling them to assist the backend solvers by providing the necessary reasoning steps to resolve challenging properties.

The architecture is illustrated below:

![image](https://github.com/user-attachments/assets/41037bb6-62f3-424c-a38d-5decc4333260)
*Architecture of the automate verification tool (yellow: components developed in this project, green: already existing software components, white: potential future developments)* 

Key features of our solution include:

* Proof Automation: The tool will leverage normalization, simplification techniques, and SMT solving to automatically prove or refute as many proof obligations as possible. Proof artifacts will be automatically reconstructed based on SMT results and applied rewriting rules, streamlining the verification process.
* Execution Traces: Counterexample generation across multiple blockchain events for falsified properties, which will help users identify potential issues and vulnerabilities within the DApp protocol with greater accuracy.
* User Experience: Designed for a seamless experience, developers will continue working in their preferred smart contract language without needing to interact with Lean4 code in most cases. There's no requirement for users to model the proof execution environment (e.g., blockchain state, transaction submission, or fund transfers). Instead, they will only need to annotate their smart contracts with global state properties aligned with their business logic and security specifications.
* Automated property generation: The tool will be able to automatically generate properties related to common vulnerabilities, such as double satisfaction, large datum attacks, and large value attacks. This will allow users to focus on defining properties specific to their smart contracts without being burdened by common security concerns.
* Advanced Capabilities for Experts: In rare cases where the automated solver cannot resolve complex problems, advanced users can manually intervene using Lean, though this will be an exception rather than the norm.

![image](https://github.com/user-attachments/assets/90198a6f-1f94-4af6-9366-d09470f9408e)
*Mock of the future integration into VSCode, where users only have to annotate their code with the specification (left panel), run the tool through a simple CLI command (bottom panel), and can see realistic traces of execution leading to a property violation (right panel)*

Our tool will primarily benefit two key user groups:

Smart contract developers: They will be able to formally verify their contracts without needing specialized expertise, leading to more secure and reliable contracts for the Cardano ecosystem.
Auditors: Auditors will gain a powerful tool to automatically check contract properties, enhancing the thoroughness of their audits and reducing the time required for manual review.

By greatly simplifying formal verification, this tool will set a new standard for smart contract security in Cardano. The impact will be demonstrable through reduced security vulnerabilities, fewer on-chain failures, and an overall increase in trust for Cardano smart contracts. We expect the Cardano ecosystem to benefit from more efficient audits and more secure smart contracts, promoting wider adoption of blockchain technology for critical applications.
