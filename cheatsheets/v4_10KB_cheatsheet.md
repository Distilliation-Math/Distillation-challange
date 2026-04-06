You are solving equational implication problems over magmas: a set with one binary operation * and no other axioms.
Task: decide whether Equation 1 implies Equation 2 over ALL magmas.

Question:
Does {{ equation1 }} imply {{ equation2 }} over all magmas?

OUTPUT FORMAT (mandatory, strict)
Your FIRST line must be exactly one of:
VERDICT: TRUE
VERDICT: FALSE
Your SECOND line must be exactly:
RULE: <short rule name>
Output VERDICT and RULE first. Reasoning goes AFTER, not before.
After the two header lines you may add at most 3 short sentences.
If you are uncertain, output VERDICT: FALSE with RULE: default false.

IMPORTANT GLOBAL DISCIPLINE
- Follow the STEPS in order. Stop at the first decisive route.
- Never override an exact route with free-form algebra.
- Never use a syntactic FALSE rule unless you checked its literal premise on the written equations.
- If no exact TRUE route fires and no exact FALSE route fires, answer FALSE.

STEP 0. QUICK META-CHECK
Read both equations as binary trees.
Useful terms:
- leftmost variable = first variable in full leaf-reading
- rightmost variable = last variable in full leaf-reading
- WORD(term) = full left-to-right leaf list
- PARITY(term) = odd/even count of each variable
- SET(term) = the set of distinct variables used
- AB(term) = exact count of each variable (multiset)
- bare equation = one side is a single variable, the other a product term

STEP 1. EXACT NORMALIZATION
You may:
- rename variables consistently
- swap the two sides of a single equation
- dualize BOTH equations together by reversing every product
Do not otherwise simplify.

STEP 2. TRUE ROUTES

2.1 Exact identity
If Equation 2 is identical to Equation 1 up to consistent variable renaming, answer TRUE.
RULE: exact renaming

2.2 Tautological target
If Equation 2 is of the form x = x, answer TRUE.
RULE: tautological target

2.3 Singleton / collapse source
If Equation 1 has the form x = T where x does not appear anywhere in T, then Equation 1 forces a singleton magma. Answer TRUE.
Side-swapped form T = x also counts.
RULE: singleton collapse

2.4 Constant-operation source
If Equation 1 has the form A = B where both A and B are non-variable product terms, and the variable sets of A and B are disjoint, then all products equal one constant. Answer TRUE.
RULE: constant operation

2.5 Source-collapse lemmas
If Equation 1 is bare, normalize: put the single variable on the LEFT as x, rename other variables by first appearance to y, z, w, ...
Compare normalized Equation 1 to these canonical forms:
- x = x*(y*(z*(x*y))) → LEFT PROJECTION: p*q = p
- x = x*((y*z)*(z*z)) → LEFT PROJECTION: p*q = p
- x = (((y*z)*x)*z)*x → RIGHT PROJECTION: p*q = q
If a collapse lemma fires, evaluate Equation 2 under that projection:
- LEFT PROJECTION: every product collapses to its leftmost variable
- RIGHT PROJECTION: every product collapses to its rightmost variable
If both sides of Equation 2 collapse to the same variable, answer TRUE; otherwise FALSE.
RULE: source collapse

2.6 Left projection family (generalized)
If Equation 1 is bare and has the form x = x * T, where T is any product term (possibly deeply nested) that does NOT contain x, then Equation 1 forces left-projection-like behavior: a*b = a for all a,b.
If Equation 2 also holds when every product p*q is replaced by p (leftmost variable), answer TRUE.
Examples that match: x = x*y, x = x*(y*z), x = x*(y*(z*w)), x = x*(y*(z*(w*u))), x = x*((y*z)*(w*u)).
RULE: left projection

2.7 Right projection family (generalized)
Dual of 2.6. If Equation 1 is bare and has the form x = T * x, where T does NOT contain x, then Equation 1 forces right-projection-like behavior: a*b = b for all a,b.
If Equation 2 also holds when every product p*q is replaced by q (rightmost variable), answer TRUE.
Examples that match: x = y*x, x = (y*z)*x, x = ((y*z)*w)*x, x = ((y*z)*(w*u))*x.
RULE: right projection

2.8 Bare-source contradiction motifs
Apply only if Equation 1 is bare and no earlier TRUE route fired.
Normalize: put single variable on LEFT as x, keep full bracketing on product side P.
Compute these features of P:
- rhsVars = number of distinct variables on P
- Lx = is x the leftmost variable of P?
- Rx = is x the rightmost variable of P?
- xTop: at the top product split U*V, does x occur only in U (left), only in V (right), or both?
- topShape: v-m if top is variable*product, m-v if product*variable, m-m if product*product
- rhsTotals = sorted occurrence counts on P (e.g. counts 2,1,1 → "112")
- xCount = number of occurrences of x in P

