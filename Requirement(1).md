# Document de Spécifications Fonctionnelles

## 1. Aperçu de l'application

**Nom :** SiteTrack — Suivi du temps de séjour des camions

**Description :** Application PWA responsive permettant de digitaliser et tracer le parcours complet d'un camion sur un site industriel, depuis son entrée jusqu'à sa sortie, avec calcul automatique des temps à chaque étape, tableau de bord analytique et gestion des rôles utilisateurs.

---

## 2. Utilisateurs et scénarios d'utilisation

### 2.1 Profils utilisateurs

| Rôle système | Désignation | Responsabilités principales |
|---|---|---|
| AGENT_ENTREE | Agent d'entrée | Créer un passage, enregistrer l'entrée (T1), consulter les camions présents |
| OPERATEUR | Opérateur carrousel | Voir camions en attente, enregistrer arrivée carrousel (T2), démarrer (T3) et terminer (T4) le remplissage |
| AGENT_SORTIE | Agent de sortie | Rechercher un camion, enregistrer la sortie (T5) |
| RESPONSABLE | Responsable | Consulter tous les camions, statistiques, historique, exporter les données |
| ADMIN | Administrateur | Toutes les fonctions Responsable + gestion des utilisateurs, configuration des seuils d'alerte |

### 2.2 Scénarios principaux

- Un camion arrive sur site : l'agent d'entrée crée le passage et enregistre T1.
- Le camion rejoint la file d'attente du carrousel : l'opérateur enregistre T2.
- Le remplissage démarre puis se termine : l'opérateur enregistre T3 puis T4.
- Le camion quitte le site : l'agent de sortie enregistre T5.
- Le responsable consulte les KPIs et exporte les données en fin de journée.

---

## 3. Structure des pages et fonctionnalités

### 3.1 Arborescence

```
Application SiteTrack
├── Authentification
│   ├── Connexion
│   └── Déconnexion
├── Tableau de bord (RESPONSABLE, ADMIN)
├── Vue site en temps réel (tous rôles)
│   └── Fiche camion / Action étape suivante
├── Création de passage (AGENT_ENTREE)
├── File d'attente carrousel (OPERATEUR)
├── Enregistrement sortie (AGENT_SORTIE)
├── Historique des passages (RESPONSABLE, ADMIN)
├── Export des données (RESPONSABLE, ADMIN)
├── Alertes (tous rôles concernés)
└── Gestion des utilisateurs (ADMIN)
```

### 3.2 Page Authentification

- Connexion par identifiant et mot de passe sécurisé.
- Après connexion, redirection vers la vue adaptée au rôle de l'utilisateur.
- Déconnexion automatique après une période d'inactivité configurable.
- Journalisation de chaque connexion et déconnexion (audit log).

### 3.3 Vue site en temps réel

Accessible à tous les rôles. Affiche la liste des camions actuellement sur site.

**Informations affichées par camion :**
- Immatriculation
- Transporteur
- Chauffeur
- Type de camion
- Type d'opération
- Statut coloré :
  - EN ATTENTE : orange
  - AU CARROUSEL : bleu
  - REMPLISSAGE EN COURS : bleu
  - REMPLISSAGE TERMINÉ : vert
- Temps écoulé depuis l'étape en cours
- Bouton d'action pour l'étape suivante (visible uniquement pour le rôle habilité)

**Principe UX :** Une étape = une action. Les boutons d'action sont larges, adaptés à une utilisation sur tablette et smartphone.

### 3.4 Création de passage (AGENT_ENTREE)

Formulaire de création d'un nouveau passage avec saisie des données d'identification du camion :

- Immatriculation (obligatoire)
- Transporteur (obligatoire)
- Chauffeur (obligatoire)
- Type de camion (obligatoire)
- Type d'opération (obligatoire)
- Quantité prévue (optionnel)

À la validation, T1 (entrée sur le site) est enregistré automatiquement avec l'horodatage courant et l'identifiant de l'agent. Le statut du camion passe à EN ATTENTE.

### 3.5 File d'attente carrousel (OPERATEUR)

