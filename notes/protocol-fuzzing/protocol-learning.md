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
        2. Reduce the problem of trace minimization with a NFA 
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
        - Given the message abstractions, we can derive a Mealy-NFA:
            - A mealy-NFA is a DFA where its output depends *on the state and its input*
                - I.e. it is possible to assign an output to each outgoing edge of a state.
            - $\langle S, s_0, A_{In}, A_{Out}, f_{next}, f_{output}\rangle$
                - $S$: The finite set of states
                - $s_0$: The initial state of the NFA
                - $A_{In}$ input alphabet
                - $A_{Out}$ output alphabet
                - $f_{next}: S \times A_{In}$ is the transition function of the NFA ($s_i \to s_j$)
                - $f_{output}: S \times A_{Out}$ is the output function.
            - $f_{next}$ and $f_{output}$ must not be fully defined (i.e. they can be partial) for synthesizing a model.
        - A Mealy-NFA is a *tree*, if it does not have a backward edge (there is no state that is on a path twice)
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

### Theoretical insight: Active L* algorithm

- Active learning approach with a teacher that finds counter-examples
- TODO: Find more information on this 

### Passive learning algorithm

- Approach: collect traces passively and construct model from it
    1. Collect many traces
        - Every trace is a sequence of message pairs $\langle x_i, y_i \rangle$, with $x_i \in MSG_{In}$ and $y_i \in MSG_{Out}$
        - Pre-process them: remove session dependent fields
        - Effectively abstracting the message to $\alpha(x_i)$ and $\beta(y_i)$ while maintaining determinism
    2. Identify possible loops:
        - Lengths of monitored traces are finite: it is not possible to identify loops
        - Assume that loops in the traces are loops in the NFA: 
            - Ex.: $s_1 a, s_2 b, s_3 a, s_4 b$ is minimized to $s_1 a, s_2 b, s_1 a, s_2 b$
        - The loops are removed (for the next step) but the location and contents are recorded
    3. Construct a tree Mealy-NFA
        - For each trace $tr = \{ \langle x_i, y_i \ranlge | 1 \leq i \leq L \}$ do:
            - Set the starting state to $s_0$ 
            - If for $i$ the transition $s, \alpha(x_i)$ **is not defined**:
                - add a new state $s_{new}$ to the current automata
                - add a transition to reach this new state $s_{new}$: $(s, s_{new}, \alpha(x_i), \beta(y_i))$
                - set the current state $s$ to $s_{new}$
            - Else If: The transition **is defined** but conflicting with the witnessed output (i.e. $f_{output}(s, \alpha(x_i)) \neq \beta(y_i)$)
                - Refine the abstractions $\alpha$ and $\beta$ to consolidate the conflict
                - It is necessary to restart the computation from this point on
            - Else: The output corresponds to an already witnessed output (good)
                - Just take the next transition and go on with the next pair.
    4. Merge equivalent states in the tree NFA
        - Getting the minimal NFA is NP-hard and not necessarily achievable for complex protocols
        - The idea is to find pairs of nodes which are isomorphic:
            - two states are considered as roots of subtrees
            - if they produce the same output for any input, they are isomorphic
            - special case: subtrees of height 1 - they have the corresponding outgoing edges with same I/O
        - Algorithm for finding isomorphic edges
            - Start bottom up and color the nodes with 1:1 correspondence (not the leaf node but the node before the leaf)
                - prerequisite: must have the same outgoing edges
            - all matching nodes get the same color
            - Next iteration: start from one node above and compare outgoing edges as well as colors for the successor nodes. Same color $\Rightarrow$ isomorphic
            - Done until there is nothing left.
    5. Finalize the Mealy-NFA
        - Add the self-loops and edges that were removed in (2).
    6. The final Mealy-NFA is the output and taken for flaw detection

### Evaluation: MSN Instant Messenger Implementations

- MSN protocol was analyzed for open-source implementations
- Pre-Requisite: There is a message format that has been reverse-engineered
    - Requirement: A parser must be written for this protocol to parse relevant fields
- Message collection: SOCKS proxy for message interception
- Developed 26 single message fuzzing functions, which fall into four categories: (1) 
    1. Fuzzing data fields; 
        - Delete some or all fields
        - alter the value of fields
        - repeat fields
        - Change the contents but depending on the data types
    2. Fuzzing message type; 
        - change the message type to a different one 
        - either defined but invalid type or undefined type
    3. Intra-session message reordering; 
        - insert new messages
        - repeat messages
        - drop messages
        - Ideally: force unexpected transitions
    4. Transition substitution.
        - Trigger new transitions only by chaning the contents of the message
        - This is possible, because the constructed Mealy-NFA is **not accurate**
- The coverage is increased as the fuzzing is targeted to choose transitions of the state machine as a reference

- Result: 89 crashes in aMSN and 61 in Pidgin 
    - At least denial-of-service is possible
- The automata coverage is restricted to ~35% due to failing logins
    - The assumption was that it is not possible to construct a valid message $x_i \in MSG_{In}$ from $x^* in A_{IN}$

### Future Work & Conclusion

- Lookout to extending the state machine on-the-fly via feedback mechanisms
    - Effectively: combining the advantages of passive and active learning
