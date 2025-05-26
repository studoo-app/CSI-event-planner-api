![separe](https://github.com/studoo-app/.github/blob/main/profile/studoo-banner-logo.png)
# TP ApiPlatform - API de gestion d'événements
[![Version](https://img.shields.io/badge/Version-2025-blue)]()

**Objectif** : Construire une API REST complète et sécurisée avec ApiPlatform

---

## **Etape 1 : Première entité et API automatique**

### Objectifs
- Créer une entité Doctrine Event
- Générer automatiquement une API REST

### Travail à réaliser
**Projet : API de gestion d'événements**

Créez l'entité Event avec les propriétés suivantes :
- **id** : identifiant auto-généré
- **title** : titre de l'événement (string, 255 caractères max)
- **description** : description détaillée (text)
- **startDate** : date de début (DateTime)
- **endDate** : date de fin (DateTime)
- **maxParticipants** : nombre maximum de participants (integer)
- **location** : lieu de l'événement (string, 255 caractères max)

**Tâches :**
1. Annotez l'entité avec `#[ApiResource]`
2. Générez et exécutez la migration de base de données
3. Testez les endpoints générés automatiquement via l'interface Swagger
4. Vérifiez que toutes les opérations CRUD fonctionnent

---

## **Etape 2 : Authentification JWT**

### Objectifs
- Sécuriser l'API avec l'authentification JWT
- Créer le système d'utilisateurs
- Configurer les rôles de base

### Travail à réaliser

**Tâches :**
1. Installez et configurez LexikJWTAuthenticationBundle

2. Créez l'entité User implémentant UserInterface avec :
    - **email** : identifiant unique
    - **password** : mot de passe hashé
    - **roles** : tableau des rôles
    - **firstName** et **lastName** : informations personnelles

3. Configurez les rôles suivants :
    - **ROLE_USER** : utilisateur de base
    - **ROLE_ORGANIZER** : peut créer des événements
    - **ROLE_ADMIN** : accès complet

4. Configurez la sécurité dans `security.yaml` :
    - Firewall pour l'API avec JWT
    - Endpoint de connexion `/api/login_check`
    - Hiérarchie des rôles

5. Créez des fixtures pour avoir des utilisateurs de test

6. Testez l'authentification via l'endpoint de login

---

## **Etape 3 : Personnalisation des opérations**

### Objectifs
- Personnaliser les opérations disponibles
- Configurer les méthodes HTTP et sécurité basique

### Travail à réaliser

**Tâches :**
1. Configurez les opérations ApiResource pour Event avec sécurité :
    - GetCollection (lecture de la liste - public)
    - Get (lecture d'un élément - public)
    - Post (création - ROLE_ORGANIZER minimum)
    - Put (modification - propriétaire ou admin)
    - Delete (suppression - ROLE_ADMIN uniquement)

2. Modifiez l'entité Event pour intégrer les Users :
    - Ajoutez la propriété `organizer` (relation ManyToOne vers User)
    - Ajoutez la propriété `participants` (relation ManyToMany vers User)
    - L'organisateur est l'utilisateur qui crée l'événement
    - Les participants sont les utilisateurs inscrits à l'événement

3. Modifiez l'entité User pour les relations inverses :
    - Ajoutez `organizedEvents` (OneToMany vers Event, événements organisés)
    - Ajoutez `participatedEvents` (ManyToMany vers Event, événements auxquels l'utilisateur participe)

4. Appliquez les règles de sécurité sur les opérations

5. Mettez à jour les propriétés calculées :
    - `currentParticipants` devient le nombre réel de participants dans la collection

---

## **Etape 4 : Sérialisation et groupes**

### Objectifs
- Contrôler les données exposées dans l'API
- Utiliser les groupes de sérialisation

### Travail à réaliser

**Tâches :**
1. Configurez les contextes de normalisation/dénormalisation pour Event :
    - Groupe `event:read` pour la lecture
    - Groupe `event:write` pour l'écriture

2. Annotez les propriétés avec les groupes appropriés :
    - L'ID ne doit être visible qu'en lecture
    - Ajoutez une propriété `createdAt` visible uniquement en lecture
    - La propriété `currentParticipants` doit compter les vrais participants
    - L'organisateur (User) doit être visible avec les informations de base
    - Les participants doivent être visibles en lecture (liste des utilisateurs inscrits)
    - Selon les permissions, certains utilisateurs peuvent voir plus d'informations

3. Appliquez la logique à l'entité User :
    - Groupe `user:read` pour la lecture (sans le mot de passe)
    - Groupe `user:write` pour l'écriture
    - `organizedEvents` visible en lecture
    - `participatedEvents` visible en lecture pour le propriétaire du compte
    - Le mot de passe ne doit jamais être visible en lecture

4. Gérez les références circulaires entre User et Event

5. Testez les différences entre les données d'entrée et de sortie avec les participants

---

## **Etape 5 : Validation et contraintes**

### Objectifs
- Ajouter la validation des données
- Personnaliser les messages d'erreur

### Travail à réaliser

**Tâches :**
1. Ajoutez les contraintes de validation sur Event :
    - **title** : obligatoire, longueur min 3, max 255
    - **description** : obligatoire, longueur minimum 10 caractères
    - **startDate** : doit être dans le futur
    - **endDate** : doit être postérieure à startDate
    - **maxParticipants** : positif, maximum 1000
    - **location** : obligatoire

2. Personnalisez les messages d'erreur en français

3. Ajoutez des contraintes appropriées sur User :
    - **email** : format email valide, unique
    - **password** : longueur minimum 6 caractères
    - **firstName** et **lastName** : obligatoires

4. Testez les validations via l'interface Swagger (avec authentification)

5. Vérifiez que les contraintes de sécurité sont respectées

---

## **Etape 6 : Filtres et recherche**

### Objectifs
- Implémenter la recherche et le filtrage
- Ajouter le tri des résultats

### Travail à réaliser

**Tâches :**
1. Configurez les filtres de recherche sur Event :
    - Recherche partielle sur le titre
    - Recherche exacte sur le lieu
    - Recherche partielle sur le nom de l'organisateur (User.firstName ou User.lastName)

2. Ajoutez les filtres suivants :
    - Filtre par date (startDate)
    - Filtre par plage de participants (maxParticipants)
    - Tri par date de début, titre, nombre de participants

3. Testez tous les filtres via l'interface (en tant qu'utilisateur authentifié)

4. **Etape bonus :** Ajoutez un filtre par statut d'événement (à venir, en cours, terminé)

5. Vérifiez que les filtres respectent les permissions (certains utilisateurs peuvent voir plus d'informations)

---

## **Etape 7 : Opérations personnalisées**

### Objectifs
- Créer des endpoints personnalisés
- Implémenter une logique métier spécifique

### Travail à réaliser

**Tâches :**
1. Créez les opérations métier suivantes avec sécurité appropriée :
    - **POST /events/{id}/register** : inscription de l'utilisateur connecté à un événement (ROLE_USER)
    - **DELETE /events/{id}/unregister** : désinscription de l'utilisateur connecté (ROLE_USER)
    - **GET /events/stats** : statistiques globales des événements (public)
    - **GET /events/my-events** : événements organisés par l'utilisateur connecté (ROLE_ORGANIZER)
    - **GET /events/my-participations** : événements auxquels l'utilisateur connecté participe (ROLE_USER)

2. Implémentez les contrôleurs correspondants :
    - **Inscription** : Ajoutez l'utilisateur connecté à la collection des participants
        - Vérifiez les places disponibles
        - Empêchez la double inscription
        - Vérifiez que l'événement n'est pas terminé
    - **Désinscription** : Retirez l'utilisateur de la collection des participants
    - **Statistiques** : Calculez avec les vrais participants
    - **Mes événements** : Filtrez par organisateur = utilisateur connecté
    - **Mes participations** : Filtrez les événements où l'utilisateur participe

3. Intégrez la gestion complète des utilisateurs :
    - Récupérez l'utilisateur connecté via le token JWT
    - Gérez les relations ManyToMany pour les participations
    - Vérifiez les permissions selon les rôles et propriétés des objets
    - Empêchez qu'un organisateur s'inscrive à son propre événement

4. Testez ces nouveaux endpoints avec différents utilisateurs et rôles

5. Vérifiez les cas limites :
    - Inscription à un événement complet
    - Inscription à un événement passé
    - Désinscription d'un événement non inscrit

---

## **Etape 8 : Tests et finalisation**

### Objectifs
- Tester l'API complète
- Documenter les fonctionnalités

### Travail à réaliser

**Tâches :**
1. Écrivez des tests d'intégration pour :
    - L'authentification JWT (déjà implémentée)
    - Les opérations CRUD sur les événements avec authentification
    - Les inscriptions aux événements avec gestion des utilisateurs
    - Les validations et contraintes de sécurité
    - Les permissions par rôle

2. Optimisez les requêtes (lazy loading, requêtes N+1)

3. Documentez votre API :
    - Listez tous les endpoints disponibles avec leurs permissions
    - Décrivez les rôles et leur hiérarchie
    - Fournissez des exemples de requêtes avec authentification
    - Documentez le processus d'inscription/connexion

4. Préparez une démonstration complète de votre API sécurisée

---

## **Récapitulatif des fonctionnalités à implémenter**

### Fonctionnalités obligatoires
- ✅ API REST complète avec CRUD
- ✅ Authentification JWT dès le début du développement
- ✅ Système d'utilisateurs avec rôles (USER, ORGANIZER, ADMIN)
- ✅ Relations entre entités (Event/User) avec double fonction :
    - **Organisation** : Un User peut organiser des Events (OneToMany)
    - **Participation** : Un User peut participer à des Events (ManyToMany)
- ✅ Validation et sérialisation avec groupes
- ✅ Filtres, recherche et tri
- ✅ Opérations métier sécurisées :
    - Inscription/désinscription réelle aux événements
    - Gestion des organisateurs et participants distincts
    - Endpoints personnalisés par rôle utilisateur
- ✅ Tests d'intégration avec authentification et gestion des participations

### Fonctionnalités bonus
- Gestion des statuts d'événements avec participants
- Tableau de bord utilisateur personnalisé (mes événements organisés/mes participations)
- Notifications par email lors des inscriptions/désinscriptions
- Upload d'images pour les événements (avec permissions par organisateur)
- Système de commentaires/avis sur les événements par les participants
- Export des listes de participants en CSV (selon les rôles)
- Rate limiting par utilisateur
- Gestion des listes d'attente quand l'événement est complet
- Système de badges/récompenses pour les participants actifs

---

## **Ressources techniques**

### Architecture des relations
Le système implémente une double relation entre User et Event :
- **Organisation** (OneToMany) : Un utilisateur peut organiser plusieurs événements
- **Participation** (ManyToMany) : Un utilisateur peut participer à plusieurs événements, et un événement peut avoir plusieurs participants

### Documentation utile
- Documentation officielle ApiPlatform
- Documentation Symfony
- Spécification OpenAPI/Swagger