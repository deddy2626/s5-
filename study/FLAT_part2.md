# FLAT — Part 2: PDA, CSG, LBA, Turing Machines, Decidability

## 5. PDA (Pushdown Automaton)

A finite automaton + a **stack**. Recognizes exactly the **context-free languages**.

**Formal definition (7-tuple):** M = (Q, Σ, Γ, δ, q0, Z0, F)
- Q states, Σ input alphabet, Γ stack alphabet, q0 start state, Z0 initial stack symbol, F final states
- δ : Q × (Σ ∪ {ε}) × Γ → finite subsets of Q × Γ*
  e.g. δ(q, a, X) = {(p, γ)}: in state q, reading a, with X on top: go to p, replace X by string γ (γ = ε pops).

**Instantaneous Description (ID):** (q, remaining input, stack). A move is written ⊢.

**Acceptance:** by **final state** or by **empty stack**. The two are equivalent in power.

**DPDA vs NPDA:** NPDA is strictly more powerful (e.g. {wwᴿ} needs nondeterminism; DPDA languages are a proper subset of CFLs).

**Example 1: L = {aⁿbⁿ | n ≥ 1}** (accept by final state)
```
δ(q0, a, Z0) = (q0, aZ0)      push first a
δ(q0, a, a)  = (q0, aa)       push more a's
δ(q0, b, a)  = (q1, ε)        first b pops a
δ(q1, b, a)  = (q1, ε)        pop for each b
δ(q1, ε, Z0) = (q2, Z0)       all matched → q2 (final)
```
Trace for aabb: (q0,aabb,Z0) ⊢ (q0,abb,aZ0) ⊢ (q0,bb,aaZ0) ⊢ (q1,b,aZ0) ⊢ (q1,ε,Z0) ⊢ (q2,ε,Z0) → accept.

**Example 2: L = {wwᴿ | w ∈ {a,b}*}**
```
q0: push every symbol:   δ(q0,a,Z0)=(q0,aZ0)  δ(q0,b,Z0)=(q0,bZ0)
                         δ(q0,a,a)=(q0,aa) (or (q1,ε))   δ(q0,a,b)=(q0,ab)  ... (all combinations)
guess the middle:        δ(q0, ε, X) = (q1, X)   for X = a, b, Z0
q1: pop on match:        δ(q1,a,a)=(q1,ε)   δ(q1,b,b)=(q1,ε)
accept:                  δ(q1, ε, Z0) = (q2, Z0)
```

---

## 6. PDA → CFG

**Construction** (PDA accepting by empty stack). Variables are of the form **[p X q]**, meaning "starting in state p with X on top, the PDA pops X and ends in state q (consuming the generated string)".

1. **Start:** S → [q0 Z0 q] for every state q.
2. For δ(p, a, X) ∋ (r, Y1Y2…Yk) (a may be ε):
   **[p X s_k] → a [r Y1 s1][s1 Y2 s2] … [s(k−1) Yk s_k]** for **all** choices of s1…s_k ∈ Q.
3. For δ(p, a, X) ∋ (r, ε): **[p X r] → a**.

**Example:** PDA with Z start symbol and transitions (accepts by empty stack)
```
δ(q0, a, Z) = (q0, AZ)     δ(q0, a, A) = (q0, AA)
δ(q0, b, A) = (q1, ε)      δ(q1, b, A) = (q1, ε)     δ(q1, ε, Z) = (q1, ε)
```
- S → [q0 Z q0] | [q0 Z q1]
- [q0 Z r2] → a [q0 A r1][r1 Z r2]   (r1, r2 ∈ {q0, q1}: 4 rules)
- [q0 A r2] → a [q0 A r1][r1 A r2]   (4 rules)
- [q0 A q1] → b ; [q1 A q1] → b ; [q1 Z q1] → ε

**Simplify (remove useless variables, e.g. any with [q0 A q0] which never terminates).** Let X=[q0 Z q1], Y=[q0 A q1], W=[q1 A q1], V=[q1 Z q1]:
S → X, X → aYV, Y → aYW | b, W → b, V → ε
Hence X → aY, Y → aYb | b → language **aⁿ⁺¹bⁿ⁺¹ = aⁿbⁿ (n ≥ 1)** ✔

---

