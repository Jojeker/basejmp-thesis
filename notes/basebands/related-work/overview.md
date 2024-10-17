# Overview Related Work

## Related Work in Baseband Exploitation

- OTA-Fuzzing corresponds to Hardware-In-The-Loop Fuzzing

| Work                                      | Research Approach (Category) | Academic/Industrial Research | Target Baseband                                                                       | Target Protocol Version | Target Protocol Component                                                                                                                        | Open Source | Description                                                                                                           |
|-------------------------------------------|------------------------------|------------------------------|---------------------------------------------------------------------------------------|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|-------------|-----------------------------------------------------------------------------------------------------------------------|
| [BaseComp](./static-analysis/basecomp.md) | Static-Analysis              | Academic                     | 16 Images: 10 Samsung (ARM), 5 MTK, 1 srsRAN (all versions provided)                  |                         | Integrity Protection Function during Non-Access-Stratum (NAS) setup. Located on Layer 3 (session management, call control & user authentication) | Yes         | Create a factor graph for the integrity protection function                                                           |
|                                           |                              |                              | - Samsung: Galaxy S8, S8+, S9, S9+, S10e, S10+, S10 5G, S21 5G, S21+ 5G, S21 Ultra 5G |                         | - After receiving a message, the BP has to check whether the message is security protected                                                       |             | - Graph with variables and functions, where input -> fn -> output                                                     |
|                                           |                              |                              | - srsRAN: open source UE                                                              |                         | - If yes: Integrity (i.e. MAC) has to be checked before being processed (e.g. decrypted)                                                         |             | - Use probability theory to compute the relative probability of a variable being true (depending on the inputs & fns) |


## Assumed Threat Model

- BaseComp: Attacker intercepts, drops, injects and modifies messages between the base station and the victim
    - Cryptographic keys are assumed to be secure: Attacker cannot counterfeit messages with a correct MAC


## Related Work in Protocol Learning

k
