# Journal IA — Kilasy

Outil utilisé : Claude (Anthropic), via claude.ai, avec IntelliJ IDEA pour le code.

## Changement : C1 — Pénalité de retard

### Échange 1 — [08:02]
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


## Changement : C2 — SMS de notification

### Échange 1 — [12:40]

Demande : « Je souhaite que l'étudiant reçoive également un SMS lorsque sa note est publiée. En utilisant les fichiers existants, propose une solution simple qui permet d'ajouter cette fonctionnalité sans perturber la publication des notes. »

Proposition : dans SubmissionController, ajout d'une interface SmsSender et d'une classe LogSmsSender qui écrit « [SMS] numéro : message » dans les logs. L'envoi du SMS est effectué après celui de l'email, dans un try/catch. Une propriété sms.simulate-failure permet également de simuler une panne d'envoi.

Décision : gardé.

Pourquoi : cette solution permet de séparer l'envoi du SMS de la logique de notation grâce à l'interface SmsSender. Le try/catch garantit que la note reste publiée même si l'envoi du SMS échoue.

Vérification : Felix reçoit l'email et le SMS simulé. Avec la panne simulée, la note de Bob reste publiée (200, grade 14,4). OK.

## Bilan C2
- Fichiers modifiés : SubmissionController.java

## Changement : C3 — Exercices minutés

### Échange 1 — [19:00]

**Demande :** « Je souhaite ajouter la possibilité de définir une durée en minutes pour un exercice. Le chronomètre doit commencer lorsque l'étudiant ouvre l'exercice et la soumission doit être refusée lorsque le temps est écoulé. Il faut conserver le fonctionnement des exercices existants et afficher le temps utilisé dans le relevé de notes. »

**Proposition :** Claude a proposé d'ajouter le champ `durationMinutes`, de rendre l'échéance facultative et de créer une table `exercise_start` pour enregistrer le début du chronomètre de chaque étudiant. Il a également proposé un nouvel endpoint `/exercises/{id}/start`, une vérification du temps lors de la soumission et l'affichage du temps utilisé dans le relevé de notes et l'export CSV.

**Décision :** gardé, avec les adaptations nécessaires.

**Pourquoi :** cette solution permet de gérer les exercices minutés sans supprimer la possibilité d'utiliser une échéance classique. Elle conserve également la pénalité de retard du C1.

**Vérification :** création d'un exercice minuté, démarrage du chronomètre et vérification de la soumission après expiration. L'API renvoie bien une erreur HTTP 422 avec le message indiquant que les 2 minutes sont écoulées. Les résultats attendus s'affichent également.

### Bilan C3

- Fichiers modifiés :
    - `CsvExport.java`
    - `Exercise.java`
    - `ExerciseController.java`
    - `GradeReport.java`
    - `SubmissionController.java`
    - `schema.sql`

- Fonctionnalités ajoutées : durée en minutes, démarrage du chronomètre, refus des soumissions après expiration et affichage du temps utilisé.