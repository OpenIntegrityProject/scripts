# Claude CLI Agent Guidelines for Open Integrity Project

This document contains specific requirements and guidelines for the Claude CLI agent when working with the Open Integrity Project codebase. These guidelines ensure consistent, maintainable, and secure code changes.

## Overview

The Open Integrity Project integrates cryptographic trust mechanisms into Git repositories, establishing verifiable chains of integrity, provenance, and authorship. For a comprehensive overview, see the [README.md](README.md).

## Key Process Requirements for Claude

When working with files and directories:

1. **Always check if files or directories exist before creating them:**
   - Use `test -f` or `[[ -f ]]` to check for file existence
   - Use `test -d` or `[[ -d ]]` to check for directory existence
   - Check `.gitignore` to see if a path is already excluded before suggesting additions
   - Look for existing configuration before creating new files

2. **Run all scripts and tests from the repository root:**
   - Never `cd` into subdirectories to run scripts or tests
   - Use absolute or relative paths from the repository root in examples
   - Maintain consistent working directory references in documentation

3. **Check git tracking status before removing files:**
   - Use `git ls-files <path>` to check if a file is tracked by git
   - Use `git rm <path>` instead of `rm <path>` for tracked files
   - For directories, use `git rm -r <directory>` for tracked directories
   - For untracked files, regular `rm` or `rm -r` can be used

4. **Use git status before making changes:**
   - Run `git status` to check the repository state before modifying files
   - Note any already staged changes or untracked files
   - Check branch status and any pending commits

5. **Update OPEN_TASKS.md when completing tasks:**
   - Mark completed tasks with [x] in the checklist
   - Add progress notes with dates when applicable
   - Update status fields to reflect current state (OPEN, IN PROGRESS, RESOLVED)

## Progressive Trust Terminology

The Open Integrity Project uses specific terminology for each phase of the Progressive Trust model. For the comprehensive terminology guidelines, please refer to:

- [Progressive Trust Terminology Requirements](src/requirements/REQUIREMENTS-Progressive_Trust_Terminology.md)

## Common Commands
- Audit this repository: `./src/audit_inception_commit-POC.sh`
- Audit another repository: `./src/audit_inception_commit-POC.sh -C /path/to/repo`
- Create repository with signed inception commit: `./src/create_inception_commit.sh -r my_new_repo`
- Get a repository's DID: `./src/get_repo_did.sh -C /path/to/repo`

## Development Commands
- Run regression tests (from repository root): `./src/tests/TEST-audit_inception_commit.sh`
- Run regression tests with verbose output: `./src/tests/TEST-audit_inception_commit.sh --verbose`
- Update regression test output reference:
  ```bash
  # Always run tests from the repository root
  # First capture regular output
  ./src/tests/TEST-audit_inception_commit.sh > src/tests/OUTPUT-TEST-audit_inception_commit.txt 2>&1
  
  # Then append verbose output
  ./src/tests/TEST-audit_inception_commit.sh --verbose >> src/tests/OUTPUT-TEST-audit_inception_commit.txt 2>&1
  ```

## Post-Test Process
When tests pass successfully (after modifying scripts, paths, or test expectations), follow these steps:

1. Ensure test output file naming is consistent:
   - Test output files should match the test script name (e.g., TEST-script-name.sh → OUTPUT-TEST-script-name.txt)
   - Avoid redundant or duplicate output files with variant names
   - When renaming/removing tracked files, use git commands (git mv, git rm) rather than filesystem commands

2. For each test script, process it individually:
   ```bash
   # Always run tests from the repository root
   # Run the test and capture both standard and verbose output
   ./src/tests/TEST-script-name.sh > src/tests/OUTPUT-TEST-script-name.txt 2>&1
   ./src/tests/TEST-script-name.sh --verbose >> src/tests/OUTPUT-TEST-script-name.txt 2>&1
   
   # Add and commit both the test script and its output reference together
   git add src/tests/TEST-script-name.sh src/tests/OUTPUT-TEST-script-name.txt
   git commit -S -s -m "Update TEST-script-name.sh and test output reference
   
   - Fix test expectations to match current behavior
   - Update path references to work with new directory structure
   - Capture both regular and verbose test outputs
   
   Tests reflect current functionality accurately."
   ```

