# Terraform Set with StateFunc results in unexpected additional element

This is a repro for an issue in the Terraform SDK that leads a config
that contains a set with single element to result in a create/update
that has multiple elements. This occurs when a set is used with a nested
object schema, and a nested attribute contains a `StateFunc` that modifies
the input.

To reproduce, check out this repo, cd into it, and run the following commands:
```
# ensure the repo is in a clean state
$ git clean -fdx

# install the provider to the local plugins directory
$ make install

# apply the initial state
$ make apply

panic: got unexpected jobs: [{"name":"","q":"foo "},{"name":"ignored","q":"foo"}]
```

This is the plan for the create:
```
  # tftest_dummy.d will be created
  + resource "tftest_dummy" "d" {
      + id = (known after apply)

      + job {
          + name = "ignored"
          + q    = "foo"
        }
    }
```

Despite the plan matching the config (a single job element), the provider calls Create
with multiple elements:
 * The first element is invalid (no name), with the `q` field set to the config
   value (no `StateFunc`).
 * The second element has a valid name, and the `q` field is set to the post-`StateFunc` value.

There should only be a single element in the set.
