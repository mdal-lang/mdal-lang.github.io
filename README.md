# mdAL: A DSL for AL Extension Development

`mdAL`is a Domain Specific Language (DSL) that enables a Model-Driven approach to extension module development for the ERP System Microsoft Dynamics 365 Business Central. `mdAL` stands for **m**odel-**d**riven **AL**.

## Motivation

## Features

`mdAL` is a descriptive language used to define the needed core entities and views of an AL solution. In addition, the data flow between entities can be specified. General features and integrations which have to be provided in an AL solution (e. g. comment line table and pages, source code and navigate integration) and features that can be derived from the entity and view specification are added automatically.

Thus, `mdAL` provides AL code generation for:

* Tables
  * Master
  * Supplemental
  * (Posted) Document Header and Line
  * Document Comment Line
  * Journal Template, Batch and Line
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

Specific code that cannot be generated form a `mdAL` model file can be integrated by subscribing to the various event publishers available in the generated AL code. Hence, you can use `mdAL` to automatically generate a base AL extension and create an additional AL extension that depends on the base extension and adds your specific code. This way you do not have to change generated code in order to do customizations. Take a look at the [demo projects](#demo-projects) to see how this could be done.

## Language Elements

```mdAL
// Test
master "Seminar" {
    ShortName = "Sem.";

    fields {
        template("Description"; Description)
        field("Duration Days"; Decimal)
        field("Minimum Participants"; Integer)
        field("Maximum Participants"; Integer)
        field("Language Code"; Code[10]) {
            TableRelation = "Language";
        }
        field("Seminar Price"; Decimal)
        field("Gen. Prod. Posting Group"; Code[20]) {
            TableRelation = "Gen. Product Posting Group";
        }
        template("Dimensions"; Dimensions)
    }

    cardPage {
        group("General") {
            field("Description")
            field("Duration Days")
            field("Minimum Participants")
            field("Maximum Participants")
            field("Language Code")
        }
        group("Posting Details") {
            field("Gen. Prod. Posting Group")
            field("Seminar Price")
            field("Dimensions")
        }
    }

    listPage {
        field("Description")
        field("Duration Days")
        field("Minimum Participants")
        field("Maximum Participants")
        field("Seminar Price")
        field("Language Code")
    }

    true
}
```

## VS Code Extension

## CLI Tool

## GitHub Action

