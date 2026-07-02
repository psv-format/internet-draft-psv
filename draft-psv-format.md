---
title: "Pipe Separated Value"
abbrev: "psv"
category: info

docname: draft-psv-format-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
keyword:
- markdown
- json
venue:
github: "psv-format/internet-draft-psv"
latest: "https://psv-format.github.io/internet-draft-psv/draft-psv-format.html"

author:
 -
    fullname: "Brian"
    organization: PSV Format Group
    email: "mofosyne@gmail.com"

normative:

informative:


--- abstract

The Pipe Separated Value (PSV) format is a structured, typed alternative to CSV for representing tabular data. It uses a strict subset of GitHub Flavored Markdown table syntax, so that a PSV document is simultaneously human-readable, diffable in version control, renderable by common Markdown tools, and mechanically convertible to other data formats like JSON or CBOR. It supports data annotations for specifying data types or formats, cell properties for enforcing constraints, and an optional table ID using the Consistent Attribute Syntax. A PSV document may stand on its own as a data file, comparable to how a `.csv` file is used, or be embedded directly within a larger Markdown document. This document defines the specification for the PSV format.

--- middle

# Introduction

The PSV format aims to provide a standardized way to represent tabular data, either as a standalone data file or embedded within a Markdown document. It extends the existing Markdown table syntax with additional features and conventions to enable structured data representation and interoperability with other data formats.

# Motivation: Why Pipes Instead of Commas

CSV is the incumbent plain-text tabular format, but it has three practical weaknesses that motivate PSV:

1. **No agreed dialect.** RFC 4180 documents a common interpretation of CSV, but decades of independently developed tools (spreadsheets, shells, databases) disagree on quoting, escaping, embedded newlines, and even the delimiter itself. The exact grammar of a given `.csv` file is usually discovered by trial and error against whichever tool produced it, rather than read off a specification.
2. **No types.** Every consumer of a CSV file has to determine, out-of-band, whether a column holds a string, an integer, a date, or a UUID. The schema, if it exists at all, lives in a separate file, in documentation, or in someone's memory.
3. **Not legible as text.** Because columns are not aligned, a raw CSV file is hard to read or review directly, and a version-control diff of a wide CSV file collapses a single changed cell into a line of undifferentiated commas.

Markdown's pipe-table syntax already solves the third problem: it is designed to be read as plain text, it stays aligned under hand-editing, and it renders in essentially every code-hosting platform, editor, and static-site generator without extra tooling. It also already has a widely implemented cell-escaping convention (`\|`), defined by the CommonMark/GFM table specification, so PSV reuses that rather than inventing another quoting scheme.

GFM table syntax, however, was designed for prose formatting, not data interchange, and leaves ambiguous things a data format cannot leave ambiguous — for example, whether the leading and trailing `|` on a row are required, which affects how many columns a parser sees. PSV therefore defines a strict, deterministic subset of GFM table syntax: every valid PSV table is a valid, correctly rendered GFM Markdown table, but not every valid GFM table is a valid PSV table. The goal is that a `.psv` file can be used directly as a practical, typed alternative to `.csv` — parsed on its own, without any surrounding prose — while remaining fully readable, diffable, and renderable using nothing but existing Markdown tooling.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# PSV Table Structure

A PSV table consists of the following components:

