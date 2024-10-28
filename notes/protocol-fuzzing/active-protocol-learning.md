# Active Automata Learning 

- Background taken from [DIKEUE](../basebands/related-work/ota-fuzzing/dikeue.md)
    - TODO: More from [] and []
- For FSM and Mealy machine Background refer to [(Passive) Protocol
Learning](./protocol-learning.md)

## General Idea of - $\mathcal{L}^*$

- Most well-known **active** automata learning approach is $\mathcal{L}^*$
- Assumption: the input alphabet $\Sigma$ and the output alphabet $\Psi$ are known 
- There is an oracle, which specifies, whether a word (message sequence) is part of the language

### Query Types

#### Membership Query

- Is a word, i.e. a message sequence sent to the UE $x$ accepted by it? ($x \in^{\text{?}} L$)
- The learner wants to find out, whether a concrete message is in the unknown language $L$
- The oracle replies with "yes", if $x \in L$, otherwise "no"

#### Equivalence Query

- The learner has prepared a hypothesis DFA $\mathcal{H}$ and wants to find out, whether it is
equivalent to the DFA of the language $L$ (let this be $D_L$)
- Formally, this checks whether $\forall x \in \Sigma^*, D_L(x) = \mathcal{H}(x)$
    - If the model is not good enough, the oracle returns one $y \in \Sigma^{*}$, where the two
    Mealy automata mismatch

##### Practical Equivalence

- The formal checking is not feasible for all $x \in \Sigma^*$
- Thus, an approximation is used: a number of membership queries
- The learned automaton is not "correct" anymore but instead "observationally correct"

### Stages of Learning

- The learning is iterative 
- Usually first a hypothesis is created by asking a series of membership queries
- Then the model is validated, by asking an equivalence query to the oracle

- If the models are equivalent, the learning is complete and the automaton is returned
- Otherwise, the learning is continued



