# State Machine Library API Reference

[← Back to Tool Guide](state_machine_tool_guide.md) | [Wiki Index](wiki_index.md)

## Core Classes

### `StateMachine<TContext>`
**Namespace**: `StateMachine`
**Inherits**: `MonoBehaviour`

The core component that manages states and transitions for an entity.

#### Public Properties
- `IState<TContext> CurrentState { get; }`: The currently active state.
- `IState<TContext> PreviousState { get; }`: The state active before the current one.
- `TContext Context { get; }`: The context data container shared by all states.

#### Public Methods
- `void Initialize(TContext context)`: Initializes the state machine with the given context.
- `T RegisterState<T>(string stateId, T state)`: Registers a state instance with a string ID. Returns the registered state.
  ```csharp
  var idle = RegisterState("idle", new IdleState());
  ```

- `IState<TContext> GetState(string stateId)`: Retrieves a registered state by its ID. **Throws** if not registered.
- `bool TryGetState(string stateId, out IState<TContext> state)`: Safe lookup — returns `false` (no throw) if not registered. Prefer this for optional/fallback lookups.
- `T GetState<T>()`: Retrieves the first registered state of the specified type.
- `Transition<TContext> AddTransition(IState<TContext> from, IState<TContext> to, Func<bool> condition, Action onTransition = null, int priority = 0)`: Adds a transition between two states.
  ```csharp
  AddTransition(idle, run, () => context.velocity > 0.1f);
  ```

- `void SetInitialState(string stateId)`: Sets the starting state by ID.
- `void SetInitialState(IState<TContext> state)`: Sets the starting state by reference.
- `void ChangeState(string stateId)`: Exits current state and enters the new state by ID.
- `void ChangeState(IState<TContext> newState)`: Exits current state and enters the new state by reference.
- `void RevertToPreviousState()`: Changes state back to `PreviousState`.

#### Events
- `event Action<IState<TContext>, IState<TContext>> OnStateChanged`: Fired when a state change occurs (Previous, Current).

---

### `StateConfig<TContext>`
**Namespace**: `StateMachine`
**Inherits**: `ScriptableObject`

Base class for all state configuration assets. Acts as a factory for runtime states.

#### Abstract Methods
- `IState<TContext> CreateState()`: Creates and returns a new instance of the state logic associated with this config.

---

### `StateMachineDefinition<TConfig>`
**Namespace**: `StateMachine`
**Inherits**: `ScriptableObject`

Base class for defining the set of states available to an entity.

#### Public Fields
- `List<TConfig> States`: List of state configurations available to the entity.
- `TConfig InitialState`: The configuration for the starting state.

---

### `Transition<TContext>`
**Namespace**: `StateMachine`

Represents a conditional link between two states.

#### Public Properties
- `IState<TContext> TargetState { get; }`: The state to transition to.
- `int Priority { get; }`: The priority of this transition (higher values checked first).
- `int TransitionCount { get; }`: Number of times this transition has occurred.

#### Public Methods
- `bool ShouldTransition()`: Evaluates the condition. Returns true if the transition should occur.
- `void PerformTransitionAction()`: Executes the optional action associated with the transition.

---

## Interfaces

### `IState<TContext>`
**Namespace**: `StateMachine`

Interface that all states must implement.

#### Methods
- `void Init(StateMachine<TContext> stateMachine, TContext context)`: Called once during registration to inject dependencies.
- `void Enter()`: Called when the state becomes active.
- `void Exit()`: Called when the state becomes inactive.
- `void Update()`: Called every frame while active.
- `void FixedUpdate()`: Called every physics frame while active.

---

## Editor Classes

### `StateMachineDefinitionEditor`
**Namespace**: `StateMachine.Editor`
**Inherits**: `UnityEditor.Editor`

UI Toolkit custom editor for `StateMachineDefinition` assets (`CreateInspectorGUI`): a styled title banner,
live validation, a Configuration card, and an inline States list (each state expands to its config editor).
Styled by `StateMachineInspector.uss`.

#### Protected Methods (Overridable — return a `VisualElement`, or `null` for none)
- `virtual VisualElement CreateCustomContent()`: Feature-specific UI shown between Configuration and the
  States list (e.g. the enemy "Global Behaviors" toggle).
- `virtual VisualElement CreatePostStateListContent()`: Feature-specific UI shown after the States list
  (e.g. the enemy "Executions" list).

---

# Enemy System API Reference

## Core Classes

### `EnemyStateMachine`
**Inherits**: `StateMachine<EnemyContext>`, `IHittable`

The concrete state machine implementation for enemies.

#### Key Responsibilities
- **Initialization**: Sets up `EnemyContext` with components (`Rigidbody2D`, `Animator`, `CombatStatusManager`).
- **State Registration**: Loads states from `EnemyDefinition`.
- **Global Transitions**: Manages transitions to `JuggledState` and `HitState`.
- **Hit Handling**: Implements `IHittable.TakeHit` to process damage, status effects, and trigger Executions.

