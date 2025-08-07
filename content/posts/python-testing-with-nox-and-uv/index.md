---
title: "Python testing with UV and Nox: A perfect pairing"
description: "Streamlining Python project CI with Nox and UV for easy efficient testing."
date: 2025-08-06
keywords: ['Python', 'CI', 'Nox', 'UV', 'Testing', 'Automation']
showDate: true
showTableOfContents: true
---

{{< timeline >}}
{{< timelineItem icon="lightbulb" header="In this post, you'll learn how to:" >}}

• Set up Nox for automated testing across Python versions<br>
• Integrate UV into Nox as the environment manager<br>
• Build a blazing-fast™ GitHub Actions CI pipeline<br>
• Avoid common mistakes when combining Nox and UV<br>
{{< /timelineItem >}}
{{< /timeline >}}

If you have been following the world of Python tooling at all the past year, you've probably heard of [UV](https://docs.astral.sh/uv/). In fact there's a good chance you're already using it. If you're not, I can only recommend looking into it. For those of you that are using UV already, I'm willing to bet that you're not yet aware of the beautiful synergy that is created by the combination of UV with the [Nox](https://nox.thea.codes/en/stable/) testing automation tool. This is a new combination that is not currently well documented. I thought I'd do my part to change that.

## What Makes Nox Different from Tox?

Nox is a Python testing automation tool that is similar to [Tox](https://tox.wiki/). In case you're not familiar, Nox and Tox are tools that allow you to automate testing in multiple Python environments. This is especially useful if you're a library author and want to ensure that your code works across different versions of Python, or across different versions of your dependencies (ie. different versions of Django, Flask, etc). Tox has been around for a long time and is very popular. However, due to how Tox works, it is not easy to integrate with other package managers. Tox is limited to using the standard Python package manager (pip) to install dependencies, as well as the [virtualenv](https://github.com/pypa/virtualenv) library to create virtual environments. This means that if you want to bring your own package manager, or tool for creating virtual environments, you're out of luck. Up until recently, this was not a problem for most people, as pip and venv were the de facto standards for Python package management and virtual environments. But with the meteoric rise of UV and all the clear advantages it brings, as well as the increasing popularity of Docker and containerization, the limitations of Tox are becoming more apparent.

This is where Nox comes in. Nox is a more modern tool that has first-class support for a variety of package managers and virtual environment tools: venv, uv, [conda](https://github.com/conda/conda), [mamba](https://github.com/mamba-org/mamba), [micromamba](https://github.com/mamba-org/mamba#micromamba), or [virtualenv](https://github.com/pypa/virtualenv). You can use Nox to automate your testing regardless of which of these tools you prefer. Also, unlike Tox which uses a declarative configuration file (tox.ini), Nox uses a Python file (noxfile.py) to define your test sessions. This leverages the full power of Python to set up your tests, which can be very useful for more complex testing scenarios. If you already write a lot of Python, this will feel very natural to you.

## Installing Nox with UV

Nox is meant to be installed as a global tool. Since Nox manages environments, you can't install it as a dependency in your current environment. The official Nox documentation recommends installing it as a global tool using [Pipx](https://pipx.pypa.io/stable/) (The Tox documentation also recommends using either Pipx or UV).

If you're familiar with UV then you'll know that UV can also replace Pipx as your global tool manager. That is indeed very convenient for us, as it means we can simply install Nox using UV:

```bash
uv tool install nox
```

## First-class UV support

Support for using different environment backends is possibly the most compelling feature that Nox brings to the table. With Tox, you are stuck with pip. This is very inconvenient if you're already using UV to manage your project's dependencies because pip and UV do not share a cache. If you are using Tox together with UV, then you will have to download all of your dependencies again and store them all in a separate cache. This is not only inefficient, it also slows down your testing process. With Nox, you can use UV as your environment backend, which means that you can use the same cache for both your local development and your testing. This is a huge win for performance and possibly also your internet bandwidth.

The amount of time needed to run tests may also be significantly reduced by using UV over pip. Let's say you're installing something like [FastAPI](https://fastapi.tiangolo.com/) with `requests`, `pydantic`, `starlette`, and a few extras. On pip, that could take 8–20 seconds depending on wheel availability and platform. On UV, it might take 1–2 seconds flat. Now repeat that for every environment you want to test against. You can see this can quickly add up to a lot of time saved. It might be the difference between your tests taking 5 minutes, or 30 seconds, especially if your project has a lot of dependencies (ie. if you're using a framework).

Configuring Nox to use UV is fairly straightforward. Luckily, the [cookbook](https://nox.thea.codes/en/stable/cookbook.html) in the Nox docs includes a section on the best practices to do this.

Here is the code (slightly modified) from that section:

```python
import nox

@nox.session(
    venv_backend="uv",
    python=["3.10", "3.11", "3.12"],
)
def tests(session: nox.Session) -> None:

    session.run_install(
        "uv",
        "sync",
        "--quiet",
        f"--python={session.virtualenv.location}",
        env={"UV_PROJECT_ENVIRONMENT": session.virtualenv.location},
        external=True,
    )
    # Then run your tests:
    session.run("mypy", "src")     
    session.run("pytest", "tests", "-vvv")
```

There's two lines in the above code that are particularly important for making this work, which may not be obvious from skimming the Nox documentation (but are shown in the cookbook example). The lines are:

```python
f"--python={session.virtualenv.location}",
env={"UV_PROJECT_ENVIRONMENT": session.virtualenv.location},
```

Together, these two lines tell UV to create the new environment in the location specified by Nox, and then use the Python interpreter from that environment. Without these lines, when Nox creates an environment for the session then UV would create it in the default location and overwrite your existing `.venv` folder (wherever that may be). This would cause your local development environment to be changed every time you run your tests, which is probably not what you want. You can read more about these options in the UV documentation:

[Project Environment Path](https://docs.astral.sh/uv/concepts/projects/config/#project-environment-path)  
[UV_PYTHON env variable](https://docs.astral.sh/uv/reference/environment/#uv_python)

The above example is all you need to get started. Place the example in a file called `noxfile.py` in your project root, and you can now run the `nox` command in your terminal to automatically run MyPy and Pytest against Pythons 3.10, 3.11, and 3.12, fully taking advantage of your existing UV cache to install the environments. It's fast and it's reproducible. You can drop this noxfile.py into any project and you're off to the races. It's also pretty convenient to have one consistent `nox` command across all your Python projects.

## Using Nox Parameterized Sessions

What if you also want to test against numerous framework versions? For example, if you're a Django developer, you might want to test against Django 3.2, 4.1, and 5.0. Nox makes this easy with parameterized sessions. You can define a session that takes parameters and then run it with different arguments. Since we set up the sessions using Python, it's trivial to specify exactly what versions we want to test against. Here's an example of how you can do this:

```python
framework = "django"
VERSIONS = [3.2, 4.1, 5.0]

@nox.session(
    venv_backend="uv",
    python=PYTHON_VERSIONS,
)
@nox.parametrize("version", VERSIONS)
def tests(session: nox.Session, version: int) -> None:

    session.run_install(
        "uv",
        "sync",
        "--quiet",
        f"--python={session.virtualenv.location}",
        env={"UV_PROJECT_ENVIRONMENT": session.virtualenv.location},
        external=True,
    )

    major, minor = str(version).split(".")
    next_minor = f"{major}.{int(minor)+1}"
    session.run_install(
        "uv", "pip", "install",
        f"{framework}>={version},<{next_minor}.0",
        external=True,
    )
    # Run your tests here
```

In this example we define a list of Django versions that we want to test against. We then use the `@nox.parametrize` decorator to create a parameterized session that will run the tests for each version in the list. The `version` parameter is passed to the session function, allowing us to install the specific version of Django for each run. Then this line:

```py
f"{framework}>={version},<{next_minor}.0",
```

...ensures that it grabs the latest patch for the specified minor version of Django (For example for version 3.2 this would result in `Django>=3.2,<3.3.0`).

You can start to imagine how easy it would be to re-use this file almost exactly between projects if you already manage all of them with UV. Just change the `framework` variable to the name of your framework, and the `VERSIONS` list to the versions you want to test against, and everything else remains the same.

See the [Nox docs on parameterized sessions](https://nox.thea.codes/en/stable/config.html#parametrizing-sessions) for more information.

## Using Nox and UV in GitHub Actions CI

Both Nox and UV provide official Github actions that you can use to run your tests in a workflow. This means that you can use the same noxfile.py to run your tests locally and in your CI/CD pipeline. Behold the simplicity of the following workflow file:

```yaml
# .github/workflows/ci-checks.yml

name: CI Checks

on:
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup uv
        uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true

      - name: Setup Nox
        uses: wntrblm/nox@2025.05.01

      - name: Run Nox sessions
        run: nox
```

That's it. You can see the official [astral-sh/setup-uv](https://github.com/marketplace/actions/astral-sh-setup-uv) and [wntrblm/nox](https://github.com/wntrblm/nox) Github actions being used (wntrblm/nox seems to use the repository itself as the action). It checks out your code, sets up UV, sets up Nox, and then runs everything you've defined in your noxfile. Nox will handle the installation of all the Python versions that it needs to test against. The `enable-cache: true` option in the 'setup-uv' step ensures that the UV cache is used, making the process much faster.

Doing this same thing with Tox in CI is a bit more complicated. There is no official support for Github actions the same way that Nox has, and it requires you to set up a matrix of jobs to test against multiple Python versions. There's various solutions to try to make this easier such as the [tox-gh](https://github.com/tox-dev/tox-gh) plugin. But none of them are as simple as the above Nox setup. With Nox, you configure everything in your noxfile, making it easy to re-use the above workflow across all your Python projects. You only need to worry about changing the noxfile to suit each project, while the CI workflow remains the same.

## Conclusion

This setup is a game-changer for a few reasons:

- **Copy-Paste Simplicity**: Two files, minimal config, and you're done. Port it to any Python project in minutes.
- **UV's Speed**: UV is _much_ faster than pip for dependency resolution and installation. My CI runs take seconds even when testing numerous Python versions, and local testing is so fast that running it often is no issue. I've even started using Nox as the testing interface for terminal coding agents, and it works surprisingly well.
- **Nox's Flexibility**: Testing multiple Python versions and framework versions is a breeze with Nox's parameterized sessions.

With UV becoming more popular every day, there's going to be more people looking for more ways to use it in CI effectively. If you already use UV for managing your project then it`s only natural to want to look for solutions to this. Until recently, this has generally been a headache to achieve with traditional tooling. But now, by leveraging how UV and Nox have a practically symbiotic relationship, this process is so much simpler and faster that I don't think I'll ever have reason to use Tox again. I believe anyone who is a fan of UV will feel the same way after seeing it in action for themselves.

{{< alert icon="lightbulb" cardColor="#1B462F" iconColor="white">}}
**Pro Tip**: If you're using a GUI framework, you can add visual regression tests (aka Snapshot testing) into your unit testing regimen. Combine that with Nox's multi-version testing, and you've got full-stack UI coverage.
{{< /alert >}}

External Links:
- [Nox Documentation](https://nox.thea.codes/en/stable/)
- [UV Documentation](https://docs.astral.sh/uv/)