1. **Table ID (Optional)**: A table can have an optional ID specified using the [Consistent Attribute Syntax](https://talk.commonmark.org/t/consistent-attribute-syntax/272), enclosed in curly braces followed by a hash (`#`) and the desired ID. e.g., `{#tableID}`

2. **Header Row**: The first row of the table, containing the column names. Column headers can include data annotations within square brackets to specify the expected data type or format of values in that column. e.g., `| Name | Age [int] | City [str] |` in addition with column id `{#headerID}`.

3. **Delimiter Row**: A row where each cell consist of at least three dashes (`---`) separating the header row from the data rows. All characters between the dashes are ignored (This is so that `:` being used for left/right alignment in a typical markdown table is not affected).

4. **Data Rows**: The subsequent rows containing the actual data for each column.

5. **Delimiters**: Every row (the header row, the delimiter row, and all data rows) MUST begin and end with a `|`. General GFM tables permit omitting the leading and/or trailing pipe, but PSV requires both on every row so that the number of columns is unambiguous when a PSV document is parsed on its own, without a Markdown renderer to fall back on.

Each row can have `|` in each cell as long as it is escaped via `\|` (escaping rule is based on github tables).

## Standalone and Embedded Documents

A PSV document consists of a single PSV table, optionally surrounded by blank lines, and MAY be saved directly as a file — conventionally with a `.psv` extension — the same way a `.csv` file is used today, but with types and constraints encoded directly in the header row instead of tracked separately.

A PSV table MAY instead be embedded within a larger Markdown document, as in the examples below. Embedded and standalone PSV tables use identical syntax: a PSV table is not a special case carved out of Markdown, it is a Markdown table that happens to also be strict enough to stand alone.

# Features

## Data Annotations

Data annotations within square brackets in the header row specify the expected data type or format of values in that column. The following data annotations are supported:

- **string** or **str**: Represents a string value (default if no annotation is provided).
- **integer** or **int**: Represents an integer value.
- **float**: Represents a floating-point number.
- **bool**: Represents a boolean value. Accepted values: `true`, `false`, `yes`, `no`, `active`, `inactive`.
- **hex**: Represents a hexadecimal string (encoded using CBOR tag `CBOR_TAG_HEXADECIMAL_STRING`).
- **base64**: Represents Base64-encoded binary data (encoded using CBOR tag `CBOR_TAG_BASE64`).
- **datauri**: Represents a data URI (encoded using CBOR tag `CBOR_TAG_BASE64URL`).
- **datetime**: Represents a date-time value (encoded using CBOR tag `CBOR_TAG_STD_DATE_TIME_STRING`).
- **uuid**: Represents a UUID value (encoded using CBOR tag `CBOR_TAG_STD_DATE_TIME_STRING`).

Additional data annotations can be added in the future to support other data types or formats.

### Cell Properties Data Annotation

These properties provide additional metadata or constraints for the values in that column. The following cell properties are supported:

- **unique**: Indicates that the values in this column must be unique across all rows.
- **key**: Marks this column as a key column, which can be used for identifying or grouping rows.
- Custom properties can be defined and used as needed.

# Examples

## Example 1: Basic Table

    {#basic-table}
    | Name    | Age [int] | City [str]    |
    | ---     | ---       | ---           |
    | Alice   | 25        | New York      |
    | Bob     | 32        | San Francisco |
    | Charlie | 19        | London        |

This example demonstrates a basic PSV table with columns for "Name", "Age" (annotated as an integer), and "City" (annotated as a string). The table has an optional ID "basic-table" specified using the Consistent Attribute Syntax.

## Example 2: Table with Properties

    {#employee-table}
    | ID [str] [uuid] [unique] [key]       | Name    | Birthday [date] | Want Candy [bool] | Candy Count [int] |
    | ---                                  | ---     | ---             | ---               | ---               |
    | 0b32a75e-e190-4a71-b0e1-45e0d826584f | Alice   | 1994-05-15      | yes               | 10                |
    | 1e6e3e5a-f6cf-4b30-bc42-5365418604a1 | Bob     | 1989-08-20      | no                |                   |
    | 9a7f548f-0854-4c23-bb39-1aefb48b47ae | Charlie | 1996-12-10      | yes               | 32                |

This example showcases a PSV table with various data annotations and cell properties. The "ID" column is annotated as a string, with the "uuid", "unique", and "key" properties specified. The "Birthday" column uses the "date" annotation, while the "Want Candy" column is annotated as a boolean, and the "Candy Count" column is an integer.

# Security Considerations

TODO: Security considerations for the PSV format.

# IANA Considerations

This document has no IANA actions at this time. A future revision may register a media type (e.g., `text/psv`) to support the standalone `.psv` file usage described in "Standalone and Embedded Documents" above.


--- back

# Acknowledgments
{:numbered="false"}

TODO: Acknowledge contributors and relevant sources.
