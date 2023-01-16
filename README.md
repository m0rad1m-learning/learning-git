# Git

## Basiskonzepte

- Working directory --> Staging area --> Git Repository
- Git file lifecylce: Untracked --> Modified --> Staged --> Unmodified

## Basisbefehle
 
Übersicht Git Repository / Status / Datei zum commiten etc.

`git status`

Übertragung von Änderungen aus dem Working Directory in die Staging Area

`git add <filename>`

Alle Dateien auf einmal hinzufügen

`git add .`

Löschen von Datein aus der Staging Area

`git rm --cached <filename>`

Übertragung von Änderungen aus der Staging Area in das Git Repository

`git commit -m  "<Nachricht>"`

Stage & Commit in einem Befehl

`git commit -a -m "<Nachricht>"`

### Übersicht aller (commit) Änderungen

Textbasierte Übersicht

`git log`

Übersicht mit grafischer Ansicht (Verlinkung der jeweiligen parrent trees

`git log --graph`

## Branches

### Übersicht aller Branches

Nur lokale Branches

`git branch`

Nur remote Branches

`git branch -r`

Alle Branches

`git branch -a`

Anzeigen der upstream branches meiner local branches

`git branch -vv`

### Branch erstellen / wechseln

Einen neuen local branch erstellen
- Dies funktioniert auch wenn der Branch nur remote existiert
- In diesem Fall wird der neu erzeugte local branch automatisch zu einem tracking branch des remote Branches

`git branch <branch name>`

Einen neuen local branch erstellen und sofort in diesen wechseln

`git checkout -b <branch name>`

Wechseln in einen bestehenden Branch

`git checkout <branch name> ODER <hash of respective commit>` (erste 6 Zeichen ausreichend)
 
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

### Branches löschen (z.B. nach einem Merge)

Löchen eines (bereits gemergten) branches

`git branch -d <name>`

Löschen eines nicht nicht gemergedten branch (z.B. wenn der Branch nicht mehr gebraucht wird)

`git branch -D <name>`

## .gitignore

In der Datei .gitignore können untracked Dateien / Verzeichnise verwaltet werden. Die Datei .gitignore muss dazu selbst commited werden.
```
# Ignore file 1
file-1.txt
# Igorne all filed in bin folder
/bin
# Ignore all temp files
*.tmp
```

Ignorieren und löschen einer bereits commiteten Datei:
1. Datei in .gitignore aufnehmen
2. `git rm --cached <file>`
3. `git commit -m "File ignored`
4. `git push`


## Git Config

### Globale config

Globale config auslesen

`git config --list`

Gobalen user name und eail setzen

```
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
 
```
git config user.name "<Name>"
git config user.email <E-Mail>
```

## Git under the hood

Informationen zu Git objects
```
git cat-file -p <hash> // Content
git cat-file -t <hash> // Type
git cat-file -s <hash> // Size
```

Informationen zu Dateien in der Staging area
`git ls-files -s`

## Fortgeschrittenes Git

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

### Reset

Mit reset können vergangene commits rückgängig gemacht werden. Hierbei werden drei reset arten unterschieden

- ***soft***: Nur commit werden rückgängig gemacht, in working directory und der staging area bleiben diese aber erhalten.
- ***mixed*** (default): Änderungen in commits und staging area werden rückgängig gemacht, im working directory bleiben alle Änderungen aber enthalten.
- ***hard***: Commit und alle Änderungen in staging area und working directory werden rückgängig gemacht --> Mit bedacht zu nutzen

Reset ist immer 'destruktiv', da es die Historie ändert.
Reset ermöglich es mehrere commit zurück zu nehmen.

```shell
git resest <commit hash value to reset to> OR <HEAD~5 to resetz the last 5 commit>
git reset --<soft|hard> <commit hash value to reset to> OR <HEAD~5 to resetz the last 5 commit>
```

### Revert

Mit revert kann die letzte vergangene Änderungen rückgängig gemacht werden. Anders als bei reset wird bei revert jedoch nicht "die Zeit zurück gedreht", sondern es wird ein neues commit erzeugt, welche die letzten Änderungen Rückgängig macht. Die Historie bleibt bei revert also erhalten und wird nicht gelöscht. Die Verwendung von revert ist sinnvoll, wenn Änderungen bereits in ein remote repository eingeflossen sind und andere Entwickler ggf. bereits ein pull auf diese remote repository ausgeführt haben.

Revert ist nicht destruktiv, da die Historie bestehen bleibt.
Revert ermöglich es immer nur einen commit zurück zu nehmen.

`git revert <commit hash value to reset to> OR <HEAD~5 to resetz the last 5 commit>`

### Amend

# GitHub / Bitbucket / GitLab

Nach Erstellung eines Git Hub Repositories

## Verknüpfen eines local mit einem remote repositories 

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

## Aktualisieren des local repository mit Änderungen im remote repository

Laden aller branches und/or tags aus dem remote repository in das local repository
Dies hat keine Auswirkung auf working directory / staging area im local repository: Es erfolgt kein sync von Versionen / keine Erzeugung eines local (tracking) branches

`git fetch`

Nach dem fetch Befehl kann der remote branch ausgecheckt und alle Daten übertragen werden

`git checkout <remote branch name>`

Anzeige aller branches und deren letzten commit

`git show-ref`

Vergleich der commits zweier branches zwischen local und remote repositories

`git show-ref <branch name>`

Daten aus dem remote repository laden

`git pull`

Änderungen aus dem remote repository sollten regelmäßig in das local repository überspielt werden.
"Git pull" führt implit zwei Schritte aus:

1. ***git fetch***: Übertragen aller Updates / Changes etc. des remote repository in das local repository als objects (blob, tree, commit), jedoch ohne Änderungen am local working directory / staging area
2. ***git merge*** (Basis: FETCH_HEAD) : 2- oder 3-way merge zwischen local repository und local working area & staging area. Dies betrifft alle tracking branches, beginnend beim aktuell ausgecheckten branch

Vor der Durchführung mittels "git branch -vv" prüfen, ob und welche local branches tracking branches sind

Aktualisierung des tracking status eines local repository branches, z.B. nachdem der zugehörige remote branch gelöscht wurde

`git remote update <origin> --prune`

Der local branch kann hiernach gelöscht werden

`git branch -D <local branch name>`

Löschen eines local repository branches, den es im remote repository nicht mehr gibt

`git remote prune <remote repository name, z.B. origin>`

## Hochladen von Änderungen im local repository an das remote repository

Übertragung aller Änderungen des aktuellen local repositroy branches in den bestehenden remote repository branch

`git push`

Erstellung eines neuen remote repositroy branches, basierend auf einem bestehenden local repository branch und Übertragung aller Daten in den remote repository branch

`git push -u <remote repository name, z.B. origin> <local branch name>`

Löschen eines remote branches

`git push <remote repository name, z.B. origin> -d <branch name>`

Hier nach kann der local branch gelöscht werden

`git branch -d <local branch name>`

## Tracken eines remote repository von einem bestehenden local reporistory

```
git remote add origin <URL>
git push -u origin main
```

## Pull requests (auch "Merge requests")

Mit pull requestes (PR) bittet man andere Entwickler um ein Review der eigenen Änderungen im local repository branch, bevor diese in den remote repository branch eingespielt werden.
