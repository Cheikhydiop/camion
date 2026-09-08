# SiteTrack
## Document de Spécifications Fonctionnelles
**Suivi du temps de séjour des camions — Version 2**

### Sommaire

#### 1. Aperçu de l'application
**Nom :** SiteTrack — Suivi du temps de séjour des camions
**Description :** Application PWA responsive permettant de digitaliser et tracer le parcours complet d'un camion sur un site industriel, depuis son entrée jusqu'à sa sortie, avec calcul automatique des temps à chaque étape, tableau de bord analytique et gestion des rôles utilisateurs. L'interface est conçue pour être utilisable par des opérateurs de terrain non-alphabètes ou peu à l'aise avec la lecture, grâce à une ergonomie basée sur les couleurs, les pictogrammes et le retour visuel/sonore (voir section 3.12).

#### 2. Utilisateurs et scénarios d'utilisation
**2.1 Profils utilisateurs**
Mise à jour : l'application compte 4 profils. Le rôle « Responsable » a été supprimé ; ses fonctions d'analyse, d'historique et d'export sont intégrées au rôle ADMIN, qui reste strictement en lecture, analyse et gestion (aucune action terrain).

| Rôle système | Désignation | Responsabilités principales |
|---|---|---|
| **ADMIN** | Administrateur | Tableau de bord, KPIs, historique complet, export des données (CSV/Excel/PDF), gestion des utilisateurs, configuration des seuils d'alerte, consultation du journal d'audit. Aucune action sur les étapes T1 à T5. |
| **CONTROLLEUR_ENTREE** | Contrôleur d'entrée camion | Créer un passage, enregistrer l'entrée (T1), consulter les camions présents |
| **CONTROLLEUR_CHARGEMENT** | Contrôleur de chargement | Voir camions en attente, enregistrer arrivée carrousel (T2), démarrer (T3) et terminer (T4) le remplissage |
| **CONTROLLEUR_SORTIE** | Contrôleur de sortie | Rechercher un camion, enregistrer la sortie (T5) |

*Le rôle ADMIN n'intervient jamais sur les actions terrain : il ne peut ni créer de passage, ni enregistrer T1 à T5. Toute action sur le cycle de vie d'un camion reste exclusivement du ressort des trois rôles contrôleurs.*

**2.2 Scénarios principaux**
- Un camion arrive sur site : le contrôleur d'entrée crée le passage et enregistre T1.
- Le camion rejoint la file d'attente du carrousel : le contrôleur de chargement enregistre T2.
- Le remplissage démarre puis se termine : le contrôleur de chargement enregistre T3 puis T4.
- Le camion quitte le site : le contrôleur de sortie enregistre T5.
- L'administrateur consulte les KPIs et exporte les données en fin de journée.

#### 3. Structure des pages et fonctionnalités
**3.1 Arborescence**
- Authentification (Connexion / Déconnexion)
- Tableau de bord (ADMIN)
- Vue site en temps réel (tous rôles)
- Création de passage (CONTROLLEUR_ENTREE)
- File d'attente carrousel (CONTROLLEUR_CHARGEMENT)
- Enregistrement sortie (CONTROLLEUR_SORTIE)
- Historique des passages (ADMIN)
- Export des données (ADMIN)
- Alertes (tous rôles concernés)
- Gestion des utilisateurs (ADMIN)

**3.2 Page Authentification**
- Connexion par identifiant et mot de passe sécurisé.
- Après connexion, redirection vers l'unique écran adapté au rôle de l'utilisateur — pas de menu de navigation à parcourir.
- Déconnexion automatique après une période d'inactivité configurable.
- Journalisation de chaque connexion et déconnexion (audit log).

**3.3 Vue site en temps réel**
Accessible à tous les rôles. Affiche la liste des camions actuellement sur site, mise à jour en direct via WebSocket (sans rafraîchissement manuel).
Informations affichées par camion :
- Immatriculation (affichage en gros caractères)
- Transporteur (avec logo si disponible)
- Chauffeur
- Type de camion (pictogramme)
- Type d'opération (pictogramme)
- Statut coloré avec icône associée (voir tableau 3.12)
- Temps écoulé depuis l'étape en cours
- Bouton d'action pour l'étape suivante (visible uniquement pour le rôle habilité)
*Principe UX : une étape = une action. Les boutons d'action sont larges, dominés par un pictogramme, adaptés à une utilisation tactile sur tablette et smartphone, y compris avec des gants.*

**3.4 Création de passage (CONTROLLEUR_ENTREE)**
Formulaire de création d'un nouveau passage avec saisie des données d'identification du camion :
- Immatriculation : Obligatoire (Clavier alphanumérique simplifié, gros caractères, affichage de contrôle)
- Transporteur : Obligatoire (Liste déroulante avec logos/photos des transporteurs habituels)
- Chauffeur : Obligatoire (Liste des chauffeurs récurrents avec photo, ou saisie libre en secours)
- Type de camion : Obligatoire (Pictogrammes cliquables)
- Type d'opération : Obligatoire (Pictogrammes cliquables)
- Quantité prévue : Non (Pavé numérique)

À la validation, T1 (entrée sur le site) est enregistré automatiquement avec l'horodatage courant et l'identifiant de l'agent. Le statut passe à EN ATTENTE.

