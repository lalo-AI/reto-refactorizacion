# CLAUDE.md

Project-specific instructions for Claude Code in the **"Refactorización Asistida por IA"** exercise.

## 1. Project purpose

Python console application for **inventory and sales management** of a small store
("La Esquina"): product registration, sales with discounts and VAT (IVA), quotes,
low-stock alerts, best-seller reports, and JSON persistence.

The program **works correctly** and all tests pass. The goal is to improve code
quality **without changing behavior**.

## 2. Project structure

```
.
├── src/                  # Application source code (refactoring target)
│   ├── gestor.py         # Product and sales logic
│   ├── almacen.py        # JSON load/save
│   ├── reportes.py       # Reports and indicators
│   └── main.py           # Interactive console menu
├── tests/                # Existing pytest suite (black-box) — READ-ONLY
├── pyproject.toml        # Ruff + pytest configuration — READ-ONLY
├── requirements.txt      # pytest, ruff
├── datos_ejemplo.json    # Sample data for the interactive menu
└── BITACORA_TEMPLATE.md  # Template for the prompt log (copy to BITACORA.md)
```

## 3. Environment

- Python **3.10 or newer** (Ruff `target-version = "py310"`).
- Setup:
  ```bash
  python -m venv .venv
  source .venv/bin/activate
  pip install -r requirements.txt
  ```
- Run the app (optional, manual check): `cd src && python main.py`

## 4. Coding conventions

- Follow **PEP 8** (line length 88, as configured in `pyproject.toml`).
- Use `snake_case` for functions/variables, `PascalCase` for classes,
  `UPPER_CASE` for constants. Exception: `agregarProducto` and `buscarProducto`.
- Prefer **descriptive names** over cryptic abbreviations.
- Keep functions **focused and maintainable** (one responsibility; Ruff enforces
  max cyclomatic complexity of 10).
- Add **type hints** where appropriate, using modern 3.10+ syntax
  (`list[str]`, `dict[str, int]`, `X | None`).
- Avoid unnecessary complexity and **duplicated code**.
- Replace magic numbers with named constants.
- Use `with` for file handling.
- Prefer early returns over deeply nested `if` blocks.

## 5. Validation commands

```bash
pytest
ruff check src
```

Both must succeed after every refactoring. (`ruff check src` reports issues at
the start of the exercise; the final target is **0 errors**.)

## 6. Mandatory restrictions

- **NEVER modify files under `tests/`.**
- **NEVER modify `pyproject.toml`** (no disabling rules, no `ignore` additions).
- **Preserve observable program behavior**: same return values, same exceptions,
  same console output, same JSON format.
- **All tests must pass after every refactoring.** A failing test means behavior
  changed — fix the refactoring, never the test.
- **Run Ruff after every refactoring** (`ruff check src`).
- **Preserve the public function names `agregarProducto` and `buscarProducto`**
  (signatures included) because the tests depend on them.
- Perform only **ONE significant refactoring at a time**.
- **Do not automatically fix unrelated issues.** Mention them, but leave them for
  a separate step.
- **Review and validate each refactoring before starting another one.**
- Do not add `# noqa` comments to silence Ruff instead of fixing the underlying smell.
- Do not add new third-party dependencies.

## 7. Refactoring workflow

```
Analyze -> Propose -> Modify one concern -> pytest -> ruff check src -> Review diff -> Continue?
```

1. **Analyze**: read the relevant code and identify one specific code smell.
2. **Propose**: explain the planned change and why, before editing.
3. **Modify one concern**: apply only that refactoring.
4. **Run `pytest`**: all tests must pass.
5. **Run `ruff check src`**: no new issues introduced; ideally fewer.
6. **Review diff** (`git diff`): confirm only the intended change was made.
7. **Continue only if behavior remains intact.** Otherwise revert or fix before
   moving on.

Each validated refactoring should be a single atomic commit on the
`refactorizacion` branch and recorded in `BITACORA.md`.

## 8. Examples

### Acceptable changes

- Splitting a giant function into smaller helpers while keeping its name,
  signature, and return value unchanged.
  ```python
  # Before: one function validates, computes discount, applies VAT and prints
  # After:
  def registrar_venta(...):
      _validar_venta(...)
      subtotal = _calcular_subtotal(...)
      return _aplicar_iva(subtotal)
  ```
- Replacing magic numbers with constants: `total * 1.16` → `total * (1 + TASA_IVA)`
  where `TASA_IVA = 0.16`.
- Extracting duplicated product-lookup logic into one helper used everywhere.
- Renaming a cryptic internal variable (`x`, `tmp2`) to a descriptive name.
- Replacing `f = open(...); ...; f.close()` with `with open(...) as f:`.
- Flattening nested `if` statements using guard clauses.
- Removing dead code that is truly unreachable or unused (verified with search).
- Adding type hints to an existing function.

### Unacceptable changes

- Editing a test or `conftest.py` to make it pass.
- Adding `ignore = [...]` or lowering `max-complexity` in `pyproject.toml`.
- Renaming `agregarProducto` → `agregar_producto` (breaks the tests).
- Changing a printed message, rounding, error type, or JSON key names.
- Combining several refactorings in one step (e.g. split function + rename
  variables + add type hints + reorganize modules all at once).
- "While I'm here" fixes to unrelated code outside the agreed concern.
- Fixing an apparent bug that changes observable behavior without explicit
  approval from the user.
- Silencing Ruff with `# noqa` instead of refactoring.
