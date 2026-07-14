# Prompting Fundamentals on Real Tasks

**Task chosen from FL-01:** Debugging Python/C++ code errors

## Baseline — naive prompt
**Prompt:** "fix my code"

**Output:** I'd be happy to help fix your code! Could you share the 
code you're working with and the error message you're seeing?

---

## Version 1 — technique: role assignment
**Prompt:** "You are a senior Python debugger with 10 years of 
experience. Fix my code."

**Output:** As a senior debugger, I'll need to see the actual code and 
error message to diagnose the issue. Please share both.

**Note:** Role assignment alone didn't change much - Claude still 
correctly refused to guess without real code. Assigning a role shapes 
tone, not capability, when the core input is missing.

---

## Version 2 — technique: context and motivation
**Prompt:** "You are a senior Python debugger. I'm working on an ML 
internship notebook and got this error: KeyError: 'is_declining_labl' 
from this line: declining_rate = df['is_declining_labl'].mean(). Fix it."

**Output:** The error is a typo: 'is_declining_labl' is missing the 
'e' in "label". It should be: 
declining_rate = df['is_declining_label'].mean()
This is a classic KeyError caused by a misspelled column name.

**Note:** Huge jump - as soon as real context (actual code + error) 
was included, the response went from useless to fully correct. The 
biggest lever isn't technique, it's giving the model something real to 
work with.

---

## Version 3 — technique: output structure
**Prompt:** "...Fix it. Structure your answer as: 1) Root cause 2) The 
fix (code only) 3) How to avoid this in future."

**Output:**
1) Root cause: Typo in the column name.
2) Fix: declining_rate = df['is_declining_label'].mean()
3) Avoid: Use df.columns to check exact column names before 
referencing them, or use tab-autocomplete.

**Note:** Structure made the answer far more scannable and added 
genuinely new value - the "avoid" section wasn't explicitly requested 
content-wise, only format-wise, but appeared because the structure 
demanded it.

---

## Version 4 — technique: few-shot examples
**Prompt:** "...same structure... Example of the style I want: 'Root 
cause: Off-by-one in the loop range. Fix: changed range(n) to 
range(n+1). Avoid: always test loop bounds with a 1-element list first.'"

**Output:**
1) Root cause: 'is_declining_labl' is a typo.
2) Fix: declining_rate = df['is_declining_label'].mean()
3) Avoid: Always run print(df.columns.tolist()) right after loading a 
dataframe.

**Note:** The example mainly tightened terseness and made the "avoid" 
tip more concrete (a specific command instead of general advice). 
Smaller improvement than Version 2 or 3, but real.

---

## Version 5 — technique: step decomposition
**Prompt:** "...Break your process into explicit steps: Step 1 - read 
the error type. Step 2 - locate the line/variable. Step 3 - diagnose 
typo/type/missing-data. Step 4 - fix. Step 5 - prevention tip. Then 
give the same structured answer."

**Output:** (steps shown explicitly, then same 3-part answer as 
Version 4)

**Note (honest "didn't help" moment):** The final answer was nearly 
identical in substance to Version 4. Step decomposition mostly 
restated the same reasoning as visible steps rather than uncovering 
anything new - the bug was too simple to need multi-step reasoning. 
Likely matters more for genuinely complex, multi-cause bugs.

---

## Cross-model comparison (Claude vs ChatGPT, same final prompt)

**Tone:** Claude's answer was terse, matching the few-shot style. 
ChatGPT was noticeably more verbose - it answered the "steps" and the 
"3-part structure" as two separate full deliverables back to back, 
rather than merging them into one response.

**Accuracy:** Identical - both correctly diagnosed the typo and gave 
the same fix.

**Structure:** Claude merged the requested steps into one unified 
3-part answer. ChatGPT treated them as two separate outputs, roughly 
doubling the length for the same content - it followed the instruction 
literally rather than by intent.

**Failure points:** ChatGPT added unrequested defensive-code 
suggestions (an `if column in df.columns` safety check, appearing 
twice) - not wrong, but scope creep for a simple typo fix. Claude 
stayed tighter to exactly what was asked.

**Bottom line:** Same correct diagnosis from both, but Claude respected 
the requested format more precisely, while ChatGPT over-delivered with 
redundant structure and unrequested extras that would need manual 
trimming before reuse.

---

## Final reusable template

You are a senior Python debugger. I got this error: [PASTE ERROR TYPE 
AND MESSAGE] from this line: [PASTE THE EXACT LINE OF CODE].

Structure your answer as exactly one unified response with 3 parts:
1) Root cause
2) The fix (code only)
3) One specific prevention tip (not generic advice - a concrete 
   command or habit)

Keep it terse - no repeated explanations, no unrequested extras like 
defensive error-handling unless I ask for it.
