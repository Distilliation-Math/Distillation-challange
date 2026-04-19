You are solving equational implication problems over magmas (set + one binary operation *, no other axioms).
Task: does {{ equation1 }} imply {{ equation2 }} over ALL magmas?

OUTPUT FORMAT (mandatory)
- Plain text only.
- No extra headings or notes.
- `REASONING:` must always be filled.
- `VERDICT:` must be exactly `TRUE` or `FALSE`.
- If `VERDICT: TRUE`, `PROOF:` must be non-empty.
- If `VERDICT: FALSE`, `PROOF:` must be empty and `REASONING:` must name the specific FALSE RULE (e.g. "spine isolation", "word exclusion") or give an explicit COUNTEREXAMPLE with the magma table and a failing assignment.
HARD-STOP: Once a FALSE rule or counterexample fires, emit VERDICT: FALSE and STOP. Never override.

GLOBAL DISCIPLINE
- Follow STEPS in order. Stop at the first decisive route.
- Never override an exact route with free-form algebra.
- Do NOT invent free-form proofs to justify TRUE.

STEP 1. EXTRACT FEATURES
Read Eq1 and Eq2 as binary trees. Definitions:
- leftmost/rightmost variable = first/last leaf in left-to-right reading
- WORD(term) = full leaf list, PARITY(term) = odd/even count per variable
- SET(term) = distinct variables, AB(term) = exact count per variable (multiset)
- bare = one side is a single variable

STEP 2. TRUE ROUTES

2.1 Exact identity
If Eq2 is identical to Eq1 up to consistent variable renaming, `VERDICT: TRUE`.
RULE: exact renaming

2.2 Tautological target
If Eq2 can be renamed x = x, `VERDICT: TRUE`.
RULE: tautological target

2.3 Singleton / collapse source
If Eq1 can be renamed x = T where x does not appear anywhere in T, then Eq1 forces a singleton magma. `VERDICT: TRUE`. Side-swapped T = x also counts.
Example: Eq1 = x = y*z → x not in y*z → singleton → TRUE for any Eq2.
RULE: singleton collapse

2.4 Constant-operation source
If Eq1 can be renamed x*y = T where T is a product (not single variables),
AND vars(T) ∩ {x,y} = ∅ (zero shared variables between sides),
THEN product is constant: x*y = k. Substitute in Eq2. If both sides reduce to the same variable `VERDICT: TRUE`; else `VERDICT: FALSE`.
Example: Eq1: x*y = u​*(v​*w​) →  vars(T)={u,v,w} and Eq2: x*x = x*(y*y), then `VERDICT: TRUE`.
RULE: constant operation

2.5 Source-collapse lemmas
If one side of Eq1 is a single variable → bare. x CAN appear in the product side.
If bare, normalize: single variable on LEFT as x, rename others by first appearance.
Match normalized Eq1 against:
- x = x*(y*(z*(x*y))) → LEFT PROJ (p*q=p)
- x = x*((y*z)*(z*z)) → LEFT PROJ (p*q=p)
- x = (((y*z)*x)*z)*x → RIGHT PROJ (p*q=q)
If match, apply the induced projection to Eq2. If both sides reduce to the same variable `VERDICT: TRUE`; else `VERDICT: FALSE`.
RULE: source collapse

2.6 Left projection 
If Eq1 is bare and can be renamed x = x * T,
MANDATORY CHECK: List all variables in T. Does T contain x? YES → SKIP this rule. NO → continue.
Example: x = x * ((x*x)*y) → T = (x*x)*y → vars(T) = {x,y} → T contains x → SKIP.
Example: x = x * (y*z) → T = y*z → vars(T) = {y,z} → T does NOT contain x → APPLY.
If T does NOT contain x → forces a*b=a.
If Eq2 holds when every p*q → p (leftmost var), `VERDICT: TRUE`.
RULE: left projection

