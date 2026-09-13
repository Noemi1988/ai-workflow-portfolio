# Human-in-the-Loop Word Search Workflow

**Status:** CASE STUDY IN PROGRESS

## Overview

This project explores how a multimodal AI model can be used to solve simple word-search puzzles in a repeatable and verifiable way.

The goal was not simply to ask AI to find words in an image.

The project gradually evolved into a structured workflow separating:

- logical solving
- coordinate-based verification
- visual rendering
- quality control
- final-answer validation

The workflow was developed iteratively through testing, human verification, failure analysis and process refinement.

---

## Problem

A multimodal AI model could often find words in a word-search puzzle, but a response such as:

> "The word is here."

was difficult to verify.

A useful workflow needed to answer more precise questions:

- Where exactly is the word?
- Was every word found?
- Which cells belong to which words?
- Which letters remain unused?
- Do the unused letters form the expected final answer?
- Does the visual solution actually match the logical solution?

This required converting a visual puzzle into structured information.

---

## Approach

The puzzle grid was treated like a spreadsheet.

Columns were labelled:

`A, B, C, D...`

Rows were labelled:

`1, 2, 3, 4...`

Each cell therefore received a unique coordinate such as:

`A1`, `D4`, `H9`

Each detected word could then be stored in a structured form:

`WORD | START-END | DIRECTION | COLOR | FOUND | DRAWN`

Example:

`MONOKLINA | D1-D9 | vertical | pink | YES | NO`

This made it possible to verify the logical solution independently from the graphical output.

---

## Workflow

The current workflow can be represented as:

~~~text
IMAGE
  ↓
GRID ORIENTATION
  ↓
COORDINATE SYSTEM
  ↓
WORD SEARCH
  ↓
LOGICAL SOLUTION
  ↓
FOUND VALIDATION
  ↓
CELL USAGE MAP
  ↓
RENDERER
  ↓
DRAWN VALIDATION
  ↓
UNUSED CELLS
  ↓
FINAL ANSWER
  ↓
APPROVED / REQUIRES REVIEW
~~~

Each word is searched in all eight possible directions:

- horizontal left → right
- horizontal right → left
- vertical top → bottom
- vertical bottom → top
- diagonal down-right
- diagonal down-left
- diagonal up-right
- diagonal up-left

The workflow is designed to stop when the result is uncertain instead of forcing a final answer.

---

## Visual Convention

A consistent visual legend was introduced:

- **green** — horizontal words
- **pink** — vertical words
- **orange** — diagonal words
- **brown** — overlapping word cells
- **red circles** — unused letters forming the final answer

The visual layer is treated as presentation rather than the primary source of truth.

---

## First Validation Loop

Before any final graphic is created, every word must be checked.

Each word receives a `FOUND` status.

The workflow verifies:

- whether every word from the list was found
- whether coordinates are correct
- whether the direction is consistent
- whether the letters along the path match
- whether short, diagonal and vertical words were checked separately

If any word has:

`FOUND = NO`

the workflow returns to the search stage.

The graphic is not produced yet.

---

## Key Failure Case

One test revealed an important weakness.

The logical solver correctly identified:

`SEKRECJA — F2:F9`

The puzzle was also logically solved to the final answer:

`MONTAŻOWNICA`

However, during the graphical reconstruction of the solution, the word `SEKRECJA` was not correctly displayed.

This demonstrated an important distinction:

> A correct logical solution does not automatically produce a correct visual solution.

The failure occurred between the verified logical result and its graphical representation.

The exact technical root cause was not confirmed.

There is not enough evidence to state that the failure was specifically caused by:

- coordinate-to-pixel mapping
- layer ordering
- data transfer between stages
- or another single rendering mechanism

---

## Architectural Change After the Failure

The failure led to a significant workflow change.

Two separate states were introduced.

### FOUND

The word has been correctly identified in the grid.

### DRAWN

The word has actually been rendered on the final graphic.

The workflow now includes two separate quality-control loops:

~~~text
SOLVE
  ↓
VALIDATE ALL FOUND
  ↓
RENDER
  ↓
VALIDATE ALL DRAWN
~~~

A word is no longer considered complete simply because it was found logically.

---

## Second Validation Loop

After the graphic is created, the workflow checks:

- whether every `FOUND = YES` word also has `DRAWN = YES`
- whether every word is visible on the graphic
- whether the correct color was used
- whether overlaps are represented correctly

