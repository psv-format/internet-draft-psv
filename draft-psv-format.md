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

The Pipe Separated Value (PSV) format is a structured way to represent tabular data in Markdown, designed to be easily convertible to other data formats like JSON or CBOR. It supports data annotations for specifying data types or formats, cell properties for enforcing constraints, and an optional table ID using the Consistent Attribute Syntax. This document defines the specification for the PSV format.

--- middle

# Introduction

The PSV format aims to provide a standardized way to represent tabular data within Markdown documents. It extends the existing Markdown table syntax with additional features and conventions to enable structured data representation and interoperability with other data formats.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# PSV Table Structure

A PSV table consists of the following components:

1. **Table ID (Optional)**: A table can have an optional ID specified using the [Consistent Attribute Syntax](https://talk.commonmark.org/t/consistent-attribute-syntax/272), enclosed in curly braces followed by a hash (`#`) and the desired ID. e.g., `{#tableID}`

2. **Header Row**: The first row of the table, containing the column names. Column headers can include data annotations within square brackets to specify the expected data type or format of values in that column. e.g., `| Name | Age [int] | City [str] |` in addition with column id `{#headerID}`.

3. **Delimiter Row**: A row where each cell consist of at least three dashes (`---`) separating the header row from the data rows. All characters between the dashes are ignored (This is so that `:` being used for left/right alignment in a typical markdown table is not affected).

4. **Data Rows**: The subsequent rows containing the actual data for each column.

Each row can have `|` in each cell as long as it is escaped via `\|` (escaping rule is based on github tables).

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

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO: Acknowledge contributors and relevant sources.
