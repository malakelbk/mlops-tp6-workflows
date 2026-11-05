# Orchestrateur ou integration continue ? (TP06)

| Critere                         | Prefect (orchestrateur)                                                  | GitHub Actions (CI)                                                  |
|---------------------------------|--------------------------------------------------------------------------|----------------------------------------------------------------------|
| Ce qui declenche l'execution    | Planification cron (lundi 6 h), Quick run depuis l'interface, script     | Chaque `git push` (et bouton Run workflow / workflow_dispatch)       |
| Ou s'execute le traitement      | Sur notre poste : processus `serve` ou worker du work pool `pool-tp`     | Machine Linux ephemere de GitHub (ubuntu-latest)                     |
| Qui surveille, et ou            | Interface Prefect (127.0.0.1:4200) : runs, taches, durees, journaux      | Onglet Actions du depot, commit marque vert/rouge                    |
| Ce qui se passe en cas d'echec  | Reessais (2x sur dvc repro) ; tache `valider` en Failed, `notifier` sautee | Etape rouge, job en echec, commit marque en rouge, rapports conserves |
| Type de tache adapte            | Traitements recurrents ou declenches : reentrainement, collecte, scoring | Verification du code a chaque modification avant integration         |
| Ce qui est versionne dans Git   | `flow_mlops.py`, `planifier.py`, `prefect.yaml` (l'historique des runs reste sur le serveur) | `.github/workflows/ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` |

## Conclusion (5 lignes)

La CI GitHub Actions valide le code a chaque push : un changement qui degrade le modele (max_depth: 2, F1 = 0.25) est bloque en rouge.
Prefect execute et surveille le reentrainement recurrent, planifie chaque lundi, et s'arrete seul si F1 < seuil_f1.
Le serveur garde les deploiements ; le travail n'est execute que si un processus (serve ou worker) est actif, sinon le run reste en attente.
Les deux outils appliquent la meme regle de qualite, mais a des moments differents : avant integration (CI) et en production (orchestrateur).
Ils sont complementaires : aucun ne remplace l'autre.