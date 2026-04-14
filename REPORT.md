
DÉPARTEMENT DE MATHÉMATIQUES, D’INFORMATIQUE ET DE GÉNIE

Gestion de projets informatiques 
(8INF847-MS)
Session hiver 2026

Rapport de projet de maîtrise en informatique

Par 
Nzuzi Mpingi Doudou 

Présenté à Ismail Khriss 

Date de remise : 19 avril 2026 à 23h59
Université du Québec à Rimouski
Table des matières
1. Introduction 
2. Fonctionnalités de l’application 
3. Modèle de données - Diagramme de classes 
4. System Architecture
5. Tests unitaires
6. Integration Tests
7. Tests fonctionnels
8. Quality Metrics
9. CI/CD Pipeline
10. Choix technologiques
11. Défis et solutions
12. Conclusions
13. Références 
1. Introduction
Ce rapport présente le processus complet d’assurance qualité des logiciels appliqué à Mpingi Market, une plateforme gratuite de petites annonces en ligne permettant aux utilisateurs de publier et de parcourir des annonces dans plus de 239 pays à travers le monde. L’application a été initialement développée en tant que projet de dernière année sous la supervision du professeur Mehdi Adda lors de la session d’hiver 2023.
L’objectif de cette extension de projet est d’appliquer les pratiques professionnelles d’intégration continue et de déploiement continu (CI/CD) à la plateforme Mpingi Market, notamment :
•	Tests automatisés à plusieurs niveaux (unité, intégration et fonctionnel)
•	Analyse de code statique avec des indicateurs de qualité mesurables
•	Un pipeline Jenkins CI/CD orchestré
•	Déploiement automatisé utilisant la gestion de configuration Ansible
•	Surveillance des performances applicatives en temps réel via Datadog
L’application est déployée dans un environnement de production accessible à www.mpingimarket.com.
2. Fonctionnalités de l’application
2.1 Présentation
Mpingi Market est une plateforme de petites annonces en ligne gratuite qui met en relation acheteurs et vendeurs dans 239 pays, 4120 régions et 47576 villes. La plateforme prend en charge la publication et la recherche de produits, services et offres d’emploi dans 12 catégories principales et 40 sous-catégories.
2.2 Rôles utilisateur
L’application prend en charge trois rôles d’utilisateur distincts :
Rôle	Description
Visiteur	Peut parcourir et rechercher des annonces sans compte
Utilisateur enregistré	Peut publier des annonces, enregistrer des favoris et contacter des vendeurs
Administrateur	Gère les catégories, les utilisateurs et le contenu de la plateforme

2.3 Liste des fonctionnalités
Authentification et gestion des utilisateurs
•	Inscription avec vérification de l’âge : seuls les utilisateurs âgés de 18 ans et plus peuvent s’inscrire, appliquée à la fois dans le formulaire (restriction du sélecteur de date HTML) et dans la logique métier (checkUserAge fonction d’utilité)
•	Connectez-vous / déconnectez-vous : en utilisant l’authentification par courriel et mot de passe Supabase
•	Gestion du profil : les utilisateurs peuvent mettre à jour leur nom, leur numéro de téléphone et leur emplacement
•	Gestion des sessions : les sessions persistantes sur les recharges de page à l’aide de jetons JWT Supabase.
Gestion des annonces
•	Publier une annonce un assistant guidé en 4 étapes :
1.	Sélection de catégorie (12 catégories principales + sous-catégorie facultative)
2.	Détails de l’annonce (titre, description, condition, prix avec sélecteur de devise, URL des photos)
3.	Emplacement (pays, région, ville) et choix du plan (gratuit ou avec option)
4.	Examen et soumission
•	Forfait gratuit : visibilité automatique sur 31 jours sans frais pour l’utilisateur
•	Plan principal : visibilité étendue et améliorée gérée par l’équipe d’administration
•	Cycle de vie du statut de l’annonce : Actif → Vendu / Expiré / Supprimé
•	Marquer comme vendu : les utilisateurs peuvent mettre à jour le statut de leur annonce depuis le tableau de bord
Découverte et recherche
•	Recherche en texte intégral : recherche de titres insensibles à la casse avec des résultats en temps réel sur la page d’annonces
•	Puces de filtrage des catégories : barre de filtres horizontaux déroulante affichant les 12 catégories
•	Panneau de filtrage avancé : état (neuf / d’occasion / reconditionné), prix minimum, prix maximum, filtre par pays
•	Options de tri : Premier le plus récent, Premier le plus ancien, Prix : Du plus bas au plus élevé, Prix : Du plus élevé au moins élevé
•	Pagination : 20 annonces par page avec la navigation Précédent / Suivant



Tableau de bord utilisateur
•	Onglet Mes annonces : afficher toutes les propres annonces avec des badges de statut, le nombre de vues et le temps écoulé depuis la publication
•	Onglet Éléments enregistrés : annonces enregistrées avec l’icône de cœur pour consultation ultérieure
•	Onglet Profil : modifier les informations du profil en ligne
Page de détail d’annonce
•	Galerie de photos : avec navigation par vignettes et carrousel d’images
•	Affichage des prix : avec indicateur de négociabilité
•	Badges de condition et de type de plan
•	Métadonnées de localisation et d’heure de publication
•	Voir le compteur : incrémenté à chaque visite de page
•	Enregistrer / Annuler l’enregistrement : (nécessite une authentification)
•	Partager : (copie l’URL dans le presse-papiers)
•	Grilles d’annonces associées de la même catégorie
•	Panneau Contacter le vendeur : révèle les coordonnées du vendeur uniquement aux utilisateurs authentifiés
Structure de classification
La plateforme met en œuvre la classification hiérarchique suivante :
Niveau	Nombre
Principales catégories	12
Sous-catégories	40
Types de produits (conception du système)	1401
Types de sous-produits	1177
Types d’articles	3461

