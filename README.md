# Backlog initial — PrêtMatériel

Source des issues GitHub créées par `scripts/setup_github.py`. Chaque section `##` devient une issue.
La ligne de métadonnées (commentaire HTML) donne la priorité, les points, le milestone et les labels.
Une fois les issues créées, **c'est GitHub qui fait foi** : on modifie les issues, pas ce fichier.

Codes HTTP : 200 lecture/modification · 201 création · 204 suppression · 400 requête invalide · 404 introuvable · 409 conflit.

## M1 — Ajouter un équipement à l'inventaire
<!-- priorite: Must | points: 3 | milestone: Livrable 2 | domaine: inventaire -->

### User story
En tant que préposé·e,
je veux ajouter un équipement à l'inventaire (nom, catégorie, numéro d'inventaire, état)
afin de pouvoir le prêter et suivre sa disponibilité.

### Critères d'acceptation
- [ ] **Étant donné** des données valides (nom « Canon EOS R50 », catégorie « Caméra », numéro « CAM-001 », état « bon »), **quand** j'envoie `POST /equipements`, **alors** je reçois `201` avec l'équipement créé, son `id` et `disponibilite = "Disponible"`.
- [ ] **Étant donné** qu'un équipement avec le numéro « CAM-001 » existe déjà, **quand** j'en ajoute un autre avec « CAM-001 », **alors** je reçois `409` avec un message indiquant que le numéro d'inventaire est déjà utilisé, et rien n'est créé.
- [ ] **Étant donné** une requête sans nom ou sans numéro d'inventaire, **quand** je l'envoie, **alors** je reçois `400` avec le champ manquant indiqué.
- [ ] **Étant donné** un état hors de la liste permise (`neuf`, `bon`, `usage`, `hors_service`), **quand** je l'envoie, **alors** je reçois `400`.

### Estimation
3 points

### Priorité
Must

### Notes techniques
Contrainte `UNIQUE` en base sur `numero_inventaire`, en plus de la vérification dans le service. Catégories : texte libre normalisé (ex. « Caméra », « Portable », « Projecteur », « Trépied »).

## M2 — Consulter la liste des équipements et leur disponibilité
<!-- priorite: Must | points: 2 | milestone: Livrable 2 | domaine: inventaire -->

### User story
En tant qu'usager,
je veux consulter la liste des équipements et leur disponibilité
afin de savoir ce que je peux emprunter sans me déplacer au comptoir.

