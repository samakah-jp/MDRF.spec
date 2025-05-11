## Markdown Diff Review Format (MDRF) v3.0 Specification

### Version

3.0

### 1. Purpose

The Markdown Diff Review Format (MDRF) is a format for recording and sharing code review content—including code changes (diffs), associated metadata (commit information, file paths, file-specific details, etc.), discussions about specific code sections (comment threads), their metadata (status, type, context snippets, etc.), and reply relationships between comments—in a single Markdown file that is both human-readable and machine-processable.

This format is defined as a **subset of Markdown** and requires adherence to specific structures and rules. It is intended for use in documenting the process and results of code reviews.

### 2. Main Features

* **Flexible Grouping:** H2 level allows grouping by units other than commits (e.g., features, folders). Standard types `Version` and `Path` are defined.
* **Strict Metadata Definition via JSON Schema:** The structure of metadata is clearly defined.
* **Markdown Subset Constraints:** Format ambiguity is eliminated by enforcing stricter rules.
* **Standardized Timestamps:** Unified to ISO 8601 format.
* **Clear and Parsable Reply Specification:** The `:reply_to[ID]` notation is limited to a separate line immediately following the header.
* **Recommended Comment ID Naming Convention:** The `<thread_number>.<comment_number>` format is recommended.
* **Context Snippet Reinforcement (Recommended):** Helps identify commented code sections when line numbers shift. Guidance for multi-line snippets is provided.
* **Comment Threads:** Enables tracking of discussion flow. Thread numbers are sequential per file.
* **Thread-Level Metadata:** Reduces overall complexity.
* **Clear Version Management:** The `mdrf_version` explicitly states the format version.
* **Flexible Comment Metadata:** Allows arbitrary strings for `status` and `type` while defining recommended values (warning for non-recommended).
* **Case-Insensitive Keywords:** Keywords in headers (e.g., `Commit`, `Thread`, `Renamed`) are case-insensitive.
* **Clear Whitespace Rules:** At least one space is required between keywords and identifiers in headers.
* **Prohibition of Empty Comments:** Each comment must have content.
* **HTML Headings in Comment Body (Discouraged):** Markdown headings are forbidden, but `<h1>`-`<h5>` tags are permitted (though not recommended).
* **Recommended Encoding and Newlines:** UTF-8 (no BOM) and LF (`\n`) are recommended for compatibility.
* **Readability:** Easy to read and write with standard Markdown and YAML.
* **Tool-Friendliness:** Structured for relatively easy processing by tools.

### 3. Basic Structure

The entire file is structured as a Markdown document with specific rules. Key information is organized hierarchically:

#### 3.1. Entire File (H1 Level)

* The file must begin with an H1 heading in the format `# <Review Title>`. No other text or heading levels should precede it.
* **YAML Front Matter:** Immediately following the H1 heading, a YAML block enclosed in `---` **must** be placed.
    * `mdrf_version` (Required): The value must be `3.0`.
    * Timestamp Format: All timestamps used within this file **must** conform to the **ISO 8601 format** (e.g., `2025-05-01T20:15:30+09:00` or `2025-05-01T11:15:30Z`).
    * Other keys (e.g., `project`, `review_date`, `participants`) are optional but their structure must adhere to the JSON Schema (`FrontMatter`).

#### 3.2. Grouping Unit (H2 Level)

* Each review group is delimited by an H2 heading in the format `## <Group Type>:<Group Name/ID>`.
    * `<Group Type>` is an arbitrary string defined by the file creator (e.g., `Commit`, `Feature`, `Folder`, `Review Unit`). **Keywords are case-insensitive** (e.g., `Commit` and `commit` are the same).
        * **Standard Group Types (Recommended):** Use of `Version` and `Path` is recommended.
        * Custom types (e.g., `Feature`, `Module`) are also permissible.
    * `<Group Name/ID>` is a non-empty string identifying the group.
    * There **must be at least one whitespace character (space, tab, etc., excluding newlines)** between `<Group Type>` and `:`, and between `:` and `<Group Name/ID>`.
* **Group Metadata (Optional):** An optional YAML block (```yaml ... ```) with the key `group_metadata` can be placed immediately after the H2 heading.
    * Its content must adhere to the JSON Schema (`GroupMetadata`).
    * `group_type` (Recommended): It's recommended to write the `<Group Type>` used in the H2 heading in lowercase (e.g., `version`, `path`, `feature`).

#### 3.3. File Unit / Diff (H3 Level)

