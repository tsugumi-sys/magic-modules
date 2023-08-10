# How we use TeamCity

## Contributing to the TeamCity configuration code

Tools needed:

- `brew install openjdk@17`
- `brew install kotlin`

To learn more about using TeamCity's Kotlin DSL see:
- A step by step [tutorial series published by JetBrains on YouTube](https://www.youtube.com/playlist?list=PLQ176FUIyIUaW-RqAJLbSZe59l6r7t8wp)
- (Official documentation for the Kotlin DSL)[https://www.jetbrains.com/help/teamcity/kotlin-dsl.html#Getting+Started+with+Kotlin+DSL]
- (Reference documentation for the Kotlin DSL)[https://teamcity.jetbrains.com/app/dsl-documentation/index.html]

## How testing works in TeamCity

### When do tests run?

We run nightly tests in TeamCity every night of the week. They're scheduled to start in the early morning (in UTC) so that they finish during the morning of those on the west coast of the US.

Currently we test both the main branch and a feature branch used to prepare the next major release. The major release branch's tests run only one night a week, while tests from the main branch are run every other night. The night that the major release branch tests run is staggered: GA on Thursdays, Beta on Fridays.

### Where are tests run in TeamCity?

There are separate projects for the GA and Beta provider in TeamCity. They both run acceptance tests using separate GCP projects, so GA and Beta tests can run in parallel without clashing.

### What builds run in TeamCity?

The projects in TeamCity contain build configurations for each package found in the GA/Beta providers. This means that each night multiple builds are created and are enqueued to be run. Builds that run acceptance tests can run in parallel with each other.

There are also builds that run sweepers ([see HashiCorp documentation here](https://developer.hashicorp.com/terraform/plugin/sdkv2/testing/acceptance-tests/sweepers)) to remove lingering resources from our test GCP projects. Our TeamCity configuration creates depedencies between builds to ensure that sweepers run once _before_ acceptance tests run and again _after_ all tests finish.