2. Repeat for each test script individually, rather than committing them all at once
   - This approach creates a logical connection between test script changes and output
   - Makes it easier to track which changes affected specific test results
   - If tests break in the future, the commit history clearly shows the last working state

## Version Update Process
When preparing a new version release, follow these steps:

1. Update version numbers in affected scripts:
   - Update `VERSION:` header comment
   - Update `Script_Version` constant
   - Add CHANGE LOG entry with details of changes

2. Run regression tests and update reference output:
   ```bash
   # Always run tests from the repository root
   # Verify all tests pass
   ./src/tests/TEST-audit_inception_commit.sh --verbose
   
   # Update reference output with both regular and verbose outputs
   ./src/tests/TEST-audit_inception_commit.sh > src/tests/OUTPUT-TEST-audit_inception_commit.txt 2>&1
   ./src/tests/TEST-audit_inception_commit.sh --verbose >> src/tests/OUTPUT-TEST-audit_inception_commit.txt 2>&1
   ```

3. Update any relevant issue documents:
   - Mark resolved issues as `RESOLVED` with version number
   - Update version references in issues documents
   - Document any architectural decisions or open questions

4. Create separate commits for each file:
   ```
   git add <single-file>
   git commit -S -s -m "Update <file> to version X.Y.Z

   - Bullet point list of specific changes
   - Be thorough but concise
   - Focus on what changed and why

   Broader explanation of the impact and rationale
   behind these changes, if needed."
   ```

5. Repeat for each modified file:
   - Main script (e.g., audit_inception_commit-POC.sh)
   - Test script (e.g., TEST-audit_inception_commit.sh)
   - Issue documents
   - Test output references

6. Push commits and create tags as appropriate:
   ```
   # Push commits to origin
   git push origin main
   
   # For script changes only: Create signed tag with script-specific name
   git tag -s <script-name>-v<version> -m "Release <version> (<date>) of <script-name>"
   # Example: git tag -s audit_inception_commit-POC-v0.1.04 -m "Release 0.1.04 (2025-03-04) of audit_inception_commit-POC.sh"
   
   # Push the tag (for script changes only)
   git push origin <script-name>-v<version>
   ```
   
   Important tagging notes:
   - Create and push script-specific version tags only for changes to scripts (not for documentation-only changes)
   - Documentation-only changes (to requirements, issues, etc.) should be pushed but do not need version tags
   - Each script maintains its own version numbering
   - Always push changes to the upstream repository when complete, regardless of whether a tag is created

## Debugging Strategies

When encountering issues with script behavior:

1. **Add temporary debug output statements**:
   ```zsh
   z_Output debug "Variable value: $variable_name"
   z_Output debug "Exit code from function: $?"
   ```

2. **Debug critical control flow points**:
   - Add debug output before/after key decision points
   - Track values of variables that influence control flow
   - Pay special attention to exit code propagation

3. **Test with --debug flag** before making changes permanent:
   ```
   # Always run scripts from the repository root
   ./src/script_name.sh --debug [other options]
   ```

4. **Remove or comment out debug statements** after resolving issues unless they provide ongoing value for maintenance.

## Implementing Architectural Decisions

When implementing system-wide architectural decisions:

1. **Progressive implementation approach**:
   - First implement in one script as a reference implementation
   - Document the architectural decision in the relevant ISSUES document
   - Mark as "PARTIALLY RESOLVED" or "IN PROGRESS (implemented in X script)"
   - Plan for system-wide standardization

2. **Document implementation details** in both:
   - Script-specific issue document (e.g., ISSUES-script_name.md)
   - System-wide issue document (e.g., ISSUES-Open_Integrity_Scripting_Infrastructure.md)

3. **Update regression tests** to align with new architectural decisions
   - Consider both immediate and long-term impact on test expectations
   - Document reason for changes in test expectations

## Version Control Best Practices

When working with version numbers and release management:

1. **Always update version numbers consistently:**
   - Update script version in header comments (VERSION: line)
   - Update Script_Version constant in the code 
   - Update CHANGE LOG entries with detailed bullet points
   - Use consistent version format (X.Y.ZZ) across all references
   - Update corresponding test script versions when main script versions change

