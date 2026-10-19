# FLAT — Part 1: DFA Minimization, CNF, GNF

## 1. DFA Minimization — Table Filling Method (Myhill–Nerode)

**Idea:** Two states are *distinguishable* if some string takes one to a final state and the other to a non-final state. Mark all distinguishable pairs; unmarked pairs are equivalent and can be merged.

**Algorithm**
1. Remove unreachable states.
2. Make a table of all pairs (p, q) (lower triangle).
3. **Mark** every pair where one state is final and the other is non-final.
4. Repeat until no change: for each unmarked pair (p, q) and each input symbol a, look at (δ(p,a), δ(q,a)). If that pair is **marked**, mark (p, q).
5. Unmarked pairs = equivalent states → merge them. Build the new DFA.

**Example.** DFA over {0,1}, start A, final = {C, D, E}

| State | 0 | 1 |
|---|---|---|
| A | B | C |
| B | A | D |
| C | E | F |
| D | E | F |
| E | E | F |
| F | F | F |

*Step 1 (mark final vs non-final):* A–C, A–D, A–E, B–C, B–D, B–E, F–C, F–D, F–E are marked.

*Step 2 (check remaining pairs A–B, A–F, B–F, C–D, C–E, D–E):*
- **A–F:** on 1 → (C, F), which is marked → **mark A–F**
- **B–F:** on 1 → (D, F), which is marked → **mark B–F**
- **A–B:** on 0 → (B, A) itself; on 1 → (C, D), unmarked → stays unmarked
- **C–D, C–E, D–E:** on 0 → (E, E); on 1 → (F, F) (same state) → stay unmarked

*Pass 2:* no new marks.

**Equivalent:** A ≡ B, and C ≡ D ≡ E. F stays alone.

**Minimal DFA (3 states)**

| State | 0 | 1 |
|---|---|---|
| →[AB] | [AB] | [CDE] |
| *[CDE] | [CDE] | [F] |
| [F] | [F] | [F] |

---

## 2. DFA Minimization — Equivalence (Partition Refinement) Method

**Definition:** states p, q are **k-equivalent** if they behave the same on all strings of length ≤ k. Start with 0-equivalence and refine until two successive partitions are equal.

**Algorithm**
1. **P0** = {final states}, {non-final states}.
2. **P(k+1):** within each group of Pk, put two states in the same group only if, for every input symbol, their transitions go to the **same group** of Pk.
3. Stop when P(k+1) = Pk. Each group becomes one state.

**Same example**
- **P0** = {C, D, E} , {A, B, F}
- **P1:** In {A,B,F}: A → (0: B∈G2, 1: C∈G1); B → (0: A∈G2, 1: D∈G1); F → (0: F∈G2, 1: F∈G2).
  A and B have signature (G2, G1), F has (G2, G2) → split F.
  {C,D,E}: all go (E→G1, F→G2) → stay together.
  **P1** = {C,D,E}, {A,B}, {F}
- **P2:** {A,B}: A→(0: B, 1: C), B→(0: A, 1: D) → both (same group, {CDE}) → no split. {C,D,E} no split.
- **P2 = P1 → stop.**

Result is the same 3-state DFA as above.

**Table-filling vs Equivalence:** both give the same minimal DFA. Table filling works on pairs, partitioning works on groups (usually faster to write).

---

## 3. CNF — Chomsky Normal Form

**Definition:** every production is of the form **A → BC** (two variables) or **A → a** (one terminal). (S → ε allowed only if ε ∈ L and S is not on any right side.)

**Conversion steps**
1. Add new start symbol S0 → S.
2. **Eliminate ε-productions** (find nullable variables, add all versions of rules with them removed).
3. **Eliminate unit productions** (A → B): replace with B's non-unit productions.
4. **Eliminate useless symbols** (non-generating, unreachable).
5. **Terminals:** in any right side of length ≥ 2, replace terminal a with a new variable Ua → a.
6. **Long right sides:** break A → B1B2…Bk into A → B1C1, C1 → B2C2, … (k − 2 new variables).

**Example:** S → ASA | aB ; A → B | S ; B → b | ε

1. New start: S0 → S
2. Nullable = {A, B}. Remove B → ε. Expand:
   S → ASA | SA | AS | S | aB | a ; A → B | S ; B → b
3. Remove units (S0→S, S→S, A→B, A→S):
   S0 → ASA | SA | AS | aB | a
   S → ASA | SA | AS | aB | a
   A → b | ASA | SA | AS | aB | a
   B → b
4. No useless symbols.
5–6. Replace `aB` by `UB` (U → a), `ASA` by `A A1` with A1 → SA.

**CNF**
```
S0 → A A1 | S A | A S | U B | a
S  → A A1 | S A | A S | U B | a
A  → b | A A1 | S A | A S | U B | a
A1 → S A
U  → a
B  → b
```
*Fact:* a string of length n needs exactly **2n − 1** derivation steps in CNF.

---

## 4. GNF — Greibach Normal Form

**Definition:** every production is **A → aα**, where a is a terminal and α is a (possibly empty) string of variables. Every derivation step produces exactly one terminal, so a string of length n needs n steps. GNF is used to build PDAs without ε-moves.

**Steps**
1. Convert to CNF (no ε, unit, useless).
2. Number the variables A1, A2, …, An.
3. Ensure every Ai → Aj γ has **j > i**: use the **substitution lemma** (replace Aj by its right sides).
4. Remove **left recursion**. **Lemma:** A → Aα1 | … | Aαr | β1 | … | βs becomes
   A → βi | βi Z, and Z → αj | αj Z (Z is a new variable).
5. Back-substitute from An up to A1 so every production starts with a terminal.

**Quick example (substitution):** S → AB ; A → aA | bB | b ; B → b
A and B already start with terminals. Only S → AB is not: substitute A →
S → aAB | bBB | bB. All in GNF.

**Left recursion example:** A → Aa | b becomes **A → bZ | b ; Z → aZ | a**.

**Full example:** A1 → A2A3 ; A2 → A3A1 | b ; A3 → A1A2 | a
- A1 → A2A3 and A2 → A3A1 are fine (j > i). A3 → A1A2 violates (1 < 3).
- Substitute A1 then A2 into A3 → A1A2: A3 → A2A3A2 → A3A1A3A2 | bA3A2. So
  **A3 → A3A1A3A2 | bA3A2 | a** (left recursive).
- Remove left recursion (Z):
  A3 → bA3A2 | a | bA3A2 Z | aZ ;  Z → A1A3A2 | A1A3A2 Z
- Back-substitute:
  A2 → A3A1 gives **A2 → bA3A2A1 | aA1 | bA3A2ZA1 | aZA1 | b**
  A1 → A2A3 gives **A1 → bA3A2A1A3 | aA1A3 | bA3A2ZA1A3 | aZA1A3 | bA3**
  Z → A1A3A2 | A1A3A2 Z: replace A1 by each production above (each followed by A3A2 and A3A2Z).

All productions now start with a terminal → GNF.

---

## Practice questions (try without looking)
1. Minimize a 6-state DFA with both methods and check you get the same answer.
2. Convert S → aSb | ab | ε to CNF.
3. Convert S → AB ; A → BS | b ; B → SA | a to GNF.
4. Why do we need CNF/GNF? (CNF: CYK parsing, derivation bounds, pumping lemma proof; GNF: PDA construction, no left recursion.)