### Critères d'acceptation
- [ ] **Étant donné** un inventaire de 3 équipements dont 1 est en prêt, **quand** j'appelle `GET /equipements`, **alors** je reçois `200` et une liste de 3 éléments où exactement 1 a `disponibilite = "En prêt"` et 2 ont `"Disponible"`.
- [ ] **Étant donné** un inventaire vide, **quand** j'appelle `GET /equipements`, **alors** je reçois `200` et une liste vide `[]`.
- [ ] **Étant donné** un équipement existant d'`id` 7, **quand** j'appelle `GET /equipements/7`, **alors** je reçois `200` avec ses champs et sa disponibilité ; **et** `GET /equipements/9999` renvoie `404`.
- [ ] **Étant donné** un inventaire de 100 équipements, **quand** j'appelle `GET /equipements`, **alors** la réponse arrive en moins d'une seconde (mesuré dans un test d'intégration).

### Estimation
2 points

### Priorité
Must

### Notes techniques
La disponibilité est **calculée** (un prêt actif sans date de retour réelle) et non stockée, pour éviter toute incohérence.

## M3 — Enregistrer un emprunt
<!-- priorite: Must | points: 5 | milestone: Livrable 2 | domaine: prets -->

### User story
En tant que préposé·e,
je veux enregistrer un emprunt (équipement, emprunteur, date de retour prévue)
afin de savoir qui a quel équipement et jusqu'à quand.

### Critères d'acceptation
- [ ] **Étant donné** un équipement disponible et un emprunteur existant, **quand** j'envoie `POST /prets` avec une date de retour prévue future, **alors** je reçois `201` avec le prêt créé (date d'emprunt = aujourd'hui) **et** l'équipement apparaît « En prêt ».
- [ ] **Étant donné** un équipement déjà en prêt, **quand** j'essaie de l'emprunter une deuxième fois, **alors** je reçois `409` et aucun nouveau prêt n'est créé.
- [ ] **Étant donné** un équipement dont l'état est `hors_service`, **quand** j'essaie de l'emprunter, **alors** je reçois `409` avec un message explicite.
- [ ] **Étant donné** une date de retour prévue antérieure à aujourd'hui, **quand** j'envoie la requête, **alors** je reçois `400`.
- [ ] **Étant donné** un `equipement_id` inexistant, **quand** j'envoie la requête, **alors** je reçois `404`.

### Estimation
5 points

### Priorité
Must

### Notes techniques
Dépend de M1. Tant que M6 n'est pas fait, l'emprunteur peut être créé directement en base par une fixture ; M6 ajoutera la validation complète. La règle « pas de double prêt » vit dans `PretService` et est testée unitairement.

## M4 — Enregistrer le retour d'un équipement
<!-- priorite: Must | points: 3 | milestone: Livrable 2 | domaine: prets -->

### User story
En tant que préposé·e,
je veux enregistrer le retour d'un équipement
afin qu'il redevienne disponible pour les autres usagers.

### Critères d'acceptation
- [ ] **Étant donné** un prêt actif, **quand** j'appelle `POST /prets/{id}/retour`, **alors** je reçois `200`, le prêt a une `date_retour_reelle` égale à aujourd'hui **et** l'équipement redevient « Disponible ».
- [ ] **Étant donné** un prêt déjà retourné, **quand** j'appelle de nouveau `POST /prets/{id}/retour`, **alors** je reçois `409` et la date de retour initiale n'est pas modifiée.
- [ ] **Étant donné** un `id` de prêt inexistant, **quand** j'appelle le retour, **alors** je reçois `404`.
- [ ] **Étant donné** un équipement retourné, **quand** j'enregistre un nouvel emprunt de cet équipement, **alors** je reçois `201`.

### Estimation
3 points

### Priorité
Must

## M5 — Modifier ou retirer un équipement de l'inventaire
<!-- priorite: Must | points: 3 | milestone: Livrable 2 | domaine: inventaire -->

### User story
En tant que préposé·e,
je veux modifier ou retirer un équipement de l'inventaire
afin de garder l'inventaire à jour (changement d'état, équipement perdu ou mis au rebut).

### Critères d'acceptation
- [ ] **Étant donné** un équipement existant, **quand** j'envoie `PATCH /equipements/{id}` avec `etat = "usage"`, **alors** je reçois `200` et la consultation suivante affiche le nouvel état.
- [ ] **Étant donné** deux équipements « CAM-001 » et « CAM-002 », **quand** je modifie le numéro de « CAM-002 » en « CAM-001 », **alors** je reçois `409`.
- [ ] **Étant donné** un équipement disponible, **quand** j'appelle `DELETE /equipements/{id}`, **alors** je reçois `204` **et** un `GET /equipements/{id}` subséquent renvoie `404`.
- [ ] **Étant donné** un équipement en prêt, **quand** j'appelle `DELETE /equipements/{id}`, **alors** je reçois `409` et l'équipement est toujours présent.

### Estimation
3 points

### Priorité
Must

### Notes techniques
Retrait « logique » (champ `retire_le`) pour conserver l'historique des prêts (utile pour S2), mais l'équipement retiré n'est plus visible et renvoie `404`.

## M6 — Créer et consulter les emprunteurs
<!-- priorite: Must | points: 3 | milestone: Livrable 3 | domaine: emprunteurs -->

### User story
En tant que préposé·e,
je veux créer et consulter les emprunteurs
afin d'associer chaque prêt à une personne identifiée et joignable.

### Critères d'acceptation
- [ ] **Étant donné** un nom, un courriel valide et un matricule, **quand** j'envoie `POST /emprunteurs`, **alors** je reçois `201` avec l'emprunteur créé.
- [ ] **Étant donné** un emprunteur avec le matricule « 20251234 », **quand** j'en crée un autre avec le même matricule, **alors** je reçois `409`.
- [ ] **Étant donné** un courriel mal formé (ex. « abc »), **quand** j'envoie la requête, **alors** je reçois `400`.
- [ ] **Étant donné** des emprunteurs existants, **quand** j'appelle `GET /emprunteurs` puis `GET /emprunteurs/{id}`, **alors** je reçois `200` avec la liste, puis le détail ; un `id` inexistant renvoie `404`.
- [ ] **Étant donné** un `emprunteur_id` inexistant, **quand** j'envoie `POST /prets`, **alors** je reçois `400` avec le message « emprunteur inexistant » et aucun prêt n'est créé.

