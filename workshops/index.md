# Welcome to AVD Workshops

![Tools Bar](assets/images/tools_bar_light.png#only-light){: style="width:900px"}
![Tools Bar](assets/images/tools_bar_dark.png#only-dark){: style="width:900px"}

## Fundamentals

- [Git](git.md)
- [VS Code](vscode.md)
- [Ansible](ansible.md)
- [Jinja/YAML](jinja-yaml.md)
- [CI/CD Basics](cicd-basics.md)

## AVD

- [AVD](avd.md)

## Contributing

If you would like to edit any content or contribute a new workshops, please follow the following guidelines.

- [Fork](https://github.com/aristanetworks/avd-workshops) this repository
- Create a working branch
- Run pre-commit on all updates
- Create pull request

### Prepare contributor environment

This repository comes with a devcontainer that you may leverage. If you'd like to work locally, preparing your contributor environment is relatively straight forward.

```shell
git clone https://github.com/aristanetworks/avd-workshops.git
cd avd-workshops
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Run pre-commit

Pre-commit should be installed in your environment.

```shell
pre-commit install
pre-commit run --all-files
```

### Preview site updates

!!! Note
    This site uses mkdocs material insiders, some changes may not reflect locally.

```shell
mkdocs serve
```
