# JFLAP Practical Assignment — README

This package contains 5 JFLAP (`.jff`) files implementing the Theory of
Computation assignment: 2 DFAs, 2 PDAs, and 1 Turing Machine.

## Files included

| File | Machine | Language / Task |
|---|---|---|
| `Q1_DFA_EndsWith101.jff` | DFA | Binary strings ending in `101` |
| `Q2_DFA_EvenZeros_OddOnes.jff` | DFA | Even number of 0s, odd number of 1s |
| `Q3_PDA_AnBn.jff` | PDA | L = { aⁿbⁿ \| n ≥ 1 } |
| `Q4_PDA_WcWr.jff` | PDA | L = { w c wᴿ \| w ∈ {a,b}* } |
| `Q5_TM_UnaryAddition.jff` | Turing Machine | Unary addition, 1ᵐ 0 1ⁿ → 1ᵐ⁺ⁿ |

How to open: launch JFLAP → File → Open → select the `.jff` file. Use
**Input → Step by Step** or **Input → Multiple Run** to test strings, and
**Input → Step by Step with Tape** for the Turing Machine.

---

## Question 1 — DFA: strings ending in 101

States: `q0` (initial), `q1`, `q2`, `q3` (final)

| State | Input 0 | Input 1 |
|---|---|---|
| → q0 | q0 | q1 |
| q1 | q2 | q1 |
| q2 | q0 | q3 |
| * q3 | q2 | q1 |

Each state tracks the longest suffix read so far that is also a prefix of
`101`.

**Test results (verified by simulation):**

| Input | Result | Reason |
|---|---|---|
| 101 | Accepted | ends with 101 |
| 0101 | Accepted | ends with 101 |
| 100 | Rejected | does not end with 101 |
| 1010 | Rejected | ends with 010 |

---

## Question 2 — DFA: even 0s, odd 1s

States: `qEE` (initial), `qEO` (final), `qOE`, `qOO`
Naming: first letter = parity of 0s seen, second letter = parity of 1s
seen (E = even, O = odd).

| State | Input 0 | Input 1 |
|---|---|---|
| → qEE | qOE | qEO |
| * qEO | qOO | qEE |
| qOE | qEE | qOO |
| qOO | qEO | qOE |

A `0` flips the 0-parity; a `1` flips the 1-parity. Accept only in `qEO`
(even 0s, odd 1s).

**Test results (verified by simulation):**

| Input | 0 count | 1 count | Result |
|---|---|---|---|
| 1 | 0 (even) | 1 (odd) | Accepted |
| 001 | 2 (even) | 1 (odd) | Accepted |
| 101 | 1 (odd) | 2 (even) | Rejected |
| 11 | 0 (even) | 2 (even) | Rejected |

---

## Question 3 — PDA: L = { aⁿbⁿ \| n ≥ 1 }

States: `qS` (initial), `q0`, `q1`, `qF` (final)
Stack alphabet: `Z` (bottom marker), `A` (counter). Initial stack symbol: `Z`.

| From | Input | Pop | Push | To | Purpose |
|---|---|---|---|---|---|
| qS | ε | Z | Z | q0 | start |
| q0 | a | Z | AZ | q0 | push first A |
| q0 | a | A | AA | q0 | push another A |
| q0 | b | A | ε | q1 | first b, pop A |
| q1 | b | A | ε | q1 | pop A for each b |
| q1 | ε | Z | Z | qF | accept when counts match |

**Test results (verified by simulation):**

| Input | Result | Reason |
|---|---|---|
| ab | Accepted | 1 a, 1 b |
| aabb | Accepted | 2 a, 2 b |
| aaabbb | Accepted | 3 a, 3 b |
| aab | Rejected | unequal counts |
| abb | Rejected | unequal counts |
| abab | Rejected | a's and b's not grouped |
| aabbb | Rejected | unequal counts |

Acceptance mode in JFLAP: **final state**.

---

## Question 4 — PDA: L = { w c wᴿ \| w ∈ {a,b}* }

States: `qS` (initial), `qP`, `qM`, `qF` (final)
`c` marks the midpoint, so this machine is deterministic.
The PDF's "X" shorthand (any of Z, a, b) is expanded into 3 explicit
transitions per case, as instructed.

| From | Input | Pop | Push | To | Purpose |
|---|---|---|---|---|---|
| qS | ε | Z | Z | qP | start |
| qP | a | Z/a/b | a·(Z/a/b) | qP | push a |
| qP | b | Z/a/b | b·(Z/a/b) | qP | push b |
| qP | c | Z/a/b | same top | qM | cross midpoint |
| qM | a | a | ε | qM | match/pop a |
| qM | b | b | ε | qM | match/pop b |
| qM | ε | Z | Z | qF | accept |

**Test results (verified by simulation):**

| Input | Result | Reason |
|---|---|---|
| c | Accepted | w = ε |
| aca | Accepted | a c a |
| abcba | Accepted | ab c ba |
| acb | Rejected | a does not match b |
| abcbb | Rejected | extra unmatched symbol |

Acceptance mode in JFLAP: **final state**.

---

## Question 5 — Turing Machine: unary addition

States: `qS` (initial), `q0`, `q1`, `q2`, `qF` (final)
Input format: `1ᵐ 0 1ⁿ` (m, n ≥ 1). Blank symbol: `_`.

| From | Read | Write | Move | To | Purpose |
|---|---|---|---|---|---|
| qS | 1 | 1 | R | q0 | read first symbol |
| q0 | 1 | 1 | R | q0 | scan first operand |
| q0 | 0 | 0 | R | q1 | cross separator |
| q1 | 1 | 1 | R | q1 | scan second operand |
| q1 | _ | _ | L | q2 | hit blank, turn around |
| q2 | 1 | 1 | L | q2 | return over second operand |
| q2 | 0 | 1 | R | qF | replace separator, halt |

### ⚠ Known issue — please read before submitting

This machine is built **exactly** as specified in the assignment PDF. When
actually run, it produces **m + n + 1** ones instead of m + n, because it
never removes a symbol — it only converts the separator `0` to `1`. This
is a flaw in the assignment's own example table, confirmed by simulation:

| Input | Assignment's listed output | What this exact machine actually outputs |
|---|---|---|
| 101 | 11 (2 ones) | 111 (3 ones) |
| 11011 | 1111 (4 ones) | 11111 (5 ones) |
| 111011 | 11111 (5 ones) | 111111 (6 ones) |
| 1110111 | 111111 (6 ones) | 1111111 (7 ones) |

**What to do:** the file as given matches the diagram/transition table you
were handed, so it is the safe choice if grading is based on matching
that design. If you need the *tape output* to equal the listed answers,
add one extra step before `q0` that erases the very first `1` (write
blank, move right) — this removes exactly one unit and brings the final
count down to m + n. Ask if you'd like this corrected variant generated
as a separate file.

---

## Submission checklist

- [ ] FA Question 1 — DFA diagram and `.jff` file
- [ ] FA Question 2 — DFA diagram and `.jff` file
- [ ] PDA Question 3 — PDA diagram and `.jff` file
- [ ] PDA Question 4 — PDA diagram and `.jff` file
- [ ] TM Question 5 — Turing Machine diagram and `.jff` file
- [ ] Test each machine with accepted and rejected inputs
- [ ] For the TM, capture the final tape/output after execution
