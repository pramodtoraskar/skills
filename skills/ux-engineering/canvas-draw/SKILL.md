---
name: canvas-draw
description: Create original visual art in PNG and PDF documents using custom design philosophy movements. Use when users request posters, designs, or static visual pieces. Do NOT use for copying existing artists' work or non-visual content.
compatibility: Cursor IDE and Claude Code
metadata:
  agentskills-spec: "https://agentskills.io/specification"
  version: "0.1.0"
---

# Canvas Draw

## When to use

- User requests creation of a poster, artwork, design, or other static visual piece
- User asks to "create a design philosophy" or "make art based on a movement"
- User wants original visual content in PNG or PDF format
- Example user phrasing: "Create a poster for...", "Design artwork that...", "Make a visual piece inspired by..."

## Procedure

1. **Name the movement** (1-2 words) - Create an original aesthetic movement name like "Brutalist Joy", "Chromatic Silence", or "Metabolist Dreams"

2. **Articulate the philosophy** in 4-6 concise but complete paragraphs, emphasizing throughout that this work represents master-level execution and meticulous craftsmanship

3. **Define visual manifestation** through exactly these five aspects (mention each once):
   - Space and form
   - Color and material  
   - Scale and rhythm
   - Composition and balance
   - Visual hierarchy

4. **Stress craftsmanship repeatedly** - Use phrases like "meticulously crafted", "product of deep expertise", "painstaking attention", "master-level execution" multiple times throughout the philosophy

5. **Create the visual work** - Generate the actual PNG or PDF artwork that embodies the philosophy, ensuring it appears to have taken countless hours and comes from someone at the absolute top of their field

6. **Output files** - Deliver only .md (philosophy documentation), .png, and .pdf files as appropriate

## Defaults

- Movement names: 1-2 words maximum
- Philosophy length: 4-6 paragraphs
- File formats: PNG for digital art, PDF for documents/posters
- Quality standard: Master craftsperson level execution
- Originality: Always create new movements, never copy existing artists

## Anti-patterns

- Do not copy or imitate existing artists' copyrighted work
- Do not create non-visual content or formats outside PNG/PDF/MD
- Do not repeat design aspects - mention color theory, spatial relationships, typography principles only once unless adding new depth
- Do not create generic or low-effort looking designs
- Do not skip the craftsmanship emphasis in the philosophy

## Evaluation suite

See `evals/README.md` and `evals/evals.json` for test cases covering movement creation, visual output quality, and file format compliance. Run evaluations with `bash cli/run-evals.sh canvas-draw` in skills-hub.

## Additional detail

See `references/design-philosophy-guide.md` for detailed examples of movement creation and `references/visual-principles.md` for comprehensive coverage of the five visual manifestation aspects.