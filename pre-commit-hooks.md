# Pre-commit hooks

Imagine this:
You have to test some code and you got some development credentials that let you
do mostly anything. For convenience you have a little script with the credentials
written on it. It's just for you; you would certainly never submit that to your
code repository, right? Later that sprint you end up issuing a `git add *`, you
commit and push to the repository. Everything is fine until the first angry
reviewer calls and demands an explanation about leaked credentials.

It happens that your script ended up being commited and those credentials are
there for posterity. And revoking the leaked credentials is going to piss a lot
of people off.

Sometimes it is possible to remove that cursed commit from the history, but if
you don't own the repo, it is going to take some time and anybody can browse the
history and see (and probably use) the leaked credentials.
So how about not commiting sensitive stuff in the first place? If only there was
something to block unsafe commits. Enter pre-commit hooks.

## What are pre-commit hooks?

A hook, in the context of Version Control Systems, is an action that is triggered
when certain event occurs.In Git, for example, you can assign actions before an
event takes place (pre-) or afterwards (post-). These hooks can be set on the
client for actions such as commiting and merging, or on the server for receiving
a push for instance.

If we want our client to perform some checks before commiting, we will create a
pre-commit hook to run a validation script.

## Why use pre-commit hooks?

In general because it is better to avoid the damage rather than fixing it.
You can see pre-commit hooks as a safety net that will keep your Git history
clean and safe.

These are a couple of examples of the kind of help the hooks offer:

* Get rid of commits that fix silly things like syntax errors.
* Identify problems on your workspace rather than in the CI.
* Be sure your changes comply with your team's standards (linters, scanners,
  quality gates, ...).

## How to use pre-commit hooks?

Hooks in Git are binaries with a certain name inside the `.git/hooks` directory
of your workspace. To use a pre-commit hook, create an executable named
`pre-commit`. If the executable return success (exit code is 0), the commit takes
place, otherwise the commit is blocked. A very basic example would look like this:

```bash
$ cat .git/hooks/pre-commit
#!/bin/sh
set -e
! grep -qR "-----BEGIN RSA PRIVATE KEY-----" .
```

The script looks for the typical header of a private key. If it is found anywhere
in the workspace, the commit is blocked.

The executable can (and should) be more complex and can start several tools.
Some recommended tools to put a in a pre-commit hook:

* Syntax checkers like **pylint** or **shellcheck**
* Security scanners like **Talisman**
* Language specific scanners.

## Part 1: set up your workspace

As the purpose of the hooks is to avoid dangerous commits, there is a bunch of
stuff that has to be done on the developer's workstation. I will use an example
for a Python project that uses **pylint** to check coding style and syntax and
**Talisman** to scan for sensitive information. I will not go into detail on how
to configure the tools to your liking, however keep in mind, that every tool
must be configured to match the real necessities of your project. The default
configuration are nothing but an educated (sometimes opinionated) guess on your
code.

### Set-up Talisman

Talisman can be installed globally (recommended) or for a single repository.

For the global installation as a pre-commit hook:

```bash
curl --silent  https://raw.githubusercontent.com/thoughtworks/talisman/master/global_install_scripts/install.bash > /tmp/install_talisman.bash && /bin/bash /tmp/install_talisman.bash
```

Follow the instructions of the installer and restart your shell.

### Set-up pylint

Installation is done with pip:

```bash
pip install pylint
```
You can then have pylint scan your project like so:

```bash
pylint $(git ls-files '*.py')
```

### Chaining several tools.

To run several tools at once, there are a couple of options:

* Make your pre-commit executable a wrapper for the tools. In general this works
  well when you have one or two tools, but as complexity increases this may not
  be practicable.
* Use a hook manager like **pre-commit** tool or **Husky**. Certainly overkill
  for simple scenarios, but sooner than later they pay off.

## Part 2: set up the server

Awesome! Now all our team-mates use pre-commit hooks and we all comply to the same
coding guidelines and there is no way that the awkward situation with the leaked
credentials repeats ever again. All of us? Not quite, there is always one or two
indomitable (or oblivious) developers who won't run the tools as hooks.

There so much we can do to persuade them, but in the end, if they want to push
unchecked code in the repository there is nothing we can do. Or can we?

One approach is using a pre-receive hook on the server to run the same tools a
developer is supposed to run. While this will effectively avoid polluting the
repository's history, this might be too strict, and a false positive of a poorly
configured tool can block the whole team. These hooks are intended for enforcing
policies and usually developers do not have control over the server settings.

Another option is running the tools independently as part of the CI to validate
the code. Although this will let potentially dangerous commits in, the team can
still review what went wrong and decide if it actually is a blocking issue.
Besides, running the scanners and linters as part of the pipeline let the team
gather, review, and learn from these "bad commits".
This validation can even be a quality-gate only for certain branches or stages,
but not for others; and this kind of fine-tuning is easier to implement in the
DevOps cycle.
After all, everybody works on a feature-branch and in the worst case the
offending branch can be deleted.

##References

[Git documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
[Pylint documentation](https://pylint.pycqa.org/en/latest/)
[Talisman](https://thoughtworks.github.io/talisman/)

