# Campus Complaint Resolution Agent

An automated campus complaint intake, analysis, routing, and
notification workflow built in **n8n**. Students submit complaints
through an online form; the workflow analyses each submission, assigns a
category, department, and priority, stores the complaint, and sends
acknowledgement and department notification emails.

## Overview

The workflow is designed to make it easier for students to report campus
issues and help the relevant university teams receive complaints in a
structured format.

### Main capabilities

-   Collects student complaints through an n8n form.
-   Analyses complaint descriptions using keyword-based logic and an
    OpenAI-powered AI Agent.
-   Assigns a complaint category, responsible department, and priority.
-   Generates a unique complaint ID.
-   Saves complaint details in the n8n Data Table named
    `Campus_Complaints`.
-   Sends an acknowledgement email to the student.
-   Routes a department-specific notification to the relevant team
    through Gmail.

## Complaint Form

The form is titled **Campus Complaint Form** and collects these required
fields:

  Field                   Description
  ----------------------- -----------------------------------------------
  Student Name            Name of the student submitting the complaint
  Student Email           Email address for acknowledgement and updates
  Complaint Category      Student-selected complaint category
  Location                Location where the issue occurred
  Complaint Description   Details of the issue
  Date of Complaint       Date the complaint was raised

Available categories:

1.  Hostel complaints
2.  Classroom and maintenance
3.  Internet and Wi-Fi
4.  Cleanliness and hygiene
5.  Administrative complaints

## Workflow

The n8n workflow follows this sequence:

1.  **Student Complaint Form** -- receives the student's submission.
2.  **Analyse Complaint** -- uses JavaScript keyword matching to
    identify a likely category and department, estimate priority,
    generate a unique complaint ID, and set the initial status to
    `Submitted`.
3.  **AI Agent** -- uses the OpenAI Chat Model (`gpt-4o-mini`) to review
    the complaint and return structured category, department, priority,
    and reason fields.
4.  **Complaint Output Parser** -- validates the AI response against the
    expected JSON structure.
5.  **Complaint Record** -- prepares a consistent complaint record,
    using the AI output when available and the initial analysis as a
    fallback.
6.  **Insert row** -- stores the complaint in the `Campus_Complaints`
    n8n Data Table.
7.  **Student Acknowledgement Email** -- sends the student a
    confirmation containing the complaint ID, category, assigned
    department, priority, and status.
8.  **Switch** -- checks the assigned department and routes the record
    to the corresponding email node.
9.  **Department Email** -- sends the complaint details to the relevant
    team.

The student acknowledgement and department-routing branches are
initiated after the complaint is stored. The acknowledgement email is
connected before the Switch node in the exported workflow.

## Department Routing

  Complaint category          Assigned department
  --------------------------- -------------------------
  Hostel complaints           Hostel Administration
  Classroom and maintenance   Maintenance Department
  Internet and Wi-Fi          IT Department
  Cleanliness and hygiene     Housekeeping Department
  Administrative complaints   Administration Office

The Switch node checks the assigned department and selects the matching
Gmail notification node.

## Priority Classification

The JavaScript analysis identifies three priority levels:

-   **High** -- keywords indicating urgency or possible safety concerns,
    such as fire, electrical danger, threats, harassment, injury,
    flooding, or an immediate deadline.
-   **Medium** -- issues suggesting disruption or delay, such as broken
    facilities, leaks, slow internet, pending requests, or repeated
    problems.
-   **Low** -- complaints that do not match the configured high- or
    medium-priority keyword lists.

The AI Agent is also instructed to reserve the phrase
`IMMEDIATE HUMAN ATTENTION REQUIRED:` for genuine safety risks. Routine
Wi-Fi, maintenance, cleanliness, and administrative issues should not be
described as safety risks.

Priority assignment is automated and should be reviewed by a responsible
human, especially for urgent or safety-related reports.

## Complaint ID and Status

Each complaint receives a generated ID in this general format:

``` text
CMP-YYYYMMDD-HHMMSS-XXXX
```

-   The timestamp is generated using UTC date and time.
-   The final four-character section is a randomly generated suffix.
-   The initial complaint status is `Submitted`.

