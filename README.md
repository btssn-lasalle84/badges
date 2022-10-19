# Les badges GitHub

- [Les badges GitHub](#les-badges-github)
  - [Badge d'état d'un workflow GitHub Actions](#badge-détat-dun-workflow-github-actions)
  - [Badge de points pour GitHub Classroom](#badge-de-points-pour-github-classroom)
  - [Les badges badgen.net](#les-badges-badgennet)
  - [Les badges img.shields.io](#les-badges-imgshieldsio)
  - [Les badges Operating System](#les-badges-operating-system)
  - [GitHub Profile](#github-profile)
  - [Simple Icons](#simple-icons)
  - [Liens](#liens)

## Badge d'état d'un workflow GitHub Actions

GitHub Actions permet d'automatiser des tâches associées à un dépôt GitHub.

La "tâche" (le travail) est décrite dans un fichier `.yml` stocké dans l'arborescence `.github/workflows/`.

Le fichier peut être créé directement à partir de l'interface web de GitHub dans l'onglet `Actions` puis `New workflow`.

GitHub Actions propose alors des modèles prêts à l'emploi. On peut créer son propre _workflow_ en cliquant sur `set up a workflow yourself`. Un squelette est alors fourni.

Il faut commencer par donner un nom au fichier, par exemple : `build.yml`

Le _workflow_ doit avoir un **nom** (c'est lui qui apparaîtra dans le badge) :

```yml
name: Build C/C++
```

Puis on définit l'évènement (_on_) déclencheur (_trigger_), ici un `git push` sur la branche principale `main` :

```yml
on:
  push:
    branches: [main]
```

> Il est possible de déclencher l'action sur une _Pull Request_ : `pull_request: branches: [main]`

Ensuite, on décrit le travail (_job_) à exécuter (_runs-on_) sur une machine (virtuelle, par exemple Ubuntu), qui est généralement composé d'étapes (_step_) :

- Extraction du dépôt
- Installation et configuration de PlatformIO
- Fabrication du projet PlatformIO

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: make all
        run: make all
```

_Remarque :_ Le mot clé `uses` permet de récupérer des actions existantes fournies par des contributeurs.

Il est possible ensuite d'associer un badge d'état (une image) à un _workflow_ de GitHub Actions.

Pour cela, il faut aller dans l'onglet `Actions` puis cliquer sur le _workflow_ (dans la liste à gauche). Il faut ensuite cliquer sur les trois petits points `...` (à droite de la zone de recherche) et sélectionner `Create status badge`. Une boîte de dialogue s'affiche et il faut récupérer le code Markdown (`Copy status badge Mardown`).

Ce code est à placer dans le fichier `README.md` du dépôt (généralement au tout début du fichier) pour afficher le badge sur la page d'accueil.

```markdown
[![Build C/C++](https://github.com/<organisation>/<depot>/actions/workflows/build.yml/badge.svg?branch=main)](https://github.com/<organisation>/<depot>/actions/workflows/build.yml)
```

Ou avec un code HTML :

```html
<p align="center">
  </a>
    <a href="https://github.com/<organisation>/<depot>/actions/workflows/build.yml">
    <img src="https://github.com/<organisation>/<depot>/actions/workflows/build.yml/badge.svg?branch=main"/>
  </a>
</p>
```

![](images/makefile.png)

Il est possible de générer de badge à partir du _workflow_ (cela peut être utile avec GitHub Classroom où le dépôt de départ est un _template_).

Ce _workflow_ génère un badge `makefile.svg` :

```yml
name: Makefile

on:
  push:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0 # otherwise, you will failed to push refs to dest repo
    - name: make
      id: make
      continue-on-error: true
      shell: bash
      run: |
        label="Makefile"
        echo "##[set-output name=label;]${label}"
        make
        res=$?
        if [ $res -eq 0 ]
        then
          retour="passing"
          color="green"
        else
          retour="failing"
          color="red"
        fi
        echo "##[set-output name=status;]${retour}"
        echo "##[set-output name=color;]${color}"

    # switch to badges branch
    - run: |
        git checkout badges || git checkout -b badges
        mkdir -p .github/badges

    # create badge
    - name: badge makefile
      uses: emibcn/badge-action@v1.2.1
      with:
        label: ${{ steps.make.outputs.label }}
        status: ${{ steps.make.outputs.status || 'failing' }}
        color: ${{ steps.make.outputs.color || 'red' }}
        path: '.github/badges/makefile.svg'

    # commit and push badges if badges have changed
    - name: Commit changes to badge makefile
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add '.github/badges/makefile.svg'
        git commit -m "Add/Update badge makefile" || exit 0
        git push origin badges
```

Il suffit maintenant d'intégrer l'image du badge dans le `README.md` :

```markdown
[![Makefile badge](../../blob/badges/.github/badges/makefile.svg)](../../actions)
```

Ou avec un code HTML :

```html
<img alt="badge makefile" align="right" src="../../blob/badges/.github/badges/makefile.svg" />
```

Lien : https://github.com/emibcn/badge-action

## Badge de points pour GitHub Classroom

GitHub Classroom permet de "noter" un devoir.

La notation s'établit dans un fichier `.github/classroom/autograding.json`, par exemple :

```json
{
  "tests": [
    {
      "name": "Make",
      "setup": "",
      "run": "make",
      "input": "",
      "output": "",
      "comparison": "included",
      "timeout": 10,
      "points": 10
    }
  ]
}
```

Il est possible d'obtenir un badge de points avec un _workflow_ de GitHub Actions `.github/workflows/classroom.yml` :

```yml
name: GitHub Classroom Workflow

on:
  push:
    branches:
      - "*"
      - "!badges"

jobs:
  build:
    name: Autograding
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0 # otherwise, you will failed to push refs to dest repo

      # add id to action so outputs can be used
      - uses: education/autograding@v1
        id: autograder
        continue-on-error: true

      # switch to badges branch
      - run: git checkout badges || git checkout -b badges

      # create points bar
      - name: points bar
        uses: markpatterson27/points-bar@v1
        with:
          points: ${{ steps.autograder.outputs.points }}
          path: ".github/badges/points-bar.svg"

      # commit and push badges if badges have changed
      - name: Commit changes to points bar
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add '.github/badges/points-bar.svg'
          git commit -m "Add/Update points bar" || exit 0
          git push origin badges
```

![](images/points-bar.svg)

Liens :

- https://github.com/markpatterson27/points-bar
- https://github.com/stevenbitner/autograding
- https://github.com/emibcn/badge-action

## Les badges badgen.net

Liens : https://badgen.net/ et https://badgen.net/github

Format :

```txt
https://badgen.net/badge/:subject/:status/:color?icon=github
                   ──┬──  ───┬───  ──┬───  ──┬── ────┬──────
                     │       │       │       │       └─ Options (label, list, icon, color)
                     │       │       │       │
                     │      TEXT    TEXT    RGB / COLOR_NAME ( optional )
                     │
                  "badge" - default (static) badge generator
```

Exemples :

```
https://badgen.net/badge/tvaira/actif/blue
```

![](https://badgen.net/badge/tvaira/actif/blue)

```
https://badgen.net/badge/icon/github?icon=github&label
```

![](https://badgen.net/badge/icon/github?icon=github&label)

```
https://badgen.net/badge/tvaira/actif/blue?icon=github
```

![](https://badgen.net/badge/tvaira/actif/blue?icon=github)

```
https://badgen.net/badge/tvaira/%E2%98%85%E2%98%85%E2%98%85%E2%98%85%E2%98%86
```

![](https://badgen.net/badge/tvaira/%E2%98%85%E2%98%85%E2%98%85%E2%98%85%E2%98%86)

## Les badges img.shields.io

Lien : https://shields.io/

Format :

```txt
https://img.shields.io/badge/<SUBJECT>-<STATUS>-<COLOR>.svg
```

Exemples :

```
https://img.shields.io/badge/Build-yes-green.svg
```

![](https://img.shields.io/badge/Build-yes-green.svg)

```
https://img.shields.io/badge/Build-oui-1abc9c.svg
```

![](https://img.shields.io/badge/Build-oui-1abc9c.svg)

```
https://img.shields.io/badge/R%C3%A9alis%C3%A9%20avec-Markdown-1f425f.svg
```

[![made-with-Markdown](https://img.shields.io/badge/R%C3%A9alis%C3%A9%20avec-Markdown-1f425f.svg)](http://commonmark.org)

```
https://img.shields.io/badge/License-GPL-blue.svg
```

[![GPL license](https://img.shields.io/badge/License-GPL-blue.svg)](http://perso.crans.org/besson/LICENSE.html)

```
https://img.shields.io/badge/c++-%2300599C.svg?style=for-the-badge&logo=c%2B%2B&logoColor=white
```

![C++](https://img.shields.io/badge/c++-%2300599C.svg?style=for-the-badge&logo=c%2B%2B&logoColor=white)

```
https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white
```

![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)

```
https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white
```

![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

```
https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black
```

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

## Les badges Operating System

```
https://svgshare.com/i/Zhy.svg
```

[![Linux](https://svgshare.com/i/Zhy.svg)](https://svgshare.com/i/Zhy.svg)


```
https://svgshare.com/i/ZjP.svg
```

[![macOS](https://svgshare.com/i/ZjP.svg)](https://svgshare.com/i/ZjP.svg)


```
https://svgshare.com/i/ZhY.svg
```

[![Windows](https://svgshare.com/i/ZhY.svg)](https://svgshare.com/i/ZhY.svg)

## GitHub Profile

```
https://github-profile-trophy.vercel.app/?username=tvaira&row=1
```

[![tvaira github trophy](https://github-profile-trophy.vercel.app/?username=tvaira&row=1)](https://github.com/tvaira/github-profile-trophy)

```
https://github-readme-stats.vercel.app/api?username=tvaira&theme=blue-green
```

[![tvaira github stats](https://github-readme-stats.vercel.app/api?username=tvaira&theme=blue-green)](https://github.com/tvaira/github-readme-stats)

```
https://github-readme-stats.vercel.app/api/top-langs/?username=tvaira&theme=blue-green
```

[![tvaira top languages](https://github-readme-stats.vercel.app/api/top-langs/?username=tvaira&theme=blue-green)](https://github.com/tvaira/github-readme-stats)

## Simple Icons

```
[![git](https://img.shields.io/badge/--F05032?logo=git&logoColor=ffffff)](http://git-scm.com/)
```

[![git](https://img.shields.io/badge/--F05032?logo=git&logoColor=ffffff)](http://git-scm.com/)

```
[![GitHub](https://img.shields.io/badge/--181717?logo=github&logoColor=ffffff)](https://github.com/)
```

[![GitHub](https://img.shields.io/badge/--181717?logo=github&logoColor=ffffff)](https://github.com/)

```
[![Visual Studio](https://badgen.net/badge/icon/visualstudio?icon=visualstudio&label)](https://visualstudio.microsoft.com)
```

[![Visual Studio](https://badgen.net/badge/icon/visualstudio?icon=visualstudio&label)](https://visualstudio.microsoft.com)

```
[![Jira](https://badgen.net/badge/icon/jira?icon=jira&label)](https://https://jira.com/)
```

[![Jira](https://badgen.net/badge/icon/jira?icon=jira&label)](https://https://jira.com/)

```
![Terminal](https://badgen.net/badge/icon/terminal?icon=terminal&label)
```

![Terminal](https://badgen.net/badge/icon/terminal?icon=terminal&label)

## Liens

- https://github.com/Naereen/badges
- https://github.com/Ileriayo/markdown-badges

Voir aussi :

- https://www.runforesight.com/blog/top-5-badges-that-will-show-your-github-repository-is-well-tested-trusted

---
©️ LaSalle Avignon - 2022 - <thierry.vaira@gmail.com>
