# Project instructions for AI assistants (Claude, pi, and others)

Read this file before you change the repository.

## Repository purpose

This repository holds Home Assistant automation blueprints for `michondr`.

## Mandatory rules

1. **Blueprint location.** Put every automation blueprint in
   `blueprints/automation/`. Do not put blueprint YAML files in `blueprints/`
   or in the repository root. Home Assistant ignores these files.
2. **README update.** Every new blueprint needs a new section in `README.md`.
   Every changed blueprint needs an updated section in `README.md`.
   Do the README change in the same commit as the blueprint change.
3. **README section content.** A README section must contain:
   - the blueprint file name and the blueprint `name`,
   - a short summary of the behavior,
   - a table of the gestures, modes, or trigger-to-action mapping,
   - a table of the important inputs with the default values,
   - a complete `use_blueprint` YAML example,
   - the required helpers or entities, if the blueprint needs them.
4. **Example path.** Use `path: <file name>.yaml` in each example. The path is
   relative to `blueprints/automation/`.
5. **Validation.** Parse the YAML before you commit it. The custom tags `!input`
   and `!include` are valid Home Assistant syntax.
6. **Numbering.** Keep the numbered order of the README sections. Add a new
   section at the end and increase the number.
7. **Git.** Use `git mv` to move a file, so the history stays. Do not delete and
   add the same file.

## README template for a new blueprint

````markdown
## <N>. <Short title> (`<file name>.yaml`)

**Summary.** <One paragraph: what the blueprint does and when it runs.>

| <Gesture or mode> | <Result> |
| --- | --- |
| <value> | <result> |

**Useful inputs**

| Input | Default | Purpose |
| --- | --- | --- |
| `<input>` | `<default>` | <purpose> |

**Example**

```yaml
- id: <unique_id>
  alias: "<Area> — <description>"
  mode: single
  use_blueprint:
    path: <file name>.yaml
    input:
      <input>: <value>
```
````

## Style

- Use ASD-STE100 Simplified Technical English.
- Limit procedural sentences to 20 words. Limit descriptive sentences to 25 words.
- Use the active voice. Use the imperative form for instructions.