Affiche la liste des camions en statut EN ATTENTE ou AU CARROUSEL ou REMPLISSAGE EN COURS.

**Actions disponibles selon le statut du camion :**
- Camion EN ATTENTE → bouton « Arrivée carrousel » : enregistre T2, statut passe à AU CARROUSEL.
- Camion AU CARROUSEL → bouton « Démarrer remplissage » : enregistre T3, statut passe à REMPLISSAGE EN COURS.
- Camion REMPLISSAGE EN COURS → bouton « Terminer remplissage » : enregistre T4, statut passe à REMPLISSAGE TERMINÉ.

### 3.6 Enregistrement de la sortie (AGENT_SORTIE)

- Recherche du camion par immatriculation ou depuis la liste des camions en statut REMPLISSAGE TERMINÉ.
- Bouton « Enregistrer la sortie » : enregistre T5, statut passe à SORTI.
- Le passage est clôturé et les temps calculés sont finalisés.

### 3.7 Tableau de bord (RESPONSABLE, ADMIN)

**KPIs affichés :**
- Nombre de camions actuellement sur site
- Nombre de camions en attente
- Nombre de camions en remplissage
- Nombre de camions sortis aujourd'hui
- Temps moyen d'attente (T2 − T1)
- Temps moyen de remplissage (T4 − T3)
- Temps moyen total sur site (T5 − T1)

**Graphiques :**
- Nombre de camions par jour
- Temps moyen d'attente par jour
- Temps moyen de séjour par jour
- Temps moyen de remplissage par jour
- Évolution hebdomadaire et mensuelle

### 3.8 Historique des passages (RESPONSABLE, ADMIN)

**Filtres de recherche :**
- Immatriculation
- Transporteur
- Chauffeur
- Date (plage)
- Statut

**Colonnes du tableau :**
Camion | Transporteur | Entrée (T1) | Carrousel (T2) | Début remplissage (T3) | Fin remplissage (T4) | Sortie (T5) | Temps total

**Détail d'un passage :** accès à la fiche complète avec tous les horodatages, les temps calculés, et le journal de traçabilité (qui a effectué chaque action et à quelle heure).

### 3.9 Export des données (RESPONSABLE, ADMIN)

- Export de la liste filtrée ou de l'historique complet au format CSV, Excel ou PDF.
- Le PDF est destiné aux rapports.

### 3.10 Alertes