### Estimation
3 points

### Priorité
Must

## M7 — Filtrer les équipements par catégorie, état ou disponibilité
<!-- priorite: Must | points: 3 | milestone: Livrable 3 | domaine: inventaire -->

### User story
En tant qu'usager,
je veux filtrer les équipements par catégorie, état ou disponibilité
afin de trouver rapidement un équipement qui me convient.

### Critères d'acceptation
- [ ] **Étant donné** 2 caméras disponibles, 1 caméra en prêt et 1 projecteur disponible, **quand** j'appelle `GET /equipements?categorie=Caméra&disponibilite=disponible`, **alors** je reçois exactement les 2 caméras disponibles.
- [ ] **Étant donné** des équipements dans plusieurs états, **quand** je filtre `?etat=hors_service`, **alors** seuls les équipements hors service sont renvoyés.
- [ ] **Étant donné** un filtre qui ne correspond à rien (ex. `?categorie=Drone`), **quand** j'appelle l'endpoint, **alors** je reçois `200` et `[]`.
- [ ] **Étant donné** une valeur de filtre invalide (ex. `?disponibilite=peut-etre`), **quand** j'appelle l'endpoint, **alors** je reçois `400`.

### Estimation
3 points

### Priorité
Must

### Notes techniques
Filtre de catégorie insensible à la casse. Les filtres se combinent (ET logique).

## M8 — Voir la liste des prêts en retard
<!-- priorite: Must | points: 3 | milestone: Livrable 3 | domaine: prets -->

### User story
En tant que préposé·e,
je veux voir la liste des prêts en retard
afin de relancer les emprunteurs concernés.

### Critères d'acceptation
- [ ] **Étant donné** un prêt non retourné dont la date prévue était il y a 3 jours, **quand** j'appelle `GET /prets/en-retard`, **alors** il apparaît avec `jours_de_retard = 3`, le nom de l'équipement et celui de l'emprunteur.
- [ ] **Étant donné** un prêt non retourné dont la date prévue est **aujourd'hui**, **quand** j'appelle l'endpoint, **alors** il n'apparaît **pas** (il n'est en retard qu'à partir du lendemain).
- [ ] **Étant donné** un prêt en retard qui a été retourné, **quand** j'appelle l'endpoint, **alors** il n'apparaît plus.
- [ ] **Étant donné** plusieurs prêts en retard, **quand** j'appelle l'endpoint, **alors** ils sont triés du plus grand au plus petit nombre de jours de retard.

### Estimation
3 points

### Priorité
Must

### Notes techniques
Le calcul du retard reçoit la date du jour en paramètre (horloge injectée) pour être testable unitairement sans dépendre de la date réelle : candidat naturel pour le test avec double de test.

## S1 — Limiter le nombre de prêts simultanés par emprunteur
<!-- priorite: Should | points: 2 | milestone: Livrable 3 | domaine: prets -->

### User story
En tant que préposé·e,
je veux qu'un emprunteur ne puisse pas avoir plus de 3 prêts actifs en même temps
afin de répartir équitablement le matériel.

### Critères d'acceptation
- [ ] **Étant donné** un emprunteur avec 2 prêts actifs, **quand** j'enregistre un 3ᵉ emprunt, **alors** je reçois `201`.
- [ ] **Étant donné** un emprunteur avec 3 prêts actifs, **quand** j'enregistre un 4ᵉ emprunt, **alors** je reçois `409` avec un message indiquant la limite atteinte.
- [ ] **Étant donné** un emprunteur avec 3 prêts actifs dont 1 vient d'être retourné, **quand** j'enregistre un nouvel emprunt, **alors** je reçois `201`.
- [ ] **Étant donné** la variable d'environnement `MAX_PRETS_PAR_EMPRUNTEUR=5`, **quand** l'application démarre, **alors** la limite appliquée est 5.

### Estimation
2 points

### Priorité
Should

## S2 — Consulter l'historique des prêts d'un équipement
<!-- priorite: Should | points: 2 | milestone: Livrable 4 | domaine: prets -->

### User story
En tant que préposé·e,
je veux consulter l'historique des prêts d'un équipement
afin de savoir qui l'a utilisé en dernier en cas de bris ou de perte.

