# GitHub GraphQL API Repository Reference issue

This repo has been created to describe the bug we're facing with references and
the GitHub GraphQL API. This repository has a branch and a tag using the same
name (``branch-or-tag``) and the GraphQL API reports the same
[Ref](https://developer.github.com/v4/object/ref/) ID.

## How to reproduce the issue

### Querying the Ref ID for the tag

Query:

```graphql
{
  repository(owner: "fsouza", name: "github-ref-bug") {
    ref(qualifiedName: "refs/tags/branch-or-tag") {
      id
      target {
        oid
      }
    }
  }
}
```

Result:


```json
{
  "data": {
    "repository": {
      "ref": {
        "id": "MDM6UmVmMTY5MzExMTIxOmJyYW5jaC1vci10YWc=",
        "target": {
          "oid": "6028c1975be3c9f1f3145363f533769e201f10fe"
        }
      }
    }
  }
}
```

### Querying the Ref ID for the branch

Query:

```graphql
{
  repository(owner: "fsouza", name: "github-ref-bug") {
    ref(qualifiedName: "refs/heads/branch-or-tag") {
      id
      target {
        oid
      }
    }
  }
}
```

Result:

```json
{
  "data": {
    "repository": {
      "ref": {
        "id": "MDM6UmVmMTY5MzExMTIxOmJyYW5jaC1vci10YWc=",
        "target": {
          "oid": "a259720599486889f603739fed41210970c566b9"
        }
      }
    }
  }
}
```

Notice how ``ref.id`` is the same in both results, even though there are
different values for ``ref.target.oid``. If you run a node query for the id
``MDM6UmVmMTY5MzExMTIxOmJyYW5jaC1vci10YWc=``, you're going to get the ``oid``
"a259720599486889f603739fed41210970c566b9" (the one referenced by the branch):

```graphql
{
  node(id: "MDM6UmVmMTY5MzExMTIxOmJyYW5jaC1vci10YWc=") {
    ... on Ref {
      target {
        oid
      }
    }
  }
}
```

Result:

```json
{
  "data": {
    "node": {
      "target": {
        "oid": "a259720599486889f603739fed41210970c566b9"
      }
    }
  }
}
```

## Impacts of the issue

When creating deployments with the GraphQL API (a preview feature, I know :D),
a [Ref ID is
required](https://developer.github.com/v4/input_object/createdeploymentinput/#refid),
but this issue makes it impossible to create a deployment for a tag whenever
the repository has a branch with the same name.
