# Writing & Editing Style Playbook

How to write so a busy reader gets the answer fast. Load this for any drafting or
editing task.

## Voice & sentence craft

- **Active voice, present tense**: "The service publishes an event" not "an event is
  published". 
- **Second person** for instructions: "Run the migration", "You'll see…".
- **Short sentences, one idea each.** If a sentence has two "and"s, split it.
- **Plain words**: use, not utilize; start, not commence; help, not facilitate. Define a
  term once on first use; then use it consistently.
- **Concrete over abstract**: give the actual command, value, or example, not a
  description of one.
- **Cut filler**: "in order to" → "to"; "it is important to note that" → (delete);
  "there is a function that validates" → "validate()". If removing a word loses nothing,
  remove it.

## Structure for scanning

- **Lead with the answer / most important info first** (inverted pyramid). Detail below.
- **Meaningful headings** that describe content, so a reader jumps straight to their
  section. Keep a logical heading hierarchy.
- **Short paragraphs** (2–4 sentences). One topic each.
- **Lists** for parallel items or sequential steps; **tables** for comparisons and
  field/parameter sets; **code blocks** for code, commands, and output.
- **Bold** sparingly for the key term in a line; never whole paragraphs.

## Instructions that work

- One action per numbered step, in the order performed. Start with a verb.
- Name the exact UI element/state/command — not "click the button". State what the reader
  should see after a step when it's not obvious.
- State prerequisites before step 1; state the expected end result.

## Examples & visuals

- Prefer a runnable example over prose. Make examples realistic (use plausible IDs/values)
  and correct (test them).
- Use a diagram (Mermaid) when structure/flow beats words; a screenshot/Figma frame for
  UI; a table when comparing.
- Annotate code samples with brief comments only where the intent isn't obvious.

## Consistency & accessibility

- One term per concept across all docs; keep a glossary for domain terms.
- Consistent formatting for commands, file names, code, UI labels.
- Expand acronyms on first use. Don't rely on color alone; write descriptive link text
  ("see the API reference", not "click here"); add alt text for images.
- Numbers, units, dates, and casing consistent.

## Editing pass (do this every time)

1. Cut: remove every word/sentence that adds nothing.
2. Simplify: shorten sentences, swap jargon for plain words.
3. Verify: check each command, endpoint, field, and claim against the source.
4. Restructure: front-load the answer; add headings/lists/tables/diagrams.
5. Read as the target reader: can they do the task without getting stuck?

## Quality bar

- Active voice, plain words, short sentences, no filler.
- Scannable: headings, short paras, lists/tables/code blocks.
- Examples runnable and correct; visuals where they help.
- Terminology consistent; accessible link text and alt text.
