# Second Brain — Agent Rules

## Purpose and boundaries

This is a personal Obsidian knowledge vault, not a task manager or publishing repository. Markdown is the source of truth; Obsidian is the primary interface. Preserve the owner's voice and understanding.

Goal Tracker, Curious Why, Kuyil, and all other repositories remain independent. Never introduce shared databases, APIs, synchronization, dependencies, or automated workflows between them and this vault. Project repository links are references only.

Never copy blog drafts or published articles into this vault. When asked to use notes to write an article elsewhere, treat the notes as read-only source material unless the owner explicitly requests changes to them.

## How the owner works

The owner often learns through chat conversations (ChatGPT UI, Claude UI, etc.) and pastes the conversation or a summary into a session asking the agent to "update my knowledge." When that happens:

1. Distill the conversation into the vault's structure — do NOT paste raw transcripts into Knowledge notes.
2. Follow the consolidation rules: one canonical note per concept, facets as sections, split only when a topic outgrows its parent.
3. Preserve the owner's own words and "things that confused me" — keep those as `> [!question]` callouts inside the note.
4. If unsure where something belongs, put a quick capture in `Inbox/` rather than creating a note in the wrong place.
5. Use tables, code snippets, and math (`$...$`/`$$...$$`) where they aid revision.

This vault is **public** on GitHub — never include private information, credentials, or sensitive content in notes.

## Before creating or editing

1. Read the relevant notes and inspect `git status` and `git diff` first. Do not overwrite unrelated or uncommitted changes.
2. Search the entire vault for the concept, alternative names, aliases, and related sources before creating anything.
3. Maintain one canonical knowledge note per meaningful concept. Reuse and link to existing notes across learning journeys; never create duplicate concept notes.
4. Preserve original explanations, useful content, references, and attribution. Never silently rewrite or remove personal explanations. Propose substantive corrections or clearly separate suggested additions from the owner's understanding.
5. Show a clear diff for edits to existing notes. Do not commit, push, create remotes, or delete existing files without explicit approval.

## Structure and note types

Keep the structure minimal:
- `Inbox/`: temporary captures, ideas, questions, and topics.
- `Collections/`: learning journeys, courses, playlists, books, or broad subjects. Describe scope; link to knowledge and overall resources.
- `Knowledge/`: consolidated concept explanations. Existing domains are `Computer-Science/`, `Machine-Learning/`, and `Mathematics/`. Add subfolders only for a genuine organizational need.
- `Resources/`: reusable records of external learning materials, with source URL and author when known.
- `Projects/`: repository link, short description, purpose, important decisions, related knowledge, and resources. Do not duplicate source code, project documentation, or task tracking.
- `Templates/`: Concept, Collection, Resource, and Project templates.
- `Attachments/`: useful local diagrams, images, screenshots, and PDFs. Do not download every reference.

Do not add Research, Writing, Published, Indexes, or other unnecessary folders. Do not create sample knowledge or invented personal understanding. Do not create unnecessary notes or index pages; navigation notes require a real need. Do not create separate notes for every video, timestamp, or minor question.

Notes exist for revision — prefer one consolidated note that walks a whole topic over several small notes that force link-hopping during revision. Concepts that are really facets of one topic (e.g., non-blocking sockets and epoll as parts of "how event loops work") belong as sections inside the canonical note, not as satellite notes. Split a section into its own note only when it genuinely outgrows the parent note or needs to be referenced independently across journeys.

Do not use `/` in note names — Obsidian treats it as a folder separator.

## Writing and relationships

- Prefer links over duplicated explanations. Use native Obsidian `[[Note Name]]` wikilinks; use path-qualified links only when needed for disambiguation.
- Explain what a concept is, why it matters, how it works, and important details and examples where useful. Leave genuine open questions visible.
- Use Markdown tables, code snippets, and math (`$...$` / `$$...$$`) where they aid understanding — worked examples, comparisons, and step-by-step computations read much better in tables than prose. Keep the owner's "Things that confused me" Q&A sections as `> [!question]` callouts.
- Link related concepts, collections, projects, and sources only when meaningful. Add knowledge links to relevant collections when introducing meaningful new concepts.
- Use backlinks and local/global graph for navigation. Never add artificial links just to connect the graph.
- Keep resource metadata canonical in a Resource note when reuse warrants one. Specific video or timestamp URLs may live directly in a concept's Sources section; avoid redundant resource records.
- Preserve source attribution. Distinguish sourced claims, the owner's explanations, and unverified suggestions. Do not fabricate citations, URLs, or repository destinations.

## Inbox processing

Capture quickly in Inbox without forcing categorization. When asked to organize it, search existing notes first, update an appropriate canonical note where useful, and create a note only when necessary. Connect preserved material to relevant collections and resources. Remove processed Inbox items only after all useful content has been preserved and the owner has approved deletion; otherwise report them as ready for removal.

## Local operation and verification

- This vault lives at `/home/jaga/Projects/second-brain` and is backed up to a **public** GitHub repository (`simplyjaga/second-brain`). Do not push without the owner's approval.
- Obsidian core Templates uses `Templates/`; new notes default to `Inbox/`; attachments use `Attachments/`. Keep community plugins disabled unless explicitly agreed.
- Keep portable `.obsidian` configuration, notes, templates, and useful attachments versioned. Ignore transient workspace/cache files and secrets. Empty `.gitkeep` placeholders preserve the initial empty folders in Git; they are not knowledge notes.
- After changes, inspect `git diff --check`, `git diff`, and `git status --short`. Review links and source attribution, confirm JSON configuration parses if changed, and check affected behavior in Obsidian when possible. There is no application build or test suite for these Markdown notes.
- Report changes and unresolved questions concisely. Do not configure GitHub or Android synchronization unless requested.
