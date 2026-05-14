Audit what has changed in the codebase since user docs were last updated, then update documentation accordingly.

## Configuration

Before running this command, customize these settings for your project:

- **`<DOCS_ROOT>`** — The root directory where user-facing docs live (e.g., `docs/user-docs/`, `docs/`, `wiki/`)
- **`<PAGES_DIR>`** — The subdirectory for page documentation (e.g., `<DOCS_ROOT>/pages/`)
- **`<FEATURES_DIR>`** — The subdirectory for reusable feature documentation (e.g., `<DOCS_ROOT>/features/`)
- **`<REFERENCE_PAGE>`** — An existing page file to use as a style/structure template (e.g., `landing-page.md`)
- **`<REFERENCE_FEATURE>`** — An existing feature file to use as a style/structure template (e.g., `column-visibility.md`)

Replace these placeholders throughout the steps below with your actual paths and filenames.

## Writing style

All user docs are written for end-users with non-technical backgrounds. Write in plain, conversational English. Avoid jargon. Keep sentences short. Use numbered steps for procedures, bullet points for lists of options. A user should be able to follow any doc without knowing anything about software development.

## Doc structure

There are two types of files under `<DOCS_ROOT>`:

**Feature files** live in `<FEATURES_DIR>`. Each file documents one discrete feature (e.g., `<REFERENCE_FEATURE>`). Features are shared — if two pages offer the same capability, both page files link to the same feature file. Never duplicate a feature doc.

**Page files** live in `<PAGES_DIR>` (e.g., `<PAGES_DIR>/<REFERENCE_PAGE>`). Each file corresponds to a major page or view in the application. A page file describes what the page is for, how it's organized, and links out to the feature files for each capability available on that page.

## Steps

### 1. Find the last docs commit

Run:
```
git log --oneline -- <DOCS_ROOT> | head -1
```

Save the commit hash. This is the baseline.

### 2. Get code changes since that commit

Run:
```
git diff <baseline-hash>..HEAD -- . ':(exclude)<DOCS_ROOT>'
```

Also get a file-level summary:
```
git diff --name-only <baseline-hash>..HEAD -- . ':(exclude)<DOCS_ROOT>'
```

### 3. Analyze the changes

Read the diff carefully. Look for two categories of change:

**New or changed pages** — Does the code introduce a new major view or page in the app (e.g., a new route, a new top-level component)? If so, a new page file may be needed in docs/user-docs/.

**New or changed features** — New UI interactions, changed behavior, renamed or removed capabilities, or backend changes that affect what a user can see or do.

Ignore internal refactors, test changes, performance work, and infra changes that have no user-visible effect.

For each meaningful change, decide:
- **New page file needed** — a new major page/view exists with no corresponding docs/user-docs/*.md file
- **New feature file needed** — a feature exists that docs/user-docs/features/ doesn't cover
- **File needs updating** — a page or feature changed and existing docs are now inaccurate or incomplete
- **File should be deleted** — a page or feature was removed

### 4. Read the relevant existing docs

Before editing, read the current content of any doc files you plan to modify or that cover adjacent features. Read existing page files (e.g., `<PAGES_DIR>/<REFERENCE_PAGE>`) to understand what's already linked and whether the feature already has a doc.

If a new page shares features with an existing page, link to the existing feature file — do not write a new one.

### 5. Apply changes

**New page file:** Create `<PAGES_DIR>/<page-name>.md`. Follow the structure of `<REFERENCE_PAGE>`: describe the page's purpose and layout, then list its features with links to the corresponding feature files.

**New feature file:** Create `<FEATURES_DIR>/<feature-name>.md`. Follow the style and structure of existing feature files. Write for a non-technical user.

**Updated file:** Edit in place. Keep the plain-language tone.

**Deleted file:** Remove the file and remove its link from any page files that referenced it.

**Page file feature list:** After any changes, make sure each page file's feature list accurately reflects what's available on that page.

### 6. Report what you did

Summarize:
- Which files were added, updated, or deleted
- What code changes drove each doc change
- Any code changes you saw but chose to skip (and why)