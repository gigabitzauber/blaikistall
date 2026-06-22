# Blaikistall

AI prompts and agent instructions.

## Home

https://github.com/gigabitzauber/blaikistall

## Acknowledgements

* Ground rules inspired by [Lada Kesseler](https://github.com/lexler).
* Original work by [JanMosigItemis/gtdda](https://github.com/JanMosigItemis/gtdda)

## Ground Rules

* **Blaikistall Ground Rules** (`blaikistall_ground_rules.md`) - Core principles and guidelines for all agents.

## Agents

### Blaikistall GitHub Copilot TDD Agents

Custom GitHub Copilot agents and instructions designed to support Test-Driven Development (TDD) workflows.

#### Overview

This repository contains specialized agent configurations and instructions that extend GitHub Copilot's capabilities for TDD practitioners. The agents provide all necessary actions to follow the TDD cycle, implement things and perform reviews.

#### Agent Configurations

These files must be copied to .github/agents/ to be used by GitHub copilot.

Although the agents can be used on their own, they are designed to work together to provide a full featured TDD experience. The agents are technology agnostic, i. e. you can use them to develop a Java project as well as one based on TypeScript.

Technology specific rules are supposed to go into you copilot-instructions file. This repo provides instruction files for various technologies.

The instruction files reference the ground rules file (`blaikistall_ground_rules.md`). It should be located inside the .blaikistall directory.

* **TDD Developer** (`agents/copilot/blaikistall_agent_copilot_tdd_developer.agent.md`) - Specialized agent for test-driven development workflows
* **Senior Software Engineer** (`agents/copilot/blaikistall_agent_copilot_software_engineer.agent.md`) - Agent for architectural and design decisions
  * This agent can also be used standalone to perform any kind of implementation tasks.
* **Code Review** (`agents/copilot/blaikistall_agent_copilot_code_review.agent.md`) - Agent for code quality and review guidance
  * This agent may be used standalone. Just prompt "review" to perform a full review.

#### Instructions

One of these files must be copied to .github/copilot-instructions.md to be used by the agents. The instructions files reference the ground rules file (`blaikistall_ground_rules.md`). It should be located inside the .blaikistall directory.

* **TypeScript CLI Instructions** (`instructions/copilot/blaikistall_instructions_copilot_ts_cli.md`) - TypeScript-specific CLI development guidelines.
* **Java Spring Boot CLI Instructions** (`instructions/copilot/blaikistall_instructions_copilot_java_sb_cli.md`) - Java-Spring-Boot-specific CLI development guidelines.

#### Usage

Agents make use of [GitHub copilot's handoff feature](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure) and communicate with one another by the help of dynamically created files in the .blaikistall folder.

* **review.md**
  * Contains results of the review agent and can be used to feed prompts into the developer agent.
* **handover_prompt.md**
  * Used by agents to generate prompts for the next agent in line.
* **plan.md**
  * The file holds informations about the app to be created and - most importantly - a section called `Test Scenarios`. This section contains a list of tests that will be the starting point for the TDD cycle.

#### Example

```md
# Human Readable Time

We are going to write a CLI App. The app takes a non-negative integer (seconds) as input and returns the time in a human-readable format (HH:MM:SS)

HH = hours, padded to 2 digits, range: 00 - 99
MM = minutes, padded to 2 digits, range: 00 - 59
SS = seconds, padded to 2 digits, range: 00 - 59

The input must never exceed 359999 (99:59:59)

## Tests

[x] When input is "0" then output is "00:00:00".
[x] When input is "1" then output is "00:00:01".
[x] When input is "12345" then output is "03:25:45".
[x] When input is "123456" then output is "34:17:36".
[x] When input is "359999" then output is "99:59:59".
[x] When input is larger than 359999 then output error and exit with code 1.
[x] When input is a negative number then output error and exit with code 1.
[x] When input is not an integer then output error and exit with code 1.
[x] When input is empty string or pure whitespace then output error and exit with code 1.
[x] When there is no argument on the command line output help and exit with code 0.
[x] When multiple arguments are provided then output error and exit with code 1.
[x] When --help flag is provided then output help and exit with code 0.
[x] When --version flag is provided then output version from package.json and exit with code 0.
```

#### How to

1. Create a 'plan.md' file in `<project_root>/.blaikistall`.
2. Select the TDD agent and prompt "red". This enters the red phase.
3. (Optional) Check contents of handover_prompt.md.
4. Select "Start implementation" to trigger the developer agent.
5. When the developer agent has finished, select 'Commit' and then "Green Phase" to trigger the TDD agent's green phase.
6. (Optional) Check contents of handover_prompt.md.
7. Select "Start implementation" to trigger the developer agent.
8. When the developer agent has finished, select 'Commit' and then "Refactor Phase" to tell the review agent to look for refactorings.
9. Select "Fix next suggestion" or "Fix next error" to perform refactorings. This triggers the developer agent.
10. When satisfied continue with 2. and repeat.