### Critères d'acceptation
- [ ] **Étant donné** un équipement prêté 3 fois, **quand** j'appelle `GET /equipements/{id}/prets`, **alors** je reçois `200` et 3 prêts triés du plus récent au plus ancien, avec emprunteur, dates d'emprunt, de retour prévue et de retour réelle.
- [ ] **Étant donné** un équipement jamais prêté, **quand** j'appelle l'endpoint, **alors** je reçois `200` et `[]`.
- [ ] **Étant donné** un `id` d'équipement inexistant, **quand** j'appelle l'endpoint, **alors** je reçois `404`.

### Estimation
2 points

### Priorité
Should

## S3 — Réserver un équipement à l'avance
<!-- priorite: Should | points: 8 | milestone: none | domaine: prets -->

### User story
En tant qu'usager,
je veux réserver un équipement pour une période future
afin d'être certain de l'avoir pour un tournage ou une présentation planifiés.

### Critères d'acceptation
- [ ] **Étant donné** un équipement sans prêt ni réservation du 10 au 12, **quand** j'envoie `POST /reservations` pour le 10 au 12, **alors** je reçois `201`.
- [ ] **Étant donné** une réservation existante du 10 au 12, **quand** j'en crée une autre du 11 au 13 pour le même équipement, **alors** je reçois `409` (chevauchement).
- [ ] **Étant donné** un prêt actif avec un retour prévu le 15, **quand** je réserve du 14 au 16, **alors** je reçois `409`.
- [ ] **Étant donné** une date de début postérieure à la date de fin, **quand** j'envoie la requête, **alors** je reçois `400`.

### Estimation
8 points

### Priorité
Should

### Notes techniques
Non planifiée pour la v1.0 (pas de milestone) : à reprendre seulement si les Must et S1/S2 sont terminées avant le livrable 4.

## S4 — Afficher un tableau de bord d'utilisation
<!-- priorite: Should | points: 5 | milestone: none | domaine: prets -->

### User story
En tant que responsable du service de prêt,
je veux un tableau de bord (équipements en prêt, en retard, taux d'utilisation par catégorie)
afin de justifier les achats et repérer le matériel peu utilisé.

### Critères d'acceptation
- [ ] **Étant donné** 10 équipements dont 4 en prêt et 1 en retard, **quand** j'appelle `GET /tableau-de-bord`, **alors** je reçois `en_pret = 4` et `en_retard = 1`.
- [ ] **Étant donné** 4 caméras dont 3 en prêt, **quand** j'appelle l'endpoint, **alors** le taux d'utilisation de la catégorie « Caméra » est `0.75`.
- [ ] **Étant donné** un inventaire vide, **quand** j'appelle l'endpoint, **alors** je reçois `200` avec des compteurs à 0 et aucune division par zéro.

### Estimation
5 points

### Priorité
Should

## C2 — Exporter l'inventaire en CSV
<!-- priorite: Could | points: 2 | milestone: none | domaine: inventaire -->

### User story
En tant que préposé·e,
je veux exporter l'inventaire en CSV
afin de l'ouvrir dans un tableur pour l'inventaire annuel.

### Critères d'acceptation
- [ ] **Étant donné** 5 équipements, **quand** j'appelle `GET /equipements/export.csv`, **alors** je reçois `200`, l'en-tête `Content-Type: text/csv` et 6 lignes (en-tête + 5).
- [ ] **Étant donné** un nom d'équipement contenant une virgule ou un accent, **quand** j'exporte, **alors** le fichier s'ouvre correctement dans un tableur (champ entre guillemets, encodage UTF-8).

### Estimation
2 points

### Priorité
Could

## C1 — Authentification avec rôles préposé·e et usager
<!-- priorite: Won't | points: 8 | milestone: none | domaine: securite -->

### User story
En tant que responsable du service,
je veux que seules les personnes authentifiées comme préposé·e·s puissent modifier l'inventaire et enregistrer des prêts
afin d'éviter les modifications non autorisées.

### Critères d'acceptation
- [ ] **Étant donné** une requête sans jeton, **quand** j'appelle `POST /prets`, **alors** je reçois `401`.
- [ ] **Étant donné** un jeton valide d'usager, **quand** j'appelle `POST /prets`, **alors** je reçois `403`, mais `GET /equipements` renvoie `200`.
- [ ] **Étant donné** un compte créé, **quand** j'inspecte la base, **alors** le mot de passe est stocké haché (bcrypt ou argon2), jamais en clair.

### Estimation
8 points

### Priorité
Won't

### Notes techniques
**Won't (pour cette version)** : exclue du périmètre v1.0 (voir cadrage, section 2). Conservée dans le backlog pour une version future.
