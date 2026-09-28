# V2V Essay Revision

This folder has my essay "The Limits of Certificate Revocation in Vehicle to Vehicle Communication." I wrote it for COMP-348 (Network Security) at Loyola University Chicago in September 2026. The essay argues that revoking certificates can't stop the first false message a vehicle sends, because revocation only happens after the message is already out.

For COMP-250 I revised the essay using Markdown and Git instead of Word. Each change is its own commit, so the history shows how the essay changed.

## Files

- `eldahshan-v2v-essay.md` is the essay.
- The `.json` file is my bibliography, exported from Zotero.
- `apa.csl` sets the citation style to APA.
- `.gitignore` keeps Word files out of the repo.

## What I did

I converted the essay from Word to Markdown with Pandoc and put each sentence on its own line. I added a title block, an abstract, and section headings. I replaced the typed citations with Zotero citation keys so Pandoc builds the bibliography. Then I revised the introduction, the conclusion, and one body paragraph using The Craft of Research by Booth et al.

## Making a Word copy

```
pandoc eldahshan-v2v-essay.md --citeproc -o essay.docx
```
