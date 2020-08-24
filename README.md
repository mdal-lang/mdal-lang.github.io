# A DSL for AL Extension Development

`mdAL` is a [Domain Specific Language (DSL)](https://en.wikipedia.org/wiki/Domain-specific_language) that enables a [Model-Driven](https://en.wikipedia.org/wiki/Model-driven_engineering) approach to extension module development for the ERP System Microsoft Dynamics 365 Business Central (BC). `mdAL` stands for **m**odel-**d**riven **AL**.

!> Please note that the current `mdAL` releases are pre-releases. If you encounter errors or have suggestions please open an issue in the corresponding [repositories](https://github.com/mdal-lang).

## Motivation

While supporting a standard set of processes contained in modules such as finance, purchasing and sales, Microsoft Dynamics 365 BC is often times customized due to specific requirements. Such customizations are not limited to e. g. small configuration changes adapting the user interface to the business needs but, instead, can entail programming of completely new solutions integrating processes into the system not considered by the ERP vendor. It is common for ERP introduction projects that customizing takes a large portion of the budget. Thus, increasing programming efficiency in this area seems beneficial.

When developing an AL extension that adds a whole new solution to Microsoft Dynamics 365 BC there are many reoccurring patterns. For example:

* Every solution contains tables of different categories (e. g. Master, Supplemental, (Posted) Document Header, (Posted) Document Line, Setup).
* The tables in the different table categories have to be designed in a specific way (e. g. Master tables must implement the No. Series logic, all Document Header (Line) fields are carried over to the Posted Document Header (Line)).
* For the tables in each table category specific pages with a standard set of actions have to be provided (e. g. Card Page and List Page for Master tables).
* There is a standard design for posting routines (e. g. Post, Post (Yes/No), Jnl. Check Line, Jnl. Post Line).
* There is a standard set of customizations required to integrate the solution into the BC standard (e. g. Navigate, Source Code Setup, Comment Line Table Name).

Because of these patterns, there is a high portion of code that can be considered as "boilerplate code" because it is either completely identical in every solution or can be derived after investigating the data model. `mdAL` aims at automatically generating this "boilerplate code" from a `mdAL` model file.

## Features

`mdAL` is a descriptive language that defines the needed core entities and views of an AL solution. In addition, the data flow between entities can be specified. `mdAL` provides AL code generation for:

* Tables
  * Setup
  * Master
  * Supplemental
  * (Posted) Document Header and Line
  * Document Comment Line
  * Journal Line
  * Ledger Entry
  * Register
* Enums
  * From specified table fields
* Pages
  * Master
    * Card and List Page
  * Supplemental
    * List Page
  * (Posted) Document
    * Document and List Page
  * Ledger Entry
    * List Page
* Codeunits
  * Post
  * Post (Yes/No)
  * Jnl. Check Line
  * Jnl. Post Line
  * Reg.-Show Ledger

Moreover, these customizations to standard objects are generated:

* Event subscribers implementing the Navigate functionality for the specified Posted Document Header
* `Source Code Setup` table extension
* `Comment Line` table extension
* `Comment Line Table Name` enum extension

Specific code that cannot be generated from a `mdAL` model file can be integrated by subscribing to the various event publishers available in the generated AL code. Hence, you can use `mdAL` to automatically generate a base AL extension and create an additional AL extension that depends on the base extension and adds your specific code. This way you do not have to change generated code in order to do customizations. Take a look at the demo projects [`mdal-lang/mdal-demo`](https://github.com/mdal-lang/mdal-demo) and [`mdal-lang/mdal-demo-extenion`](https://github.com/mdal-lang/mdal-demo-extension) to see how this could be done.

## Quick Start

!> Please note that `mdAL` currently is targeting Business Central 16 and later.

Try `mdAL` with a [demo project](https://github.com/mdal-lang/mdal-demo) specifying a seminar management solution. You can install the `mdAL` [VS Code Extension](#vs-code-extension) for local development or try `mdAL` inside the Gitpod online IDE:

[![Edit with Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/mdal-lang/mdal-demo)

In the [`mdal-lang/mdal-demo-extenion`](https://github.com/mdal-lang/mdal-demo-extension) repository you can find an example on how to further customize the base app defined by the demo project (e. g. in order to complement the posting routines). The customizations are implemented as an additional VS Code extension that uses the base app as dependency.

## Language Elements

### Solution

Using the `solution` keyword a new solution can be defined. You have to specify the name of the solution and the prefix used for all AL objects. Inside of this block the tables ([Master](#master), [Supplemental](#supplemental), [Document Header (Line)](#document), [Ledger Entry](#ledger-entry)) are defined. Supplemental tables are optional.

**Example:**

```mdAL
solution "Seminar Management" {
    Prefix = "SEM";

    // Master

    // Supplemental

    // Document

    // Ledger Entry
}
```

### Table Field

Depending on the table type different types of fields can be used. [Custom Fields](#custom-field) and [Template Fields](#template-field) can be used in all table types. [Include Fields](#include-field) can only be used in [Document Header](#document-header), [Document Line](#document-line) and [Ledger Entry](#ledger-entry) tables.

#### Custom Field

A Custom Field defines a regular table field known from AL. The only difference is that you do not have to specify a field number. In `mdAL` field numbers are generated automatically based on the order of the field definitions. So the field name and type are required to define a field. You can use all of the standard AL types (`Boolean`, `Integer`, `BigInteger`, `Decimal`, `Code`, `Text`, `Date`, `Time`, `DateTime`, `Guid`, `Blob`, `Enum`, `Option`, `Media`, `MediaSet`, `DateFormula`, `Duration`, `RecordId`, `TableFilter`). The only property available in Custom Fields is the `TableRelation` property that can be used to reference AL tables contained in the symbol references. For `Enum` and `Option` fields you can define the members directly in the type definition. Enum objects are then automatically generated from the field definition.

**Example:**

```mdAL
field("Gen. Prod. Posting Group"; Code[20]) {
    // Optionally you can define a table relation to AL tables
    TableRelation = "Gen. Product Posting Group";
}
field("Resource No."; Code[20]) {
    // where and const are also supported
    TableRelation = "Resource" where("Type" = const("Person"));
}
// Option/Enum members are specified directly in the type definition
field("Internal/External"; Option[" ", "Internal", "External"])
field("Source Type"; Enum[" ", "Seminar Registration"])
```

#### Template Field

A Template Field can be used to automatically generate a set of fields and/or standard fields that need additional AL code (e. g. for validation). A field name and the template type have to be specified.

**Example:**

```mdAL
template("Contact Information"; ContactInfo)
template("Dimensions"; Dimensions)
```

The following templates types are available in `mdAL`:

* **`TemplateName`**: Generates the fields `Name`, `Name 2` and `Search Name`.
* **`TemplateDescription`**: Generates the fields `Description`, `Description 2` and `Description Name`.
* **`TemplateDimensions`**: Generates the fields `Global Dimension Code 1` and `Global Dimension Code 2` (for [Master](#master) and [Supplemental](#supplemental) tables) or `Shortcut Dimension Code 1` and `Shortcut Dimension Code 2` fields (otherwise).
* **`TemplateAddress`**: Generates the fields `Address`, `Address 2`, `City`, `Country/Region Code`, `Post Code`, and `County`.
* **`TemplateContactInfo`**: Generates the fields `Contact Person`, `Phone No.`, `Telex No.`, `Fax No.`, `Telex Answer Back`, `E-Mail`, `Home Page`.
* **`TemplateSalesperson`**: Generates the field `Salesperson Code`.
* **`TemplateContact`**: Generates a field with the name of the template field with relation to the `Contact` table.

#### Include Field

A Include Field can be used to specify the data flow between tables. Using this language element you only need to define the type of a field once and can include this field in other tables. A field name and a reference to the original field (`"Table Name"."Field Name"`) have to be specified. If the optional property `Validate` is set, field assignments are validated (e. g. assignments implemented after a Master record is entered in a Document Header table).

**Example:**

```mdAL
// Custom Fields in Master table "Seminar"
field("Duration Days"; Decimal)
field("Seminar Price"; Decimal)

// Include Fields in Document Header table "Seminar Registration"
include("Duration Days"; "Seminar"."Duration Days")
include("Seminar Price"; "Seminar"."Seminar Price") {
    Validate = true;
}
```

### Page Field

A Page Field is used to include a table field on a page. You only have to reference the name of the table field.

**Example:**

```mdAL
field("Duration Days")
```

### Page Field Group

A Page Field Group contains multiple Page Fields to be grouped on Card Pages or Document Pages. You have to specify the group name.

**Example:**

```mdAL
group("General") {
    // Page Field(s)
}
```

### Master

To define a Master table you first have to specify its name and short name. Using the keywords `fields` the table fields are specified. The keywords `cardPage` and `listPage` start a Card Page and List Page definition.

**Example:**

```mdAL
master "Seminar" {
    ShortName = "Sem.";

    fields {
        // Table Field(s)
    }

    cardPage {
        // Page Field Group(s)
    }

    listPage {
        // Page Field(s)
    }
}
```

### Supplemental

To define a Supplemental table you first have to specify its name and short name. Using the keywords `fields` the table fields are specified. The keyword and `listPage` starts a List Page definition.

**Example:**

```mdAL
supplemental "Instructor" {
    ShortName = "Inst.";

    fields {
        // Table Field(s)
    }

    listPage {
        // Page Field(s)
    }
}
```

### Document

To define a document you first have to specify its name and short name. Then the [Document Header](#document-header) and [Document Line](#document-line) tables are define inside of this block.

**Example:**

```mdAL
document "Seminar Registration" {
    ShortName = "Sem. Reg.";

    // Header

    // Line
}
```

#### Document Header

To define a Document Header table you first have to specify its name and short name. Furthermore, the status captions for the Document Header have to be provided. Using the keywords `fields` the table fields are specified. The keywords `documentPage` and `listPage` start a Document Page and List Page definition.

**Example:**

```mdAL
header "Seminar Registration Header" {
    ShortName = "Sem. Reg. Header";
    StatusCaptions = ["Planning", "Registration", "Closed", "Canceled"];

    fields {
        // Table Field(s)
    }

    documentPage {
        // Page Field Group(s)
    }

    listPage {
        // Page Field(s)
    }
}
```

#### Document Line

To define a Document Line table you first have to specify its name and short name. Using the keywords `fields` the table fields are specified. The keyword `listPartPage` starts a List Part Page definition.

**Example:**

```mdAL
line "Seminar Registration Line" {
    ShortName = "Sem. Reg. Line";

    fields {
        // Table Field(s)
    }

    listPartPage {
        // Page Field(s)
    }
}
```

### Ledger Entry

To define a Ledger Entry table you first have to specify its name and short name. Using the keywords `fields` the table fields are specified. The keyword `listPage` starts a List Page definition.

**Example:**

```mdAL
ledgerEntry "Seminar Ledger Entry" {
    ShortName = "Sem. Ledger Entry";

    fields {
        // Table Field(s)
    }

    listPage {
        // Page Field(s)
    }
}
```

## VS Code Extension

From the VS Code Marketplace you can download the `mdAL` [VS Code extension](https://marketplace.visualstudio.com/items?itemName=joneug.mdal) which adds language support for `mdAL`. To use the extension [Java JRE 8](https://www.java.com/de/download/) or newer is required (please restart after installing Java JRE). The extension provides the following features:

* Language Support
  * Syntax highlighting for `mdAL` files.
* Commands (available on `*.mdal` files)
  * Load symbol references: Loads the symbol references found in the `.alpackages` folder.
  * Generate AL Code: Generates AL Code from the currently opened `mdAL` file into the `src-gen` folder.
  * Clean: Deletes the `src-gen` folder.
* Menu items (available on `*.mdal` files)
  * Load symbol references
  * Generate AL Code
* Code Actions
* Content Assist
* Documentation hovers
* Snippets

![Extension Demo](images/extension-demo.gif)

## CLI Tool

If you want to use the `mdAL` generator as a standalone component (e. g. inside of a CI/CD pipeline) you can either download the binaries form the `mdal-lang/mdal` [releases](https://github.com/mdal-lang/mdal/releases) or use the docker image `mdal/cli`. The CLI tool takes the provided `mdAL` model file as an input and generates AL code into the `src-gen` folder.

**Unix:**

```sh
docker run --rm -v $(pwd):/project mdal/cli model.mdal
```

**Windows**

```powershell
docker run --rm -v "$(pwd):C:/project" mdal/cli model.mdal
```

## GitHub Action

If you want to use the `mdAL` generator inside of GitHub Actions workflows you can use the `mdAL` [Action](https://github.com/mdal-lang/mdal-action).

**Example:**

```yaml
uses: mdal-lang/mdal-action@v1
with:
  model-file: 'src/model.mdal'
```