* Each file section must begin with an H3 heading in one of the following formats:
    * Modified/Added: `### <Path to new file>`
    * Renamed: `### <Path to new file> (Renamed)`
    * Moved: `### <Path to new file> (Moved)`
    * Renamed and Moved: `### <Path to new file> (Moved)` (Treated as Moved)
    * Removed: `### <Path to old file> (Removed)`
    * **Note:** The keywords `(Renamed)`, `(Moved)`, `(Removed)` are case-insensitive. There **must be at least one whitespace character** between the file path and these keywords.
* **File Metadata (Optional):** An optional YAML block (```yaml ... ```) with the key `metadata` can be placed immediately after the H3 heading.
    * Its content must adhere to the JSON Schema (`FileMetadata`). The `file_path` key is required and must match the path in the H3 header.
* The label `**Diff:**` and a `diff` code block are required. **The keyword `Diff` is case-insensitive**. There should be no space between `**` and `Diff`, or between `Diff` and `:`.

#### 3.4. Comment Thread (H4 Level)

* Each comment thread must begin with an H4 heading in the format `#### Thread <Thread Number> [on Line <Line Number>]`.
    * The keyword `Thread` is **case-insensitive**.
    * There **must be at least one whitespace character** between `Thread` and `<Thread Number>`.
    * The `on Line` part is optional. If present, the keywords `on` and `Line` are **case-insensitive**. There **must be at least one whitespace character** between `on` and `Line`, and between `Line` and `<Line Number>`.
    * `<Thread Number>` is an integer starting from 1 **within that H3 level (File Unit / Diff section)**.
    * `<Line Number>` is an integer >= 1.
* **Thread Metadata (Optional):** An optional YAML block (```yaml ... ```) with the key `thread_meta` can be placed immediately after the H4 heading.
    * Its content must adhere to the JSON Schema (`ThreadMetadata`).
    * Recommended values for `status`, `type`, and recommendations for `context_snippet` are as follows:
        * `status`: Overall status of the thread (recommended: `open`, `addressed`, `resolved`, `wontfix`).
        * `type`: Subject of the thread (recommended: `issue`, `suggestion`, `question`, `nitpick`).
        * `context_snippet` (Recommended): It is recommended to include a string containing the code block being discussed (multiple lines, starting from or representing the line specified by `on Line`). **Using YAML's literal block style (`|`) is recommended for multi-line snippets.**
    * Tools are recommended to issue a warning if `status` or `type` values other than the recommended ones are used.

#### 3.5. Individual Comment (H5 Level)

* Each individual comment must begin with an H5 heading in the format `##### [<Comment ID>] <Username> (<Timestamp>)`.
    * `[<Comment ID>]` is optional. If specified, it is recommended to use the format `<Thread Number>.<Comment Number>`. **Here, `<Comment Number>` is sequential starting from 1 within that comment thread.**
    * `(<Timestamp>)` is required and must be in ISO 8601 format.
* **Reply Specifier:** `:reply_to[<Target Comment ID>]` is optional.
    * If specified, it **must** be written on the **next non-empty line immediately following the H5 header**, and **only** this specifier should be on that line.
* **Comment Body:** The comment body is written in standard Markdown after the reply specifier line (or after the H5 header if no reply specifier is present). **The comment body must not be empty.** It must contain some content.
    * **Headings within Body:**
        * Using Markdown headings (`#`, `##`, etc.) within the comment body is **not permitted** as it conflicts with the MDRF structure.
        * If heading-like expressions are desired within the body, using **HTML `<h1>` to `<h5>` tags is permissible but not recommended**. Consider using Markdown bold (`**`) or horizontal rules (`---`) for emphasis or separation instead, for better readability and tool compatibility.
        * **Note:** The interpretation or display of HTML tags other than `<h1>` to `<h5>` depends on the viewer or rendering tool used and is not defined by the MDRF specification. For security reasons, tools are recommended to sanitize HTML tags.

#### 3.6. File Format Details

* **Character Encoding:** It is **strongly recommended** that files be encoded in **UTF-8 (without BOM)**.
* **Newline Characters:** It is **strongly recommended** to use **LF (`\n`)** for newline characters within the file.

### 4. Data Interpretation Parser Implementation Guidelines

This section provides guidelines for developers implementing tools (parsers, linters, data extractors, etc.) that not only display MDRF v3.0 files but also extract, analyze, aggregate, or validate their content as structured data. Strict adherence to these rules will enhance data consistency and reliability.

**Tools intended solely for display purposes may relax these constraints and perform more lenient (best-effort) parsing.**

