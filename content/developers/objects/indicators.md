---
title: Indicators
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

# Usage

## Create an indicator

Create an indicator from the given name, type, pattern and diamond without saving it to the database.

```python
create(name: str, type: str, pattern: str, diamond: DiamondModel, **kwargs)
```

* Params
   * `name`: a string defining the name of the indicator
   * `type`: a string defining the type of the indicator
   * `pattern`: a string defining the pattern of the indicator
   * `diamond` a string defining the diamond model of the indicator
   * `kwargs`: a dictionary defining further attributes depending on the type of the indicator.
* Returns
If the function call succeeds, the created indicator is returned. Otherwise, an `ValueError` exception is raised.

## Save an entity

```python
save(name: str, type: str, pattern: str, diamond: DiamondModel, tags: List[str] = None, **kwargs)
```

Use `create` function and save indicator in the database. Tag the resulting indicator with the list of tags if defined. 

* Params
   * `name`: a string defining the name of the indicator
   * `type`: a string defining the type of the indicator
   * `pattern`: a string defining the pattern of the indicator
   * `diamond` a string defining the diamond model of the indicator
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observables.
   * `kwargs`: a dictionary defining further attributes depending on the type of the entity.
* Returns
If the function call succeeds, the saved indicator is returned. Otherwise, an `ValueError` exception is raised.