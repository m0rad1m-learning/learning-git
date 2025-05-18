# Git

## Inhaltsverzeichnis

1. [Basiskonzepte](#basiskonzepte)
2. [Basisbefehle](#basisbefehle)
3. [Branches](#branches)
4. [Änderungen Zwischenspeichern](#änderungen-zwischenspeichern)
5. [Änderungen rückgängig machen](#änderungen-rückgängig-machen)
6. [Inhalte löschen](#inhalte-löschen)
7. [Git under the hood](#git-under-the-hood)
8. [Git Ingore](#git-ignore)
9. [Git Config](#git-config)
10. [GitHub](#github)
11. [Pull requests](#pull-requests)
12. [Git Tags](#git-tags)
13. [ToDos](#todos)
14. [Backup](#backup)

## Basiskonzepte

[ToC](#inhaltsverzeichnis)

- Working Tree --> Staging area --> (Local) Git Repository
- Git file lifecylce: Untracked --> Modified --> Staged --> Unmodified

## Basisbefehle

[ToC](#inhaltsverzeichnis)

Übersicht Git Repository / Status / Datei zum commiten etc.

`git status`

Übertragung bestimmter Änderungen aus dem Working Tree in die Staging Area

`git add <filename>`

Übertragung aller Änderungen aus dem Working Tree in die Staging Area

`git add .`

Löschen bestimmter Änderungen aus der Staging Area

`git rm --cached <filename>`

Übertragung von Änderungen aus der Staging Area in das Local Repository

`git commit -m  "<Nachricht>"`

Stage & Commit in einem Befehl

`git commit -am "<Nachricht>"`

Ändern der letzten Commit Nachricht (der letzte Commit wird durch einen neuen Commit ersetzt)

`git commit --ammend -m "<Nachricht>"`

Ändern des letzten Commit

```shell
git add Datei1.txt
git commit -m "Änderungen an Datei1.txt & Datei2.txt"
# Das Hinzufügen der Datei2.txt zur Staging Area wurde vergessen, damit ist die Datei auch nicht im Local Repository
git add Datei2.txt
git commit --amend --no-edit
# In der Staging Area  liegt nun die Datei2.txt. Der 'amend' Befehlt überführt diese in das Local Repository, so dass der Commit Datei1.txt & Datei2.txt umfasst, also geändert wurde. Die Commit Nachricht selbst wurde nicht geändert.
```

### Übersicht aller (commit) Änderungen

Textbasierte Übersicht

`git log`

Textbasierte Übersicht des letzten Commit in einer Zeile

`git log --oneline`

Textbasierte Übersicht aller vergangenen Commit in einer Zeile

`git log --pretty=oneline`

Übersicht mit grafischer Ansicht (Verlinkung der jeweiligen parrent trees)

`git log --graph`

Übersicht der letzten Änderungen an einer Datei

`git log <file>` oder `git blame <file>`

> *Besser*: `git log --pretty=format:'%h %an %ad %s' merge.txt`

## Branches

[ToC](#inhaltsverzeichnis)

### Übersicht aller Branches

Nur lokale branches

`git branch`

Nur remote branches

`git branch -r`

Alle branches

`git branch -a`

Anzeigen der upstream / remote (tracking) branches meiner local branches

`git branch -vv`

### Branch erstellen / wechseln

Einen neuen local branch erstellen

> Dies funktioniert auch wenn der branch nur remote existiert. In diesem Fall wird der neu erzeugte local branch automatisch zu einem tracking branch des remote branches

`git branch <branch name>`

Einen neuen local branch erstellen und sofort in diesen wechseln

`git checkout -b <branch name>`

Wechseln in einen bestehenden branch

`git checkout <branch name | hash commit (frist 6 chars)>`

```text
   Example: Checkout of a dedicated hash commit in 'main' branch
   ========
   # Initial state
   (C1)--> (C2)-->(C3)-->(C4)-->(C5) <-- [HEAD, main]
    
   # Checkout hash commit
   git checkout c2

   # Final state (detachted HEAD state)
   (C1)--> (C2)-->(C3)-->(C4)-->(C5) <-- [main]
            |
          [HEAD]
```

### Branch umbenennen

Den aktuellen local branch umbenennen

`git branch -M <name>`

Einen anderen local branch umbennen

`git branch -m <old name> <new name>`

### Branches mergen (im aufnehmenden Branch, z.B."main")

Von Git verwendente merge Konzepte:

- ***Fast-forward merge***: main / HEAD pointer werden auf letzten commit des feature branch umgeleitet.
Dies geht nur, wenn nach Erzeugung des feature branch nur in diesem commits durchgeführten wurden und keine weiteren commits im main branch erfolgten
- ***3-way" merge***: Neuer commit im main branch wird erzeugt, der auf den letzten commit des main branch und den letzten commit des Feature commits zeigt. Der neue commit im main branch hat damit also zwei parrents.
Dieses Konzept wird gewählt, wenn nach der Erstellung des feature branch ein / mehrere commit im feature branch und im main branch erfolgten

Mergen des aktuellen branch mit einem feature branch

`git merge <feature branch name>`

#### Umgange mit merge Konflikten

1) Nutzen von `git status`, um die Dateien in den Konflikte  entstandenen zu identifizieren
2) Verwenden von `cat <file>`, um die Konflikte in den einzelnen Dateien zu identifizieren. Typischerweise sollte dies in etwa wie folgt aussehen:

```text
<<<<<<< HEAD
< content that exists in the current branch>
=======
<content that is present in our merging branch>
>>>>>>> <name of merging branch>
```

3) Lösen den Konflikts im Editor der Wahl, z.B. `vim` oder `VS Code` 
4) `git commit` ausführen, um die Änderungen zu bestätigen

> **HINWEIS**: Sind alle merge conflicts gelöst, wird der merge automatisch fortgeführt

### Rebasing

Rebasing ist eine Alternative zum branch merging, wobei 3-way merges durch fast-forward merges ersetzt werden. Wurde ein feature branch erzeugt und sowohl in diesem, als auch in main branch ein/mehrere commits ausgeführt, kann ein merge nur durch einen 3-way merge erfolgt. Durch rebasing wird der Startpunkt des feature branch auf den letzten commit im main branch verschoben. Somit zweigt der feature branch vom letzten commit im master branch ab und ein einfacher fast-forward merge ist möglich.
Faktisch wird durch einen rebase die Historie eines repositorys neu geschrieben, da durch den rebase alle vergangenen commits des feature branches neu gesetzt werden, um eine lineare Historie zu erzeugen.

```shell
git checkout feature-1
git rebase main
git checkout main
git merge feature-1
git branch -d feature-1
```

```text
   Example: Rebasing a feature branch
   ========
   # Initial state
            (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
             /
   (C1)--> (C2)-->(C3)-->(C4)-->(C5) <-- [main]
    
   # Execute command from above
   git checkout feature-1
   git rebase main

   # State
                                 (FB1)-->(FB2)-->(FB3) <-- [HEAD, feature-1]
                                  /
   (C1)--> (C2)-->(C3)-->(C4)-->(C5) <-- [main]
   ```

### Branches löschen (z.B. nach einem Merge)

Löschen eines (bereits gemergten) local branches

`git branch -d <name>`

Löschen eines (bereits gemergten) remote branches

`git push origin --delete <name>`

Löschen eines nicht nicht gemergeten branch (z.B. wenn der branch nicht mehr gebraucht wird)

`git branch -D <name>`

Aktuellisieren der remote branchen im lokalen repository, z.B. wenn remote branches gelöscht wurden, lokal aber noch angezeigt werden

`git remote update origin --prune`

## Änderungen Zwischenspeichern

[ToC](#inhaltsverzeichnis)

In manchen Situation kann es hilfreich sein durchgeführte Änderungen in einem branch zwischenzuspeichern, ohne diese direkt zu commiten. Hierzu dient `git stash`

### Stash

Speichern von Änderungen im Working Tree und der Staging Area.

> Es sind mehrere Zwischenspeicher möglich. Jeder Zwischenspeicher hat eine eindeutige ID: `stash@{#}`. Wird bei Verwendung von `git stash` keine eindeutige ID angegeben, wird implizit immer `stash@{0}` angenommen

`git stash`

Aufruf von Änderungen aus dem Zwischenspeicher

`git stash pop` // Änderungen werden aus dem Zwischenspeicher gelöscht

`git stash apply` // Änderungen bleiben im Zwischenspeicher erhalten

Zwischenspeichern von untracked / ignored Dateien

`git stash -u` // für untracked Dateien

`git stash -a` // für untracke und ignorierte Dateien

Anzeigen aller Änderungen im Zwischenspeicher

`git stash list`

Anzeigen der Inhalte eines Zwischenspeichers

```shell
git stash show
# For more details
git stash show -p
```

Zwischenspeicher löschen

```shell
`git stash drop <stash id>`
# All stahses
git stash clear
```

## Änderungen rückgängig machen

[ToC](#inhaltsverzeichnis)

### Clean

Löschen von Dateien die nur im Working Tree liegen und bisher nicht getrackt werden

```shell
# Dry-run
git clean -n
# Löschen nicht getrackter Dateien
git clean -f
# Löschen nicht getrackter Dateien & Verzeichnisse
git clean -df
```

### Reset

Reset entspricht einem "Zurück in die Vergangenheit". Alle Commits die vor dem genannten Rücksprung-Commit liegen, exisieren danach nicht mehr. Mit `git reset` können also ein oder mehrere Commits zurück genommen werden

> Reset ist immer 'destruktiv', da es die Historie ändert. Insbesondere die Verwendung von *hard* sollte mit Bedacht gewählt werden.

Folgende Varianten werden unterschieden:

- *hard*: Alle Änderungen im Working Direcoty, der Staging Area und ggf. der Commit History werden zurück gesetzt
  
  ```shell
  # Reset auf den HEAD löscht nur alle Änderungen im Working Tree und der Staging Area
  git reset --hard  # HEAD wird implizit angenommen, alternativ z.B. "origin/main"
  # Reset auf eien spezifischen Commit in der Vergangenheit, alle Änderungen danach sind verlorten
  git reset --hard "<commit ID (first 6 chars)>"
  ```

- *mixed*: Alle Änderungen in der Staging Area und im Local Repository werden zurück gesetzt, Änderungen im Working Tree bleiben erhalten

  --> Dies ist das "default" Verhalten bei Verwendung von `git reset`

  ```shell
  # Default
  git reset # HEAD wird implizit angenommen
  # Reset auf eien spezifischen Commit in der Vergangenheit, alle Änderungen danach sind verlorten
  git reset --mixed "<commit ID (first 6 chars)>"
  ```

- *soft*: Alle Änderungen im Local Repository werden zurück gesetzt, Änderungen im Working Tree und der Staging Area bleiben erhalten

  ```shell
  # Default
  git reset --soft <HEAD (implizit) | <commit hash (first 6 chars) | HEAD~5 to reset last 5 commits>
  # Reset auf eien spezifischen Commit in der Vergangenheit, alle Änderungen danach sind verlorten
  ```

### Revert

Mit `revert` kann die letzte vergangene Änderungen rückgängig gemacht werden. Anders als bei `reset` wird bei `revert` jedoch nicht "die Zeit zurück gedreht", sondern es wird ein neuer commit erzeugt, welcher die letzten Änderungen Rückgängig macht. Die Historie bleibt bei `revert` also erhalten und wird nicht gelöscht.

Die Verwendung von `revert` ist sinnvoll, wenn Änderungen bereits in ein remote repository eingeflossen sind und andere Entwickler ggf. bereits ein pull auf diese remote repository ausgeführt haben.

> Revert ist nicht destruktiv, da die Historie bestehen bleibt und ermöglich es immer nur einen Commit zurück zu nehmen.

`git revert <HEAD OR commit hash (first 6 chars)>`

## Inhalte löschen

[ToC](#inhaltsverzeichnis)

Löschen getrackter Inhalte im Working Tree und der Staging Area

```shell
git rm <name>
git rm -n <name>  # dry-run
git rm -r <folder>  # recursive delete
```

> Die Löschung im local repository wird erst wirksam, wenn diese mit `git commit` bestätigt wurde. Vorher kann eine Löschung durch `git checkout .` oder `git reset HEAD` rückgängig gemacht werden.

Löschen getrackter Inhalte nur der Staging Area, ohne Auswirkungen auf das Working Tree

`git rm --cached <name>`

## Git under the hood

[ToC](#inhaltsverzeichnis)

Informationen zu Git objects

```shell
git cat-file -p <commit hash (first 6 chars)>  # Content of the object
git cat-file -t <commit hash (first 6 chars)>  # Type of the object
git cat-file -s <commit hash (first 6 chars)>  # Size of the object
```

Informationen zu Dateien in der Staging area
`git ls-files -s`

## Git Ignore

[ToC](#inhaltsverzeichnis)

In der Datei `.gitignore` können untracked Dateien / Verzeichnise verwaltet werden. Die Datei .gitignore muss dazu selbst commited werden.

```(shell)
# Ignore file 1
file-1.txt
# Igorne all filed in bin folder
/bin
# Ignore all temp files
*.tmp
```

## Git Config

[ToC](#inhaltsverzeichnis)

Es gibt drei verschiedene Git Config Ebenen:

- *System* (optional): `git` Einstellungen für alle User des Systems (Pfad: `$/etc/gitconfig`)
- *Global*: `git` Einstellungen für alle repositiries des Users (Pfad: `$USER_HOME/.gitconfig`)
- *Local*: `git` Einstellungen für ein dediziertes repositiries eines Users (Pfad: `<repo-home>/.gitconfig`)

### Globale config

Globale config auslesen

`git config --list`

Gobalen user name und eail setzen

```(shell)
git config --global user.name "<Name>"
git config --global user.email <E-Mail>
```

Den default name eine neuen Reporistories ändern

`git config --global init.defaultBranch >main>`

### Lokale / User config

Lokale Einstellungen überschreiben immer die globalen Einstellungen in einem local reporistory

Lokale Config auslegen

`cat .git/config`

Lokalen user namen und eMail setzen

```(shell)
git config user.name "<Name>"
git config user.email <E-Mail>
```

### Beispiel einer .gitconfig

```text
[user]
    name=<YOUR-USER-NAME>
    email=<YOUR-USER-EMAIL>
    signingkey=~/.ssh/id_ed25519
[init] 
    defaultbranch=main
[gpg]
    format=ssh                               
[commit]
    gpgsign=true
```

## GitHub

[ToC](#inhaltsverzeichnis)

Nach Erstellung eines Git Hub Repositories

### Verknüpfen eines local mit einem remote repositories

Verknüpfen des local mit dem remote repository

`git remote add <remote repository name, z.B. origin> <url des remote repository>`

Anzeige aller remote repository, denen im local reporistory gefolgt wird

`git remote`

Überprüfen der fetch / push URLs des remote repository

`git remote -v`

Details zum remote repository

`git remote show <name of remote repository, z.b. origin>`

Initiales klonen des remote repository nach local
Per Default wird nur der default branch des remote repository zu einem tracking branch im local repository, alle weiteren branches werden nicht geklont

`git clone <url>`

### Aktualisieren des local repository mit Änderungen im remote repository

Anzeige aller branches und deren letzten commit IDs

`git show-ref`

Vergleich der commits zweier branches zwischen local und remote repositories

`git show-ref <branch name>`

Laden aller branches und/or tags aus dem remote repository in das local repository
Dies hat keine Auswirkung auf Working Tree / die Staging Area. Es erfolgt kein sync von Versionen / keine Erzeugung eines local (tracking) branches

`git fetch`

Nach dem fetch Befehl kann der remote branch ausgecheckt und alle Daten übertragen werden

`git checkout <remote branch name>`

Daten aus dem remote repository laden

`git pull`

    > Änderungen aus dem remote repository sollten regelmäßig in das local repository überspielt werden.
    
    Git pull" führt implit zwei Schritte aus:

    1. *git fetch*: Übertragen aller Updates / Changes etc. des remote repository in das local repository als objects (blob, tree, commit), jedoch ohne Änderungen am local working directory / staging area
    2. *git merge* (Basis: FETCH_HEAD) : 2- oder 3-way merge zwischen local repository und local working area & staging area. Dies betrifft alle tracking branches, beginnend beim aktuell ausgecheckten branch

Vor der Durchführung mittels `git branch -vv` prüfen, ob und welche local branches tracking branches sind

Aktualisierung des tracking status eines local repository branches, z.B. nachdem der zugehörige remote branch gelöscht wurde

`git remote update <origin> --prune`

Der local branch kann hiernach gelöscht werden

`git branch -D <local branch name>`

Löschen der Referenzen auf remote repository branches, den es im remote repository nicht mehr gibt

`git remote prune <remote repository name, z.B. origin>`

### Hochladen von Änderungen im local repository an das remote repository

Übertragung aller Änderungen des aktuellen local repositroy branches in den bestehenden remote repository branch

`git push`

Erstellung eines neuen remote repositroy branches, basierend auf einem bestehenden local repository branch und Übertragung aller Daten in den remote repository branch

`git push -u <remote repository name, z.B. origin> <local branch name>`

Löschen eines remote branches

`git push <remote repository name, z.B. origin> -d <branch name>`

Hier nach kann der local branch gelöscht werden

`git branch -d <local branch name>`

### Tracken eines remote repository von einem bestehenden local reporistory

```(shell)
git remote add origin <URL>
git push -u origin main
```

## Pull requests

[ToC](#inhaltsverzeichnis)


Mit pull requestes (PR), auch "Merge requests", bittet man andere Entwickler um ein Review der eigenen Änderungen im local repository branch, bevor diese in den remote repository branch eingespielt werden.

## Git tags

[ToC](#inhaltsverzeichnis)

Git Tags sind Referenzen, die auf bestimmte Zeitpunkte im Git-Verlauf verweisen (Commits). In der Regel werden mit Tags bestimmte Punkte im Verlauf erfasst, die für einen markierten Versions-Release (z. B. v1.0.1) verwendet werden.

Es wird zwischen Lightweight und Annotated-Tags unterschieden. Annotated-Tags werden von Git wie Commits behandelt und sind daher zu bevorzugen.

Übersicht aller aktuell vorhandenen Tags

`git tag`

### Annotated git tag erstellen

Erstellen eines `git tags` der auf den letzten Commit verweist

`git tag -s -a <tag-version> -m "<message>"`

Erstellen eines `git tags` der nachträglich auf einen bestimmten Commit verweist

`git tag -s -a <tag-version> -m "<message>" <commit-id (first 6 chars)>`

### Pushing von Git Tags

Git Tags müssen explizit in das remote repositiry gepusht werden, da sie nicht implizit durch `git push` übertragen werden.

`git push <remote repository name, z.B. origin> <tag-version>`

## ToDos

[ToC](#inhaltsverzeichnis)

### Amend

## Backup

[ToC](#inhaltsverzeichnis)

 ```text
              (FB1)-->(FB2)-->(FB3)
              /                   \
    (C1)--> (C2)-->(C3)-->(C4)-->(C5)
    
  ```