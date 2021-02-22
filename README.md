# github-label-management #

[![GitHub Build Status](https://github.com/cisagov/github-label-management/workflows/build/badge.svg)](https://github.com/cisagov/github-label-management/actions)
[![Coverage Status](https://coveralls.io/repos/github/cisagov/github-label-management/badge.svg?branch=develop)](https://coveralls.io/github/cisagov/github-label-management?branch=develop)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/cisagov/github-label-management.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/cisagov/github-label-management/alerts/)
[![Language grade: Python](https://img.shields.io/lgtm/grade/python/g/cisagov/github-label-management.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/cisagov/github-label-management/context:python)
[![Known Vulnerabilities](https://snyk.io/test/github/cisagov/github-label-management/develop/badge.svg)](https://snyk.io/test/github/cisagov/github-label-management)

This is a step-by-step guide for managing existing GitHub labels across all
repositories owned by a user or organization.

## Contributing ##

We welcome contributions!  Please see [`CONTRIBUTING.md`](CONTRIBUTING.md) for
details.

## License ##

This project is in the worldwide [public domain](LICENSE).

This project is in the public domain within the United States, and
copyright and related rights in the work worldwide are waived through
the [CC0 1.0 Universal public domain
dedication](https://creativecommons.org/publicdomain/zero/1.0/).

All contributions to this project will be released under the CC0
dedication. By submitting a pull request, you are agreeing to comply
with this waiver of copyright interest.

# Managing cross-organization labels in existing repositories #


## Prerequisites ##

- User has access to the GitHub organization's repositories
- User can generate a GitHub [personal access token](https://github.com/settings/tokens)

## Initial Setup ##

1. Clone this repository locally and follow the setup instructions in
[[CONTRIBUTING.md]] or run `setup-env`.

    ```console
    git clone https://github.com/cisagov/github-label-management.git
    cd github-label-management
    ./setup_env
    ```

1. Create a local configuration file by running `touch labels-config`
1. Open `labels-config` in your favorite editor and start from this template

    ```sh
    export GITHUB_USER_TOKEN="<YOUR_TOKEN_HERE>"
    export LABELS_TOKEN=${GITHUB_USER_TOKEN}
    export LABELS_USERNAME="<YOUR_USERNAME>"
    export LABELS_ORGNAME="cisagov"
    ```

1. Create a GitHub [personal access token](https://github.com/settings/tokens)
    - Give it a descriptive name like "label management"
    - **Only** select the `repo` scope
    - Copy the generated token into `GITHUB_USER_TOKEN`
1. Save your changes and run `source labels-config`

## Label management operations ##

There are several operations you'll probably want to do, now that you've
set up the environment and have the script ready.

### List all organization repositories ###

```console
python3 list-all-repos.py
```

This will yield a list of all repositories you have access to in the specified
GitHub organization. If you have not specified a token, this will yield a list
of all public repositories.

### Back up all current repository labels ###

This will generate a `.toml` file for each repository in a directory of your
choosing. Each `.toml` file will contain the details of the repository's
labels, including names, colors, and descriptions.

Please note these output files are the same format as the `default-labels.toml`
file, so you can use one as a template if you have a repository set up with
the labels you want organization-wide.

```sh
# Make the directory if it doesn't already exist
# The -p option specifies to create intermediate directories as required
mkdir -p repo-labels-backup

# Use the output of list-all-repos.py to fetch each repository's labels
python3 list-all-repos.py | xargs -I {} labels fetch --owner "$LABELS_ORGNAME" --repo {} --filename repo-labels-backup/{}-labels.toml
```

### Apply default labels across the organization ###

View and edit `default-labels.toml` to specify the default labels and their
attributes such as name, color, and description. This is the same format as
the output `.toml` files from the ["back up" step](#Back-up-all-current-repository-labels),
so if you have an example repository, you can copy straight from that
repository's file.

Please note that the `labels ... sync` operation is **destructive** and will
reduce the set of labels in each repository to _only_ those specified.

The label history of each issue and PR will show the removal of the previous
labels in case you need to restore some.

Once you've set up your `default-labels.toml` file, create a list of all the
repositories in a file so you can edit the list to remove any repositories you
do not want to alter labels for.

```sh
python3 list-all-repos.py > repolist.txt
```

Make any edits to the list to exclude repositories that are special or that
you don't want to sync the labels for whatever reason, and then apply the
labels to the remaining list of repositories.

```sh
cat repolist.txt | xargs -I {} labels --verbose sync --owner "$LABELS_ORGNAME" --repo {} --filename default-labels.toml
```

#### ⚠️ Dangerous operation: Apply labels in one step ####

The following is **not recommended**, but to apply the `sync` without an
interstitial `repolist.txt` file (and live dangerously), you can instead run
the `sync` in one step.

```sh
python3 list-all-repos.py | xargs -I {} labels sync --owner "$LABELS_ORGNAME" --repo {} --filename default-labels.toml
```

