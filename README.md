# JFrog Artifactory Query Languages (AQL) examples

This repository shows some examples of JFrog's Artifactory Query Language (`aql`) for real scenarios.

JFrog's [Artifactory Query Language](https://jfrog.com/help/r/jfrog-artifactory-documentation/artifactory-query-language) (`aql`) is specially designed to let you uncover any data related to the artifacts and builds stored within Artifactory. Although it's powerful by design, you could find yourself in a situation where setting up your `aql` queries can be hard to understand and write.

You can find these examples rules in the `aql/` folder of this repository.

<br />

>[!TIP]
> By running these `aql` queries you'll get a list of artifacts that meet the criteria configured in your queries.
> These `aql` queries examples are incredible useful for **cleanup** tasks you might need to run after using Artifactory for quite long time.

<br />

## Example #1

```
items.find
(
  {
    "repo": "test-local-repo-1",
    "updated":
    {
      "$before": "6mo"
    },
    "$msp": [
      { "path": { "$match":  "image/*" } },
      { "path": { "$nmatch": "image/latest" } }
    ],
    "$or":
    [
      {
        "stat.downloaded": { "$before": "6mo" }
      }
    ]
  }
).sort
(
  {
    "$asc":
      ["updated"]
  }
)
```

### What this does is:

- Find items:

   1. In repo `local-test-repo-1`
   2. Whose artifacts have not been either _updated_ or _downloaded_ in the last 6 months (`6mo`).
   3. For artifacts whose (by using the `$msp` operator = `match on single property`):
	   * _Path_ in the repository matches (`$match`) the `image/*` pattern.
	   * But does **not** match (`$nmatch`) the `image/stg` nor `image/latest` patterns.
   4. The result list will be sorted in ascending order based on the `updated` field.

<br />

## Example #2

```
items.find
(
  {
    "repo": "local-test-repo-2",
    "updated":
    {
      "$before": "9mo"
    },
    "$msp": [
      { "path": { "$match":  "app/*" } },
      { "name": { "$nmatch": "*develop*" } },
      { "name": { "$nmatch": "*master*" } },
      { "name": { "$nmatch": "*main*" } }
    ],
    "$or": [
      { "stat.downloaded": { "$before": "9mo" } },
      { "stat.downloads":  { "$eq":     null } }
    ]
  }
).sort
(
  {
    "$asc":
      ["updated"]
  }
)
```

### What this does is:

- Find items:

   1. In repo `local-test-repo-2`
   2. Whose artifacts have not been either:
        * _Updated_ or _downloaded_ in the last 6 months (`6mo`).
	    * Or, have never been _downloaded_.
   3. For artifacts whose (by using the `$msp` operator = `match on single property`):
	   * _Path_ in the repository matches (`$match`) the `app/*` pattern.
	   * ***AND*** their _Name_ property does not (`$nmatch`) `*develop*` nor `*master*` nor `*main*`.
   4. The result list will be sorted in ascending order based on the `updated` field.