# Inventory

Place inventory files here.

## Static Inventory

Create `hosts.yml`:
```yaml
---
all:
  children:
    webservers:
      hosts:
        web1.example.com:
    databases:
      hosts:
        db1.example.com:
```

## Dynamic Inventory

Place executable scripts that output JSON inventory.
