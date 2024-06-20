# Write Call Operand Mappings

If you want to use operators and functions of a datastore not supported by Kelon, or you want to change the
behaviour of an existing operator, you can modify/overwrite it.

## Where
You can define the directory containing all custom call operands via the `call-operands-dir`. This argument signals
Kelon to load all YAML files following the call-operands naming schema `datastore-type.yml`.

## File Structure
A simple example for a custom call operands file for the `mysql` datastore:
```yaml
call-operands:
  - op: plus    # OPA operator 
    args: 2     # Argument Count
    mapping: "$0 + $1"  # MySQL translation
```

This configuration tells Kelon how to translate the OPA `plus` operand to a MySQL operand.

## Datastore Specific functions
If you want to use datastore specific functions, which are not included in the OPA, you need to set the `Builtin` flag
in the mapping. This will register the function in the OPA and you can use it inside your policies.

**NOTE**: Registered functions are global for all policies. Defining multiple functions with the same name but different
argument count leads to warnings, and only the first found definition will be registered in the OPA.

```yaml
call-operands:
  - op: custom 
    args: 2
    mapping: "CUSTOM($0)"  # MySQL translation
    builtin: true
```