Les 12 catégories principales sont : Véhicules, Électronique, Mode, Immobilier, Emplois, Services, Sports & Loisirs, Maison & Jardin, Animaux de compagnie, Alimentation & Agriculture, Affaires & Industrie et Communauté.

3. Modèle de données — Diagramme de classes
3.1 Description de la relation d’entité
3.2 Table Descriptions
Tableau	Lignes (environ)	Objet
auth.users	Géré par Supabase	Authentification principale — courriel, hachage de mot de passe, JWT
profiles
	1 par utilisateur	Profil utilisateur étendu et données de localisation
categories
	12	Principales catégories de classification
subcategorie
	40	Sous-classification au sein de chaque catégorie
Listings
	Illimité	Principales petites annonces
saved_listings
	Illimité	Tableau de jonction des favoris / favoris des utilisateurs

3.3 Sécurité au niveau de la ligne (RLS)
Chaque table a RLS activé avec le principe du moindre privilège :
Tableau	Politique	Condition
profiles	SELECT (authenticated)	true (any authenticated user can view)
profiles	INSERT (authenticated)	auth.uid() = id
profiles	UPDATE (authenticated)	auth.uid() = id
categories	SELECT (public	true
categories	INSERT/UPDATE	profiles.is_admin = true
subcategories	SELECT (public)	true
listings	SELECT (public)	status = 'active'
listings	SELECT (authenticated)	user_id = auth.uid()`
listings	INSERT | `user_id = auth.uid()`	user_id = auth.uid()`
listings	UPDATE/DELETE	user_id = auth.uid()`
saved_listings	SELECT/INSERT/DELETE	user_id = auth.uid()`

4. Architecture système
4.1 Pile technologique 
 
Cette architecture repose sur un modèle moderne en couches combinant frontend, backend cloud et CI/CD automatisé.
•	Client Layer (Navigateur) :
L’interface utilisateur est développée avec React 18, TypeScript et Tailwind CSS, servie via Vite. Elle communique avec le backend via HTTPS ou WebSocket, avec un suivi possible via Datadog RUM. 
•	Backend as a Service (Supabase) :
Le backend est externalisé avec Supabase, qui fournit : 
o	l’authentification via JWT 
o	une base de données PostgreSQL 
o	la sécurité via Row Level Security (RLS) 
o	des fonctionnalités temps réel (subscriptions) 
o	du stockage (prévu) 
•	Serveur de production (Nginx) :
Les fichiers statiques du frontend sont déployés sur un serveur Nginx, qui agit comme : 
o	serveur web 
o	reverse proxy 
o	gestionnaire de performance (gzip) et sécurité
Le monitoring est assuré par Datadog Agent. 
•	Infrastructure CI/CD :
Jenkins orchestre le pipeline : 
o	build et tests 
o	analyse qualité avec SonarQube 
o	déploiement automatisé via Ansible vers le serveur de production
C’est une architecture DevOps moderne, avec un frontend statique performant, un backend cloud (Supabase), et un pipeline CI/CD automatisé assurant qualité et déploiement continu.

4.2 Architecture Frontend
L’interface suit une architecture basée sur des composants avec une séparation claire des préoccupations :
src/
├── main.tsx              — Entry point, initialise le RUM de Datadog
├── App.tsx               — Composant racine, état du routeur côté client
├── index.css             — Base Tailwind + extensions de services publics
│
├── contexts/
│   └── AuthContext.tsx   — État d’authentification globale (contexte React)│
├── lib/
│   ├── supabase.ts       — Supabase singleton client
│   ├── types.ts          — Interfaces de domaine TypeScript
│   ├── database.types.ts — Types de schéma Supabase généré
│   ├── utils.ts          — Fonctions utilitaires pures (testées)
│   ├── categoryIcons.tsx — Mappage du registre d’icônes
│   └── monitoring.ts     — Intégration RUM de Datadog
│
├── components/
│   └── layout/
│       ├── Header.tsx    — Barre de navigation Sticky avec recherche
│       └── Footer.tsx    — Pied de page à l’échelle du site avec des liens
│   └── listings/
│       └── ListingCard.tsx — Composant de fiche d’inscription réutilisable
│
└── pages/
    ├── HomePage.tsx          — Héros, catégories, annonces récentes
    ├── AuthPage.tsx          — Connexion / Enregistrer le split-panel
    ├── ListingsPage.tsx      — Parcourir avec des filtres et une pagination
    ├── ListingDetailPage.tsx — Vue complète avec galerie
    ├── PostListingPage.tsx   — Assistant de création de publicités en 4 étapes
    └── DashboardPage.tsx     — gestion des comptes utilisateurs

4.3 Routing
L’application met en œuvre un modèle de routage d’application à page unique (SPA) utilisant l’état React plutôt que l’API d’historique des URL du navigateur. Le composant `App.tsx` maintient un objet `pageState `{ page, params }` et transmet une fonction `navigate(page, params)` vers le bas via des props. Cette conception évite le besoin d’une bibliothèque de routage externe.
Itinéraires disponibles :
Légende de la page	Description
home	Page de destination
auth	Connexion / S’inscrire
listings	Parcourir les annonces
listing-detail	Vue unique de l’annonce
post-listing	Créer une nouvelle annonce
dashboard	tableau de bord utilisateur
profile	Gestion des profils
saved	Annonces enregistrées

5. Tests unitaires
5.1 Cadre et configuration
Les tests unitaires sont écrits en utilisant Vitest, l’infrastructure de test native pour les projets Vite. Vitest fournit une API compatible avec Jest tout en étant nettement plus rapide grâce à la prise en charge du module ES natif et au pipeline de transformation de Vite.
Configuration  (vite.config.ts):

L’environnement `jsdom` simule un DOM de navigateur, ce qui permet de tester du code qui fait référence à `document` ou `window`. Le mocking global dans `src/tests/setup.ts` évince le client Supabase pour empêcher les appels réseau réels lors des tests unitaires.
5.2 Fichiers de test et couverture
Total : 54 tests unitaires sur 3 fichiers — tous réussis. 
5.2.1 utils.test.ts — 37 tests
Ce fichier teste les 7 fonctions d’utilitaires exportées depuis `src/lib/utils.ts`.
formatDistanceToNow(dateString : string) : string
Objectif : Convertit une chaîne de date ISO en une étiquette d’heure relative lisible par l’homme à afficher sur les fiches produites.
Cas d’essai	Type	Intrant	Attendu	Justification
Renvoie « just now » pour il y a 30 ans	Nominal	now - 30 ans	« just now »	Cas le plus courant pour les annonces récemment publiées
Renvoie les minutes pour il y a 5 m	Nominal	maintenant - 5 m	« il y a 5 m »	Limite standard des minutes
Renvoie les heures pour il y a 3 heures	Nominal	maintenant - 3 heures	« il y a 3 heures »	Condition aux limites des heures
Jours de retour pour 2d ago	Nominal	now - 2d	« 2d ago »	Condition limite du jour
Retours semaines pour il y a 2 semaines	Nominal	now - 2d	« 2d ago »	Condition limite du jour
Retours semaines pour il y a 2 semaines	Nominal	now - 14d	"2w ago"	Limite de la semaine
Retours en mois pour il y a 2 mois	Limite	maintenant - 61j	« Il y a 2 mois »	Limite supérieure de l’étiquette hebdomadaire (>30 jours)

5.2.2 listing-validation.test.ts - 13 tests
Ce fichier teste les fonctions de logique métiers utilisés dans les composants de détail de liste et de carte de liste.
Fonction	Essais	Justification
buildLocation	3	L’affichage de l’emplacement est au cœur de l’UX — testé pour les entrées complètes, partielles et vides
getListingDisplayPrice	4	L’affichage des prix avec des étiquettes Free/currency/negotiable est un élément critique de l’interface utilisateur
isListingExpired	3	Contrôle si une annonce reste visible testé à la limite au seuil passé/futur
filterActiveListings	3	Utilisé dans le tableau de bord pour séparer les annonces actives des inactives

5.2.3 category-icons.test.ts - 4 tests
Teste la fonction de registre d’icônes getCategoryIcon.
Cas d’essai	Justification
Renvoie le composant défini pour l’ensemble des 12 noms connus	S’assure qu’aucune icône manquante n’interrompt l’enregistrement dans l’interface utilisateur
Renvoie un remplacement de balise pour un nom inconnu	Défensif — empêche les plantages des futures catégories sans icône
Renvoie un remplacement de balise pour une chaîne vide	Gère le nom d’icône vide/nul avec élégance
Renvoie différents composants pour différents noms	Vérifie que la carte ne renvoie pas la même icône pour toutes les clés

5.3 Exécution des tests unitaires
Bash / Visual Studio terminal / pipeline build
•	Exécuter tous les tests unitaires : npm run test:unit
•	Avec le rapport de couverture : npm run test:coverage
Résultat : 54 tests, 3 fichiers de test - tous réussis ✓



6. Tests d’intégration
6.1 Approche
Les tests d’intégration vérifient l’interaction entre la couche applicative frontend et la base de données PostgreSQL Supabase en ligne. Contrairement aux tests unitaires, ils utilisent une connexion réelle à la base de données plutôt que des mocks, ce qui confirme que :
•	Les requêtes SQL sont correctes d’un point de vue syntaxique et sémantique
•	Les politiques RLS se comportent comme prévu
•	Les données classées (catégories, sous-catégories) sont présentes et correctement structurées
•	Les requêtes de jointure entre tables renvoient les données associées correctes.
Les tests sont sautés sous condition si la variable d’environnement VITE_SUPABASE_URL est absente ou un espace réservé, ce qui permet une exécution sécurisée dans des environnements sans accès à la base de données (par exemple, développement hors ligne).
6.2 Fichiers de test et requêtes
Total : 15 tests d’intégration sur 3 fichiers. 
6.2.1 categories.test.ts - 5 tests
Teste les tables de la base de données Catégories et Sous-catégories.
Test	Interaction des couches	Ce qu’il vérifie
Récupère les 12 catégories de têtes de série	Application → Supabase PostgreSQL	Intégrité des données de migration ; toutes les catégories sont présentes après le démarrage
Renvoie les catégories avec les champs requis	Application → Supabase PostgreSQL	Complétude du schéma — id, nom, pastille, icône, couleur - toutes présentes
Fetches category by slug	App → Supabase PostgreSQL	. eq('slug', 'electronics') renvoie l’enregistrement correct
Renvoie null pour slug inexistant	Application → Supabase PostgreSQL	maybeSingle() renvoie null gracieusement sans erreur
Récupère les sous-catégories pour l’électronique	Application (→ catégories) → Supabase (→ sous-catégories)	L’interaction à deux tables ; la jointure de clé étrangère fonctionne correctement
Justification : Les catégories sont la structure de navigation centrale de la plateforme. Toute régression dans les requêtes de catégorie casserait la page d’accueil, la barre de filtrage et l’assistant de publication simultanément. Ces tests servent de test de fumée pour l’ensemble de la hiérarchie de classification.
6.2.2 listings.test.ts - 5 tests
Teste la table de base de données Listings et son interaction avec la table Categories.
Test	Interaction des couches	Ce qu’il vérifie
Récupère les annonces actives	Application → Supabase RLS + PostgreSQL	RLS status = 'active’ le filtre est appliqué au niveau de la base de données
Récupère les annonces avec des jointures de catégories	Application → Supabase (annonces + catégories)	sélectionner('*, categories(name, slug)') clé étrangère joindre les travaux
Résultat vide pour la recherche sans correspondance	Application → Supabase ilike	ilike('title', '%xyznonexistent%')` renvoie un tableau vide
Filtre de fourchette de prix	Application → Les opérateurs Premium et Premium filtrent correctement la fourchette de prix	
Ordre décroissant created_at	Ordre décroissant created_at	Ordre le plus récent en premier est maintenu sur toute la pagination
Justification : Le tableau de listes est le tableau le plus critique du système. Ces tests vérifient à la fois l’exactitude des requêtes et l’application RLS. Le test de recherche est particulièrement important car il valide la fonctionnalité de recherche orientée utilisateur de bout en bout au niveau de la base de données.

6.2.3 profiles.test.ts - 5 tests
Teste la table Profiles et sa relation avec la table auth.user.
Test	Interaction des couches	Ce qu’il vérifie
Renvoie une valeur nulle pour un faux UUID	Application → Supabase PostgreSQL	maybeSingle() gestion de la valeur nulle pour les enregistrements manquants
La requête renvoie les colonnes attendues	Application → Supabase PostgreSQL	Le schéma correspond aux définitions de type ; aucune colonne manquante
Application RLS pour les utilisateurs anonymes	Application (pas d’authentification) → Supabase RLS	Les utilisateurs non authentifiés ne peuvent pas lire le profil des autres utilisateurs
is_admin par défaut à false	App → Supabase PostgreSQL	La contrainte par défaut est appliquée ; pas d’escalade par défaut
Connexion inter-tableaux via user_id	Application → Supabase (annonces → profils)	la clé étrangère user_id permet de récupérer le profil du vendeur
Justification : Les données du profil contiennent des informations personnelles. Tester RLS sur ce tableau garantit que la confidentialité des utilisateurs est appliquée au niveau de la base de données plutôt que de se fier uniquement aux vérifications au niveau de l’application.
6.3 Exécution de tests d’intégration
bash
Nécessite VITE_SUPABASE_URL et VITE_SUPABASE_ANON_KEY dans . env
npm run test:integration

7. Tests fonctionnels
7.1 Cadre et outils
Les tests fonctionnels sont mis en œuvre en utilisant Selenium WebDriver 4 avec Python pytest comme exécuteur de test. Selenium simule des interactions utilisateur complètes dans un navigateur réel (Google Chrome en mode sans tête), fournissant une validation de bout en bout des scénarios destinés aux utilisateurs.
Dépendances (`tests/functional/requirements.txt`) :
 
Configuration partagée (`conftest.py`) :
•	Chrome est lancé en mode headless dans CI (contrôlé par la variable d’environnement HEADLESS=true)
•	Fenêtre du navigateur : 1440×900 pixels
•	Attente implicite : 10 secondes pour toutes les références d’éléments
•	Navigateur à portée de session — une instance de navigateur partagée sur l’ensemble de la session de test pour les performances
•	reset_page autouse revient à l’URL de base avant chaque test


7.2 Suites de tests
7.2.1 test_home_page.py - 5 scénarios
Fonctionnalité à l’essai : Rendu de la page d’accueil et fonctionnalité de recherche
Scénario	Type	Parcours utilisateur simulé	Assertions
La page d’accueil se charge avec une section principale	Nominal	Ouvrir l’application dans le navigateur	<h1>l’élément est présent et non vide
La barre de recherche accepte la saisie	Nominal	Type "iPhone" dans le champ de recherche	La saisie conserve la valeur tapée
La recherche accède à la page des annonces	Nominale	Type "voiture" et appuyez sur Entrée	La page des annonces se charge ; la page des résultats contient le contenu pertinent
La recherche vide ne plante pas	Erreur/Limite	Soumettre une recherche avec un champ vide	Le corps de page est toujours présent ; aucune page d’erreur JavaScript
Les cartes de catégories sont affichées	Nominal	Charger la page d’accueil et attendre pour les catégories	Au moins 3 des 6 noms de catégorie connus apparaissent dans la source de la page

7.2.2 test_auth_flow.py - 4 scénarios
Fonctionnalité en cours de test : Authentification utilisateur (connexion et enregistrement)
Scénario	Type	Parcours utilisateur simulé	Assertions
La page d’authentification affiche le formulaire de connexion	Nominal	Cliquez sur « Se connecter » dans la navigation	Les saisies par courriel, la saisie du mot de passe et le bouton d’envoi sont tous visibles
Les identifiants non valides indiquent une erreur	Erreur	Saisir un faux courriel + un mauvais mot de passe, soumettre	Un message d’erreur contenant « invalid », « incorrect » ou « wrong » s’affiche
L’onglet Inscription est accessible	Nominal	Cliquez sur l’onglet "Inscrire" dans la page d’accueil	Un champ de saisie de nom apparaît sur le formulaire d’inscription
Le formulaire vide ne crée pas de compte	Erreur/Limite	Cliquez sur soumettre sans remplir les champs	L’utilisateur reste sur la page d’autorisation (la saisie par courriel est toujours visible)

7.2.3 `test_listings_browse.py - 6 scénarios
Fonctionnalité en cours de test : Fonctionnalités de navigation, filtrage et tri des annonces
Scénario	Type	Parcours utilisateur simulé	Assertions
La page des annonces charge les contrôles de filtrage	Nominal	Accéder à la page de navigation	Le menu déroulant du tri est visible
La liste déroulante de tri contient 4 options attendues	Nominal	Inspecter l’élément de sélection du tri	Les 4 options de tri présentes dans la sélection
Le changement de tri ne s’interrompt pas dans la page	Nominal	Sélectionnez « Prix : de bas à haut »	Le corps de la page est toujours présent ; aucune erreur dans le titre
Catégorie présente	Nominale	Charger la page de listings, inspecter le contenu	Au moins 2 des 4 noms de catégories connus visibles
Le panneau de filtrage s’ouvre au clic	Nominal	Cliquez sur le bouton "Filtres"	Les étiquettes du filtre Condition/Prix/Pays apparaissent
La recherche absurde n’affiche aucun résultat	Erreur/Limite	Recherche pour "xyznonexistentitem99999"	La page indique un message "aucune annonce" ou "introuvable"

7.3 Exécution de tests fonctionnels
bash
Installer les dépendances Python : pip install -r tests/functional/requirements.txt
Démarrer l’application (requis avant d’exécuter les tests fonctionnels) : npm run preview ou npm run dev
python -m pytest tests/functional/
Exécuter tous les tests fonctionnels : cd tests/functional
pytest . -v --html=../../reports/functional-tests.html --self-contained-html

Exécuter en mode non-headless pour le débogage visuel : HEADLESS=false pytest . -v

8. Métriques de qualité
8.1 Static Analysis - SonarQube
Le projet utilise SonarQube pour l’analyse de code statique, configuré via sonar-project.properties. L’analyse produit les catégories de métriques suivantes :
Complexité cyclomatique
La complexité cyclomatique mesure le nombre de chemins linéairement indépendants à travers une fonction. Des valeurs plus faibles indiquent un code plus simple et plus facile à maintenir.
Fonction	Complexité estimée	Notes
formatDistanceToNow
	7	6 branches conditionnelles
validateListingPrice
	5	4 conditions de validation
checkUserAge
	4	Date math + comparaison
Composant “ListingsPage”	8	Construction du filtre + construction de la requête
Composant PageAuth	7	Succursales de validation de formulaire
Seuil de porte qualité	≤ 10	SonarQube applique par fonction

Duplication de code
Le projet est structuré avec des composants à responsabilité unique et des fonctions d’utilitaires partagées dans src/lib/utils.ts et src/lib/categoryIcons.tsx, ce qui minimise la duplication de code. Le seuil de qualité est fixé à 5 % de duplication maximale.

Couverture de test
Métrique	Objectif	Objectif
Couverture du relevé	60%	≥ 60% (seuil configuré)
Couverture de la ligne	60%	≥ 60%
Couverture par les succursales	50%	≥ 50%
Couverture des fonctions	60%	≥ 60%

La couverture est générée par le fournisseur v8 de Vitest et rapportée au format LCOV, consommée par SonarQube via la propriété sonar.javascript.lcov.reportPaths=coverage/lcov.info.
Vulnérabilités détectées
SonarQube analyse les 10 principales vulnérabilités d’OWASP. Le projet suit des pratiques de codage sécurisé :
Pratique	Mise en œuvre
Pas de secret dans le code	Toutes les clés supabase dans .env (gitignored)
Prévention des injections SQL	Toutes les requêtes via le client paramétré Supabase
Prévention XSS	La fuite de JSX de React empêche l’injection
Authentification	Jetons JWT gérés par Supabase Auth
Autorisation	Politiques RLS appliquées au niveau de la base de données

Quality Gate - Le pipeline échoue automatiquement si :
•	La couverture des tests descend en dessous de 60 %
•	La duplication de code dépasse 5 %
•	Tout nouveau problème de gravité CRITIQUE ou BLOQUANT est introduit
•	Toute fonction dépasse la complexité cyclomatique de 10
8.2 Analyse ESLint
ESLint est configuré avec des règles basées sur TypeScript et des hooks React linting. Il s’exécute comme l’étape Lint dans le pipeline Jenkins, produisant un rapport JSON archivé en tant qu’artefact de construction.
bash
npm run lint


9. Pipeline CI/CD
Présentation des encours 9.1
Le pipeline Jenkins est défini dans Jenkinsfile à la racine du projet et orchestre automatiquement toutes les portes de qualité et les étapes de déploiement à chaque poussée de code.
┌──────────┐   ┌─────────┐   ┌────────┐   ┌──────────┐
│ Checkout │──▶│ Install│──▶│  Lint  │──▶│  Build   │
└──────────┘   └─────────┘   └────────┘   └──────────┘
                                                  │
                                                  ▼
┌─────────────────────────────────────────────────────────┐
│          PARALLEL TEST PHASES                           │
│  ┌──────────────┐     ┌──────────────────────────────┐  │
│  │  Unit Tests  │     │    Integration Tests         │  │
│  └──────────────┘     └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                                                  │
                                                  ▼
             ┌──────────────────────────────────────┐
             │           Coverage Report            │
             └───────────────────┬──────────────────┘
                                 ▼
             ┌──────────────────────────────────────┐
             │     SonarQube Analysis + Quality Gate│
             │  (FAILS PIPELINE if gate not met)    │
             └───────────────────┬──────────────────┘
                                 │
                    (main & staging branches only)
                                 ▼
             ┌──────────────────────────────────────┐
             │      Functional Tests (Selenium)     │
             └───────────────────┬──────────────────┘
                                 │
                         (main branch only)
                                 ▼
             ┌──────────────────────────────────────┐
             │     Deploy (Ansible Playbook)        │
             └──────────────────────────────────────┘

9.2 Description étape par étape
Étape	Élément de déclenchement	Outil	Articles
1. Checkout	À chaque poussée	Jenkins SCM	GitHub 
2. Installation	À chaque poussée	npm ci	node_modules/
3. Lint	Chaque poussée	ESLint	eslint-report.json
4. Build	À chaque poussée	Vite	dist/
5. Tests unitaires	À chaque poussée	Vitest	rapports/tests unitaires.xml
6. Tests d’intégration	À chaque poussée	Vitest + Supabase	rapports/tests d’intégration.xml
7. Couverture	À chaque poussée	Vitest v8	couverage/ (HTML + LCOV)
8. SonarQube	À chaque poussée	sonar-scanner	tableau de bord SonarQube
9. Quality Gate	À chaque poussée	SonarQube webhook	Annule la canalisation en cas de défaillance
10. Tests fonctionnels	main, staging	pytest + selenium	rapports/functional-tests.html
11. Déployer	main uniquement	Ansible	Déploiement en production

9.3 Configuration de Jenkins
Plugins Jenkins requis :
•	Pipeline
•	NodeJS Plugin (pour l’outil NodeJS-20)
•	HTML Publisher Plugin (pour la couverture et les rapports de test fonctionnels)
•	JUnit Plugin (pour le rapport des résultats de test XML)
•	SonarQube Scanner Plugin
•	Plug-in Ansible
•	Plug-in SSH Credentials

Identifiants Jenkins requis :
ID d’accréditation	Type	Utilisation
SUPABASE_URL	Texte secret	Injecté sous la forme VITE_SUPABASE_URL  lors des tests de construction et d’intégration
SUPABASE_ANON_KEY	Texte secret	Injecté sous la forme VITE_SUPABASE_ANON_KEY
SONAR_TOKEN	Texte secret	Authentification SonarQube
DEPLOY_SSH_KEY	Nom d’utilisateur SSH + Clé privée	Accès Ansible SSH au serveur de production

9.4 Actions post-contrat
•	Toujours : Espace de travail propre (`cleanWs()`)
•	Success : Journal de la console avec le numéro de build
•	Échec : Notification par courriel à `nzuzi.mpingi@email.com` avec l’URL de build

10. Choix technologiques
10.1 Interface : React + TypeScript + Vite
React 18 a été choisi comme infrastructure d’interface utilisateur pour les raisons suivantes :
•	Le modèle de composant applique naturellement la séparation des préoccupations, en s’alignant sur l’architecture MVC étudiée dans le cours.
•	Sa réconciliation DOM virtuelle fournit des rendus de données performants pour une grille de listage riche en données.
•	Le grand écosystème (icônes Lucide React, contexte React pour l’état d’authentification) réduit la complexité.
•	L’intégration TypeScript offre une sécurité des types à la compilation, réduisant les erreurs d’exécution en production.
Vite 5 remplace Créer une application React pour les raisons suivantes :
•	Remplacement quasi instantané du module à chaud (HMR) pendant le développement
•	La prise en charge du module ES natif produit des paquets plus petits et plus faciles à déformer.
•	Vitest est intégré nativement, en utilisant le même fichier de configuration
TypeScript a été choisi plutôt que JavaScript simple pour les raisons suivantes :
•	Les types de schéma de base de données (`database.types.ts`) offrent une sécurité de type de bout en bout, de la requête au composant
•	La complétion automatique de l’IDE réduit les erreurs de développement
•	Les définitions d’interface servent de documentation pour le modèle de données.
10.2 Backend : Supabase
Supabase a été choisi en tant que backend-as-a-service pour les raisons suivantes :
•	Il fournit une base de données PostgreSQL complète avec une sécurité au niveau de la ligne, éliminant ainsi le besoin de maintenir une API PHP personnalisée
•	Le client `@supabase/supabase-js` remplace les appels REST/fetch personnalisés par un générateur de requêtes à type sécurisé.
•	L’authentification intégrée (JWT, gestion de session, déclencheurs) réduit les risques liés à la mise en œuvre de la sécurité
•	Les capacités en temps réel (pas encore utilisées) activent de futures fonctionnalités de chat/notification sans modifications du backend

10.3 Style : CSS Tailwind
Tailwind CSS a été choisi plutôt que Bootstrap ou un CSS personnalisé pour les raisons suivantes :
•	L’approche Utility-first élimine les conflits de noms CSS dans les grandes arborescences de composants
•	L’intégration PurgeCSS (intégrée à Tailwind v3) supprime les styles inutilisés, produisant un ensemble CSS gzippé de 5–6 Ko
•	Les utilitaires de points de rupture réactifs (md:, lg:) simplifient la disposition de grille réactive.
10.4 Tests : Vitest + Selenium
Vitest a été choisi pour les tests unitaires et d’intégration parce que :
•	Il partage la configuration Vite, ce qui signifie qu’aucune configuration séparée Babel/Jest n’est nécessaire
•	L’API compatible Jest signifie que les connaissances existantes de Jest s’appliquent directement
•	Le fournisseur de couverture v8 produit le format LCOV requis par SonarQube sans plugins supplémentaires
•	La vitesse d’exécution est généralement 3–5× plus rapide que Jest sur des suites de tests équivalentes
Selenium WebDriver a été choisi pour les tests fonctionnels parce que :
•	C’est l’outil standard de l’industrie explicitement mentionné dans les exigences du cours
•	Python pytest fournit une structure de test paramétrique propre et un bon rapport HTML
•	Webdriver-manager automatise la correspondance des versions de ChromeDriver, éliminant ainsi la gestion manuelle des pilotes
10.5 CI/CD : Jenkins
Jenkins a été choisi car il est explicitement requis par le cahier des charges et les offres du cours :
•	Syntaxe de pipeline déclarative (Jenkinsfile) qui se trouve dans le dépôt à côté du code
•	Contrôle d’étape à grain fin avec des étapes conditionnelles de branche (`lorsque { branch 'main' }`)
•	Écosystème de plugins enrichis pour JUnit, rapports HTML, SonarQube, Ansible et identifiants SSH

10.6 Static Analysis : SonarQube
SonarQube a été choisi parce que :
•	Il fournit les quatre métriques requises : complexité cyclomatique, duplication de code, couverture de test et détection de vulnérabilité sur une seule plateforme
•	Les rapports de couverture LCOV générés par Vitest sont directement consommés par SonarQube
•	Les portes de qualité peuvent bloquer le pipeline automatiquement sans script personnalisé
•	L’édition communautaire est gratuite et auto-stable sur le même serveur que Jenkins
10.7 Gestion de la configuration : Ansible
Ansible a été choisi plutôt que Chef ou Puppet parce que :
•	Il est sans agent — aucun démon ne doit être installé sur le serveur de production
•	Les playbooks sont écrits en YAML, ce qui est lisible et auto-documentant
•	Le module synchronize s’intègre en natif avec l’espace de travail Jenkins pour le transfert d’artefacts
•	Les tâches idempuissantes garantissent que les déploiements répétés produisent le même résultat sans effets secondaires
10.8 Surveillance : Datadog
Datadog a été choisi plutôt que New Relic ou Dynatrace parce que :
•	Le SDK RUM (Real User Monitoring) peut être chargé via un CDN sans ajouter de dépendance npm
•	Le niveau d’essai gratuit est suffisant à des fins de démonstration académique
•	Les actions personnalisées (`addAction`) permettent de suivre des événements spécifiques à l’activité (vues de listage, recherches, enregistrements) non disponibles dans les outils APM génériques
•	Le Datadog Agent 7 pour le côté serveur est provisionné directement par le playbook Ansible, créant ainsi une pile de surveillance unifiée

11. Défis et solutions
Défi 1 : Stratégie de simulation pour les tests unitaires
Problème : Les tests unitaires qui importent depuis src/lib/supabase.ts tenteraient des appels réseau réels vers l’API Supabase lors de l’exécution du test, ce qui entraînerait l’échec des tests dans des environnements hors ligne et produirait des résultats non déterministes.
Solution : Une simulation globale a été mise en œuvre dans src/tests/setup.ts en utilisant la fonction vi.mock() de Vitest. L’ensemble . /lib/supabase` est remplacé par un module de retour de chaîne qui simule l’interface du générateur de requête Supabase (from().select().eq().order().limit()). Cela permet aux tests unitaires de s’exécuter hors ligne tandis que les tests d’intégration utilisent le client réel en important @supabase/supabase-js directement (en contournant la simulation).
Défi 2 : Types d’icônes Lucide React
Problème : Les icônes Lucide React sont exportées en tant qu’objets React.ForwardRefExoticComponent (créés par `React.forwardRef`), pas des fonctions JavaScript simples. Un test initial a utilisé typeof Icon == 'function' qui a échoué car `typeof forwardRef(...)` renvoie object’ dans le moteur V8.
Solution : L’assertion de test a été mise à jour pour utiliser expect(Icon). toBeDefined() et expect(Icon).not.toBeNull(), qui vérifient que l’icône est enregistrée sans faire d’hypothèses sur sa représentation interne. Le fichier `categoryIcons.tsx` a également été corrigé pour utiliser un import de type LucideIcon (`import type { LucideIcon }`) plutôt que de l’importer en tant que valeur.
Défi 3 : Dépendance de construction RUM de Datadog
Problème : Une mise en œuvre initiale de monitoring.ts utilisait des instructions dynamiques import(@datadog/browser-rum). Comme ce package n’a pas été installé en tant que dépendance npm, Vite a tenté de le résoudre lors de la compilation et a lancé une erreur module-not-found, interrompant ainsi la compilation de production.
Solution : Le module de surveillance a été réécrit pour utiliser une approche d’injection de script CDN (document.createElement('script')), qui charge le SDK Datadog RUM de manière asynchrone à l’exécution. Ce modèle est l’approche recommandée par Datadog pour la surveillance côté navigateur et n’a pas de dépendance au moment de la construction sur les packages npm.


Challenge 4 : Exécution du test d’intégration conditionnelle
Problème : Les tests d’intégration qui interrogent une base de données Supabase en ligne ne peuvent pas s’exécuter dans des environnements sans identifiants valides (par exemple, un nouveau clone de développeur ou un environnement CI sans secrets configurés).
Solution : Chaque fichier de test d’intégration vérifie en haut si les variables d’environnement sont présentes et non plagiées :ts const skipIfNoEnv = ! supabaseUrl || ! supabaseKey || supabaseUrl.includes('your_');
Chaque fonction de test commence par ` si (skipIfNoEnv) renvoie `, ce qui la fait passer silencieusement plutôt que d’échouer. Dans le pipeline Jenkins, les véritables identifiants sont injectés via `withCredentials()`, donc tous les tests s’exécutent pleinement dans CI.
Défi 5 : Routage SPA et configuration Nginx
Problème : Un SPA React utilise l’acheminement côté client. Lorsqu’un utilisateur actualise le navigateur sur une route comme /listings, Nginx tente de servir un fichier à ce chemin, qui n’existe pas sur le disque, en renvoyant une erreur 404.
Solution : La configuration Nginx dans le playbook Ansible utilise la directive try_files : nginx
emplacement / { try_files $ure $uri/ /index.html; }
Cela demande à Nginx de servir `index.html` pour tout chemin qui ne correspond pas à un fichier physique, ce qui permet à React de gérer l’itinéraire du côté client.
Défi 6 : Robustesse du test de sélénium
Problème : Les tests de sélénium qui reposent sur des étiquettes de texte exactes ou des noms de classe CSS sont fragiles — toute refonte de l’interface utilisateur peut casser des dizaines de tests. Un sélecteur comme button.bg-blue-600 se casserait si la couleur d’arrière-plan du bouton changeait.
Solution : Les tests utilisent des localisateurs basés sur le contenu et des sélecteurs sémantiques dans la mesure du possible :
input[espace réservé*='Recherche'] — correspond à un attribut d’espace réservé partiel
 Par.XPATH avec contains(text(), Signer l’entrée) — correspond à l’étiquette visible
Par le sélecteur.CSS_SELECTOR, « input[type=‘courriel’] »—correspond au type d’entrée sémantique WebDriverWait avec expected_condition` est utilisé à la place des appels time.sleep() fixes afin de réduire les défauts, avec le repli `time.sleep() uniquement là où un chargement de contenu dynamique est attendu.


12. Conclusion
Ce projet démontre avec succès l’ensemble des pratiques professionnelles d’assurance qualité appliquées à une application web dans le monde réel. La plateforme de petites annonces Mpingi Market a été équipée de :
•	54 tests unitaires sur 3 fichiers couvrant 7 fonctions utilitaires avec les cas nominal, limite et erreur
•	15 tests d’intégration validant les interactions de la base de données en direct sur 3 tables avec application RLS
•	15 scénarios de test fonctionnel sur 3 suites de tests Selenium couvrant l’authentification, la recherche et la navigation dans les annonces
•	SonarQube analyse statique avec contrôles qualité renforcés sur la couverture, la complexité, la duplication et les vulnérabilités
•	Pipeline CI/CD Jenkins avec 11 étapes, du paiement au déploiement automatisé en production
•	Guide de déploiement Ansible automatisant la configuration Nginx, le transfert d’artefacts et la configuration de l’agent Datadog
•	Intégration de Datadog RUM pour le suivi des performances en temps réel, les Core Web Vitals et le suivi personnalisé des événements professionnels.
La combinaison de ces pratiques transforme la base de code d’un projet à développeur unique en un système prêt pour la production capable d’une livraison continue, avec des filets de sécurité automatisés empêchant les régressions à chaque couche de la pile applicative.


13. References 
1.	https://www.selenium.dev/selenium-ide/
2.	https://www.youtube.com/watch?v=soIpt3c7tVE&t=432s
3.	https://www.youtube.com/watch?v=m4KpTvEz3vg&ab_channel=Simplilearn
4.	https://en.wikipedia.org/wiki/Selenium_(software)?fbclid=IwAR1Nsjt-VM5_0z1TiRvLtuw2lW0x77gNJVR-2Tp-CDml0xdNKnmXYoWTn_U
5.	https://www.selenium.dev/selenium-ide/
6.	https://www.dev2qa.com/how-to-install-selenium-ide-in-firefox-and-google-chrome/
7.	www.mpingimarket.com
8.	https://www.browserstack.com/guide/selenium-webdriver-tutorial
9.	https://www.selenium.dev/downloads/
10.	https://www.youtube.com/watch?v=OWaJ7Iu4dHQ&ab_channel=Simplilearn
11.	
12.	https://www.guru99.com/select-option-dropdown-selenium-webdriver.html
13.	https://www.youtube.com/watch?v=eGaImwD8fPQ
14.	

