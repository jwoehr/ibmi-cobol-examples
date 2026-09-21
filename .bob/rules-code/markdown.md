# Bob's Coding Rules and Standards

## Markdown Formatting Rules

### MD022/blanks-around-headings

When generating or editing markdown files, always observe MD022/blanks-around-headings:

- Add a blank line before each heading (except at the start of the file)
- Add a blank line after each heading
- This improves readability and follows markdown best practices

**Example (Correct):**

```markdown
# Main Title

This is content after the heading.

## Section Heading

More content here.
```

**Example (Incorrect):**

```markdown
# Main Title
This is content without blank line.
## Section Heading
More content here.
```

### MD047/single-trailing-newline

When generating or editing markdown files, always observe MD047/single-trailing-newline:

- Files should end with a single newline character
- No multiple trailing newlines
- No missing trailing newline
- This ensures consistent file endings across different editors and systems

**Example (Correct):**

```markdown
# Document Title

Content here.
[single newline at end]
```

**Example (Incorrect):**

```markdown
# Document Title

Content here.[no newline]
```

or

```markdown
# Document Title

Content here.
[multiple newlines]



```

### MD032/blanks-around-lists

When generating or editing markdown files, always observe MD032/blanks-around-lists:

- Add a blank line before each list (ordered or unordered)
- Add a blank line after each list
- This improves readability and ensures proper rendering

**Example (Correct):**

```markdown
Here is some text.

- List item 1
- List item 2
- List item 3

More text after the list.
```

**Example (Incorrect):**

```markdown
Here is some text.
- List item 1
- List item 2
- List item 3
More text after the list.
```

### MD031/blanks-around-fences

When generating or editing markdown files, always observe MD031/blanks-around-fences:

- Add a blank line before each fenced code block
- Add a blank line after each fenced code block
- This improves readability and ensures proper rendering
- Applies to both ``` and ~~~ style fences

**Example (Correct):**

```markdown
Here is some text.

```bash
echo "Hello World"
```

More text after the code block.
```

**Example (Incorrect):**

```markdown
Here is some text.
```bash
echo "Hello World"
```
More text after the code block.
```
```


### MD036/no-emphasis-as-heading

When generating or editing markdown files, always observe MD036/no-emphasis-as-heading:

- Do not use emphasis (bold or italic) as a substitute for proper headings
- Use proper heading syntax (# ## ### etc.) instead of **Bold Text** or *Italic Text* for section titles
- This ensures proper document structure and accessibility
- Screen readers and document parsers rely on proper heading hierarchy

**Example (Correct):**

```markdown
## Section Title

This is the content of the section.

### Subsection Title

More content here.
```

**Example (Incorrect):**

```markdown
**Section Title**

This is the content of the section.

*Subsection Title*

More content here.
```


### MD060/table-column-style

When generating or editing markdown files, always observe MD060/table-column-style:

- Use consistent column alignment across all rows in a table
- Align the separator row (dashes) with the header and data rows
- Pad cells with spaces to maintain visual alignment
- The separator row dashes should always be preceded and followed by 1 space
- This improves readability and makes tables easier to maintain
- **Avoid em-dashes (`—`) in table cells.** Prefer a plain hyphen-minus (`-`) or
  a spaced double-hyphen (` -- `) instead. Em-dashes are 3 UTF-8 bytes but only
  1 display column wide; naive byte-length padding will produce cells that appear
  correct in source but fail the alignment check. If an em-dash is unavoidable,
  padding must be calculated using **display width** (Unicode character count),
  not byte length.

**Example (Correct):**

```markdown
| Command                              | Description                      |
| ------------------------------------ | -------------------------------- |
| `mvn clean`                          | Remove all build artifacts       |
| `mvn compile`                        | Compile source code              |
| `mvn test`                           | Run unit tests                   |
| `mvn package`                        | Create JAR files                 |
| `mvn install`                        | Install to local Maven repository|
```

**Example (Incorrect):**

```markdown
| Command | Description |
|---------|-------------|
| `mvn clean` | Remove all build artifacts |
| `mvn compile` | Compile source code |
| `mvn test` | Run unit tests |
| `mvn package` | Create JAR files |
| `mvn install` | Install to local Maven repository|
```

**Note:** While the incorrect example will render correctly, the aligned version is more maintainable and easier to read in source form.
**Note:** Emphasis is fine for highlighting words within paragraphs, just not as standalone headings.
**Note:** Multi-byte Unicode characters (e.g. em-dash `—`, en-dash `–`, copyright `©`) occupy fewer display columns than bytes. Always pad to **display width**, not byte length, when such characters appear in table cells.
