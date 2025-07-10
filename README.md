# Overview

A system that allows you to create and modify documents using only natural language instructions with agent AI (**ClaudeCode** or **GeminiCLI**).

[日本語](README_JP.md)

# Quick Start
## For GeminiCLI
1. When you have an idea, write rough instructions in [@WORKS/idea.md](/WORKS/idea.md).
2. Type **mktickets** in the terminal to generate ticket files.
3. If you have a general idea of what you want to write, you may generate ticket files in @WORKS/TICKETS yourself.
4. Type **mkreport** in the terminal to execute article creation. The AI will consume tickets and create documents one after another.
5. Review the documents, and if revisions are needed, embed `<TODO>revision content</TODO>` and move the documents to the @WORKS/FIX directory.
6. Type **fixreport** in the terminal to execute article creation.

## For ClaudeCode
1. When you have an idea, write rough instructions in [@WORKS/idea.md](/WORKS/idea.md).
2. Use the **/mktickets** command to generate ticket files.
3. If you have a general idea of what you want to write, you may generate ticket files in @WORKS/TICKETS yourself.
4. Execute the **/mkreport** command. The AI will consume tickets and create documents one after another.
5. Review the documents, and if revisions are needed, embed `<TODO>revision content</TODO>` and move the documents to the @WORKS/FIX directory.
6. Execute the **/fixreport** command.

## Snippet Configuration
If you can use snippets in VSCode or similar editors, using markdown snippets is convenient.

```json
// markdown.json
{
    "Add TODO tag for AI instructions": {
        "prefix": "todo_ai",
        "body": [
            "<TODO>$1</TODO>"
        ],
        "description": "TODO Tag for AI Agent."
    }
}
```

To use markdown snippets in VSCode, you need to edit `.vscode/settings.json`.

```json
// settings.json
{
    "[markdown]": {
    "editor.quickSuggestions": {
        "comments": "on",
        "strings": "on",
        "other": "on"
    },
    }
}
```

# System Description

## Abstraction
We abstract the tasks that can be expected to be handled by AI agents:

1. **Human Task**: Recall content from ideas (human).
2. **Ticket Task**: Consider what kind of document to create from the recalled content.
3. **Research Task**: Conduct research according to a specified format and generate documents as deliverables.
4. **Human Fix Task**: Review deliverables and issue correction instructions.
5. **Fix Task**: Continuously proofread the completed deliverables to enhance quality.

## Definition

* **Ideas**: idea.md
* **Work Instruction Files**: ticket.md
* **Deliverables**: report.md
## Architecture of Documents Making System
```mermaid
graph TD
    subgraph human
      human_task["Instruct task"]
      human_fix_task["Fix task"]
    end
    subgraph AI Agent
        ticket_task
        research_task
        fix_task
    end
  
    subgraph repositories
        idea[/"idea.md"/]
        subgraph ticket_dir["TICKETS"]
            ticket_0[/"TICKET.md"/]
        end
        subgraph ticket_comp_dir["TICKETS_COMP"]
            ticket_1[/"TICKET.md"/]
        end
        subgraph report_dir["REPORTS"]
            report_1[/"REPORT.md"/]
        end
        subgraph fix_dir["FIX"]
            report_2[/"REPORT.md"/]
        end
    end
    human_task --"make ticket" --> ticket_dir
    human_task --"recollection"--> idea
    human_fix_task --"request for correction"--> fix_dir
    report_dir --"confirm files"--> human_fix_task
    idea --"in"--> ticket_task
    ticket_task --"out"--> ticket_dir
    ticket_dir --"in"--> research_task
    ticket_dir --"move"--> ticket_comp_dir
    research_task --"out"--> report_dir
    fix_task --"out"--> report_dir
    fix_dir --"in"--> fix_task 
```
## Design Points
* Enables asynchronous task execution between AI agents and humans through markdown file-based instruction and deliverable management.
* Human-readable and editable `markdown files` make human intervention easy.
* By separating ticket issuance and execution tasks, human review and intervention are possible.
* Ticket issuance can be performed by either humans or agents.
* Retaining used tickets enhances reusability.
* Retaining used tickets allows for checking and improving prompt accuracy.
* Revision work can be instructed individually within deliverable files, simplifying terminal prompt input.
* Eliminates the need to write prompts to the terminal during execution.

# FAQ

## Can I add a ticket file template?
1. Create your own ticket file template in any location. A sample is available in @WORKS/TICKETS/_temp.
2. Edit [issue_ticket.md](./WORKS/TASKS/issue_ticket.md) and provide a path to the ticket file you created.

## How can I change the content of the ticket file template?
1. Check the third line of [issue_ticket.md](./WORKS/TASKS/issue_ticket.md) to see which file is being referenced.
2. Edit the referenced sample.md.

## How can I add more types of tasks?
1. Create your own task file template in any location. The default location is @WORKS/TASKS.
2. If you are using GeminiCLI, edit [GEMINI.md](GEMINI.md) to set up shortcuts for executing the task files.
3. If you are using ClaudeCode, create a custom slash command to execute the task file in `.claude/commands`.

## How can I adjust the contents of a task?
1. Edit the relevant task file located in @WORKS/TASKS.
2. If you change the task file name and are using GeminiCLI, edit [GEMINI.md](GEMINI.md).
3. If you change the task file name and are using ClaudeCode, edit the custom slash command for the corresponding task in `.claude/commands`.
