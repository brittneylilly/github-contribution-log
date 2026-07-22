# github-contribution-log

# Contribution [#1]: Resolve doc build warnings #3863


**Contribution Number:** 1  
**Student:** Brittney Lilly  
**Issue:** https://github.com/pytorch/ao/issues/3863#event-26695524401  
**Status:** Phase IV Complete and PR Merged

---

## Why I Chose This Issue

I chose this issue because it has an immediate and visible impact on the documentation that contributors and users of the TorchAO library rely on. I wanted to work on something where I could clearly trace the impact. This issue also matched my current skill level, as this is my first issue in the pytorch/ao repository, so I wanted to start with an issue that didn’t require robust knowledge of this codebase. I am currently familiar with HTML but I hope to learn how Sphinx builds documentation from plain text files.

---

## Understanding the Issue

### Problem Description

The pytorch/ao repository (also known as TorchAO) uses Sphinx, a document generator, to read plain text markdown and reStructuredText (RST) files in TorchAO’s docs folder and convert them into publishable HTML documents that are hosted on the official TorchAO documentation website. The problem is that when Sphinx is trying to build the publishable documents, it encounters files with RST/Markdown formatting issues, missing or broken links and references, missing files, and/or import failures. When Sphinx encounters these issues, it produces a warning but continues to build and publish the file. The result is that published material displayed may be visually malformed, not visible to users, or contain missing or broken parts. The goal of this issue is to find and fix the files that generate these warnings. As of 6/18/2026, there are currently 3 warnings being generated. The 3 warnings are:
1. WARNING: [autosummary] failed to import torchao.prototype.quantization.Int8DynamicActivationInt4WeightConfig
2. WARNING: image file not readable: `eager_tutorials/output.png`
3. WARNING: document isn't included in any toctree for `docs/source/performant_kernels.rst`

### Expected Behavior

When `make html` is run, no warnings should be produced.

### Current Behavior

Running `make html' produces 3 warnings.


### Affected Components

Based on the warning messages, the parts of the codebase involved are: `prototype/quantization`, `eager_tutorials/output.png`, `docs/source/performant_kernels.rst`

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

- **Commit showing reproduction:**
  https://github.com/brittneylilly/pytorch_ao/commit/2138f14a51dd60f4ac2f93c4beddc6237a8006a5
  
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

### Manual Testing
I tested my fix manually by running `make html` in the docs folder before and after adding `:orphan:` to `performant_kernels.rst`. Before making this change, the result of running `make html` the produced 3 warnings including the warning that `performant_kernels.rst` isn't included in any toctree. After adding `:orphan:` to `performant_kernels.rst`, I reran `make html` and the build produced only 2 warnings, confirming the toctree warning was completely resolved. I also used sphinx-serve to serve the documentation locally at http://localhost:8081 and visually confirmed that the navigation structure and site content looked identical before and after the fix, verifying that marking the file as orphan did not break or remove anything from the published documentation.

---

## Implementation Notes

### Week [2] Progress

I set up my local development environment on windows using wsl2, ubuntu and miniconda. The biggest challenge I ran ran into was authenticating Git pushs from Ubuntu to GitHub. I resolved this issue by setting up and using an SSH Key.

### Week [3] Progress

I reproduced the issue locally and identified 3 remaining warnings. 

### Code Changes

- **Files modified:** Modified `performant_kernels.rst` file which exists in the repository at docs/source/ 
- **Key commits:**
  https://github.com/brittneylilly/pytorch_ao/commit/d18d039ab42cde7db2c983479f8a4c6fc0ebf20d
  
- **Approach decisions:**
The `performant_kernels.rst` file causing the warning that this PR suppresses, only contains a "TBA" placeholder,
and Git history shows it has not been modified since its creation on April 17, 2024. Therefore, per the Sphinx
documentation, I added the `:orphan:` directive to the file so that it is ignored during the Sphinx build.
---

## Pull Request

**PR Link:** 
https://github.com/pytorch/ao/pull/4515

**PR Description:** 

Fix Sphinx toctree warning for performant_kernels.rst by:
added `:orphan:` directive to `docs/source/performant_kernels.rst` to suppress the Sphinx build warning: document isn't included in any toctree.
Rationale:
The `performant_kernels.rst` file only contains a "TBA" placeholder, and Git history shows it has not been modified since its creation on April 17, 2024.
Resolves 1 doc build warning from issue #3863.

Remaining doc build warnings after this fix (2):
- [autosummary] failed to import torchao.prototype.quantization.Int8DynamicActivationInt4WeightConfig
- image file not readable: eager_tutorials/output.png

**Maintainer Feedback:**
- June 19, 2026: pytorch-bot ran - no issues returned 
- June 19, 2026: Meta CLA bot and checks ran - output: No conflicts with base branch. Changes can be cleanly merged.
- July 7, 2026: Maintainer said to removed a .txt file and the PR would be good to be merged.

**Status:** 
PR merged on 7/20/2026
---

## Learnings & Reflections

### Technical Skills Gained

I learned about sphinx document builder and how it's used to convert raw files into html files for viewing on website

### Challenges Overcome

One challenge I ran into was that GitHub Desktop could not recognize my repository because it lives inside the WSL file system rather than directly on my Windows C drive, so I still need to resolve how to authenticate and push branches from Ubuntu to GitHub.

### What I'd Do Differently Next Time

From the beginning I would have focused on choosing a recently created issue, like one created within the last 2 weeks to 1 month max.

---

## Resources Used

- Sphinx documentation: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html
