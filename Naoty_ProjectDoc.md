# Naoty — Documentation Projet Complète

> Application d'annotation harmonique automatique pour partitions en Tonic Sol-fa

---

## Table des matières

1. [Vue d'ensemble du projet](#1-vue-densemble-du-projet)
2. [Contexte et problématique](#2-contexte-et-problématique)
3. [Glossaire technique musical](#3-glossaire-technique-musical)
4. [Architecture générale et feuille de route](#4-architecture-générale-et-feuille-de-route)
5. [Phase 1 — Application Web](#5-phase-1--application-web)
6. [Phase 2 — Application Mobile avec Backend en ligne](#6-phase-2--application-mobile-avec-backend-en-ligne)
7. [Phase 3 — Application Mobile entièrement locale](#7-phase-3--application-mobile-entièrement-locale)
8. [Défis transversaux](#8-défis-transversaux)
9. [Évolutions futures possibles](#9-évolutions-futures-possibles)

---

## 1. Vue d'ensemble du projet

**Naoty** est une application qui prend en entrée un fichier PDF contenant une partition musicale écrite en notation **Tonic Sol-fa** (do, ré, mi… sous forme de lettres : d, r, m, f, s, l, t), et qui en retourne une version annotée avec les **symboles harmoniques** (accords en chiffres romains : I, IV, V, ii, vi, etc.) superposés directement sur la partition d'origine.

L'application s'adresse principalement aux :

- Chanteurs et chefs de chœur travaillant avec des partitions sol-fa (courant dans les traditions protestantes malgaches, africaines, et anglo-saxonnes)
- Étudiants en harmonie souhaitant analyser visuellement une pièce
- Arrangeurs ayant besoin d'une analyse harmonique rapide d'une partition existante

Le projet se déroule en **trois phases successives**, chacune livrant un produit utilisable, et chacune servant de base à la suivante.

---

## 2. Contexte et problématique

### 2.1 Le format Tonic Sol-fa

Le Tonic Sol-fa est un système de notation musicale relatif, développé au XIXe siècle, très utilisé dans les traditions chorales. Au lieu de notes sur une portée (Do, Ré, Mi…), il utilise des lettres :

| Lettre | Degré | Note en Do majeur |
| ------ | ----- | ----------------- |
| `d`    | 1er   | Do                |
| `r`    | 2e    | Ré                |
| `m`    | 3e    | Mi                |
| `f`    | 4e    | Fa                |
| `s`    | 5e    | Sol               |
| `l`    | 6e    | La                |
| `t`    | 7e    | Si                |

Des modificateurs indiquent l'octave (virgule `,` pour grave, apostrophe `'` pour aigu) et des altérations (ex. `fi` = fa dièse, `ta` = si bémol).

Les partitions SATB (Soprano, Alto, Ténor, Basse) sont organisées en **systèmes**, chaque système contenant 4 lignes de notes superposées, séparées par des lignes de paroles.

### 2.2 La problématique

Les partitions Sol-fa ne comportent généralement **aucune annotation harmonique**. Pour analyser la progression d'accords d'un cantique ou d'un chœur, un musicien doit actuellement :

1. Lire chaque temps colonne par colonne sur les 4 voix
2. Identifier mentalement l'accord formé
3. Annoter manuellement la partition

Ce processus est lent, propice aux erreurs, et inaccessible aux musiciens moins expérimentés. Naoty automatise entièrement cette démarche.

### 2.3 Pourquoi les PDF sol-fa sont exploitables sans OCR

Les PDF de partitions sol-fa produits par des logiciels courants (doPDF, LibreOffice, Word) contiennent le texte sous forme **sélectionnable** — c'est-à-dire que les notes et les paroles sont des caractères de texte réels avec des **coordonnées x/y précises** dans l'espace de la page. Cela permet d'extraire directement la position et la valeur de chaque note sans passer par la reconnaissance optique de caractères (OCR), ce qui est à la fois plus rapide et beaucoup plus fiable.

---

## 3. Glossaire technique musical

| Terme                 | Définition                                                                          |
| --------------------- | ----------------------------------------------------------------------------------- | --- |
| **Degré**             | Position d'une note dans la gamme (1 = tonique, 5 = dominante…)                     |
| **Accord**            | Superposition de plusieurs notes jouées simultanément                               |
| **Chiffre romain**    | Notation d'un accord selon son degré (I = tonique, V = dominante…)                  |
| **Système**           | Bloc de 4 lignes de voix (S/A/T/B) dans la partition                                |
| **Temps**             | Unité rythmique ; les notes au même temps dans les 4 voix forment un accord         |
| **Mesure**            | Groupe de temps délimité par des barres verticales `                                | `   |
| **Tonalité**          | Gamme de référence (ex. Fa majeur) qui détermine le nom absolu des degrés           |
| **Accord de passage** | Accord instable entre deux accords stables, souvent non harmoniquement significatif |
| **V⁷**                | Accord de septième de dominante (s + t + r + f), très courant avant I               |
| **vii°**              | Accord de septième diminuée sur le 7e degré                                         |
| **It⁶ / Fr⁶ / All⁶**  | Accords de sixte augmentée (chromatiques, rares)                                    |

---

## 4. Architecture générale et feuille de route

```
Phase 1 ──────────────────────────────────────────────────────────────
  Application Web (React)
  Backend Python (FastAPI) hébergé sur serveur
  Accessible via navigateur, aucune installation

Phase 2 ──────────────────────────────────────────────────────────────
  Application Mobile (Flutter — iOS & Android)
  Même backend Python Phase 1, hébergé en ligne
  L'app est autonome côté UI, le traitement reste sur serveur

Phase 3 ──────────────────────────────────────────────────────────────
  Application Mobile (Flutter)
  Traitement 100% local, aucun serveur
  Fonctionne hors-ligne
```

Chaque phase est indépendamment utilisable. Le code de l'algorithme d'analyse harmonique est partagé entre phases (extrait en module séparé).

---

## 5. Phase 1 — Application Web

### 5.1 Objectif

Livrer une version fonctionnelle accessible depuis n'importe quel navigateur, sans installation. L'utilisateur télécharge un PDF sol-fa, l'application retourne un PDF annoté avec les accords.

### 5.2 Stack technique

| Composant            | Technologie            | Rôle                                |
| -------------------- | ---------------------- | ----------------------------------- |
| Frontend             | React + TypeScript     | Interface utilisateur web           |
| Styling              | Tailwind CSS           | Mise en forme responsive            |
| Affichage PDF        | PDF.js (Mozilla)       | Rendu du PDF dans le navigateur     |
| Backend              | Python 3.11+ / FastAPI | API REST, traitement du PDF         |
| Extraction PDF       | pdfplumber             | Lecture texte + coordonnées x/y     |
| Overlay PDF          | PyMuPDF (fitz)         | Écriture des annotations sur le PDF |
| Déploiement backend  | Railway / Render / VPS | Hébergement de l'API                |
| Déploiement frontend | Vercel / Netlify       | Hébergement de l'interface          |

### 5.3 Flux utilisateur détaillé

```
1. L'utilisateur ouvre l'application web dans son navigateur
2. Il glisse-dépose (ou sélectionne) un fichier PDF sol-fa
3. Un aperçu de la première page s'affiche via PDF.js
4. Il configure optionnellement :
     - La tonalité (si non détectable automatiquement)
     - Les voix à inclure dans l'analyse (S/A/T/B ou sous-ensemble)
     - Le niveau de détail des annotations (accord simple, avec renversement, avec 7e)
5. Il clique sur "Analyser"
6. Le frontend envoie le PDF en multipart/form-data à l'API FastAPI
7. L'API traite le PDF et retourne le PDF annoté
8. Le frontend affiche le PDF annoté côte à côte avec l'original
9. L'utilisateur peut télécharger le PDF annoté
```

### 5.4 Architecture backend — modules

#### Module A : Extraction et parsing

Ce module reçoit le PDF et en extrait une représentation structurée.

**Étape A1 — Extraction brute**

- Utilisation de `pdfplumber` pour extraire tous les tokens texte avec leurs coordonnées x/y, largeur et hauteur
- Chaque mot est un objet `{text, x0, x1, top, bottom}`

**Étape A2 — Regroupement en lignes**

- Les tokens sont regroupés en lignes par proximité verticale (tolérance de ±5 points typographiques)
- Chaque ligne est identifiée par sa position y médiane

**Étape A3 — Classification des lignes**

- Chaque ligne est classée dans une des catégories suivantes :
  - **En-tête de système** : contient la tonalité (`Dô dia F`, `Dô dia E`…) et le chiffrage de mesure (`2/4`, `4/4`…)
  - **Ligne de notes** : contient uniquement des tokens sol-fa (`d`, `r`, `m`, `fi`, `s,`, `d'`, `-`, `|`, `.`, `:`)
  - **Ligne de paroles** : contient du texte syllabé correspondant au chant
  - **Ligne de texte libre** : couplets, titres, informations de composition

**Étape A4 — Construction des systèmes**

- Les lignes de notes sont regroupées en **systèmes SATB** : blocs consécutifs de 4 lignes de notes séparées par leurs lignes de paroles correspondantes
- Chaque système retient sa position y dans la page pour le placement ultérieur des annotations

**Étape A5 — Parsing des tokens sol-fa**

- Les tokens de chaque ligne de notes sont filtrés : seuls les tokens musicaux sont conservés (notes, tirets de liaison, barres de mesure)
- Chaque token conserve sa position x (position horizontale) qui représente son **temps** dans la mesure

#### Module B : Analyse harmonique

Ce module transforme les séquences de notes extraites en une séquence d'accords annotés.

**Étape B1 — Alignement temporel**

- Pour chaque système, les 4 voix sont alignées horizontalement
- Les tokens situés à la même position x (avec une tolérance de ±10 pts) sont considérés comme appartenant au **même temps**
- Les barres de mesure `|` servent de points d'ancrage pour affiner l'alignement

**Étape B2 — Formation des accords**

- À chaque temps, l'ensemble des degrés présents dans les 4 voix est extrait
- Les modificateurs d'octave (`,` et `'`) sont ignorés pour l'identification harmonique (seul le degré de base compte)

**Étape B3 — Identification des accords**

- L'ensemble de degrés est comparé à une bibliothèque de templates d'accords, ordonnée par priorité :
  - Accords triades (3 notes) : I, ii, iii, IV, V, vi, vii°
  - Accords de septième : V⁷, ii⁷, I⁷, IV⁷
  - Accords avec notes de passage ou manquantes (2 notes) : identification approximative avec label incertain
  - Accords chromatiques : V/V, It⁶, etc.
- Si aucun template ne correspond : l'accord est marqué comme `?` avec les degrés entre parenthèses

**Étape B4 — Simplification de la séquence**

- Les accords répétés consécutivement (même accord sur plusieurs temps) sont fusionnés
- Les accords isolés de courte durée entre deux occurrences du même accord (accords de passage) peuvent être optionnellement marqués différemment ou ignorés selon un paramètre de sensibilité

**Étape B5 — Détection des renversements (optionnel)**

- Si la note la plus grave (Basse) n'est pas la fondamentale de l'accord, un renversement est noté
- Exemples : I⁶ (premier renversement), I⁶₄ (second renversement), V⁶₅ (V⁷ premier renversement)

#### Module C : Génération du PDF annoté

Ce module prend le PDF original et les données d'annotation, et produit un nouveau PDF.

**Étape C1 — Calcul des positions d'annotation**

- Pour chaque accord identifié, la position x du premier temps de cet accord est connue
- La position y est calculée comme l'espace libre au-dessus de la première voix du système concerné
- Si l'espace est insuffisant (systèmes très serrés), les annotations sont placées dans la ligne d'en-tête du système

**Étape C2 — Gestion des collisions**

- Si deux annotations consécutives sont trop proches en x (moins de 12 pts), la seconde est légèrement décalée vers le bas ou omise si elle redonde avec la précédente

**Étape C3 — Écriture des annotations**

- Utilisation de PyMuPDF pour insérer les labels texte directement dans le PDF
- Police : petite (7-8pt), couleur contrastante (rouge par défaut, personnalisable)
- Les annotations n'altèrent pas le contenu original : elles sont ajoutées en surimpression

**Étape C4 — Export**

- Le PDF résultant est retourné à l'API en bytes
- L'API le renvoie au frontend comme réponse binaire avec le header `Content-Type: application/pdf`

### 5.5 Interface utilisateur — détail des écrans

**Écran 1 : Accueil / Upload**

- Zone de dépôt de fichier (drag & drop ou clic)
- Formats acceptés : PDF uniquement
- Message d'erreur si le PDF ne semble pas contenir de sol-fa (aucun token musical détecté)

**Écran 2 : Configuration (optionnel)**

- Sélecteur de tonalité détectée (modifiable si erronée)
- Choix du niveau d'annotation : basique (I, IV, V…) / détaillé (avec renversements et 7e)
- Toggle : afficher ou non les accords de passage
- Toggle : inclure ou non les accords incertains (`?`)
- Sélecteur de couleur pour les annotations

**Écran 3 : Résultat**

- Affichage côte à côte : PDF original (gauche) / PDF annoté (droite)
- Navigation entre les pages si le PDF en comporte plusieurs
- Bouton de téléchargement du PDF annoté
- Bouton de retour pour analyser un autre fichier

### 5.6 API REST — endpoints

| Méthode | Route      | Description                                                          |
| ------- | ---------- | -------------------------------------------------------------------- |
| `POST`  | `/analyze` | Reçoit le PDF + paramètres, retourne le PDF annoté                   |
| `GET`   | `/health`  | Vérification de disponibilité du serveur                             |
| `POST`  | `/preview` | Retourne uniquement les accords détectés (JSON), sans générer le PDF |

### 5.7 Format de réponse JSON de `/preview`

```
{
  "key": "F",
  "meter": "2/4",
  "systems": [
    {
      "index": 1,
      "chords": [
        { "beat_x": 16, "degrees": ["d","m","s"], "chord": "I", "certain": true },
        { "beat_x": 144, "degrees": ["s","t","r","f"], "chord": "V7", "certain": true },
        ...
      ]
    }
  ]
}
```

### 5.8 Gestion des cas limites et erreurs

| Situation                        | Comportement                                                                                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| PDF scanné (image, pas de texte) | Message d'erreur : "Ce PDF ne contient pas de texte sélectionnable. Veuillez utiliser un PDF généré numériquement." |
| PDF multi-pages                  | Traitement de toutes les pages, annotations sur chacune                                                             |
| Système à moins de 4 voix        | Analyse avec les voix présentes, accord potentiellement incomplet signalé                                           |
| Tonalité non détectée            | Demande de saisie manuelle à l'utilisateur                                                                          |
| Token inconnu dans le sol-fa     | Ignoré dans l'analyse, logué côté serveur                                                                           |
| PDF protégé par mot de passe     | Message d'erreur explicite                                                                                          |

### 5.9 Considérations de déploiement Phase 1

- Le backend est **stateless** : chaque requête est indépendante, aucune donnée n'est conservée
- Les fichiers PDF ne sont pas stockés sur le serveur (traitement en mémoire)
- Limite de taille de fichier : 10 MB (suffisant pour les partitions sol-fa habituelles)
- Timeout de traitement : 30 secondes
- CORS configuré pour autoriser uniquement l'origine du frontend

---

## 6. Phase 2 — Application Mobile avec Backend en ligne

### 6.1 Objectif

Porter l'expérience sur iOS et Android avec une application native Flutter, en réutilisant le même backend Python déployé en Phase 1. L'application mobile offre une meilleure ergonomie pour les musiciens qui travaillent souvent sur téléphone ou tablette.

### 6.2 Stack technique

| Composant            | Technologie                       | Rôle                                |
| -------------------- | --------------------------------- | ----------------------------------- |
| Framework mobile     | Flutter (Dart)                    | Application iOS + Android           |
| State management     | Riverpod                          | Gestion d'état réactive             |
| Affichage PDF        | `pdfrx` (Flutter)                 | Rendu PDF natif haute qualité       |
| Sélection de fichier | `file_picker`                     | Import PDF depuis le stockage local |
| Requêtes réseau      | `dio`                             | Communication HTTP avec le backend  |
| Stockage local       | `shared_preferences`              | Préférences utilisateur             |
| Navigation           | GoRouter                          | Routing déclaratif                  |
| Backend              | Même que Phase 1 (Python/FastAPI) | Inchangé                            |

### 6.3 Flux utilisateur mobile

```
1. L'utilisateur ouvre l'app sur son téléphone ou tablette
2. Écran d'accueil avec bouton "Importer une partition"
3. Le sélecteur de fichier natif s'ouvre (accès aux fichiers locaux, Drive, iCloud…)
4. Le PDF sélectionné s'affiche dans un viewer intégré
5. Un panneau de configuration s'ouvre par le bas (bottom sheet)
6. L'utilisateur valide → l'app envoie le PDF au backend (avec indicateur de progression)
7. L'app reçoit le PDF annoté et l'affiche
8. Options : partager, sauvegarder dans la galerie de l'app, annoter manuellement
```

### 6.4 Fonctionnalités additionnelles Phase 2

**Galerie de partitions**

- Historique des PDF analysés (stockés localement)
- Possibilité de ré-ouvrir un résultat annoté sans ré-analyser
- Organisation par date ou par nom

**Mode comparaison intégré**

- Swipe gauche/droite pour alterner entre original et annoté
- Ou affichage split-screen sur tablette

**Annotations manuelles**

- L'utilisateur peut corriger ou ajouter un accord manuellement en tapant sur la partition
- Les corrections sont sauvegardées avec le fichier

**Paramètres persistants**

- La tonalité préférée, le niveau de détail et la couleur des annotations sont mémorisés entre sessions

### 6.5 Gestion du réseau et des erreurs

- Détection de l'état de la connexion avant envoi
- Message clair si le serveur est inaccessible : "Connexion au serveur impossible. Vérifiez votre connexion internet."
- Retry automatique (3 tentatives avec backoff exponentiel)
- Timeout configurable (défaut : 60 secondes pour les PDF volumineux)
- En mode hors-ligne : les partitions déjà analysées restent accessibles depuis la galerie locale

### 6.6 Considérations de déploiement Phase 2

- Le backend Phase 1 doit exposer une URL stable (domaine propre, certificat HTTPS obligatoire pour les apps mobiles modernes)
- Authentification légère recommandée (clé API dans les headers) pour éviter l'usage abusif
- Rate limiting côté serveur : max 20 requêtes/heure par IP
- Monitoring : logs des erreurs, temps de traitement moyen, taux de succès

---

## 7. Phase 3 — Application Mobile entièrement locale

### 7.1 Objectif

Rendre l'application **totalement indépendante d'un serveur**, en déplaçant l'intégralité du traitement sur l'appareil de l'utilisateur. L'application fonctionne sans connexion internet, dans des contextes où les musiciens n'ont pas accès au réseau (zones rurales, connexions limitées).

### 7.2 Choix d'architecture locale

La difficulté centrale de cette phase est de remplacer les bibliothèques Python (`pdfplumber`, `PyMuPDF`) par des équivalents fonctionnant nativement sur iOS/Android dans une app Flutter.

#### 7.2.1 Extraction de texte avec positions : `syncfusion_flutter_pdf`

Syncfusion propose un package Flutter complet qui permet :

- L'extraction de texte avec les bounds exacts (x, y, width, height) de chaque mot ou caractère
- La lecture de la structure du PDF (pages, métadonnées)
- L'écriture d'annotations texte sur le PDF existant

La licence communautaire est gratuite (avec logo Syncfusion dans les dialogs). Une licence commerciale supprime cette contrainte.

#### 7.2.2 Alternative : `pdfrx` (extraction) + `syncfusion` (écriture)

`pdfrx` est basé sur PDFium (le moteur de Chrome/Edge), très robuste pour l'extraction de texte avec coordonnées caractère par caractère. Il peut être combiné avec `syncfusion_flutter_pdf` pour l'écriture des annotations si sa précision d'extraction s'avère supérieure.

#### 7.2.3 Logique d'analyse : Dart pur

L'algorithme d'analyse harmonique développé en Python (Phases 1 et 2) est réécrit en **Dart** pour tourner nativement dans l'app Flutter. La logique est identique :

- Regroupement par lignes
- Classification des tokens
- Alignement temporel par position x
- Identification des accords par templates
- Calcul des positions d'annotation

### 7.3 Architecture des modules Flutter Phase 3

```
lib/
├── core/
│   ├── pdf_parser/
│   │   ├── pdf_extractor.dart        # Extraction texte + coords via syncfusion ou pdfrx
│   │   ├── line_grouper.dart         # Regroupement en lignes par y
│   │   ├── line_classifier.dart      # Classification : en-tête / notes / paroles
│   │   └── system_builder.dart       # Construction des systèmes SATB
│   ├── harmonic_analyzer/
│   │   ├── beat_aligner.dart         # Alignement temporel des 4 voix par x
│   │   ├── chord_detector.dart       # Identification des accords
│   │   ├── chord_templates.dart      # Bibliothèque de templates d'accords
│   │   └── sequence_smoother.dart    # Simplification de la séquence
│   └── pdf_annotator/
│       ├── annotation_positioner.dart # Calcul des positions y d'annotation
│       ├── collision_resolver.dart    # Gestion des chevauchements
│       └── pdf_writer.dart           # Écriture via syncfusion
├── features/
│   ├── home/                         # Écran d'accueil et import
│   ├── viewer/                       # Affichage du PDF (original + annoté)
│   ├── config/                       # Panneau de configuration
│   └── gallery/                      # Historique des partitions
└── shared/
    ├── models/                       # Structures de données (Note, Chord, System…)
    └── providers/                    # State management Riverpod
```

### 7.4 Structures de données principales

**Note** : représente un token sol-fa extrait

- Degré brut (texte : `"d"`, `"fi"`, `"s,"`)
- Degré de base normalisé (sans octave : `"d"`, `"f"`, `"s"`)
- Modificateur d'octave (grave / normal / aigu)
- Position x dans la page
- Position y dans la page (ligne d'appartenance)
- Voix (S / A / T / B)
- Indice de système

**Beat** : ensemble des notes au même temps

- Position x de référence
- Liste des notes (une par voix)
- Accord identifié
- Niveau de certitude

**ChordAnnotation** : annotation prête à être écrite

- Position x, y dans la page
- Texte de l'accord (ex. `"V⁷"`)
- Taille et couleur de police
- Indice de page

### 7.5 Performances et optimisations Phase 3

| Défi                            | Solution                                                                                     |
| ------------------------------- | -------------------------------------------------------------------------------------------- |
| Extraction lente sur grands PDF | Traitement page par page en `Isolate` Dart (thread séparé) pour ne pas bloquer l'UI          |
| Mémoire limitée sur mobile      | Traitement streamé : une page en mémoire à la fois                                           |
| PDFs avec encodages exotiques   | Fallback sur `pdfrx` (PDFium) si `syncfusion` retourne des caractères incorrects             |
| Temps de traitement perçu       | Barre de progression avec étapes détaillées ("Extraction...", "Analyse...", "Génération...") |

### 7.6 Fonctionnalités spécifiques Phase 3

**Fonctionnement hors-ligne complet**

- Aucune requête réseau lors de l'analyse
- Toutes les données restent sur l'appareil de l'utilisateur

**Confidentialité totale**

- Aucune partition n'est envoyée à un serveur
- Aucune télémétrie sans consentement explicite

**Correction interactive en temps réel**

- L'utilisateur peut modifier un accord identifié directement en touchant l'annotation
- La correction est propagée si le même accord apparaît ailleurs dans la partition (option)

**Export enrichi**

- Export du PDF annoté
- Export de la progression harmonique en texte (pour copier dans un document)
- Export éventuel vers un format de notation (future évolution)

---

## 8. Défis transversaux

### 8.1 Précision de l'alignement temporel

Le défi principal de l'analyse est d'aligner correctement les 4 voix au même temps. Plusieurs facteurs compliquent cet alignement :

- **Valeurs de notes différentes** : une voix peut avoir une blanche (note tenue, symbolisée par `-`) pendant que les autres ont deux noires. La position x du `-` ne correspond pas à un temps indépendant.
- **Points de prolongation** : le symbole `.` entre deux notes indique que la première note est prolongée d'une demi-valeur — il ne représente pas une note séparée.
- **Silences** : représentés aussi par `-` ; à distinguer des liaisons.
- **Variations typographiques** : selon le logiciel ayant généré le PDF, les espacements entre tokens peuvent varier.

La stratégie adoptée est d'utiliser les barres de mesure `|` comme ancres absolues, puis de répartir proportionnellement les temps à l'intérieur de chaque mesure.

### 8.2 PDFs à mise en page complexe

Certaines partitions sol-fa ont :

- Deux tonalités sur la même page (ex. un couplet en Fa, un refrain en Sol)
- Des systèmes à 2 ou 3 voix seulement (chœur mixte simplifié)
- Des systèmes décalés horizontalement (introductions instrumentales)
- Des annotations existantes (numéros de couplets, indications de reprise)

Chaque cas nécessite une logique de détection et de traitement spécifique dans le module de parsing.

### 8.3 Gestion des accords ambigus

Un accord composé de seulement 2 degrés peut correspondre à plusieurs harmonies possibles. Par exemple, `d + m` peut être I (Do-Mi-Sol avec Sol absent) ou iii (Mi-Sol-Si avec Sol et Si absents). L'algorithme doit :

- Tenir compte du contexte (accord précédent et suivant)
- Donner la priorité aux accords les plus courants dans le style musical concerné
- Signaler clairement l'incertitude à l'utilisateur plutôt que de choisir silencieusement

### 8.4 Internationalisation

Le sol-fa est utilisé dans de nombreuses langues et conventions :

- La tonalité peut être notée `Dô dia F` (français malgache), `Key of F` (anglais), `Ton: F` (allemand)
- Les notes peuvent être orthographiées `do` ou `doh`, `si` ou `ti`
- Les accords peuvent être attendus en notation anglaise (I, IV, V), en notation fonctionnelle allemande (T, S, D), ou en notation française (chiffres arabes)

L'application doit permettre de configurer la convention d'affichage des accords.

### 8.5 Accessibilité et ergonomie mobile

- Les annotations doivent être lisibles sur petit écran sans agrandir — taille de police adaptative
- Le mode tablette doit exploiter l'espace supplémentaire (split view, annotations plus grandes)
- Support du mode sombre (annotations en couleur adaptée)
- Gestes de zoom/pan fluides sur le PDF annoté

---

## 9. Évolutions futures possibles

Ces fonctionnalités ne font pas partie du scope actuel, mais sont architecturalement compatibles avec la base du projet :

**Lecture audio**

- Synthèse audio du sol-fa pour permettre d'entendre la partition
- Mise en évidence synchronisée de l'accord en cours de lecture

**Transposition automatique**

- Changer la tonalité d'une partition et recalculer automatiquement toutes les annotations

**Comparaison de versions**

- Comparer deux arrangements du même chant (ex. version 2/4 vs 4/4) et voir comment la progression harmonique diffère

**Partage de partitions annotées**

- Fonctionnalité de partage direct par lien (backend requis), utile pour les chefs de chœur qui distribuent des partitions

**Apprentissage de l'harmonie**

- Mode pédagogique : l'application explique pourquoi chaque accord a été identifié ainsi (degrés présents, règle appliquée)

**Support d'autres formats de notation**

- LilyPond, MusicXML, ABC Notation — formats textuels ouverts pour lesquels l'extraction est encore plus fiable que sur PDF

**API publique**

- Exposer l'API d'analyse pour que d'autres développeurs puissent intégrer l'analyse harmonique dans leurs propres outils

---

_Document rédigé pour le projet Naoty — Version 1.0_
_Phases : Web → Mobile + Backend → Mobile Local_