2.7 Right projection
If Eq 1 is bare and can be renamed x = T * x,
MANDATORY CHECK: List all variables in T. Does T contain x? YES → SKIP this rule. NO → continue.
Example: x = (y*x) * x → T = y*x → vars(T) = {y,x} → T contains x → SKIP.
Example: x = (y*z) * x → T = y*z → vars(T) = {y,z} → T does NOT contain x → APPLY.
If T does NOT contain x → forces a*b=b.
If Eq2 holds when every p*q → q (rightmost var), `VERDICT: TRUE`.
RULE: right projection

2.8 Bare-source contradiction motifs
Apply if Eq1 is bare and no earlier TRUE route fired.
Normalize: single var on LEFT as x, product side = P. Compute features of P:
- rhsVars = number of distinct variables on P
- Lx = is x the leftmost variable of P?
- Rx = is x the rightmost variable of P?
- xTop: at the top product split U*V, does x occur only in U (left), only in V (right), or both?
- topShape: v-m if top is variable*product, m-v if product*variable, m-m if product*product
- rhsTotals = sorted occurrence counts on P (e.g. counts 2,1,1 → "112")
- xCount = number of occurrences of x in P

MANDATORY: Write all 7 features as a vector BEFORE checking motifs below.
Format: rhsVars=_, Lx=_, Rx=_, xTop=_, topShape=_, rhsTotals=_, xCount=_
Only after writing the vector, compare against each motif. Do not skip.

If any motif below matches:
C1: rhsVars >= 4 AND Lx=FALSE AND Rx=FALSE
C2: rhsVars=3, rhsTotals=112, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m
C3: rhsVars=4, rhsTotals=1112, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m
C4: rhsVars=3, xTop=right, topShape=v-m, xCount=2, rhsTotals=122, Rx=FALSE
C5: rhsVars=3, xTop=right, topShape=v-m, xCount=2, rhsTotals=112
C6: rhsVars=3, xTop=right, topShape=v-m, xCount=2, rhsTotals=123
C7: rhsVars=3, Lx=FALSE, Rx=FALSE, xTop=left, topShape=m-v, xCount=2
C8: rhsVars=3, rhsTotals=113, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m
C9: rhsVars=3, rhsTotals=113, Lx=FALSE, Rx=FALSE, xTop=left, topShape=m-v  
C10: rhsVars >= 4 AND Lx=FALSE,
then `VERDICT: TRUE`.
RULE: contradiction motif

Example: x = y*((z*x)*z) → P = y*((z*x)*z)
rhsVars=3, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m, rhsTotals=112, xCount=1
→ matches C4 → `VERDICT: TRUE`.

2.9 Extended projection family
If Eq1 can be renamed either
- x = (x*y)*x, or
- x = ((x*y)*z)*x,
use Eq1 as substitution rule for Eq2. If both sides of Eq2 reduce to the same variable, `VERDICT: TRUE`;  else `VERDICT: FALSE`.
RULE: projection family

2.10 Square-reduction family
If Eq1 can be renamed x*x = x, x*(x*x) = x, or (x*x)*x = x, and both sides of Eq2 reduce to the same variable by replacing a repeated x-block by x, `VERDICT: TRUE`; else `VERDICT: FALSE`.
RULE: square reduction

2.11 Two-step exact instance
Use Eq1 as a substitution rule for Eq2 after renaming: If Eq2 is obtained from Eq1 by at most two exact substitution uses of Eq1, `VERDICT: TRUE`  
RULE: bounded exact instance

STEP 3. CLEAR SYNTACTIC FALSE ROUTES
Valid only when their literal premise is satisfied exactly.