#### 4.1. Strict Validation of Structure and Format

* **Heading Levels and Order:** Parsers must strictly validate the H1, H2, H3, H4, H5 hierarchical structure and the order defined in the specification. Unexpected order or levels of headings should be treated as errors.
* **Heading Format:** Each level of heading must **structurally match** the format described in the specification (e.g., `## <Type>:<Name/ID>`, `#### Thread <Number> [on Line <Number>]`).
    * Keywords (e.g., `Commit`, `Thread`, `Renamed`, `Moved`, `Removed`, `on`, `Line`, `Diff`) must be matched **case-insensitively**.
    * Parsers must validate that **at least one whitespace character (space, tab, etc., excluding newlines)** exists where specified between keywords and identifiers/numbers/paths.
    * Other format mismatches (e.g., missing colons, missing required parts) should be treated as errors.
* **Required Elements:** The absence of elements specified as "required" (e.g., H1 heading, YAML Front Matter, `mdrf_version`, `**Diff:**` label, `diff` code block) should be treated as an error.

#### 4.2. JSON Schema Validation for Metadata

* The content of each YAML metadata block (Front Matter, `group_metadata`, `metadata`, `thread_meta`) should be strictly validated against its corresponding JSON Schema defined in the specification.
* Schema violations (e.g., missing required properties, incorrect data types, `const` value mismatches) should be treated as errors.
* For schemas where `additionalProperties: true` is specified, unknown keys are permitted, but tools may consider issuing warnings for such keys.

#### 4.3. Timestamp Format Validation

* All timestamps in the file (e.g., `timestamp` in `commit_metadata`, `(<Timestamp>)` in H5 headings) must be validated for conformance to the ISO 8601 format. Invalid formats should be treated as errors.

#### 4.4. ID and Reference Validation

* **Comment ID Uniqueness:** If a `[<Comment ID>]` is specified in an H5 heading, its uniqueness within the containing comment thread should be validated. Duplicates should be treated as errors.
* **`:reply_to` Target Validation:** If `:reply_to[<Target Comment ID>]` is specified, parsers should validate that `<Target Comment ID>` refers to a valid, existing comment ID within the same comment thread. References to non-existent IDs should be treated as errors.

#### 4.5. `:reply_to` Placement Validation

* `:reply_to[<Target Comment ID>]` must be on the "next non-empty line" immediately following the H5 header, and it must be the *only* content on that line.
    * Empty lines between the H5 header and `:reply_to`, or between `:reply_to` and the comment body, are permissible.
    * Any other text on the `:reply_to` line, or finding the specifier within the comment body, should be treated as an error.

#### 4.6. Handling of Recommended Values for `status`/`type`

* If the values for `status` or `type` keys in thread metadata do not match the recommended values (e.g., `open`, `addressed` / `issue`, `suggestion`), parsers are recommended to generate a **Warning** rather than an error. This allows data processing tools to rely on standard values for features while still recognizing user-defined values.

#### 4.7. Error Handling and Reporting

* When errors are detected during validation, parsers should generate specific error messages indicating which part of the file (with line numbers, if possible) violates which rule.
* It is desirable to report as many issues as possible if multiple errors are detected.
* The parser's error handling strategy (e.g., stop on first error, or attempt to continue parsing while logging errors) should be defined, but at a minimum, users must be clearly informed if errors occur.

#### 4.8. Prohibition of Empty Comment Bodies

* Parsers should validate that the comment body following an H5 heading (and optional `:reply_to` line) is not empty. Empty comment bodies should be treated as an error.

#### 4.9. Handling of Headings within Comment Bodies

* Parsers **must not** interpret Markdown headings (`#`, `##`, etc.) appearing within an H5 comment body as structural MDRF headings. These should be treated as plain text (or, optionally, reported as an error).
* Parsers may **optionally interpret** HTML heading tags (`<h1>` to `<h5>`) within comment bodies for structured data or display purposes. This is not a required feature.
* Parsers are not required to interpret or validate HTML tags other than `<h1>` to `<h5>` within comment bodies. For security, sanitizing HTML tags is strongly recommended.

### 5. License

This specification and the MDRF format are provided under the **Unlicense**. This effectively means they are placed in the public domain, and anyone is free to copy, modify, publish, use, compile, sell, or distribute this software, either in source code form or as a compiled binary, for any purpose, commercial or non-commercial, and by any means.

```
This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <https://unlicense.org/>
```

### 6. Acknowledgements

This specification was created through dialogue with Gemini 2.5 Pro.
