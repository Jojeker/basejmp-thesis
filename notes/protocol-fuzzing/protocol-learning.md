# Protocol Learning

## Motivation

- Protocol design is well-studied and several validation techniques exist
- Many issues may be introduced in the implementation phase
    - Unexpected user input (buffer overflows)
    - Protocol attacks: enforce a lower (unsafe) handshaking mechanism during version negotiation
- Issues:
    - *Black box* nature of implementations does not allow thorough security testing
    - *Lack of formal specification*, i.e. a machine understandable protocol specification is missing (especially problematic for cellular standards)
        - For this reason the manual derivation from the human-readable specification leads to implementation issues

## Background: Dumb and Smart Fuzzing

- Flaws in network protocols are one of the most serious problems
    - Enables remote attacks or via the Internet
- A fuzzer can be classified either as 'dumb' or 'smart' depending on its knowledge of the underlying protocol
    - 'Dumb': sends random inputs to its target
        - No knowledge of the underlying protocol
        - Less effective, as it is unable to understand the current state of the target
    - 'Smart': Rely on the understanding of the protocol
        - Leverage the type fields and the message syntax to fuzz deep into the implementation code.
        - Require to have a protocol specification at hand, otherwise they cannot apply their advantage.
        - Conclusion: 'smart' fuzzing is labor intensive and requires manual adaptation for each new protocol (revision)

## Previous Work on Protocol Extraction

- A Model-based Approach to Security Flaw Detection of Network Protocol Implementations (2008)
    - General Idea
        1. Synthesize a behavioral model of the implementation
        2. Reduce the problem of trace minimization with a DFA 
    - Model Construction:
        - Active black-box checking on protocol (i.e. the procedure that derives the protocol can *ask* the implementation)
        - Passive black-box checking: monitor the implementation for traces and derive conclusions from the results
        - Advantageous compared to only message based extraction: 
            - Example: ACKs can serve different purposes and trigger different processing logic for the *same* message
    
    - Correspondence between formal model and protocol implementation:
        - Define two sets of messages: $MSG_{In}$ and $MSG_{Out}$ where the two sets denote all *valid* input and output messages as byte-strings
            - $f_B: MSG_{In} \to MSG_{Out}$ is the function that models a protocol implementation $B$ from its *initial state*
        - Define *message abstractions*: $A_{In}$ and $A_{Out}$ represent *finite* input and output symbols.
            - We define abstraction functions $\alpha: MSG_{In} \to A_{In}$ and $\beta: MSG_{Out} \to A_{Out}$ to get a finite representation for the protocol implementation
            - Intention: $\alpha(x) = \alpha(y) \Rightarrow \beta(f_B(x)) = \beta(f_B(y))$
                - I.e. the I/O mapping and the abstractions commute: we can work with the finite representation from this point on if the above holds.
                - The above condition is fulfilled if the abstraction is *deterministic* regarding the implementation $B$: 
                    - The implementation must be deterministic as a pre-condition (which is practically fulfilled)
                    - The abstraction must maintain this deterministic behavior (e.g. only protocol-relevant fields must affect protocol states)
        - Given the message abstractions, we can derive a Mealy-DFA:
            - A mealy-DFA is a DFA where its output depends *on the state and its input*
                - I.e. it is possible to assign an output to each outgoing edge of a state.
            - $\langle S, s_0, A_{In}, A_{Out}, f_{next}, f_{output}\rangle$
                - $S$: The finite set of states
                - $s_0$: The initial state of the DFA
                - $A_{In}$ input alphabet
                - $A_{Out}$ output alphabet
                - $f_{next}: S \times A_{In}$ is the transition function of the DFA ($s_i \to s_j$)
                - $f_{output}: S \times A_{Out}$ is the output function.
            - $f_{next}$ and $f_{output}$ must not be fully defined (i.e. they can be partial) for synthesizing a model.
        - A Mealy-DFA is a *tree*, if it does not have a backward edge (there is no state that is on a path twice)
        - A trace is defined as a sequence of pairs starting from the initial state $tr = \{ \langle s_0, s_{i_1} \rangle, \langle s_{i_1}, s_{i_2} \rangle, \ldots \}$

    - Finding an implementation flaw requires a predicate, which indicates whether an *insecure* state was hit:
        - This can be modeled as a predicate of output sequences $goal$ where every output sequence that violates a safety condition, is added as a clause.
        - Fulfilling the predicate corresponds to finding a sequence $seq \in A_{In}$ such that $goal(f_B(seq)) = true$, i.e. one clause is fulfilled for a particular output. 
        - By deriving a mealy automaton, the set of inputs can be decreased to a subset of $A_{In}$ that is computationally more feasible to check.
        - The fuzzer changes the input sequence from run to run, i.e. it behaves as a function $Z: A_{In}^* to A_{In}^*$

    - Assumptions that must be held:
        - The protocol must be deterministic (otherwise paths might be conflicting).
        - This means that the *abstraction* function can be implemented automatically by parsing protocol fields.
            - Or: a Neural Network can do it without prior knowledge of the protocol but with higher probability of errors
            - The abstraction removes session specific fields and parses protocol state (the reverse usually does not hold: the fields cannot provide the session information)

### Theoretical insight: Active  L* algorithm

- Active learning approach with a teacher that finds counter-examples
- TODO: Find more information on this 

### 
