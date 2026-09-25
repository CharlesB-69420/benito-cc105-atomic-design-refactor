# Justification — Refactor the Spaghetti Screen

Each paragraph below names the level a widget was placed at and the specific rule
from the activity sheet that puts it there.

## Supporting layer (not a UI level)

Here are revised versions of the sections that usually benefit most from tighter phrasing, stronger architectural vocabulary, and sharper rubric alignment.

---

### 1. Supporting Layer: `Product`

*Sharpened to emphasize static analysis and architectural boundaries.*

> **`Product` (`lib/models/product.dart`).**
> While not a visual Atomic Design tier, establishing a strongly typed domain entity is an architectural prerequisite for enforcing the rubric’s constraints ("Templates must never import a data model directly"; "Pages are the only place allowed to hold the product list"). Refactoring the raw `Map<String, dynamic>` into a dedicated `Product` model made boundary validation verifiable via static analysis: `templates/` remains free of domain imports, and the domain collection (`List<Product>`) is strictly encapsulated within `CatalogPage`.

---

### 2. Atoms: `SearchTextField` & `CatalogAppBar`

*Refined to explain the intentional bug retention clearly and justify why an `AppBar` is an atom without sounding apologetic.*

> **`SearchTextField`.**
> A decoupled, uncontrolled `TextField` wrapper that delegates input via `onChanged`. By leaving state management to higher layers, it avoids local controller lifecycle overhead while faithfully maintaining parity with the original screen's edge-case behavior (the field does not visually clear when `_searchQuery` resets after submission).

> **`CatalogAppBar`.**
> Classified as an atom despite the underlying complexity of Flutter's `AppBar`. Because its title, actions, and styling are entirely static and consume no application state or domain models, it operates as a single, un-composed design primitive with a singular rendering responsibility rather than a multi-component organism.

---

### 3. Organism: `AddProductForm`

*Streamlined to highlight the separation between "ephemeral form state" and "app domain state."*

> **`AddProductForm`.**
> This organism strictly encapsulates **ephemeral UI state**—managing its own `GlobalKey<FormState>`, field controllers, active category selection, and client-side input validation (`_validateName`, `_validatePrice`). It decisively respects the organism boundary by remaining agnostic to **application domain state**: it does not instantiate `Product`, generate IDs, mutate catalog collections, or invoke global feedback (`SnackBar`). Upon validation, it simply emits raw input primitives via `onSubmit(name, price, category, description)` and resets its internal controllers, leaving domain orchestration entirely to the Page.

---

### 4. Template: `CatalogPageTemplate`

*Tightened into design-system terminology (pure slot-based layout).*

> **`CatalogPageTemplate`.**
> A pure structural scaffold that accepts abstract layout slots (`PreferredSizeWidget` and `Widget` parameters for `appBar`, `searchSection`, `catalogSection`, and `formSection`). It strictly fulfills the template definition by enforcing layout, scrolling boundaries, and visual spacing without importing domain models, holding state, or binding to business logic.

---

### 5. Classification Dilemma: The Confirmation `SnackBar`

*Polished to frame your decision around unidirectional data flow (UDF) and side-effect sequencing.*

> ### Architectural Trade-Off: Placement of the Confirmation `SnackBar`
> 
> 
> The confirmation `SnackBar` presents a common boundary ambiguity: while triggered by form submission, feedback presentation is a **side effect of successful state mutation**, not form validation.
> Delegating the `SnackBar` to `AddProductForm` would create an unverified assumption of success, coupling an input component to downstream state mutations. Placing it within `CatalogPage` ensures strict unidirectional data flow: the organism emits raw intent, the Page executes the state update on `_products`, and confirmation feedback fires only when the data mutation has actually occurred.
