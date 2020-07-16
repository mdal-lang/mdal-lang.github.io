# mdAL: A DSL for AL Extension Development

`mdAL`is a Domain Specific Language (DSL) that enables a Model-Driven approach to extension module development for the ERP System Microsoft Dynamics 365 Business Central. `mdAL` stands for **m**odel-**d**riven **AL**.

## Motivation

## Features

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

