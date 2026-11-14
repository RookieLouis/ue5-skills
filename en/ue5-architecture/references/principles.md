# Architecture Fundamentals: SOLID / SoC / Loose Coupling

Principles improve design decisions; they are not a scoring system requiring more types. **Clarity, correctness, and configurability matter more than the number of patterns.**

## SOLID

| Principle | UE5 guidance | Counterexample |
| --- | --- | --- |
| SRP | Organize each class/module around a clear reason to change; Combat, Inventory, and UI own their rules separately. | Character handles damage formulas, saves, UI creation, and item stacking. |
| OCP | Provide extension points through DataAssets, composition, interfaces, or controlled inheritance when real variants exist. | Build complex base classes in advance for dozens of hypothetical enemies. |
| LSP | Subclasses honor the base contract; callers should not need special-case subtype checks to be safe. | An override silently rejects previously valid operations. |
| ISP | An interface describes one clear capability, such as Interactable. | `IEverything` contains pickup, damage, trading, and quest methods. |
| DIP | Across independent systems, prefer small stable contracts; direct dependencies under clear ownership are acceptable. | Inventory depends on a specific Widget Blueprint type. |

## Separation of Concerns

- Separate Gameplay, presentation, configuration, and persistence by responsibility and reason for change, not simply by file length.
- CombatComponent owns combat state/rules; Widget presents them; DataAssets/DataTables provide configuration; the Save System persists data.
- Character/Pawn can coordinate and hold references to its own components; coordination does not automatically make it a God Object.
- Every cross-system data flow has an explicit authoritative state owner. Avoid UI or duplicate Actors maintaining another “true inventory.”

## Loose Coupling

- Reduce unnecessary compile-time and runtime object dependencies and dependencies on concrete Blueprint classes.
- An interface is not always necessary: an Owner directly calling its own Component is natural, clear, and easy to debug.
- Consider event unbinding, destruction, hard/soft asset references, Build.cs dependencies, and Subsystem lifetimes.
- Loose coupling does not mean zero dependencies. Excessive global service location, string messages, and universal event buses reduce traceability.

## Priorities and Conflicts

1. Gameplay correctness and data consistency.
2. UObject lifetime and safety, Editor/Blueprint asset compatibility.
3. Readability, verifiability, and clear responsibilities and dependencies.
4. Designer convenience, performance, and extension cost.
5. Anticipated future extension; speculation must not override the first four.

An architecture review must explain the concrete existing issue, why change is needed, why a minimal solution is insufficient, and the maintenance cost of the abstraction.
