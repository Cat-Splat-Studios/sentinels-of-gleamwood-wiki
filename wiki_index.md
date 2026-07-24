# State Machine System Wiki

> **Looking for the game-authoring wiki?** Open [`index.html`](index.html) in a browser — the
> living wiki covers content authoring end to end and summarizes these framework pages on its
> *Framework* page. The markdown files below remain the framework's full source of record.

Welcome to the **Sentinels of Gleamwood** State Machine System Wiki. This documentation covers the architecture, tools, and API for creating complex entity behaviors.

> **Project layout (post-refactor):** the FSM framework lives at `Assets/_Framework/StateMachine/`, gameplay
> under `Assets/_Game/<feature>/{Code,Data}`, and automated tests in `Assets/Tests/PlayMode/`. For the
> current refactor status, architecture summary, and the open backlog, see
> `Documentation/History/REFACTOR_CONTEXT.md` and `History/BACKLOG.md` (archived 2026-07-19;
> open items were extracted into HANDOVER.md §6.A0).

## 📚 Documentation Sections

### 1. [User Manual](state_machine_manual.md)
**Start Here!** A comprehensive guide to the core concepts, architecture, and workflow.
- [Overview & Core Concepts](state_machine_manual.md#1-overview)
- [Getting Started](state_machine_manual.md#3-getting-started)
- [Advanced Usage](state_machine_manual.md#5-advanced-usage)

### 2. [Tool Guide](state_machine_tool_guide.md)
Detailed instructions on using the **State Machine Creator** Unity Editor tool.
- [State Creator](state_machine_tool_guide.md#22-tab-1-state-creator)
- [Config Creator](state_machine_tool_guide.md#23-tab-2-config-creator)
- [Assembler](state_machine_tool_guide.md#24-tab-3-assembler)

### 3. [Code Examples](state_machine_code_examples.md)
**Copy-Paste Ready!** Real-world snippets for common tasks.
- [Classic Workflow](state_machine_code_examples.md#2-classic-workflow-code-only)
- [Data-Driven Workflow](state_machine_code_examples.md#3-data-driven-workflow-recommended)
- [Setup & Context](state_machine_code_examples.md#1-setup--context)

### 4. [API Reference](state_machine_api_reference.md)
Technical documentation for programmers extending the system.
- [Core Classes](state_machine_api_reference.md#core-classes)
- [Enemy System](state_machine_api_reference.md#enemy-system-api-reference)
- [Interfaces](state_machine_api_reference.md#interfaces)

---

## 🚀 Quick Links

- **How do I create a new Enemy?** -> [See Manual: Getting Started](state_machine_manual.md#3-getting-started)
- **How do I add a new Attack?** -> [See Tool Guide: State Creator](state_machine_tool_guide.md#22-tab-1-state-creator)
- **How do I make a Boss?** -> [See Manual: Custom Definition Editors](state_machine_manual.md#51-custom-definition-editors)

---

*This wiki is auto-generated from the project documentation.*