The workflow's exported configuration does not show a separate
resolution or status-update process. The acknowledgement email mentions
a further notification after resolution, but a resolution-triggered
workflow is not included in this export.

## Data Storage

Complaints are inserted into the n8n Data Table named
**Campus_Complaints**.

The configured columns are:

-   `Complaint_ID`
-   `Student_Name`
-   `Student_Email`
-   `Location`
-   `Complaint_Description`
-   `Date_of_Complaint`
-   `Category`
-   `Assigned_Department`
-   `Priority`
-   `Status`

## Email Notifications

### Student acknowledgement

After submission, the student receives an automated confirmation email
with the complaint details and generated complaint ID.

### Department notification

The Switch node routes a notification to one of five department-specific
Gmail nodes:

-   Hostel Administration Email
-   Maintenance Department Email
-   IT Department Email
-   Housekeeping Department Email
-   Administration Office Email

Each department email includes the complaint ID, student details,
location, description, priority, and status.

## Technology Stack

  -----------------------------------------------------------------------
  Technology                          Purpose
  ----------------------------------- -----------------------------------
  n8n                                 Workflow automation and
                                      orchestration

  n8n Form Trigger                    Student complaint submission

  JavaScript Code node                Initial keyword-based analysis,
                                      priority classification, and ID
                                      generation

  OpenAI AI Agent                     AI-assisted complaint
                                      categorisation and prioritisation

  OpenAI `gpt-4o-mini`                Language model used by the AI Agent

  Structured Output Parser            Enforces the expected AI response
                                      structure

  n8n Data Table                      Stores complaint records

  Gmail                               Sends student and department email
                                      notifications
  -----------------------------------------------------------------------

## Setup and Configuration

This project is based on an exported n8n workflow JSON file.

1.  Open your n8n workspace.
2.  Import the workflow JSON into n8n.
3.  Review the imported nodes and their connections.
4.  Configure or reconnect the required OpenAI and Gmail credentials in
    your own n8n workspace.
5.  Confirm that the `Campus_Complaints` Data Table exists and that its
    columns match the workflow configuration.
6.  Review the recipient email addresses in the department Gmail nodes
    and replace them with the correct operational addresses.
7.  Check the form fields, category options, AI prompt, and priority
    rules.
8.  Test the workflow with sample complaints from each category.
9.  Verify that the complaint is stored and that the student and correct
    department receive their emails.
10. Activate the workflow and use the generated form URL to collect
    submissions.

**Important:** Credentials and recipient addresses in an exported
workflow may be workspace-specific. Do not assume that credentials will
work after importing the file into another n8n account. Reconfigure and
test them before using the workflow.

## Testing Checklist

-   [ ] Submit a complaint from each of the five categories.
-   [ ] Confirm that a complaint ID is generated.
-   [ ] Confirm that category and assigned department are appropriate.
-   [ ] Check High, Medium, and Low priority examples.
-   [ ] Confirm that the complaint is inserted into `Campus_Complaints`.
-   [ ] Confirm that the student receives an acknowledgement email.
-   [ ] Confirm that only the intended department receives the routed
    notification.
-   [ ] Test a genuine safety-related complaint and verify that it is
    clearly flagged for human attention.
-   [ ] Review the workflow execution log for failed nodes.

## Limitations and Considerations

-   Category and priority assignment depend on the configured keyword
    rules and AI output; incorrect or ambiguous descriptions may be
    misclassified.
-   Human review is important for safety-related, sensitive, or disputed
    complaints.
-   The exported workflow stores the initial status as `Submitted`; it
    does not include a department-facing resolution dashboard or a
    completed status-update process.
-   Although the acknowledgement email refers to a later resolution
    notification, the exported workflow does not contain a separate
    resolution notification trigger.
-   The workflow sends emails and stores records; it does not itself
    guarantee that a complaint has been resolved.
-   Protect student names, email addresses, and complaint details. Limit
    access to the n8n workspace, Data Table, and email accounts to
    authorised users.

## Project Files

-   `Campus ComplaDint Resolution Agent (2).json` -- exported n8n
    workflow configuration.

## License

No license information is specified in the supplied workflow export. Add
a license if you intend to distribute or publish this project.
