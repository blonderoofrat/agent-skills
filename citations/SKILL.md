---
name: citations
description: "Verify every citation before it is published. Fetch each DOI, PMID, arXiv or ISBN from the issuing registry and confirm the title and authors match the claim being made. Use whenever an agent writes, edits or reviews content containing scientific, medical, legal or statistical claims, and whenever a citation is added, reused or inherited from another document."
---

# Citations: fetch it, or do not cite it

**The rule this skill exists to enforce: a citation is not a citation until the identifier has been
fetched from the issuing registry and the returned record has been read.** Not "looks plausible". Not
"the model produced it confidently". Fetched, and read.

## Why this needs to be a rule

Language models generate citations that are structurally perfect and factually invented. The failure
is not random noise: it is worse than that, because the output is *well-formed*:

- The DOI has a valid prefix and a plausible suffix, and resolves to nothing.
- The PMID is eight digits and belongs to a completely different paper.
- The authors are real researchers who work in that field and never wrote that paper together.
- The journal, volume and year are internally consistent and the article does not exist.
- Most dangerous of all: **the paper is real, and does not say what it is cited for.**

That last one survives every check short of reading the abstract, and it is the most common form in
practice. A reviewer skimming the reference list sees a real paper by real authors in a real journal.

Nothing downstream catches this. Spell-checkers do not. Style guides do not. A human reviewer who
trusts the writer does not. The identifier is the only part that can be mechanically checked against
an authority, so it must be.

## The procedure

**1. Resolve every identifier against its registry.** These are free, need no key, and are stable:

| identifier | authority | endpoint |
|---|---|---|
| DOI | Crossref | `https://api.crossref.org/works/<doi>` |
| PMID | PubMed (NCBI E-utilities) | `esummary.fcgi?db=pubmed&id=<pmid>&retmode=json` |
| PMCID | PubMed Central | same E-utilities, `db=pmc` |
| arXiv | arXiv | `http://export.arxiv.org/api/query?id_list=<id>` |
| ISBN | Open Library | `https://openlibrary.org/isbn/<isbn>.json` |

A 404, an empty result set, or a network error is **not** a pass. It is a refusal to publish.

> ⚠ **And a 200 is not a pass either.** Verified while writing this: a deliberately fabricated ISBN
> (`9999999999999`) returns **HTTP 200** from Open Library with valid JSON describing a real
> book: an art-museum exhibition catalogue in Spanish, because the lookup resolved to *something*
> rather than refusing. Crossref returns a clean 404 for a fabricated DOI; Open Library does not.
>
> So registries differ in whether they fail closed, and **you cannot tell which by reading their
> documentation**: you find out by sending a deliberately bad identifier and looking. Do that once
> per registry you rely on. If a registry answers 200 for garbage, status checking is worthless there
> and step 2 below is the only thing standing between you and a confidently wrong citation.

**2. Compare the returned record to what you wrote.** Fetching proves the identifier exists; it does
not prove it is the right one. Compare, at minimum:

- **title**: a fuzzy match is fine, a different subject is not;
- **first author surname**;
- **year**;
- **the claim itself**: does the abstract actually support the sentence carrying this citation?

If the identifier resolves but describes a different paper, that is the dangerous case, and it is a
**hard stop**, not a warning. Fix the citation or delete the claim.

**3. Never let an agent's proposed citation through unfetched.** This includes citations that arrive
from a subagent, a research tool, a previous draft, or a document someone else wrote. Inherited
citations are not pre-verified; they are unverified citations with a longer history.

**4. Record what verification actually happened.** Store, alongside each citation, the date it was
verified and against which registry. Without that, a later reader cannot distinguish "checked" from
"has been sitting here a long time", and the second decays into the first silently.

**5. Re-verify on reuse.** Papers are retracted. DOIs are reassigned when a publisher migrates.
Preprints are superseded by versions of record with different conclusions. A citation verified two
years ago and reused today has been verified for a different world.

## Refuse rather than soften

When a claim cannot be supported, the temptation is to keep the sentence and weaken the wording:
"some studies suggest", "it is thought that". That converts a checkable claim into an uncheckable one
and is strictly worse: the reader still receives the impression, and now nothing can be verified.

Delete the claim, or find a source that genuinely supports it.

## What good looks like

> The compound reduced marker expression in cultured cells.<sup>[4]</sup>
>
> [4] Author S, Author T. *Title of the actual paper.* Journal. 2021;12(3):45–58.
>     doi:10.xxxx/yyyyy, verified against Crossref YYYY-MM-DD.

## What this skill does NOT do

- It does not judge whether a source is *good*: impact, methodology and peer-review status are
  editorial questions for a human with domain knowledge.
- It does not verify claims with no citable source. "Fetched and matched" is a floor, not a ceiling.
- It does not replace a subject-matter reviewer. It removes one specific, mechanical, high-frequency
  failure so that reviewer's attention goes to the parts that need judgment.

## Failure modes of this skill itself

- **A verifier that cannot fail.** If your check reports success when the network is down, when the
  registry returns an empty body, or when the identifier list is empty, it verifies nothing and will
  report clean forever. Test it by feeding it a deliberately fabricated identifier and confirming it
  refuses: **for every registry separately**, because as above they do not all behave the same way.
- **Trusting the status code.** The single most useful thing in this skill is step 2, and the ISBN
  example is why: a fabricated identifier can return 200 and a well-formed record for a real, wrong
  work. If your verifier stops at the status code, it will pass exactly the citations that matter.
- **Verifying identifiers nobody uses.** Check the citations actually referenced by the text being
  published, not a catalogue that may have drifted from it. If the two can disagree, the text wins.
- **Counting without a denominator.** "All citations verified" means nothing unless you also report
  how many there were. Zero of zero passes trivially.
