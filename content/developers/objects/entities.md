---
title: Entities
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

# Usage

## Create an entity

Create an entity from the given name and type without saving it to the database.

```python
create(name: str, type: str, **kwargs)
```

* Params
   * `name`: a string defining the name of the entity
   * `type`: a string defining the type of the entity
   * `kwargs`: a dictionary defining further attributes depending on the type of the entity.
* Returns
If the function call succeeds, the created entity is returned. Otherwise, an `ValueError` exception is raised.

## Save an entity

```python
save(name: str, type: str, tags: List[str] = None, **kwargs)
```

Use `create` function and save entity in the database. Tag the resulting entity with the list of tags if defined. 

* Params
   * `name`: a string defining the name of the entity to create
   * `type`: a string defining the type of the entity to create
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observables.
   * `kwargs`: a dictionary defining further attributes depending on the type of the entity.
* Returns
If the function call succeeds, the saved entity is returned. Otherwise, an `ValueError` exception is raised.