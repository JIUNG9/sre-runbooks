# Refactoring with moved blocks

```hcl
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

moved {
  from = module.old_module.aws_instance.this
  to   = module.new_module.aws_instance.this
}
```

Benefits:
- No state surgery needed
- Plan shows move, not destroy+create
- Remove moved block after apply