## 7. CFG → PDA

**Construction:** a PDA with **one state q**, accepting by **empty stack**, start stack symbol = S (grammar start).
- For each production **A → α**: **δ(q, ε, A) ∋ (q, α)** (replace variable on top by its right side)
- For each terminal **a**: **δ(q, a, a) = (q, ε)** (match and pop)

**Example:** S → aSb | ab
```
δ(q, ε, S) = {(q, aSb), (q, ab)}
δ(q, a, a) = (q, ε)
δ(q, b, b) = (q, ε)
```
Trace for aabb: (q,aabb,S) ⊢ (q,aabb,aSb) ⊢ (q,abb,Sb) ⊢ (q,abb,abb) ⊢ (q,bb,bb) ⊢ (q,b,b) ⊢ (q,ε,ε) → accept.

*(If the grammar is in GNF, A → aα gives δ(q, a, A) ∋ (q, α), with no ε-moves.)*

---

## 8. Context Sensitive Grammar (CSG, Type 1)

**Definition:** all productions **α → β** with **|α| ≤ |β|** (non-contracting), α contains at least one variable. (S → ε allowed only if S doesn't appear on any right side.) Equivalent form: αAβ → αγβ with γ ≠ ε ("A is rewritten only in context α…β").

**Example: L = {aⁿbⁿcⁿ | n ≥ 1}** (not context-free)
```
S  → aSBC | aBC
CB → BC
aB → ab
bB → bb
bC → bc
cC → cc
```
Derivation of aabbcc:
S ⇒ aSBC ⇒ aaBCBC ⇒ aaBBCC (CB→BC) ⇒ aabBCC (aB→ab) ⇒ aabbCC (bB→bb) ⇒ aabbcC (bC→bc) ⇒ aabbcc (cC→cc)

**Properties:** recognized by a **Linear Bounded Automaton**. Closed under union, intersection, complement, concatenation, Kleene star. Membership is decidable.

**Chomsky hierarchy**

| Type | Grammar | Language | Machine |
|---|---|---|---|
| 0 | Unrestricted | Recursively enumerable | Turing machine |
| 1 | Context sensitive | Context-sensitive | Linear bounded automaton |
| 2 | Context free | Context-free | Pushdown automaton |
| 3 | Regular | Regular | Finite automaton |

Each type is a proper subset of the one above it: Type 3 ⊂ 2 ⊂ 1 ⊂ 0.

---

## 9. LBA (Linear Bounded Automaton)

A **nondeterministic Turing machine whose tape head cannot leave the part of the tape holding the input**. The input is surrounded by end markers **⟨** and **⟩**, which can't be overwritten and the head can't move beyond them. So memory is linear in input length.

**Definition:** (Q, Σ, Γ, δ, q0, ⟨, ⟩, F), with δ: Q × Γ → Q × Γ × {L, R} (nondeterministic).

**Facts**
- LBA accepts exactly the **context-sensitive languages** (CSL ⇔ LBA).
- Number of configurations is finite: |Q| · n · |Γ|ⁿ → so **acceptance (A_LBA) is decidable**, but **emptiness (E_LBA) is undecidable**.
- Example: {aⁿbⁿcⁿ} is accepted by an LBA by repeatedly marking one a, one b, one c.
- Open question: is deterministic LBA = nondeterministic LBA?

---

## 10. Turing Machine and Problems

**Definition (7-tuple):** M = (Q, Σ, Γ, δ, q0, B, F)
- Γ tape alphabet (Σ ⊂ Γ), B = blank, δ : Q × Γ → Q × Γ × {L, R}
- Infinite tape, one read/write head. Accepts by **halting in a final state**.
- Variants (multitape, multitrack, nondeterministic, two-way) are all **equivalent in power**.
- A Turing machine **computes functions** and **recognizes** languages (type 0).

### Problem 1: TM for L = {aⁿbⁿ | n ≥ 1}
Idea: mark an a with X, go right to mark a b with Y, return, repeat. At the end only X's and Y's remain.
```
δ(q0, a) = (q1, X, R)     mark first unmarked a
δ(q1, a) = (q1, a, R)     skip a's
δ(q1, Y) = (q1, Y, R)     skip Y's
δ(q1, b) = (q2, Y, L)     mark matching b
δ(q2, a) = (q2, a, L)     go back left
δ(q2, Y) = (q2, Y, L)
δ(q2, X) = (q0, X, R)     back at X → next a
δ(q0, Y) = (q3, Y, R)     no a's left → verify only Y's remain
δ(q3, Y) = (q3, Y, R)
δ(q3, B) = (q4, B, R)     q4 = final (accept)
```
Trace for aabb: aabb → Xabb → XaYb (q2 goes left) → XXYb ... → XXYY → q3 reads Y,Y,blank → **accept**. For aab or abb no move is defined → **reject**.

### Problem 2: Unary addition 1ᵐ 0 1ⁿ → 1ᵐ⁺ⁿ
```
δ(q0, 1) = (q0, 1, R)
δ(q0, 0) = (q1, 1, R)     replace 0 by 1
δ(q1, 1) = (q1, 1, R)
δ(q1, B) = (q2, B, L)     reached the end
δ(q2, 1) = (q3, B, R)     erase one extra 1 → q3 final
```
(m + 1 + n ones minus one = m + n.)

### Problem 3: Palindromes over {a, b}
Read the first symbol, erase it (write B) and remember it (state qa / qb). Move to the right end. If the last symbol matches, erase it and return left, otherwise reject. Repeat until the tape is empty (accept) or one symbol is left (accept for odd length).

*Tip for the exam:* write the idea first, then the transition table, then trace on a short string.

---

## 11. Recursive and Recursively Enumerable Languages

| | Recursive (Decidable) | Recursively Enumerable (Recognizable) |
|---|---|---|
| TM behaviour | Halts on **every** input: accepts if w ∈ L, rejects if w ∉ L | Accepts if w ∈ L; if w ∉ L may reject **or loop forever** |
| Called | Turing-decidable | Turing-recognizable / semi-decidable |
| Grammar | — | Type 0 |

**Hierarchy:** CFL ⊂ CSL ⊂ **Recursive** ⊂ **RE** ⊂ all languages.

**Key theorems**
- **L is recursive ⇔ both L and its complement are RE.**
- Recursive languages are closed under complement, union, intersection, concatenation, star.
- RE languages are closed under union, intersection, concatenation, star, but **not complement**.
- If L is RE but not recursive, then its complement is **not RE**.
- There exist languages that aren't even RE (Turing machines are countable, languages are uncountable; diagonalization).

**Examples:** A_TM = {⟨M, w⟩ : M accepts w} is **RE but not recursive**; its complement is **not RE**. HALT_TM is RE, not recursive.

---

## 12. Halting Problem is Undecidable — Proof

**HALT = {⟨M, w⟩ | Turing machine M halts on input w}**

**Claim:** no Turing machine decides HALT.

**Proof (by contradiction, diagonalization)**
1. Assume a TM **H** decides HALT: on input ⟨M, w⟩ it halts and **accepts** if M halts on w, and **rejects** if M loops on w.
2. Construct a new TM **D** which takes as input ⟨M⟩ (a TM's description):
   - Run H on ⟨M, ⟨M⟩⟩ (M on its own description).
   - If H **accepts** (M halts on ⟨M⟩) → D **loops forever**.
   - If H **rejects** (M loops on ⟨M⟩) → D **halts**.
3. Run D on its own description ⟨D⟩. Then
   - If D halts on ⟨D⟩, H accepts ⟨D, ⟨D⟩⟩, so by construction D loops. Contradiction.
   - If D loops on ⟨D⟩, H rejects, so D halts. Contradiction.
4. Both cases are contradictory, so H cannot exist. **HALT is undecidable.** ∎

**Remarks to add:** HALT is **recursively enumerable** (just simulate M on w and accept when it halts), so it is RE but not recursive. The proof is used to show other problems undecidable by reduction (A_TM, E_TM, Rice's theorem).

---

## Practice questions
1. Design a PDA for {aⁿb²ⁿ} and for equal number of a's and b's.
2. Convert S → aSb | ε to a PDA, then trace aabb.
3. Design a TM for {aⁿbⁿcⁿ} and for 2's complement of a binary number.
4. Give two languages: one recursive but not CFL, one RE but not recursive.
5. Write the halting problem proof from memory (10 marks!).
