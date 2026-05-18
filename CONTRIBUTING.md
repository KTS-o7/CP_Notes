# Contributing

Thanks for improving these notes. The goal is to make the repository useful for beginners preparing for competitive programming and SDE interviews.

## What Makes a Good Note

Each topic note should include:

1. A clear title and table of contents.
2. Short explanations before formulas or code.
3. Small examples with expected output or final answer.
4. Interview-style questions for revision.
5. Practice exercises for beginners.
6. References when the topic depends on a standard, book, or external documentation.

## Style Guidelines

- Prefer simple language over dense textbook phrasing.
- Define abbreviations before using them repeatedly.
- Use fenced code blocks with a language tag, such as `cpp`, `sql`, `c`, or `bash`.
- Keep diagrams readable in plain markdown.
- Avoid dumping code without explaining the idea first.
- Use relative links for files inside this repository.

## Pull Request Checklist

Before opening a PR:

1. Read the rendered markdown locally or on GitHub.
2. Check that every table renders correctly.
3. Check that relative links point to existing files.
4. Remove trailing whitespace.
5. Add practice questions when adding a new topic.
6. Mention whether the PR adds new content, fixes accuracy, or improves formatting.

## Accuracy Expectations

For CS fundamentals, small wording differences can create confusion. If a statement depends on a specific DBMS, operating system, network standard, compiler, or language version, say so clearly.

Examples:

- Say "PostgreSQL implements this as..." instead of implying all databases do.
- Say "on many modern systems..." instead of implying every operating system behaves identically.
- Say "historically" when covering older models such as classful IPv4 addressing.
