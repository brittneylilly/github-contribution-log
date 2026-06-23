# github-contribution-log

# Contribution [#1]: Resolve doc build warnings #3863


**Contribution Number:** 1  
**Student:** Brittney Lilly  
**Issue:** https://github.com/pytorch/ao/issues/3863#event-26695524401  
**Status:** Phase I Complete, Phase II Complete, Phase III Complete

---

## Why I Chose This Issue

I chose this issue because it has an immediate and visible impact on the documentation that contributors and users of the TorchAO library rely on. I wanted to work on something where I could clearly trace the impact. This issue also matched my current skill level, as this is my first issue in the pytorch/ao repository, so I wanted to start with an issue that didn’t require robust knowledge of this codebase. I am currently familiar with HTML but I hope to learn how Sphinx builds documentation from plain text files.

---

## Understanding the Issue

### Problem Description

The pytorch/ao repository (also known as TorchAO) uses Sphinx, a document generator, to read plain text markdown and reStructuredText (RST) files in TorchAO’s docs folder and convert them into publishable HTML documents that are hosted on the official TorchAO documentation website. The problem is that when Sphinx is trying to build the publishable documents, it encounters files with RST/Markdown formatting issues, missing or broken links and references, missing files, and/or import failures. When Sphinx encounters these issues, it produces a warning but continues to build and publish the file. The result is that published material displayed may be visually malformed, not visible to users, or contain missing or broken parts. The goal of this issue is to find and fix the files that generate these warnings. As of 6/18/2026, there are currently 3 warnings being generated. 

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

Setting up my local development environment was more involved than I expected. Because I am on a Windows computer, I had to use WSL2 (Windows Subsystem for Linux) with Ubuntu to run Linux commands like make html that don't work natively on Windows. I installed Miniconda inside Ubuntu to create an isolated virtual environment for the project, then installed the project dependencies by running the four install commands from the doc_build.yml workflow file. One challenge I ran into was that GitHub Desktop could not recognize my repository because it lives inside the WSL file system rather than directly on my Windows C drive, so I still need to resolve how to authenticate and push branches from Ubuntu to GitHub.

### Steps to Reproduce

1. Created and activated a conda virtual environment using Python 3.11 inside Ubuntu
2. Installed the project dependencies by running the four install commands derived from the doc_build.yml workflow file
3. Navigated into the docs folder and ran make html to trigger Sphinx to build the documentation and reproduce the warnings
4. Repeated step 3 three times to ensure the warning were triggered consistently
5. Observed 3 warnings in the build output, confirming the issue exists in the current state of the repository

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** Screenshot showing 3 warnings produced
  <img width="1172" height="562" alt="before_screenshot_3warnings" src="https://github.com/user-attachments/assets/f65dff25-5a40-42a2-913f-46105da3d650" />

- **My findings:** Observed 3 warnings in the build output, confirming the issue exists in the current state of the repository

---

## Solution Approach

### Analysis

The root cause is that a document called `performant_kernels.rst` that exists in the repository at docs/source/ is not referenced in any toctree located in `docs/source/index.rst`. Toctree is a Sphinx directive that inserts a table of contents tree at the specified location. According to the Sphinx documentation, all documents in the source directory must occur in some toctree directive, else Sphinx will throw a warning if it finds a file that is not included. Since performant_kernels.rst is neither included in a toctree or marked as an orphan, Sphinx throws the warning when make html is run. The `performant_kernels.rst` file only contains the word “TBA.” The Git history of the file reveals that it was created on April 17, 2024 and has never been updated since or integrated into the documentation structure.

### Proposed Solution

Add the `:orphan:` Sphinx directive to the very first line of the `performant_kernels.rst` file. This tells Sphinx that the file is intentionally excluded from the toctree and suppresses the warning. The only file I expect to modify is docs/source/performant_kernels.rst.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 
The file `performant_kernels.rst` exists in the docs folder but is not included in any toctree, causing Sphinx to throw a warning when running make html. Instead of throwing a warning, Sphinx should silently accept the file as intentionally excluded from any toctree.

**Match:** 
Before implementing the fix, I searched the codebase using `grep -r "orphan"` docs/source and found that two other files already use the `:orphan:` directive: `docs/source/tutorials/index.rst` and `docs/source/tutorials/sg_execution_times.rst`, confirming this is an accepted pattern in this project.

**Plan:** Step-by-step implementation plan
1. Open `docs/source/performant_kernels.rst`
2. Add `:orphan:` as the very first line of the file
3. Run `make html` locally to confirm the warning is gone
4. Verify the documentation site still looks correct using `sphinx-serve`

**Implement:** 
https://github.com/pytorch/ao/pull/4515 

**Review:** 
I reviewed the pytorch/ao `CONTRIBUTING.md` document. These were the pull request requirements along with information on how I enacted them:
1.	Fork the repo and create your branch from main:  I created `fix-issue-3863` from main
2.	If you've added code that should be tested, add tests:  Not applicable, this is a documentation-only change with no code added
3.	If you've changed APIs, update the documentation: Not applicable, no API changes were made
4.	Ensure the test suite passes: Not applicable, documentation-only change
5.	Make sure your code lints, using ruff for linting: Not applicable, `:orphan:` is an RST directive, not Python code, so Ruff linting does not apply to this change
6.	Complete the Contributor License Agreement ("CLA"): completed


**Evaluate:** 
Since this is a documentation-only change with no Python code modified, no test suite was required. Instead, I verified the fix worked by:
1.	Running `make html` locally before the fix and confirming the warning “document isn't included in any toctree” was present for `performant_kernels.rst`
2.	Adding `:orphan:` to `performant_kernels.rst` and running `make html` again. The warning was completely gone and the total number if warnings dropped from 3 to 2
3.	Using ‘sphinx-serve’ to serve the documentation locally at ‘http://localhost:8081’ and visually confirmed the navigation and site structure looked identical before and after the fix


---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