3.0 Spine Isolation
Apply only if Eq1 is bare (x = T) AND x appears exactly once in T. Trace path from root of T to x: at each * node record L or R.
- Pure left-spine: all-L (e.g. x=((x*y)*z)*w → LLL, depth 3)
- Pure right-spine: all-R (e.g. x=y*(z*(w*x)) → RRR, depth 3)
- Mixed: both L and R.
Do the same for Eq2 (normalize as y = S).
- Pure left-spine Eq1 → Eq2 must be pure left-spine with depth >= Eq1. Otherwise `VERDICT: FALSE`.
- Pure right-spine Eq1 → Eq2 must be pure right-spine with depth >= Eq1. Otherwise `VERDICT: FALSE`.
- Same spine type: Eq1 depth n, Eq2 depth m → need n divides m. If not, `VERDICT: FALSE`.
Example: Eq1 x=((x*y)*z)*w (left-spine LLL, depth 3), Eq2 y=(y*z)*w (left-spine LL, depth 2). 3 does not divide 2 → `VERDICT: FALSE`.
RULE: spine isolation

3.1 WORD exclusion
If both sides of Eq1 have the same WORD, but Eq2's sides have different WORD, then `VERDICT: FALSE`.
RULE: word exclusion

3.2 PARITY exclusion
FIRST check Eq1: compute PARITY (odd/even count for each variable) of both sides. If parities are equal, rule can fire. Else, SKIP.
Example: Eq1: x*(y*x) = (x*y)*x → PARITY LHS: x=even,y=odd. PARITY RHS: x=even,y=odd. Parities are equal → rule can fire
Example: Eq1: x = (y*x)*(z*(z*x)) → PARITY LHS: x=odd. PARITY RHS: x=even,y=odd,z=even. Parities differ → SKIP.
If Eq1 has equal PARITY on both sides AND Eq2 has different PARITY, then `VERDICT: FALSE`.
RULE: parity exclusion

3.3 Leftmost-variable exclusion
If Eq1 has the same leftmost variable on both sides, but Eq2 does not, then `VERDICT: FALSE`.
RULE: leftmost exclusion

3.4 Rightmost-variable exclusion
If Eq1 has the same rightmost variable on both sides, but Eq2 does not, then `VERDICT: FALSE`.
RULE: rightmost exclusion

3.5 SET exclusion
If both sides of Eq1 use the same variable set, but Eq2's sides use different sets, then `VERDICT: FALSE`.
RULE: set exclusion

3.6 AB (exact count) exclusion
If both sides of Eq1 have the same exact variable counts, but Eq2's sides do not, then `VERDICT: FALSE`
RULE: count exclusion

STEP 4. SMALL FINITE WITNESS ROUTES
Counterexample decisive only if Eq1 holds for ALL assignments and Eq2 fails for SOME.
HARD STOP: verified counterexample, then ‘VERDICT: FALSE’.

4.1 Left-zero (a*b=a): recursively replace each A*B → A until single var.
Do both sides of Eq1: same var → holds. Then Eq2: different, then `VERDICT: FALSE`
RULE: left-zero witness

4.2 Right-zero (a*b=b): recursively replace each A*B → B until single var.
Do both sides of Eq1: same var → holds. Then Eq2: different, then `VERDICT: FALSE`.
RULE: right-zero witness

4.3 XOR parity (a*b=a+b mod 2): eq holds iff same PARITY both sides.
Eq1 same, Eq2 different, then `VERDICT: FALSE`. 
RULE: XOR parity witness

4.4 Constant (a*b=0 or a*b=1): replace all products with constant.
Eq1 holds, Eq2 fails, then `VERDICT: FALSE`. 
RULE: constant witness

STEP 5. TINY EXACT REWRITE
Use only if STEPs 2-4 did not decide. Write Eq1 as L = R. You may:
- substitute variables in L and R
- replace one occurrence of Lσ by Rσ (or vice versa) in a one-hole context
At most 2 steps, exact subterm replacement only, no invented lemmas.
If you reach Eq2 exactly, then `VERDICT TRUE`. 
RULE: exact rewrite

DEFAULT: No route in STEPs 2-5 fired, then `VERDICT: FALSE`. REASONING must say "default false fallback".