**3.5 File d'attente carrousel (CONTROLLEUR_CHARGEMENT)**
Affiche la liste des camions en statut EN ATTENTE, AU CARROUSEL ou REMPLISSAGE EN COURS, triée par ordre chronologique.
Actions disponibles :
- Camion EN ATTENTE → bouton « Arrivée carrousel » : enregistre T2, statut passe à AU CARROUSEL.
- Camion AU CARROUSEL → bouton « Démarrer remplissage » : enregistre T3, statut passe à REMPLISSAGE EN COURS.
- Camion REMPLISSAGE EN COURS → bouton « Terminer remplissage » : enregistre T4, statut passe à REMPLISSAGE TERMINÉ.

**3.6 Enregistrement de la sortie (CONTROLLEUR_SORTIE)**
- Liste des camions en statut REMPLISSAGE TERMINÉ, triée par heure de fin de remplissage.
- Recherche rapide par fragment d'immatriculation en complément.
- Bouton « Enregistrer la sortie » : enregistre T5, statut passe à SORTI.

**3.7 Tableau de bord (ADMIN)**
KPIs affichés :
- Nombre de camions actuellement sur site, en attente, en remplissage, sortis aujourd'hui.
- Temps moyens (attente T2-T1, remplissage T4-T3, total T5-T1).
Graphiques : Nombre et temps par jour, évolution hebo/mensuelle.

**3.8 Historique des passages (ADMIN)**
Filtres : Immatriculation, Transporteur, Chauffeur, Date, Statut.
Détail d'un passage : Accès à la fiche complète avec horodatages et traçabilité.

**3.9 Export des données (ADMIN)**
Export CSV, Excel ou PDF.

**3.10 Alertes**
- Alerte visuelle (temps d'attente excessif, configurable ex: 2h30).
- Alerte de blocage sans progression.
- Notification in-app pour camion REMPLISSAGE TERMINÉ.

**3.11 Gestion des utilisateurs (ADMIN)**
Créer, modifier, désactiver des comptes. Attribution des rôles.

**3.12 Accessibilité et ergonomie terrain**
Cette section formalise une contrainte transversale applicable à toutes les pages de l'application : l'interface doit pouvoir être utilisée sans lecture, par des opérateurs non-alphabètes ou peu à l'aise avec l'écrit, dans un contexte de terrain (bruit, gants, urgence).

**3.12.1 Statuts : couleur + icône + libellé**
- EN ATTENTE : Orange / Sablier ou horloge
- AU CARROUSEL : Bleu / Camion + flèche
- REMPLISSAGE EN COURS : Bleu (animé) / Pompe animée ou pulsante
- REMPLISSAGE TERMINÉ : Vert / Coche
- SORTI : Gris / Porte ou flèche sortante

**3.12.2 Boutons d'action**
- Un bouton = une icône dominante (environ 80% de la surface) + un mot court, jamais une phrase.
- Une seule couleur d'action possible par écran (ex. toujours vert = « avancer à l'étape suivante »).
- Zone tactile large, utilisable avec des gants.

**3.12.3 Retour utilisateur (feedback)**
- Confirmation plein écran (1 à 2 secondes) après chaque action validée.
- Son de confirmation distinct et son d'erreur.
- Vibration courte sur tablette/smartphone.
- Messages d'erreur sous forme de pictogramme.

**3.12.4 Saisie sans clavier complet**
Listes déroulantes avec logos, icônes cliquables, clavier alphanumérique restreint.

**3.12.5 Navigation**
Un seul écran utile par rôle après connexion.

**3.13 Recherche et sélection rapide d'un camion**
**3.13.1 Liste pré-filtrée et triée** : Le mode principal, aucune saisie nécessaire.
**3.13.2 Recherche par fragment** : Mode de secours. Recherche instantanée, résultats en cartes cliquables.
**3.13.3 Écran de confirmation avant action** : Fiche géante pour validation visuelle (évite les erreurs).
*Recommandation V1.5 : OCR par caméra pour scan de plaque.*

#### 4. Données enregistrées par passage
**4.1 Identification** : Immatriculation, Transporteur, Chauffeur, Type camion, Type opération (obligatoires), Quantité (optionnel).
**4.2 Horodatages automatiques** : T1 (Entrée), T2 (Carrousel), T3 (Début Remplissage), T4 (Fin Remplissage), T5 (Sortie).
**4.3 Calculs automatiques** : Déduits des horodatages.

#### 5. Règles métier et logique
**5.1 Workflow** : EN ATTENTE → AU CARROUSEL → REMPLISSAGE EN COURS → REMPLISSAGE TERMINÉ → SORTI.
**5.2 Contrôles** : Impossible de sortir sans remplissage terminé. Pas de T4 sans T3. Horodatages croissants. Pas de double entrée sans sortie.
**5.3 Correction et annulation** : (Option 2 recommandée) L'ADMIN dispose d'un droit de correction exceptionnel, hautement tracé.
**5.4 Seuils d'alerte** : Configurable, défaut 1h30.
**5.5 Traçabilité & 5.6 Sécurité** : Audit log, authentification, déconnexion auto.

#### 6. Anomalies et cas limites
Toutes erreurs bloquées avec pictogrammes. Déconnexion automatique des utilisateurs inactifs.

#### 7. Critères de recette
Parcours testables de bout-en-bout pour chaque rôle, validant la transition des statuts, les calculs, l'absence de saisies textuelles inutiles et la génération d'alertes. L'objectif ultime étant l'opérabilité par un utilisateur non-alphabète.

#### 8. Fonctionnalités hors périmètre (V1)
- Reconnaissance de plaque par caméra (OCR) — recommandée V1.5.
- Scan QR code.
- Application mobile native.
- Intégration ERP/TMS.
- Push notifications mobiles.
- Personnalisation PDF avancée.
