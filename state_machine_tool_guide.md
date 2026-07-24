# State Machine Creator Tool Guide

[← Back to Manual](state_machine_manual.md) | [Wiki Index](wiki_index.md) | [API Reference →](state_machine_api_reference.md)

## 1. Introduction
The **State Machine Creator** is a Unity Editor tool designed to accelerate the development of state-based behaviors. It automates the creation of scripts, assets, and configurations, allowing you to focus on gameplay logic rather than boilerplate code.

**Location**: `Cat Splat State Machine > Tools > State Machine Creator`

---

## 2. Interface Overview

### 2.1. Target System Selection
At the top of the window, the **Target System** dropdown allows you to switch between different state machine implementations in your project (e.g., `Enemy`, `NPC`, `Boss`).
- **Effect**: Changing this updates all tabs to use the types associated with that system (e.g., `EnemyStateConfig` vs `NPCStateConfig`).

### 2.2. Tab 1: State Creator
**Purpose**: Generate C# scripts for new states.

#### Workflow
1.  **State Name**: Enter a descriptive name (e.g., `ChasePlayer`).
2.  **Base Config**: Choose the parent class.
    - Standard states usually inherit from the system's base config (e.g., `EnemyStateConfig`).
    - Specialized states (like Executions) might have their own base classes.
3.  **Variables**: Define the tunable parameters for this state.
    - Click `+` to add a variable.
    - Select Type (Float, Int, Bool, Vector3, GameObject, etc.).
    - Enter Name (e.g., `MoveSpeed`).
    - Set Default Value.
4.  **Generate**: Click to create the scripts.
    - **Output**: Two files are created in the system's Definitions folder:
        - `[Name]Config.cs`: Holds the data.
        - `[Name]State.cs`: Holds the logic (with `OnEnter`, `OnUpdate`, `OnExit` stubs).

### 2.3. Tab 2: Config Creator
**Purpose**: Create ScriptableObject assets from your Config scripts.

#### Workflow
1.  **Config Type**: Select the class you want to instantiate (e.g., `ChasePlayerConfig`).
2.  **Asset Name**: (Optional) Enter a name for the file. Defaults to the class name.
3.  **Preview Inspector**: Adjust default values before creation.
4.  **Create Asset**: Generates the `.asset` file in the project.

### 2.4. Tab 3: Assembler
**Purpose**: Assemble a complete Definition by assigning Config assets.

#### Workflow
1.  **Target Definition**: Select the Definition asset you want to edit (e.g., `GoblinArcherDef`).
    - *Note*: You can create a new Definition by right-clicking in the Project window -> `Create > [System] > Definition`.
2.  **Inspector View**: The tool displays the custom inspector for the selected Definition.
    - **States List**: Add the Config assets you created earlier.
    - **Initial State**: Select the starting state from the list.
    - **Custom Properties**: Any system-specific settings (like "Global Behaviors" for Enemies) will appear here.

### 2.5. Tab 4: Config Editor
**Purpose**: Edit existing Config assets without searching the Project window.

#### Workflow
1.  **Select Config**: Drag and drop a Config asset into the field.
2.  **Edit**: Modify variables and transitions directly in the tool window.

---

## 3. Tips & Tricks
- **Hot Reload**: The tool supports hot reloading. If you generate a new script, wait for Unity to compile, and the tool will automatically refresh the type lists.
- **Error Handling**: If a tab is disabled, it usually means the selected System is missing a required base class (Config or Definition). Check the Console for details.
- **Customization**: The tool's code generation templates can be modified in `StateCodeGenerator.cs` if you need to change the default script structure.
