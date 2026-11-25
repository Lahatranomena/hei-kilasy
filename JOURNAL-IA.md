# Journal IA — Kilasy

Outil utilisé : Claude (Anthropic), via claude.ai, avec IntelliJ IDEA pour le code.

## Changement : C1 — Pénalité de retard

### Échange 1 — 08:02
Demande : « Aide-moi à configurer le projet, pourquoi la connexion à la base échoue ? »

Proposition : un PostgreSQL installé sur Windows occupait déjà le port 5432 et refusait l'utilisateur kilasy. L'IA a proposé de changer le port du conteneur Docker en 5433 dans compose.yaml et dans l'URL de application.properties.

Décision : gardé.

Pourquoi : cela évite d'arrêter le PostgreSQL local de ma machine.

Vérification : docker compose ps affiche 5433->5432 et l'application démarre. OK.

### Échange 2 — 08:12
Demande : « Ajoute une fonction pour calculer les pénalités de retard : 10 % par jour de retard, avec un maximum de 50 %, sans que la note puisse devenir négative. »

Proposition :
- ajout d'une nouvelle fonction penalizedGrade dans GradeReport pour calculer la pénalité (−10 % par jour de retard, plafond −50 %, jamais négative), appelée aussi par SubmissionController et CsvExport ;
- remplacement des arrondis différents (Math.round, HALF_UP, Math.floor) par HALF_UP partout ;
- l'échéance est une date sans heure : un rendu le jour même n'est pas pénalisé, un rendu le lendemain compte 1 jour.

Décision : gardé.

Pourquoi : une seule formule donne la même note dans la correction, le relevé et le CSV, sans réorganiser le code.

Vérification : une requête SQL indépendante donne 14,4 (Bob), 11,2 (Clara), 8,0 (David, plafond), 12,0 (David), 13,8 (Emma), comme attendu. OK.

## Bilan
- Fichiers modifiés : compose.yaml, application.properties, GradeReport.java, SubmissionController.java, CsvExport.java.