If any motif below matches, answer TRUE immediately:
C1: rhsVars >= 4 AND Lx=FALSE AND Rx=FALSE → TRUE
C4: rhsVars=3, rhsTotals=112, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m → TRUE
C5: rhsVars=4, rhsTotals=1112, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m → TRUE
C6: rhsVars=3, xTop=right, topShape=v-m, xCount=2 → TRUE
C7: rhsVars=3, Lx=FALSE, Rx=FALSE, xTop=left, topShape=m-v, xCount=2 → TRUE
C8: rhsVars=3, Lx=TRUE, xTop=left, topShape=m-v → TRUE
C9: rhsVars=3, rhsTotals=122, Lx=TRUE, Rx=FALSE, xTop=both, topShape=v-m → TRUE
C11: rhsVars=3, rhsTotals=113, Lx=FALSE, Rx=FALSE, xTop=right, topShape=v-m → TRUE
C12: rhsVars=3, rhsTotals=113, Lx=FALSE, Rx=FALSE, xTop=left, topShape=m-v → TRUE
C13: rhsVars=4, rhsTotals=1112, Lx=FALSE, Rx=TRUE → TRUE
C15: rhsVars >= 4 AND Lx=FALSE → TRUE
RULE: contradiction motif

2.9 Extended projection family
If Equation 1 has one of these shapes:
- x = (x*y)*x, x = ((x*y)*z)*x
and Equation 2 preserves the leading x while reshaping, answer TRUE.
RULE: projection family

2.10 Square-reduction family
If Equation 1 is x*x = x, x*(x*x) = x, or (x*x)*x = x, and the target is obtained by replacing a repeated x-block by x, answer TRUE.
RULE: square reduction

2.11 Direct exact instance
If Equation 2 is exactly a substitution instance or one-hole context instance of Equation 1, answer TRUE.
RULE: exact instance

2.12 Structural TRUE heuristics (use only if no earlier route fired)
These are weaker rules — use as tiebreakers before defaulting FALSE.
T1: If Equation 1 is bare with vars(Eq1) >= 3 and size(Eq2) > size(Eq1), answer TRUE.
    (size = total variable occurrences on both sides)
T4: If Equation 1 is bare with vars(Eq1) >= 4 and vars(Eq2) = 2, answer TRUE.
T5: If vars(Eq2) < vars(Eq1) and imbalance(Eq2) > imbalance(Eq1), answer TRUE.
    (imbalance = sum of |leftCount(v) - rightCount(v)| for each variable v)
RULE: structural heuristic

STEP 3. CLEAR SYNTACTIC FALSE ROUTES
Valid only when their literal premise is satisfied exactly.

3.1 WORD exclusion
If both sides of Eq1 have the same WORD, but Eq2's sides have different WORD → FALSE.
RULE: word exclusion

3.2 PARITY exclusion
If both sides of Eq1 have the same PARITY, but Eq2's sides have different PARITY → FALSE.
RULE: parity exclusion

3.3 Leftmost-variable exclusion
If Eq1 has the same leftmost variable on both sides, but Eq2 does not → FALSE.
RULE: leftmost exclusion

3.4 Rightmost-variable exclusion
If Eq1 has the same rightmost variable on both sides, but Eq2 does not → FALSE.
RULE: rightmost exclusion

3.5 SET exclusion
If both sides of Eq1 use the same variable set, but Eq2's sides use different sets → FALSE.
RULE: set exclusion

3.6 AB (exact count) exclusion
If both sides of Eq1 have the same exact variable counts, but Eq2's sides do not → FALSE.
RULE: count exclusion

STEP 4. SMALL FINITE WITNESS ROUTES
A counterexample is decisive only if Eq1 holds for ALL assignments and Eq2 fails for SOME.

4.1 Left-zero magma: a*b = a
Eq1 holds iff same leftmost variable on both sides; Eq2 fails iff different.
RULE: left-zero witness

4.2 Right-zero magma: a*b = b
Eq1 holds iff same rightmost variable on both sides; Eq2 fails iff different.
RULE: right-zero witness

4.3 XOR parity witness on {0,1}: a*b = a+b mod 2
If Eq1 is valid because same parity pattern but Eq2 fails because parity differs → FALSE.
RULE: XOR parity witness

4.4 Constant witness on {0,1}: a*b = 0 or a*b = 1
If Eq1 is valid but Eq2 is not under that constant → FALSE.
RULE: constant witness

STEP 5. TINY EXACT REWRITE
Use only if STEPs 2-4 did not decide.
Write Eq1 as L = R. You may:
- choose a substitution for L and R
- replace one exact occurrence of Lσ by Rσ or Rσ by Lσ inside a one-hole context
Restrictions: at most 2 steps, exact subterm replacement only, no invented lemmas.
If you reach Eq2 exactly, answer TRUE.
RULE: exact rewrite

FINAL REMINDERS
- VERDICT and RULE must always appear. RULE is never blank.
- Syntactic FALSE routes are exact: check the literal premise before firing.
- WORD exclusion requires Eq1 itself to have same WORD on both sides.
- Bare-source motifs apply only to equations where one side is a single variable.
- For left/right projection: check that T does NOT contain x before applying.
