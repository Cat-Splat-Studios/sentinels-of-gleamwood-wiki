# State Machine — API reference

> **This is the only state-machine markdown page.** It replaces four that used to sit beside it —
> a manual, a tool guide, a code-examples page and a wiki index. They described a UI that no longer
> exists, an NPC module that was deleted, and a snippet that did not compile; between them they
> never once mentioned `DynamicCondition`, the two condition lists, or the `SetupTransitions`
> contract, which is most of what the data-driven path actually is.
>
> **Concepts, recipes, editor menu paths and the known-bug list live on
> [`framework.html`](framework.html)** — that page has the screenshots and is kept in step with the
> code. This page is the type-and-member reference it points at.
>
> Verified against the code 2026-08-09.

---

## Core

### `StateMachine<TContext>` — `MonoBehaviour`

The component that owns states and transitions for one entity. It has **no abstract members**: a
concrete machine builds its context in `Awake`. `Initialize(context)` is the public wiring path but
has **no game call sites** — both shipped machines assign the protected `context` field directly
(`PlayerStateMachine.cs:96`, `EnemyStateMachine.cs:62`). Tests call it to stand a machine up
without a prefab, so it is not dead code — it is just not how the game gets there.

**Properties** — `CurrentState`, `PreviousState`, `Context`.

**Methods**
- `void Initialize(TContext context)`
- `T RegisterState<T>(string stateId, T state)` — returns the registered state.
- `IState<TContext> GetState(string stateId)` — **throws** if not registered.
- `bool TryGetState(string stateId, out IState<TContext> state)` — no throw. Prefer this for optional and fallback lookups.
- `T GetState<T>()` — first registered state of that type.
- `Transition<TContext> AddTransition(IState from, IState to, Func<bool> condition, Action onTransition = null, int priority = 0)`
- `void SetInitialState(string stateId)` / `(IState<TContext>)`
- `void ChangeState(string stateId)` / `(IState<TContext>)`
- `void RevertToPreviousState()`

**Event** — `Action<IState, IState> OnStateChanged` *(previous, current)*.

> ### `AddTransition` REPLACES same-target edges — it does not stack
> `list.RemoveAll(t => t.TargetState == toState)`. A code-registered edge **deletes the authored
> one** to the same target. Four of `meleeAtk`'s five authored edges in `PlayerDef.asset` died this
> way, and one carries a condition nobody noticed was wrong *because it never ran*.
>
> The replacement itself is correct — `CheckTransitions` takes the first match, so two edges to one
> target would leave the second unreachable anyway. What was wrong was doing it silently; it now
> reports and names both sides. **The fix is always the same: one edge, one condition, ORing the two
> cases together.** Any state whose edges are half data and half code is one refactor from this.

Insertion is a **stable** highest-priority-first insert, deliberately not `List.Sort` — that is an
unstable quicksort, and since `CheckTransitions` takes the first match, re-sorting would turn
authored order into a coin flip.

---

### `IState<TContext>`

- `void Init(StateMachine<TContext> stateMachine, TContext context)` — dependency injection at registration.
- `void Enter()` / `void Exit()` / `void Update()` / `void FixedUpdate()`
- **`void SetupTransitions()`** — called after initialization. **This is the seam the whole
  data-driven path hangs off**, and it is on the interface, not a convention.

**Two-pass contract**: every state is registered *first*, then every state's `SetupTransitions()`
runs. That ordering is what lets an edge name a state that had not been created when its own config
was read.

### `StateBase<TContext, TStateMachine>`

**Two generic parameters.** There is no single-generic `StateBase<TContext>` — a snippet written
against one does not compile, which is exactly what the deleted code-examples page shipped.
`SetupTransitions`, `Enter`, `Exit`, `Update` and `FixedUpdate` are all `virtual` no-ops; `Init` is
non-virtual.

### `ConfigurableState<TContext, TConfig, TStateMachine>` — `: StateBase<TContext, TStateMachine>`

State driven by a config asset. Protected field `config`. Its `Enter()` override plays the state's
animation when `TConfig` implements `IStateConfigWithAnimation`, and it overrides
`SetupTransitions()` to register the edges the config authored.

---

### `StateConfig<TContext>` — `ScriptableObject`

- `abstract IState<TContext> CreateState()` — the factory.
- **`virtual string StateId => name`** — defaults to the asset name; subclasses override it to use a
  dedicated field. `EnemyStateConfig` does exactly that, which is why an enemy state's id is its
  `StateName` and not its filename.

### `StateMachineDefinition<TConfig>` — `ScriptableObject`

`List<TConfig> States` · `TConfig InitialState`.

### `Transition<TContext>`

`TargetState` · `Priority` (higher checked first) · `TransitionCount` ·
`bool ShouldTransition()` · `void PerformTransitionAction()`.

### `TransitionDefinition<TStateConfig, TContext>` — the authored edge

- `TStateConfig ToState`
- **`List<TransitionCondition<TContext>> Conditions`** — typed condition assets.
- **`List<DynamicCondition> DynamicConditions`** — reflection-driven: name a context property, pick
  an operator, give a value.

**Both lists gate the same edge.** This is the pair no previous page mentioned, and it matters when
reading an authored graph: an edge that "has no conditions" may have all of them in the other list.

### `DynamicCondition` — `ScriptableObject`

`PropertyName` (a property on the context) · `Operator` · `FloatValue` (int/float compares) ·
`BoolValue` (Equal/NotEqual).

> Because binding is by **name and type**, a context property that changes type breaks the edge at
> runtime with no compile error. A `bool` read by a DynamicCondition must stay a `bool`.

---

### Animation interfaces