#### Public Methods
- `void TakeHit(HitInfo hit)`: Processes incoming hits. Applies damage, status effects, and checks for Execution triggers.
- `bool CanBeHit()`: Returns true if the enemy is alive.

---

### `EnemyContext`
**Inherits**: `LightweightReactiveContext<EnemyContext>`

Runtime data container for enemies.

#### Public Properties
- `Rigidbody2D Rigidbody`: Reference to the enemy's Rigidbody.
- `Animator Animator`: Reference to the enemy's Animator.
- `CombatStatusManager CombatStatus`: Manages status effects (Poison, Stun, etc.).
- `bool IsGrounded`: True if the enemy is touching the ground layer.
- `bool ShouldLaunch`: Flag to trigger a launch into the air (for Juggles).
- `bool IsDead`: True if the enemy has 0 health.
- `HitInfo LastHit`: Stores details of the last received hit.

---

## Definitions

### `EnemyDefinition`
**Inherits**: `StateMachineDefinition<EnemyStateConfig>`

Defines the behavior profile for an enemy type.

#### Public Fields
- `bool CanBeJuggled`: If true, the enemy can be launched into the air.
- `List<ExecutionDefinition> Executions`: List of special execution attacks available to this enemy.

---

### `EnemyStateConfig`
**Inherits**: `StateConfig<EnemyContext>`, `IStateConfigWithAnimation`

Base class for all enemy state configurations.

#### Public Fields
- `string StateName`: Unique identifier for the state.
- `List<TransitionDefinition> Transitions`: Data-driven transitions to other states.

---

### `ExecutionDefinition`
**Inherits**: `ScriptableObject`

Defines the conditions and target state for a special execution attack.

#### Public Fields
- `string ExecutionName`: Descriptive name.
- `ExecutionStateConfig ExecutionConfig`: The state to enter when triggered.
- `DamageType RequiredDamageType`: The damage type required to trigger this execution (e.g., `Melee`, `Ranged`).
- `CombatStatusDefinition RequiredStatus`: Optional status required on the enemy.
- `int MinStatusCount`: Required stacks of `RequiredStatus` (e.g. 3 sticky arrows).
- `bool MustBeAirborne`: If true, enemy must be in the air.
- `bool MustBeGrounded`: If true, enemy must be on the ground.
- `bool ConsumeStatusOnTrigger`: If true, clears `RequiredStatus` when the execution fires.

---

## States

### `ConfigurableState<TContext, TConfig, TStateMachine>`
**Namespace**: `StateMachine`
**Inherits**: `StateBase<TContext, TStateMachine>`

Base class for states that are driven by a configuration asset.

#### Protected Fields
- `TConfig config`: The configuration asset for this state.

#### Public Methods
- `ConfigurableState(TConfig config)`: Constructor that initializes the state with the given config.
- `virtual void Enter()`: Automatically plays the state's animation if `TConfig` implements `IStateConfigWithAnimation`.

---

### `IStateConfigWithAnimation`
**Namespace**: `StateMachine`

Interface for state configurations that provide animation details.

#### Properties
- `AnimationClip Animation { get; }`: The default animation clip.
- `string AnimatorStateName { get; }`: The explicit state name in the Animator (optional).
- `string AnimatorLayerName { get; }`: The layer name in the Animator.

---

### `IHasAnimator`
**Namespace**: `StateMachine`

Implemented by a context that exposes an `Animator`, so `ConfigurableState` can play state animations
directly instead of via reflection. Implemented by `PlayerContext`, `EnemyContext`, `NPCContext`.

#### Properties
- `Animator Animator { get; }`

---

## States

### `EnemyState`
**Status**: *Removed* (Refactored to use `ConfigurableState`)

All enemy states now inherit from `ConfigurableState<EnemyContext, TConfig, EnemyStateMachine>`. This provides standard access to the config and automatic animation handling.

---

### Common States

#### `EnemyIdleState`
Standard idle behavior. Plays an idle animation.

#### `EnemyHitState`
Handles hit reactions (flinch/stun).
- **Config**: `EnemyHitConfig` (Duration, Animation).

#### `EnemyDeathState`
Handles death logic. Disables collider and plays death animation.

#### `EnemyJuggledState`
Handles being launched and suspended in the air.
- **Logic**: Applies gravity modification and handles "air juggle" hits to keep the enemy airborne.

---

### Execution States

#### `EnemyBowExecutionState`
Cinematic execution triggered by ranged attacks.
- **Behavior**: Freezes time (Hitstop), plays a specific animation, and applies massive damage.

#### `EnemyDashExecutionState`
Cinematic execution triggered by dash attacks.
- **Behavior**: Dashes through the enemy, applying a "slice" effect.
