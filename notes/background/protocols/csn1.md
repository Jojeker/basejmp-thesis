# Concrete Syntax Notation (CSN)

- Contrast to Abstract Syntax Notation (ASN.1)
- Contains bit conditionals + bit length fields; custom structures are already part of the specification

## Constraints of CSN.1

- Idea: The syntax encodes the format of a struct, where predefined structs can be used
- Example (from Wikipedia)

```csn.1
<Example> ::= { 1 <Apple struct> | 0 <Orange struct> } 0;
<Apple struct> ::= < Apple Code : bit(5) >;
<Orange struct> ::= <Orange Code : bit(3) > <PeelType: bit(2)>;
```

- If the first bit is a `1`, then an `Apple struct`
    - An `Apple struct` is 5 bits of `Apple Code`
- If, instead, the first bit is a `0`, then an `Orange struct` follows
    - Composed of two Structs, 3 bit `Orange Code` and 2 Bit `PeelType`
- The final bit is a `0`.
