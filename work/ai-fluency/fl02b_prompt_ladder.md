# The Prompt Ladder

## Baseline (Run 0) — weak prompt
**Prompt:** "Explain this data"

**Output:** I'd be happy to help explain data! However, I don't see any 
data attached to your message. Could you share the dataset or specific 
numbers you'd like me to explain?

---

## Version 1 — added layer: real context
**Prompt:** "Explain this data: correlation between search_volume and 
impressions_90d is 0.001, computed on 30,000 anonymized web pages."

**Output:** A correlation of 0.001 is essentially zero - it means 
there's virtually no linear relationship between search_volume and 
impressions_90d. Pages with high search volume keywords don't reliably 
get more impressions than pages with low search volume keywords.

**Notes:**
- What changed: added the actual number and sample size
- What improved: it stopped asking me for data and gave a real 
  interpretation instead of a generic definition of "correlation"
- What still failed: doesn't know who's asking, still fairly 
  textbook-level, no connection to any action
- Try next: give it an audience

---

## Version 2 — added layer: defined audience
**Prompt:** "Explain this data to a beginner with no statistics 
background: correlation between search_volume and impressions_90d is 
0.001, computed on 30,000 anonymized web pages."

**Output:** Imagine you have two numbers for each webpage: how many 
people search for a topic and how many times the page actually showed 
up in search results. A correlation of 0.001 basically means these two 
numbers don't move together at all - knowing one tells you almost 
nothing about the other.

**Notes:**
- What changed: added "beginner with no statistics background"
- What improved: dropped jargon like "linear relationship," used a 
  plain-language analogy instead
- What still failed: still purely explanatory, no format, doesn't say 
  what to do with the insight
- Try next: add an output format constraint

---

## Version 3 — added layer: specified output format
**Prompt:** "...Answer in exactly 3 bullet points: what the number 
means, why it matters, and one thing I should NOT conclude from it."

**Output:**
- What it means: correlation of 0.001 is essentially zero
- Why it matters: can't assume high search volume gets more impressions
- What NOT to conclude: doesn't mean search volume is useless for SEO 
  in general - just one sample, not proof

**Notes:**
- What changed: added a strict 3-bullet format with named categories
- What improved: became scannable, forced the "what NOT to conclude" 
  caveat to actually appear
- What still failed: doesn't know why I'm asking or what I'll use it for
- Try next: give it a clearer goal

---

## Version 4 — added layer: clearer goal
**Prompt:** "...My goal: I need 2-3 sentences I can paste directly 
into my research write-up as an 'observed finding,' following 
careful/directional language (not proof, not causal)..."

**Output:**
- What it means: search volume and impressions_90d show essentially no 
  correlation (r = 0.001)
- Why it matters: directional evidence that visibility is driven by 
  other factors, not keyword popularity alone
- What NOT to conclude: does not prove search volume is irrelevant to 
  SEO broadly - single observed correlation, not causal

**Notes:**
- What changed: added the actual downstream goal (research write-up, 
  careful language)
- What improved: wording shifted to match "observed/directional" 
  register without me asking for that phrasing specifically
- What still failed (honest miss): this version and Version 3 aren't 
  hugely different in substance - the goal layer mostly changed tone, 
  not content. This didn't help as much as I expected.
- Try next: add a verification step, since tone was solid but accuracy 
  risk remained unchecked

---

## Version 5 — added layer: verification requirement
**Prompt:** "...Before answering, double-check: does a correlation 
this close to zero actually support the interpretation you're about 
to give, or are you overstating it?"

**Output:**
- What it means: correlation of 0.001 is effectively zero, no 
  measurable linear relationship
- Why it matters: directional evidence against the assumption that 
  high search-volume keywords automatically drive visibility
- What NOT to conclude: doesn't rule out a non-linear relationship, 
  doesn't apply beyond this sample, doesn't prove anything about how 
  search engines rank pages generally

**Notes:**
- What changed: added an explicit self-check instruction before 
  answering
- What improved: the "what NOT to conclude" bullet got noticeably more 
  careful - added "doesn't rule out a non-linear relationship," a real 
  gap that wasn't caught in any earlier version
- What still failed: nothing major left for this specific use case
- Try next: for reuse, build a placeholder template instead of 
  rewriting each time

---

## Final reusable prompt

Explain this data to a beginner with no statistics background: 
[INSERT YOUR FINDING/NUMBER HERE, e.g. "correlation between X and Y is 
0.001, computed on N samples"].

My goal: I need 2-3 sentences I can paste directly into a research 
write-up as an "observed finding," using careful/directional language 
(not proof, not causal).

Answer in exactly 3 bullet points: 
1) what the number means, 
2) why it matters, 
3) one thing I should NOT conclude from it.

Before answering, double-check: does this result actually support the 
interpretation you're about to give, or are you overstating it?
