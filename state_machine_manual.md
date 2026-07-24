# State Machine Library Manual

[← Back to Wiki Index](wiki_index.md) | [Tool Guide →](state_machine_tool_guide.md)

## 1. Overview
The **State Machine Library** provides a robust, data-driven framework for creating complex behaviors in Unity. It separates logic (States) from data (Configs) and definition (Definitions), allowing for modular, reusable, and easily editable systems.

This library is designed to work seamlessly with the **State Machine Creator** tool, enabling a workflow where designers can prototype and assemble behaviors with minimal programmer intervention.

### Key Features
- **Data-Driven Design**: Behaviors are defined in ScriptableObjects, making them easy to edit and version control.
- **Generic Framework**: Works for any entity type (Enemies, NPCs, Bosses, Player).
- **Tool Integration**: Custom editor tools for creating states, configuring variables, and assembling definitions.
- **Performance**: Optimized for runtime efficiency with minimal allocation during state transitions.

---

## 2. Core Concepts

### 2.1. The Context (`TContext`)
The **Context** is the runtime data container for your entity. It holds references to components (Rigidbody, Animator, Transform) and runtime variables (Health, Target). Every state receives this context, allowing it to act upon the entity.

### 2.2. The State Machine (`StateMachine<TContext>`)
The **State Machine** component manages the current state and handles transitions. It is the "brain" of the entity.
- **Responsibilities**:
    - Initializing the Context.
    - Updating the current State.
    - Handling State transitions (Enter/Exit).

### 2.3. The State (`StateBase<TContext>`)
A **State** represents a specific behavior (e.g., `Idle`, `Chase`, `Attack`). It contains the logic for that behavior.
- **Base Classes**:
    - `StateBase<TContext, TStateMachine>`: The raw base class.
    - `ConfigurableState<TContext, TConfig, TStateMachine>`: The standard base class for states driven by a Config asset. Handles automatic animation playback.
- **Lifecycle Methods**:
    - `Enter()`: Called when the state becomes active.
    - `Update()`: Called every frame while active.
    - `Exit()`: Called when transitioning out of the state.

### 2.4. The Config (`StateConfig<TContext>`)
A **Config** is a ScriptableObject that holds the *data* for a state. It acts as a factory for the runtime State instance.
- **Purpose**: Allows designers to tweak values (Speed, Damage, Duration) without touching code.
- **Relationship**: 1 Config Asset -> Creates -> 1 Runtime State Instance.

### 2.5. The Definition (`StateMachineDefinition<TConfig>`)
A **Definition** is a "character sheet" for your entity. It lists all the states that a specific type of entity can use.
- **Example**: A `GoblinArcherDefinition` might list `IdleConfig`, `PatrolConfig`, and `ShootConfig`.

---

## 3. Getting Started

### 3.1. Setting up a New System
To create a new State Machine system (e.g., for an NPC), follow these steps:

1.  **Create the Context**:
    ```csharp
    public class NPCContext
    {
        public Transform transform;
        public NavMeshAgent agent;
        // Add other components...
    }
    ```

2.  **Create the State Machine**:
    ```csharp
    public class NPCStateMachine : StateMachine<NPCContext>
    {
        // Implement abstract methods to initialize context
    }
    ```

3.  **Create the Base Config**:
    ```csharp
    public abstract class NPCStateConfig : StateConfig<NPCContext> { }
    ```

4.  **Create the Definition**:
    ```csharp
    [CreateAssetMenu(menuName = "NPC/Definition")]
    public class NPCDefinition : StateMachineDefinition<NPCStateConfig> { }
    ```

5.  **Create the Custom Editor (Optional but Recommended)**:
    ```csharp
    [CustomEditor(typeof(NPCDefinition))]
    public class NPCDefinitionEditor : StateMachineDefinitionEditor { }
    ```

Once these classes are created, the **State Machine Creator** tool will automatically detect your new system!

---

## 4. Using the State Machine Creator Tool
Go to `Cat Splat State Machine > Tools > State Machine Creator` to open the main interface.

### 4.1. Target System
Select your system (e.g., `NPCStateMachine`) from the dropdown. The tool adapts its interface to the selected context.

### 4.2. State Creator
**Goal**: Generate the C# scripts for a new state.
1.  **State Name**: Enter a name (e.g., `Patrol`).
2.  **Base Config**: Select the base class (usually `NPCStateConfig`).
3.  **Variables**: Add variables (Speed, Radius, etc.).
4.  **Generate**: Creates `PatrolState.cs` and `PatrolConfig.cs`.

### 4.3. Config Creator
**Goal**: Create a ScriptableObject asset for your state.
1.  **Config Type**: Select the config script (e.g., `PatrolConfig`).
2.  **Preview**: Set default values.
3.  **Create Asset**: Saves a `.asset` file (e.g., `GoblinPatrol`).

### 4.4. Assembler
**Goal**: Assign states to an entity definition.
1.  **Target Definition**: Select your Definition asset (e.g., `GoblinArcherDef`).
2.  **States**: Add the Config assets you created.
3.  **Initial State**: Choose which state the entity starts in.

---

## 5. Advanced Usage

### 5.1. Custom Definition Editors
Inherit from `StateMachineDefinitionEditor` to get a standard "States" list and "Initial State" picker. You can override `DrawCustomContent` and `DrawPostStateListContent` to add system-specific UI (like Global Behaviors or specialized lists).

### 5.2. Execution States
For complex interactions (like cinematic attacks), you can create specialized Config types (e.g., `ExecutionStateConfig`) and handle them differently in your State Machine or Definition.

### 5.3. Global Behaviors
You can implement "Global States" that run in parallel or override normal logic (e.g., getting hit, being stunned) by managing them in your Definition Editor and checking for them in your State Machine's update loop.

---

## 6. Implementation Patterns

For full code examples, see the [Code Examples](state_machine_code_examples.md) page.

### 6.1. The Data-Driven Pattern
This is the recommended workflow for most complex entities.
1.  **Define Context**: Creating the data container.
2.  **Generate Scripts**: Using the Editor Tool to make `State` + `Config` pairs.
3.  **Assemble**: Dragging-and-dropping to build the character.

### 6.2. The Classic Pattern
Best for simple, one-off objects like a trap or a door.
1.  **Manual Registration**: `RegisterState("Open", new OpenState());`
2.  **Direct Transitions**: `ChangeState("Closed");`