| Interface | Member | Implemented by |
|---|---|---|
| `IHasAnimator` | `Animator Animator { get; }` | `EnemyContext` |
| `IHasAnimationDriver` | `IAnimationDriver AnimationDriver { get; }` | `PlayerContext`, `EnemyContext` |
| `IStateConfigWithAnimation` | `Animation`, `AnimatorStateName`, `AnimatorLayerName` | every `EnemyStateConfig` **and** every `PlayerStateConfig` |

**The player has no `UnityEngine.Animator` to expose.** Its rig is split across two animator layers
(legs and upper body), so it drives animation through `IHasAnimationDriver` instead.

> **`context.Animator` exists on both — and is a different type on each.** On `EnemyContext` it is a
> `UnityEngine.Animator`; on `PlayerContext` it is a `Gleamwood.Player.Animation.PlayerAnimation`
> (`PlayerContext.cs:151`), which is also what its `AnimationDriver` returns. Same member name, same
> apparent shape, no compile error until you use it — so code and docs copied from the enemy side
> read as if they apply to the player when they do not.

---

## Enemy system

### `EnemyStateMachine` — `: StateMachine<EnemyContext>`, `IHittable`

Builds `EnemyContext` from components, registers states from the `EnemyDefinition`, owns the global
edges into `juggled`/`hitstun`, and implements `TakeHit`.

- `void TakeHit(HitInfo hit)` · `bool CanBeHit()`

The order inside `TakeHit` is load-bearing and documented as a numbered pipeline on
[`enemies.html` §04](enemies.html) — including the rule that **spike beats launch beats armor**, and
that death bypasses armor entirely.

### `EnemyContext` — `: LightweightReactiveContext<EnemyContext>`, `IHasAnimator`, `IHasAnimationDriver`

`Rigidbody` · `Animator` · `CombatStatus` · `IsGrounded` · `ShouldLaunch` · `IsDead` · `LastHit`.

### `EnemyDefinition` — `: StateMachineDefinition<EnemyStateConfig>`

- `int MaxHealth = 3` — **`0` means the enemy cannot be killed by damage.** Not a bug and not a
  placeholder; it is how a scripted or invulnerable creature is authored.
- `float ExecutionThreshold = 100f` — per-enemy, so a hound and an imp need different work.
- `bool CanBeJuggled = true`
- `List<ExecutionDefinition> Executions` — **ordered, first-match-wins.** An entry asking for
  strictly less than a later one makes that later one unreachable, silently. Most-specific first,
  status-gated catch-alls last; tests enforce both.

### `EnemyStateConfig` — `: StateConfig<EnemyContext>`, `IStateConfigWithAnimation`

`StateName` (and `StateId => StateName` when set, else the asset name) · `Animation` ·
`AnimatorLayerName = "Base Layer"` · `AnimatorStateName` · `EnterSound` · `EnterSoundVolume` ·
`List<TransitionDefinition> Transitions`.

### `ExecutionDefinition` — `ScriptableObject`

`ExecutionName` · `ExecutionConfig` · `RequiredDamageType` · `MustBeAirborne` · `MustBeGrounded` ·
`RequireSpecificReaction` + `RequiredReaction` · `RequireRecentlySpiked` · `RequiredStatus` +
`MinStatusCount` · `ConsumeStatusOnTrigger`.
`CheckConditions(context, hit)` has an `out string reason` overload, and `DescribeGates()` renders
the entry for tooling — use them rather than re-deriving why an execution did not fire.

### The state roster

`Assets/_Game/Enemies/Code/States/`, each with a matching config in `Definitions/States/`:

| | |
|---|---|
| **Movement** | `EnemyIdleState` · `EnemyChaseState` · `EnemyFloatChaseState` · `EnemyRelocateState` |
| **Offence** | `EnemyAttackPhaseState` (the phase chain) · `EnemyProjectileAttackState` · `EnemySummonState` · `EnemyHazardPhaseState` |
| **Reactions** | `EnemyHitState` · `EnemyJuggledState` · `EnemyDeathState` |
| **Executions** | `EnemyBowExecutionState` · `EnemyDashExecutionState` · `EnemyExplosiveExecutionState` |

`EnemyState` as a base class is **gone** — every enemy state is a
`ConfigurableState<EnemyContext, TConfig, EnemyStateMachine>`.

**`IEnemySuperArmor`** — `bool TryAbsorbHit(in HitInfo hit)`. Implemented by
`EnemyAttackPhaseState` and `EnemySummonState`; `EnemyStateMachine.TakeHit` consults it in the
hitstun fallback branch. Any new state that should shrug off hits implements the same interface.

---

## Editor

### `StateMachineDefinitionEditor` — `UnityEditor.Editor`

UI Toolkit inspector for `StateMachineDefinition` assets (`CreateInspectorGUI`): title banner, live
validation, a Configuration card, and an inline States list where each state expands into its config
editor. Styled by `StateMachineInspector.uss`.

**Override points** — return a `VisualElement`, or `null` for none:
- `virtual VisualElement CreateCustomContent()` — between Configuration and the States list (the
  enemy "Global Behaviors" toggle).
- `virtual VisualElement CreatePostStateListContent()` — after the States list (the enemy
  "Executions" list).

### The creator window

**Code generation only: State Creator + Condition Creator.** The Config Creator, State Config Editor
and Assembler tabs were removed in the 2026-07-20 consolidation — building a machine is the Content
Workbench's States tab now. Generated code lands under the feature's Code root: the config as
`Assets/_Game/<Feature>/Code/Definitions/States/{Name}Config.cs` and the state — the file you edit —
as `Assets/_Game/<Feature>/Code/States/{Name}State.cs`, which stubs `Enter()` only.

Exact menu paths are on [`framework.html` §05](framework.html).