2. **Check repository status before and after operations:**
   - Use `git status` before starting work to understand current state
   - Verify branch is clean or has expected changes before making modifications
   - Confirm changes are as expected after edits with another `git status`
   - Use `git diff <file>` to verify specific changes before staging

## Enhanced Commit Message Guidelines

When creating commits:

1. **Structure commit messages** with:
   - Clear, concise title (50 chars or less) that identifies the change
   - Blank line after title
   - Bullet points listing specific changes (what changed)
   - Blank line after bullets
   - Paragraph explaining rationale (why it changed)

2. **Use bullet points for specific changes**:
   ```
   - Added tracking of phase numbers in Trust_Assessment_Status
   - Improved output with clear warnings for non-critical issues
   - Enhanced documentation explaining exit code behavior
   ```

3. **Sign and sign-off your commits**:
   ```
   git commit -S -s -m "Your message"
   ```
   - `-S` cryptographically signs with your key
   - `-s` adds DCO sign-off line

4. **Handle quoting in commit messages**:
   - Use single quotes for the entire message when it contains double quotes
   ```
   git commit -S -s -m 'Update "Error Handling" section'
   ```
   - Use double quotes for the message when it contains single quotes
   ```
   git commit -S -s -m "Fix issue with user's input"
   ```

5. **Create separate commits for each file** when implementing architectural changes or addressing issues that span multiple files.

6. **Stage files individually, never in batches:**
   - Always use `git add <single-file>` for each file separately
   - Never use `git add` with multiple files or wildcards
   - The only exception is for test scripts and their output files, which should be committed together
   - Example: `git add src/tests/TEST-script-name.sh src/tests/OUTPUT-TEST-script-name.txt`

## Main Scripts

### 🔍 `src/audit_inception_commit-POC.sh`
- **Purpose**: Audit a repository's inception commit
- **Requirements**: [src/requirements/REQUIREMENTS-audit_inception_commit-POC.md](src/requirements/REQUIREMENTS-audit_inception_commit-POC.md)
- **Issues**: [src/issues/ISSUES-audit_inception_commit-POC.md](src/issues/ISSUES-audit_inception_commit-POC.md)
- **Test**: [src/tests/TEST-audit_inception_commit.sh](src/tests/TEST-audit_inception_commit.sh)

### 🏗️ `src/create_inception_commit.sh`
- **Purpose**: Create a repository with a signed inception commit
- **Requirements**: [src/requirements/REQUIREMENTS-create_inception_commit.md](src/requirements/REQUIREMENTS-create_inception_commit.md)
- **Test**: [src/tests/TEST-create_inception_commit.sh](src/tests/TEST-create_inception_commit.sh)

### 🔍 `src/get_repo_did.sh`
- **Purpose**: Get a repository's DID
- **Requirements**: [src/requirements/REQUIREMENTS-get_repo_did.md](src/requirements/REQUIREMENTS-get_repo_did.md)

## Reference Documents

### Requirements
- Core principles: [src/requirements/REQUIREMENTS-Zsh_Core_Scripting_Best_Practices.md](src/requirements/REQUIREMENTS-Zsh_Core_Scripting_Best_Practices.md)
- Script best practices: [src/requirements/REQUIREMENTS-Zsh_Snippet_Script_Best_Practices.md](src/requirements/REQUIREMENTS-Zsh_Snippet_Script_Best_Practices.md)
- Framework scripts: [src/requirements/REQUIREMENTS-Zsh_Framework_Scripting_Best_Practices.md](src/requirements/REQUIREMENTS-Zsh_Framework_Scripting_Best_Practices.md)
- Utility functions: [src/requirements/REQUIREMENTS-z_Utils_Functions.md](src/requirements/REQUIREMENTS-z_Utils_Functions.md)
- Regression tests: [src/requirements/REQUIREMENTS-Regression_Test_Scripts.md](src/requirements/REQUIREMENTS-Regression_Test_Scripts.md)
- Progressive Trust terminology: [src/requirements/REQUIREMENTS-Progressive_Trust_Terminology.md](src/requirements/REQUIREMENTS-Progressive_Trust_Terminology.md)

