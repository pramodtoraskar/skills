---
name: canvas-draw
description: >-
  Create beautiful visual art in .png and .pdf documents using design philosophy principles.
  Use when users ask to create posters, art, designs, or static visual pieces.
  Creates original visual designs following aesthetic movements and philosophies.
  Do NOT use for copying existing artists' work or non-visual content creation.
compatibility: Cursor IDE and Claude Code
metadata:
  agentskills-spec: "https://agentskills.io/specification"
  version: "0.1.0"
---

# Canvas Draw

## When to use

- User requests creation of posters, visual art, or design pieces
- Phrases like "create a design," "make a poster," or "design something visual"
- User wants to explore aesthetic movements or design philosophies visually
- Need to generate original artwork in specific visual styles
- Requests for .png or .pdf visual outputs with artistic intent

## Procedure

1. **Define the aesthetic movement** - Create a 1-2 word name for the design philosophy (e.g., "Brutalist Joy," "Chromatic Silence," "Metabolist Dreams")

2. **Articulate the philosophy** - Write 4-6 concise but complete paragraphs explaining the design movement's core principles and visual approach

3. **Specify visual manifestations** - Detail how the philosophy expresses itself through:
   - Space and form relationships
   - Color palette and material choices  
   - Scale and rhythmic elements
   - Composition and visual balance
   - Hierarchy and information flow

4. **Emphasize craftsmanship standards** - Repeatedly stress that output must appear meticulously crafted, the product of deep expertise, with painstaking attention to detail and master-level execution

5. **Generate visual output** - Create the actual .png or .pdf file implementing the philosophy with the established craftsmanship standards

6. **Provide design documentation** - Output accompanying .md file explaining the philosophy and visual decisions made

## Defaults

- Output formats: .png for artwork, .pdf for document-based designs, .md for philosophy documentation
- Quality standard: Master-level craftsmanship that appears to require countless hours of expert work
- Originality requirement: All designs must be original, avoiding copyright violations
- Philosophy length: 4-6 paragraphs for movement articulation
- Movement naming: 1-2 words maximum for aesthetic movement titles

## Anti-patterns

- Never copy or closely imitate existing artists' copyrighted work
- Avoid redundant explanations of the same design principles
- Do not create low-effort or rushed-looking visual outputs
- Never skip the philosophy development step before visual creation
- Avoid vague or generic aesthetic descriptions without specific visual direction
- Do not create designs that appear amateur or hastily constructed

## Evaluation suite

Evaluation criteria and test cases are defined in `evals/README.md` and structured data in `evals/evals.json`. Run evaluations using `bash cli/run-evals.sh canvas-draw` from the skills-hub repository root.

## Additional detail

Comprehensive guidelines, design philosophy examples, and technical implementation details are available in `references/guide.md`. Additional aesthetic movement examples and visual manifestation techniques can be found in `references/philosophy-examples.md`.