If the logical solution and the graphic disagree, the graphic must be corrected before the workflow continues.

---

## Unused Cells and Final Answer

After the word list is validated, every grid cell is classified.

Cells belonging to words are marked as used.

Cells belonging to no word receive:

`UNUSED`

Unused letters are read:

1. row by row
2. from left to right
3. from top to bottom

The resulting final answer is then checked for:

- expected number of letters
- meaningful word or expression

If the result fails validation, the workflow does not guess.

It returns:

`REQUIRES_REVIEW`

---

## Further Iterations

Later experiments focused on improving the visual layer.

Additional ideas included:

- layered rendering
- drawing groups in a defined order
- separate handling of nested or overlapping matches
- `PRIMARY / SECONDARY` classification
- stricter rules for brown overlap markers
- `FINAL / AUDIT` modes
- additional review states

These iterations improved the workflow design, but the renderer is still not considered fully reliable.

---

## Human Role

My role was not limited to writing prompts.

I designed and refined the workflow by:

- preparing test cases
- solving puzzles manually to create reference answers
- introducing the coordinate system
- defining the visual legend
- comparing AI results with manually verified solutions
- identifying failure cases
- defining acceptance criteria
- separating logical solving from graphical rendering
- introducing independent `FOUND` and `DRAWN` states
- requiring the workflow to stop instead of guessing
- turning individual errors into reusable workflow rules
- testing whether manual preparation steps could eventually be removed

The human contribution therefore included:

**workflow design, test design, human QA, failure analysis, acceptance criteria and iterative refinement.**

---

## AI Role

The AI model was used to:

- analyze puzzle images
- determine grid orientation
- read the letter grid
- read the word list
- search words in multiple directions
- calculate start and end coordinates
- classify word direction
- track used and unused cells
- identify the final answer
- attempt visual reconstruction
- compare results with reference solutions
- help formalize new rules into instructions and pseudocode

This project is not presented as a finished software application.

It is a tested, multimodal, human-designed AI workflow.

---

## Quality Control

Quality control operates at several levels.

### Word Level

Each word is checked for:

- correct letters
- correct length
- correct start and end coordinates
- consistent direction

### List Level

Every word must be explicitly marked as found or missing.

### Cell Level

The workflow tracks which cells belong to which words.

This enables detection of:

- overlaps
- unused cells
- inconsistencies between logical and visual output

### Render Level

Every word with:

`FOUND = YES`

should also have:

`DRAWN = YES`

### Final Answer Level

The final answer is checked for:

- expected length
- meaningful result

### Human QA

AI output is compared with a manually prepared reference solution.

This human verification step was essential for detecting errors that the model did not identify by itself.

---

## Current Project State

| Component | Status |
|---|---|
| Solver | Works in the tested range |
| Validator | Partially working |
| Renderer / visual layer | Unresolved problem |
| Word-search generator | Prototype |

The strongest part of the project is currently the logical solving and validation workflow.

The weakest component is the graphical rendering layer.

---

## What the Project Demonstrated

The most valuable result was not simply teaching an AI model to solve a puzzle.

The project demonstrated how an AI-assisted workflow can improve through repeated failure analysis:

~~~text
TEST
  ↓
FAILURE
  ↓
ANALYSIS
  ↓
NEW PROCESS RULE
  ↓
NEW VALIDATION GATE
  ↓
RETEST
~~~

The failure of the visual layer directly led to a better process architecture.

Instead of hiding errors, the workflow was redesigned around them.

---

## Planned Portfolio Artifacts

The final public case study should include:

- a synthetic demonstration word-search puzzle
- a structured solver output table
- an example of a logical-result vs visual-result mismatch
- a workflow architecture diagram
- an example of `FOUND / DRAWN / REQUIRES_REVIEW`
- a simplified QA checklist

Original photographed puzzle-book pages will not be used in the public portfolio.

A synthetic example will be created instead.

---

## Next Steps

The next development steps are:

1. Create a synthetic demonstration puzzle.
2. Reproduce the renderer failure in a controlled example.
3. Improve the visual rendering pipeline.
4. Test the workflow on a defined set of puzzles.
5. Define measurable success criteria.
6. Document the final architecture.
7. Update this case study with visual examples.

---

## Key Lesson

A visually convincing AI output is not necessarily a correct output.

Separating:

**reasoning → validation → rendering → render validation**

made the workflow easier to verify, debug and improve.
