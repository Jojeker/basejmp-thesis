# Prospex: Protocol Specification Extraction

- Approach: Extract features from messages and determine when two sessions are similar or not
    - Done by recognizing different message types and create a message format specification

- Precondition & assumption: the protocol is not known (i.e. a CnC bot that uses a non-standard protocol) 


## Working Principle

On a high level: We capture input data by recording traces from the running application. 
Then we extract execution traces and, infer the message format.
Next, we extract the features and create a state machine from them.
Finally, the state machine is minimized. 

### Stage 1: Collection

- Read the application output as it processes incoming messages
    - One ends up with execution traces that show how the implementation performs.
    - **Important distinction**: Not only I/O is regarded, but also the implementation's state is included into the analysis!
    - Underlying assumption: the implementation can be tainted to collect execution traces

- Limitation/constraint: only one direction is monitored not both 
     - But it is equally possible to apply the algorithm to both participants, i.e. the server and the client

### Stage 2: Message Format derivation

- The session is analyzed by first splitting up into individual messages and performing message inference on each one.
    - Reason: the message could be split up in multiple segments (e.g. TCP segments that build up the application protocol)
        - The server collects the packets until a full message is set up.
    - I.e.: The message is considered to be as long as the server does not reply.

- The message inference provides a set of *features* that are conveyed by the message.
    - Representation as a tree of fields

- Not necessarily required in the context of cellular networks, as the format is well-known and automatic parsers exist(?)

### Stage 3: Clustering 

- Clustering is used to classify similar message types into types
    - For each message type it is now possible to derive a format specification.

### Stage 4: State machine inference

- First: Similar to [the protocol learning approach](./protocol-learning.md), one starts with a tree Mealy-NFA

- The next step depends on domain-specific knowledge (i.e. one must know the field of application)
    - Heuristics to label the states in the state machine
    - Similar states get the same label

- Finally: The Mealy-NFA is minimized with a minimzation Algorithm: [*Exbar*](./exbar.md)
