# Installing Prerequisites

<!--- cSpell:ignore homebrew -->

**Audience**: Application developers, Administrators

## Overview

The production reference guide uses many different command line tools to allow
you interact with the cluster from your local machine. Tools such as `oc`, `git`
and `jq` help you to understand how a cloud native deployment works in a
hands-on manner.

In this topic, we're going to:

* Learn how to install the prerequisite command line tools


=== "macOS"
=== "Linux"
=== "Windows"

Where appropriate, click on the **macOS**, **Linux** or **Windows** horizontal
tab to see the corresponding operating system instructions.

---

## Installing a package manager

A operating system specific package manager is used to install the prerequisite
tools on the different platforms.

=== "macOS"

    On macOS, we use the [**homebrew**](https://brew.sh/){:target="_blank" .external} package
    manager to install the prerequisite tools

    On the terminal command line, issue the following command to see if **homebrew**
    is installed:

    ```shell
    brew --version
    ```

    You should see the following output if homebrew is installed:

    ```{.shell .no-copy}
    Homebrew 3.3.5
    Homebrew/homebrew-core (git revision 63e2388f12d; last commit 2021-11-24)
    Homebrew/homebrew-cask (git revision b8049f243b; last commit 2021-11-24)
    ```

    If it is not installed, use the [Install Homebrew](https://brew.sh/){:target="_blank" .external}
    instructions.

=== "Linux"

    TBD

=== "Windows"

    TBD

---

## Installing `git`

The `git` command line tool is used extensively throughout the tutorial to
interact with git and GitHub. We perform GitOps operations using commands such
as `git push` to make a change to a cluster.

Install `git` using the following command:

=== "macOS"

    ```bash
    brew install git
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD



Learn more about [`git`](https://git-scm.com/about){:target="_blank" .external}.

---

## Installing `node`, `nvm` & `npm`

This trio of technologies is used to install various
[`node.js`](https://nodejs.org/){:target="_blank" .external} based utilities
used in the tutorial.

Install `node` using the following command:

=== "macOS"

    ```bash
    brew install node
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD

It is advisable to use
[nvm](https://github.com/nvm-sh/nvm#intro){:target="_blank" .external}
(node version manager) to allow you to install and use different versions of `node.js`.

Install `nvm` using the following command:

=== "macOS"

    ```bash
    brew install nvm
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD

`npm` is the package manager used to install different node.js packages.

Install `npm` using the following command:

=== "macOS"

    ```bash
    brew install npm
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD


Learn more about [`npm`](https://www.npmjs.com/){:target="_blank" .external}.

---

## Installing `jq`

`jq` is a utility used to manipulate the JSON output typically returned by the
`oc` command when reading or updating Kubernetes resources.

Install `jq` using the following command:

=== "macOS"

    ```bash
    brew install jq
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD

Learn more about [`jq`](https://stedolan.github.io/jq/){:target="_blank" .external}.

---

## Installing `tree`

`tree` is a command-line tool display a file system structure by displaying folders
and files in tree-like structure.

Install `tree` using the following command:

=== "macOS"

    ```bash
    brew install tree
    ```

=== "Linux"

    TBD

=== "Windows"

    TBD

Learn more about [`tree`](http://mama.indstate.edu/users/ice/tree/){:target="_blank" .external}.

---

## Installing `oc`

The `oc` command is used to interact with an OpenShift cluster. It is
effectively a synonym for the `kubectl` command with some additional options.

Use these
[instructions](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html)
to get the latest version of `oc`.

You can use `oc version` to confirm that you have:

  - `Client Version: 4.6` or higher
  - `Server Version: 4.7` or higher

 The version of `oc` must be compatible with the version of the cluster you previously created.

---