### Issues
- Infrastructure issues: [src/issues/ISSUES-Open_Integrity_Scripting_Infrastructure.md](src/issues/ISSUES-Open_Integrity_Scripting_Infrastructure.md)
- Core scripting issues: [src/issues/ISSUES-Zsh_Core_Scripting_Best_Practices.md](src/issues/ISSUES-Zsh_Core_Scripting_Best_Practices.md)

## Templates
- Script template: [src/snippet_template.sh](src/snippet_template.sh)
- Framework: `z_min_frame.sh` or `z_frame.sh`

## Work Stream Management Process

The Open Integrity Project uses a structured approach to manage parallel development across multiple branches:

1. **Work Stream Tasks Document**:
   - [WORK_STREAM_TASKS.md](WORK_STREAM_TASKS.md) in the repository root tracks all branches
   - The document in the `main` branch is the single source of truth
   - Each task is tagged with its branch name: `[branch-name]`
   - Unassigned tasks are tagged with `[unassigned]`

2. **Branch-Specific Work**:
   - Each feature branch owns specific sections of the document
   - Tasks are organized by development stages (Requirements, Implementation, Testing, etc.)
   - Individual tasks can be marked with priority levels (High, Medium, Low)
   - Completed tasks are moved to the "Completed in this Branch" section
   - Only update your branch's section, preserving other sections

3. **Status Update Process**:
   - When making significant progress, update the document with:
     - Task status (completed/in-progress)
     - Completion dates for finished items
     - Any new subtasks discovered during implementation
   - Create a small PR just for the status update
   - Keep documentation changes separate from code changes

4. **Branch Creation Process**:
   - When creating a new branch, add your section to WORK_STREAM_TASKS.md
   - Submit a small PR to update the main branch with your new section
   - Follow the existing format with branch name, description, and priority levels
   - Tag all tasks with your branch name: `[your-branch-name]`

5. **Pre-Commit Review Process**:
   - Before staging any files, conduct a thorough review of all changes
   - Use `git diff <file>` to examine each file's changes individually
   - Verify that changes align with requirements and branch goals
   - Check for consistency across all modified files
   - **CRITICAL**: The human author must personally review all changes before staging
     - AI tools like Claude may assist with changes, but cannot perform the final review
     - Only the human author should add their Signed-off-by certification
     - Only the human author should sign commits with their SSH key

6. **Branch Completion**:
   - Final PR from feature branch to main should include updated task status
   - All tasks should be marked complete with dates or moved to future work
   - Include a summary of accomplishments in the PR description

## Guidelines for Claude When Working on Issues

When working on fixing issues or implementing new features:

1. **Understand the full context before making changes:**
   - Read relevant documentation in README.md and requirement documents
   - Check issue descriptions in ISSUES files for background and constraints
   - Look at existing implementation patterns in similar files
   - Verify assumptions with `git status` and appropriate directory listings

2. **Follow a methodical approach:**
   - First assess what files need to be modified and their current state
   - Create a plan before making any changes
   - Make changes one file at a time with individual commits
   - Test changes immediately after making them
   - Document what was done in WORK_STREAM_TASKS.md and relevant ISSUES files

3. **Ensure thorough testing:**
   - Run the specific test for any script you modify
   - Confirm test output matches expectations
   - Update test output files if changes were intended to modify behavior
   - Verify script functionality works as expected with manual testing

4. **Follow the work stream management process:**
   - Update WORK_STREAM_TASKS.md when starting work on a new branch
   - Tag tasks with branch name: `[branch-name]`
   - Keep status updated with small, focused PRs to the main branch
   - Mark completed tasks with dates: `(YYYY-MM-DD)`
   - Only update your branch's section of the document
   
5. **Follow the pre-commit review process:**
   - Before staging any file, thoroughly review all changes
   - Use `git diff <file>` to examine each file individually
   - Verify consistency across all modified files
   - CRITICAL: The human author must personally review all changes before staging
   - Only the human author should add their Signed-off-by certification
   - Only the human author should sign commits with their SSH key

6. **When in doubt, ask for clarification:**
   - Be explicit about what's unclear rather than making assumptions
   - Provide options with pros and cons when multiple approaches exist
   - Reference specific lines or files when asking questions