- Alerte visuelle (dans l'interface) si un camion dépasse le seuil d'attente configurable (ex. : 1h30 par défaut).
- Alerte si un camion semble bloqué à une étape sans progression.
- Notification lorsqu'un camion passe au statut REMPLISSAGE TERMINÉ (camion prêt pour la sortie).

### 3.11 Gestion des utilisateurs (ADMIN)

- Créer, modifier, désactiver un compte utilisateur.
- Attribuer un rôle parmi : ADMIN, RESPONSABLE, AGENT_ENTREE, OPERATEUR, AGENT_SORTIE.
- Réinitialiser le mot de passe d'un utilisateur.

---

## 4. Données enregistrées par passage

### 4.1 Identification

| Champ | Obligatoire |
|---|---|
| Immatriculation | Oui |
| Transporteur | Oui |
| Chauffeur | Oui |
| Type de camion | Oui |
| Type d'opération | Oui |
| Quantité prévue | Non |

### 4.2 Horodatages automatiques

| Identifiant | Événement |
|---|---|
| T1 | Entrée sur le site |
| T2 | Arrivée au carrousel |
| T3 | Début du remplissage |
| T4 | Fin du remplissage |
| T5 | Sortie du site |

### 4.3 Calculs automatiques

| Indicateur | Formule |
|---|---|
| Temps avant carrousel | T2 − T1 |
| Attente au carrousel | T3 − T2 |
| Temps de remplissage | T4 − T3 |
| Temps après remplissage | T5 − T4 |
| Temps total sur site | T5 − T1 |

---

## 5. Règles métier et logique

### 5.1 Workflow des statuts

```
EN ATTENTE → AU CARROUSEL → REMPLISSAGE EN COURS → REMPLISSAGE TERMINÉ → SORTI
```

Chaque transition est irréversible et enregistrée avec l'horodatage et l'identifiant de l'utilisateur ayant effectué l'action (traçabilité complète).

### 5.2 Contrôles de cohérence

- Il est impossible d'enregistrer la sortie (T5) si le statut n'est pas REMPLISSAGE TERMINÉ.
- Il est impossible de terminer le remplissage (T4) si le remplissage n'a pas été démarré (T3 absent).
- Il est impossible de créer deux passages actifs avec la même immatriculation simultanément (un camion ne peut pas entrer deux fois sans être sorti).
- Les horodatages doivent être strictement croissants : T1 < T2 < T3 < T4 < T5.

### 5.3 Seuils d'alerte

- Le seuil de temps d'attente excessif est configurable par l'ADMIN (valeur par défaut : 1h30).
- Une alerte est déclenchée automatiquement lorsqu'un camion dépasse ce seuil sans progression d'étape.

### 5.4 Traçabilité

- Chaque changement de statut est conservé avec : timestamp, identifiant de l'utilisateur, rôle, action effectuée.
- Le journal d'audit est accessible en lecture seule depuis la fiche de passage.

### 5.5 Sécurité

- Authentification obligatoire pour accéder à toute fonctionnalité.
- Chaque rôle n'accède qu'aux fonctionnalités qui lui sont attribuées.
- Les mots de passe sont stockés de manière sécurisée.
- Toutes les actions sont journalisées (audit log).
- Déconnexion automatique après inactivité.

---

## 6. Anomalies et cas limites

| Situation | Comportement attendu |
|---|---|
| Tentative de sortie sans remplissage terminé | Blocage avec message d'erreur explicite |
| Tentative de terminer remplissage sans l'avoir démarré | Blocage avec message d'erreur explicite |
| Double enregistrement d'entrée pour le même camion | Blocage, passage déjà actif détecté |
| Camion en attente dépassant le seuil configuré | Alerte visuelle dans l'interface pour les rôles concernés |
| Utilisateur inactif | Déconnexion automatique |
| Accès à une fonctionnalité non autorisée | Redirection ou message d'accès refusé |

---

## 7. Critères de recette

1. Un agent d'entrée se connecte avec ses identifiants et accède à la vue site.
2. Il crée un nouveau passage en saisissant les informations du camion ; T1 est enregistré automatiquement et le camion apparaît en statut EN ATTENTE (orange).
3. L'opérateur carrousel voit le camion dans sa file, clique sur « Arrivée carrousel » ; T2 est enregistré, statut passe à AU CARROUSEL (bleu).
4. L'opérateur clique sur « Démarrer remplissage » ; T3 est enregistré, statut passe à REMPLISSAGE EN COURS (bleu).
5. L'opérateur clique sur « Terminer remplissage » ; T4 est enregistré, statut passe à REMPLISSAGE TERMINÉ (vert).
6. L'agent de sortie recherche le camion, clique sur « Enregistrer la sortie » ; T5 est enregistré, le passage est clôturé.
7. Le responsable consulte le tableau de bord et vérifie que les KPIs et les temps calculés sont corrects.
8. Le responsable exporte l'historique au format CSV et vérifie que le fichier contient les données du passage.
9. L'administrateur configure le seuil d'alerte et vérifie qu'une alerte s'affiche pour un camion dépassant ce seuil.
10. L'administrateur crée un nouvel utilisateur avec le rôle AGENT_ENTREE et vérifie que cet utilisateur peut se connecter et accéder uniquement aux fonctionnalités de son rôle.

---

## 8. Fonctionnalités hors périmètre (version 1)

- Scan QR code associé au passage pour retrouver automatiquement le camion.
- Application mobile native (iOS / Android).
- Intégration avec des systèmes tiers (ERP, TMS).
- Notifications push sur appareil mobile.
- Gestion multi-sites.
- Personnalisation avancée des rapports PDF.
