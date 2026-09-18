# LB 324

Tagebbbuch-Applikation mit CI/CD-Umgebung.

## Laufende Applikation

<!-- TODO: URL nach dem Azure-Deployment hier eintragen -->
https://siinoantonio-lb324.azurewebsites.net

## Ast-Strategie

| Ast         | Zweck                                                                      |
| ----------- | -------------------------------------------------------------------------- |
| `main`      | Entspricht immer der ausgelieferten Version. Merge hierhin = Auslieferung.  |
| `dev`       | Sammelt alle abgeschlossenen, getesteten Änderungen.                        |
| `feature/*` | Eigentliche Entwicklung. Wird jeweils ab `dev` erstellt.                    |

## Lokal starten

```
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
flask run
```

Die Applikation erwartet eine `.env`-Datei im Projektverzeichnis mit folgendem Inhalt:

```
PASSWORD="einSehrGeheimesPasswort"
```

Diese Datei ist über `.gitignore` von der Versionierung ausgeschlossen und darf
nicht eingebucht werden.

## Aufgabe 2

Erklären Sie hier, wie man `pre-commit` installiert.

`pre-commit` sorgt dafür, dass nur formatierter und getesteter Code in die Ablage
gelangt. Die Konfiguration steht in `.pre-commit-config.yaml`.

**Einmalige Einrichtung** (nach dem Klonen der Ablage, im aktivierten venv):

```
pip install pre-commit
pre-commit install --install-hooks
```

`pre-commit install --install-hooks` installiert dank des Eintrags
`default_install_hook_types: [pre-commit, pre-push]` in der Konfigurationsdatei
**beide** Hook-Typen auf einmal. Gleichwertig und expliziter:

```
pre-commit install --hook-type pre-commit --hook-type pre-push
```

**Was danach automatisch passiert:**

| Befehl       | Ausgelöster Hook | Wirkung                                                              |
| ------------ | ---------------- | -------------------------------------------------------------------- |
| `git commit` | `pre-commit`     | `black` formatiert den Code, Leerzeichen/Zeilenenden werden bereinigt |
| `git push`   | `pre-push`       | `pytest` führt die gesamte Test-Suite aus                             |

Formatiert `black` beim Commit eine Datei um, bricht der Commit ab und die
geänderten Dateien müssen erneut hinzugefügt und eingebucht werden:

```
git add .
git commit -m "..."
```

Schlägt beim Push ein Test fehl, wird der Push abgebrochen. So gelangt nur
getesteter Code in die Ablage.

**Manuell über alle Dateien laufen lassen** (z.B. direkt nach der Einrichtung):

```
pre-commit run --all-files
```

## Aufgabe 3

Die Datei `.github/workflows/tests.yml` führt bei jedem *pull request* auf den
`dev`-Ast die Test-Suite aus. Weitere Auslöser sind bewusst nicht konfiguriert.

## Aufgabe 4

Erklären Sie hier, wie Sie das Passwort aus Ihrer lokalen `.env` auf Azure übertragen.

Die `.env`-Datei wird bewusst nicht eingebucht und ist deshalb auf Azure nicht
vorhanden. `os.getenv("PASSWORD")` in `app.py` liest die Variable jedoch aus der
Umgebung – auf Azure wird sie darum als *App setting* hinterlegt. App settings
werden dem Prozess als Umgebungsvariablen zur Verfügung gestellt, wodurch der
Code unverändert funktioniert.

Vorgehen im Azure-Portal:

1. Im Portal den erstellten **App Service** öffnen.
2. Links unter **Settings** auf **Environment variables** wechseln
   (in älteren Portal-Versionen: **Configuration** → **Application settings**).
3. Im Reiter **App settings** auf **+ Add** klicken.
4. **Name**: `PASSWORD`
   **Value**: das Passwort (gemäss Aufgabenstellung der github-Benutzername).
5. Mit **Apply** bzw. **OK** bestätigen und anschliessend oben auf **Apply** /
   **Save** klicken, um die Änderung zu speichern.
6. Der App Service startet daraufhin neu; die Variable ist danach aktiv.

Der Wert wird ausschliesslich in Azure gespeichert und erscheint nirgends in der
Ablage auf github.com.

## Automatische Auslieferung

Der von Azure erzeugte Workflow unter `.github/workflows/` löst bei jedem Push
auf den `main`-Ast – also nach jedem erfolgreichen Merge – eine erneute
Auslieferung auf Azure aus.
