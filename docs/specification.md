---
title: Spécification — Plugin Obsidian Pandoc Publisher
status: draft
version: 0.11
date: 2026-07-21
---

# 1. Résumé

Le plugin **Pandoc Publisher** permet de publier la note Markdown active en PDF depuis Obsidian en utilisant une chaîne Pandoc centralisée, XeLaTeX et des brandings configurables.

Le plugin remplace l'usage du plugin `obsidian-pandoc`, désormais supprimé du Vault, et généralise la chaîne récemment créée pour les whitepapers.

Il distingue deux modes de publication :

1. **Document brandé** : devis, comptes rendus, offres, documents clients et whitepapers. Tous partagent la même base de mise en page, la table des matières, la numérotation des pages et le rendu des callouts.
2. **Document interne** : plans de tournage, checklists, fiches de préparation et autres documents courts. La mise en page est plus compacte et ne comporte pas de page de titre ni de table des matières par défaut.

Le plugin fournit un branding global **Generic**, complet, neutre et utilisable comme exemple. Il charge en complément des packs de branding locaux depuis un dossier configurable du Vault. Dans le Vault Full Front-End, le pack **Full Front-End** est installé localement et peut être sélectionné par défaut. Un client reçoit le même plugin accompagné uniquement de son propre pack de branding.

# 2. Contexte

Avant la suppression de l'ancien plugin, le fonctionnement reposait sur trois méthodes différentes :

- le script `99 - TOOLS/bin/build-white-paper`, qui utilise un fichier Pandoc defaults, un header LaTeX et un filtre Lua ;
- le plugin `obsidian-pandoc`, qui était configuré globalement avec Eisvogel et XeLaTeX et n'est plus installé ;
- des commandes Pandoc manuelles pour les documents internes qui ont besoin du header `latex-unicode-symbols.tex`.

La configuration historique placée dans le frontmatter des devis et comptes rendus fonctionne, mais elle duplique le branding dans chaque note et rend son évolution difficile. Le script whitepaper constitue une évolution plus maintenable : la configuration de publication est placée dans des fichiers partagés.

Le plugin doit généraliser cette approche à tous les documents.

# 3. Objectifs

Le plugin doit :

- publier la note Markdown active en PDF depuis la palette de commandes Obsidian ;
- utiliser exclusivement XeLaTeX comme moteur PDF ;
- appliquer le support Unicode à tous les exports ;
- fournir une base commune aux devis, comptes rendus, offres et whitepapers ;
- fournir une mise en page distincte et compacte pour les documents internes ;
- centraliser le branding hors des notes Markdown ;
- charger les brandings depuis un dossier local configurable dans chaque Vault ;
- fournir un branding global générique complet, toujours disponible et duplicable comme exemple ;
- permettre de choisir le branding par défaut entre le générique global et les packs locaux valides ;
- utiliser Full Front-End comme branding par défaut uniquement dans le Vault Full Front-End ;
- permettre la création, la duplication, la modification et la suppression de brandings clients ;
- permettre de distribuer un pack de branding client indépendamment du plugin ;
- garantir qu'un branding absent du Vault ne peut ni être affiché ni être utilisé ;
- permettre à une note de remplacer le branding par défaut avec la propriété frontmatter `branding` ;
- fournir une documentation utilisateur complète ;
- fournir une documentation complète pour créer, valider et distribuer un pack de branding ;
- prendre en charge les callouts Obsidian dans les PDF ;
- migrer le whitepaper de référence vers les callouts Obsidian après l'implémentation du plugin ;
- traiter correctement les chemins contenant des espaces, accents et caractères Unicode ;
- produire des erreurs compréhensibles lorsque Pandoc, XeLaTeX ou une ressource manque.

# 4. Hors périmètre initial

La première version ne doit pas :

- exporter vers DOCX, ePub, HTML ou PowerPoint ;
- reproduire le rendu HTML ou CSS du thème Obsidian ;
- exécuter Dataview pour matérialiser ses résultats dans le PDF ;
- publier automatiquement des dossiers complets ;
- fournir un éditeur visuel complet de templates LaTeX ;
- installer Pandoc, TeX Live, Eisvogel ou les polices système ;
- prendre en charge Obsidian mobile.

# 5. Terminologie

## 5.1 Socle de publication

Configuration technique appliquée sans exception à tous les exports :

- entrée Markdown avec frontmatter YAML ;
- sortie PDF ;
- génération autonome (`standalone`) ;
- moteur XeLaTeX ;
- langue française par défaut ;
- header Unicode commun ;
- chemins de ressources de la note et du Vault ;
- prétraitement des syntaxes propres à Obsidian ;
- filtre commun de callouts.

Le moteur PDF ne peut pas être remplacé au niveau d'un branding ou d'une note.

## 5.2 Branding

Un branding définit l'identité visuelle d'un document brandé :

- nom et identifiant stable ;
- logo ;
- couleurs principales et secondaires ;
- couleurs des liens ;
- polices de titres, de texte et monospace ;
- textes d'en-tête et de pied de page ;
- styles des callouts ;
- éventuels fichiers Pandoc defaults et headers LaTeX supplémentaires.

Le branding ne définit ni le moteur PDF ni le support Unicode.

## 5.3 Mode de document

Le mode définit la structure générale du PDF :

- `branded` : document destiné à être présenté, transmis ou publié ;
- `internal` : document de travail court et compact.

# 6. Architecture de configuration

La configuration est composée dans cet ordre :

1. socle technique global ;
2. mode de document ;
3. branding, uniquement pour le mode `branded` ;
4. métadonnées éditoriales de la note ;
5. paramètres d'exécution calculés par le plugin, notamment les chemins d'entrée, de ressources et de sortie.

Les paramètres techniques globaux et les valeurs visuelles du branding ne doivent pas pouvoir être remplacés silencieusement par le frontmatter.

## 6.1 Socle global attendu

Équivalent Pandoc conceptuel :

```yaml
from: markdown+smart+yaml_metadata_block
to: pdf
standalone: true
pdf-engine: xelatex
metadata:
  lang: fr-FR
include-in-header:
  - latex-unicode-symbols.tex
filters:
  - obsidian-callouts.lua
```

Le chemin absolu de XeLaTeX est configuré une fois dans les réglages globaux. Sur l'installation actuelle, la valeur attendue est `/Library/TeX/texbin/xelatex`.

## 6.2 Mode `branded`

Ce mode reprend et généralise la configuration actuelle du whitepaper.

Il active obligatoirement :

- le template Eisvogel ;
- une page de titre ;
- une table des matières ;
- une table des matières sur une page dédiée ;
- la numérotation des pages ;
- les styles de titres, tableaux, citations et liens ;
- les callouts PDF ;
- le branding sélectionné ;
- des marges adaptées à la lecture et à l'impression.

Il est utilisé indifféremment pour :

- les devis ;
- les offres ;
- les comptes rendus ;
- les documents clients ;
- les whitepapers.

Le whitepaper n'est donc pas un moteur ou un template distinct. Il s'agit d'un document brandé dont le contenu est généralement plus long.

## 6.3 Mode `internal`

Ce mode utilise le même socle XeLaTeX, Unicode et callouts, mais une présentation simplifiée.

Valeurs par défaut :

- aucune page de titre ;
- aucune table des matières ;
- titres compacts ;
- marges plus petites ;
- en-tête absent ou très discret ;
- pied de page limité au nom du document et au numéro de page ;
- logo absent ;
- rendu compact des listes et checkboxes ;
- liens Obsidian transformés en libellés lisibles.

Le mode interne n'utilise pas de branding client. Il utilise un style interne générique livré avec le plugin, sans logo ni identité Full Front-End.

# 7. Packs de branding locaux

## 7.1 Dossier configurable

Le branding global `generic` est embarqué dans le plugin. Tous les brandings métier ou client sont des packs locaux. Chaque Vault configure leur dossier au moyen d'un chemin relatif à sa racine.

Exemple de réglage dans le Vault Full Front-End :

```json
{
  "brandingDirectory": "99 - TOOLS/Pandoc/Brandings",
  "defaultBrandingId": "full-front-end"
}
```

Chemin absolu correspondant :

```text
/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings
```

Le chemin est sélectionnable dans les réglages avec un sélecteur de dossier du Vault. Il peut être différent dans chaque Vault. Un chemin absolu externe au Vault n'est pas accepté dans la première version afin que les packs restent portables.

Le plugin fournit les actions suivantes :

- choisir le dossier de brandings ;
- créer le dossier s'il n'existe pas ;
- ouvrir le dossier dans le gestionnaire de fichiers ;
- rescanner les brandings ;
- afficher les erreurs de validation d'un pack ;
- choisir un branding par défaut entre `generic` et les packs locaux valides.

## 7.2 Structure d'un pack

Un branding est un dossier autonome et copiable :

```text
Nom du branding/
├── branding.yaml
├── branding-header.tex
├── defaults.yaml
├── callouts.tex
└── assets/
    ├── logo.png
    └── autres-ressources.png
```

Seul `branding.yaml` est obligatoire. Les autres fichiers sont référencés depuis celui-ci lorsqu'ils existent.

Tous les chemins internes au pack sont relatifs au dossier du pack. Un pack valide ne dépend pas d'un chemin absolu propre à la machine de son créateur.

Le manifeste contient au minimum :

```yaml
schema-version: 1
id: client-example
name: Client Example
version: 1.0.0
```

Il peut ensuite référencer le logo, les couleurs, les polices, les textes de page, les styles de callouts, le header et les defaults Pandoc.

## 7.3 Découverte et validation

Le plugin scanne les sous-dossiers directs du dossier configuré et recherche `branding.yaml`.

Un pack n'apparaît comme disponible que si :

- son manifeste est lisible et conforme au schéma pris en charge ;
- son `id` est unique dans le dossier configuré ;
- tous ses chemins déclarés restent dans le dossier du pack ;
- ses ressources obligatoires existent ;
- ses couleurs et dimensions utilisent des formats valides ;
- ses fichiers YAML peuvent être analysés ;
- ses éventuels headers LaTeX et filtres Lua sont explicitement déclarés.

Un pack invalide est affiché séparément avec ses erreurs, mais ne peut pas être utilisé pour publier.

Si aucun pack valide n'est installé :

- le branding global `generic` reste disponible pour les documents brandés ;
- le mode interne reste disponible ;
- aucun branding métier, client ou Full Front-End n'est utilisé silencieusement.

## 7.4 Distribution et isolation

Pour équiper un client :

1. installer le plugin générique dans son Vault ;
2. copier uniquement le dossier de son branding dans un dossier de son Vault ;
3. sélectionner ce dossier dans les réglages du plugin ;
4. sélectionner son branding comme branding par défaut ;
5. tester la configuration PDF.

Le client peut alors produire ses documents sans modifier le plugin.

L'isolation repose sur l'absence physique des autres packs :

- le dépôt et les releases du plugin ne contiennent pas le branding Full Front-End ;
- le seul branding intégré est `generic`, qui ne contient aucune identité métier ;
- le pack donné à un client ne contient que son identité ;
- le plugin n'effectue aucun téléchargement de branding ;
- le plugin ne connaît pas d'identifiant Full Front-End codé en dur ;
- un branding absent du dossier configuré ne peut être sélectionné par une commande ou un frontmatter ;
- si une note demande un branding absent, la publication s'arrête avec une erreur explicite.

Le plugin ne constitue pas un mécanisme de DRM : une personne qui obtient volontairement les fichiers d'un pack peut les utiliser. La garantie recherchée est qu'aucune charte non distribuée ne soit incluse ou accessible par défaut. Le branding `generic` ne doit reprendre aucun élément identifiable d'une charte réelle.

Les packs contenant des headers LaTeX ou filtres Lua doivent être considérés comme du code local. Le plugin indique qu'un pack ne doit être installé que depuis une source de confiance.

## 7.5 Branding global Generic

Le plugin fournit un branding intégré avec :

```yaml
schema-version: 1
id: generic
name: Generic
version: 1.0.0
```

Ce branding est :

- complet et directement utilisable ;
- neutre et sans identité commerciale ;
- en lecture seule dans les ressources du plugin ;
- toujours disponible, même sans dossier local configuré ;
- proposé comme branding par défaut lors de la première installation ;
- duplicable vers le dossier local pour servir de point de départ à un nouveau pack ;
- utilisé par les exemples, captures et tests publics.

Il comprend au minimum :

- une page de titre neutre ;
- une palette sobre et accessible ;
- des polices génériques disponibles avec l'environnement documenté ;
- des en-têtes et pieds de page neutres ;
- une table des matières et une pagination ;
- tous les rôles de callouts ;
- un logo d'exemple explicitement générique ou aucune dépendance obligatoire à un logo ;
- un `branding.yaml`, un header, des defaults et les assets nécessaires pour illustrer un pack complet.

L'identifiant `generic` est réservé. Un pack local ne peut pas le remplacer ou le masquer.

## 7.6 Pack local Full Front-End

Dans le Vault Full Front-End uniquement, le pack local est installé à cet emplacement :

```text
/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings/Full Front-End
```

Il est créé pendant la migration à partir de :

- `99 - TOOLS/Pandoc/White Paper/ffe-white-paper.yaml` ;
- `99 - TOOLS/Pandoc/White Paper/ffe-white-paper-header.tex` ;
- `99 - TOOLS/Pandoc/White Paper/ffe-callouts.lua` ;
- les logos Full Front-End autorisés pour cet usage.

Le fichier `ffe-callouts.lua` sert uniquement de référence pour reprendre le rendu existant. Son mécanisme de reconnaissance des libellés dans des blockquotes n'est pas conservé.

Le nouveau filtre commun détecte exclusivement la syntaxe officielle des callouts Obsidian. Les couleurs, titres et styles Full Front-End sont définis dans le pack local Full Front-End.

# 8. Gestion des brandings

## 8.1 Liste des brandings

Les réglages affichent :

```text
Brandings
├── Dossier : 99 - TOOLS/Pandoc/Brandings
├── Generic — Global intégré
├── Full Front-End ★ Par défaut
├── Client A
├── Client B
├── Packs invalides (1)
└── Rescanner
```

Actions disponibles :

- créer ;
- dupliquer ;
- renommer ;
- définir comme branding par défaut ;
- désactiver ;
- supprimer ou déplacer vers la corbeille après confirmation ;
- dupliquer `generic` vers le dossier local ;
- tester sur une note exemple ;
- révéler le dossier du pack.

La création peut partir de zéro, dupliquer le branding global `generic` ou dupliquer un pack local déjà présent. Le plugin ne propose jamais Full Front-End comme modèle lorsqu'il n'est pas installé dans le Vault courant. Le branding `generic` ne peut être ni désactivé ni supprimé.

## 8.2 Champs d'un branding

Champs simples :

- nom ;
- identifiant généré et stable ;
- logo sélectionné dans le Vault ;
- largeur du logo ;
- couleur principale ;
- couleur d'accent ;
- couleur de texte ;
- couleur de couverture ;
- couleur des liens ;
- police des titres ;
- police du corps ;
- police monospace ;
- texte d'en-tête gauche, centre et droite ;
- texte de pied de page gauche, centre et droite.

Champs avancés :

- fichier Pandoc defaults supplémentaire ;
- headers LaTeX supplémentaires ;
- filtres Lua supplémentaires ;
- variables Eisvogel supplémentaires sous forme de paires clé/valeur ;
- correspondances de callouts personnalisés.

Tous les chemins de ressources stockés dans `branding.yaml` sont relatifs au dossier du pack afin qu'une copie du dossier suffise pour transférer le branding.

## 8.3 Modèle de données indicatif

```ts
interface Branding {
  schemaVersion: number;
  id: string;
  name: string;
  version: string;
  enabled: boolean;
  rootPath: string;

  logo?: {
    path: string;
    width: string;
  };

  colors: {
    primary: string;
    accent: string;
    coverBackground: string;
    coverText: string;
    bodyText: string;
    muted: string;
    link: string;
  };

  fonts: {
    headings: string;
    body: string;
    mono: string;
  };

  header: PageSlots;
  footer: PageSlots;
  callouts: CalloutTheme;

  defaultsFiles: string[];
  headerFiles: string[];
  luaFilters: string[];
  variables: Record<string, string | number | boolean>;
}

interface PageSlots {
  left?: string;
  center?: string;
  right?: string;
}
```

# 9. Callouts Obsidian

## 9.1 Syntaxe prise en charge

Le filtre commun doit reconnaître la syntaxe officielle Obsidian :

```markdown
> [!info] Titre personnalisé
> Contenu du callout.
```

Il doit également gérer :

- un titre implicite dérivé du type ;
- un titre personnalisé ;
- un callout sans corps ;
- les marqueurs repliables `+` et `-` ;
- les types sans distinction de casse ;
- les alias officiels ;
- les callouts imbriqués ;
- le Markdown présent dans le contenu ;
- les liens internes et images après leur normalisation par le plugin ;
- les types personnalisés configurés dans un branding.

Le caractère replié ou déplié n'a pas de sens dans un PDF : `+` et `-` sont acceptés mais n'affectent pas le contenu exporté. Le PDF contient toujours le corps complet.

## 9.2 Types et alias standards

Le filtre prend en charge au minimum :

- `note` ;
- `abstract`, `summary`, `tldr` ;
- `info` ;
- `todo` ;
- `tip`, `hint`, `important` ;
- `success`, `check`, `done` ;
- `question`, `help`, `faq` ;
- `warning`, `caution`, `attention` ;
- `failure`, `fail`, `missing` ;
- `danger`, `error` ;
- `bug` ;
- `example` ;
- `quote`, `cite`.

Les alias sont normalisés vers un type canonique avant l'application du thème.

Un type inconnu utilise le style `note`, sauf si une correspondance personnalisée existe dans le branding.

## 9.3 Rôles visuels

Le filtre ne doit pas contenir directement les couleurs Full Front-End. Il transforme les types Obsidian en rôles sémantiques, par exemple :

- information ;
- conseil ;
- succès ;
- question ;
- avertissement ;
- danger ;
- exemple ;
- citation.

Chaque branding associe ensuite ces rôles à ses couleurs, bordures et titres.

Le mode interne utilise les mêmes rôles avec un rendu plus compact et moins coloré.

### 9.3.1 Loi cognitive Full Front-End

Cette sous-section spécifie le pack local Full Front-End. Ses valeurs ne sont ni codées dans le moteur générique ni incluses dans les releases GitHub du plugin.

Le thème de callouts Full Front-End respecte en priorité la loi cognitive définie dans la charte graphique et le système de couleur narratif :

| Niveau | Type Obsidian principal | Fonction | Couleur | Fond |
| --- | --- | --- | --- | --- |
| Neutre | `note`, `example`, `quote` | Citation, exemple, conclusion ou contenu neutre | Gris `#5F5F5F` | Gris clair `#F3F3F3` |
| Pratique | `info` | **Tips & Tricks**, **Astuce pratique** ou **Trucs et astuces** : conseil simple, action ou test immédiatement applicable | Bleu FFE `#315FA8` | Bleu clair `#EAF1FB` |
| Maîtrise | `tip` | **Pro Tip**, **Le conseil Full Front-End** ou **Le regard de Fullfrontend** : principe structurant, compréhension avancée ou savoir issu de l'expérience | Jaune FFE `#FFCC00` | Jaune clair `#FFF8D6` |

Le bleu et le jaune ci-dessus correspondent exactement aux deux encarts déjà présents dans le header whitepaper. Leur aspect doit être conservé lors de la migration :

- le nouvel environnement générique `info` reproduit l'aspect bleu de `ffetipbox`, sans conserver cet ancien environnement dans le plugin ;
- le nouvel environnement générique `tip` reproduit l'aspect jaune de `ffeprotipbox`, sans conserver cet ancien environnement dans le plugin.

Le jaune ne doit jamais servir à signaler une simple information pratique. Il indique le niveau de compréhension le plus fort dans un document Full Front-End.

La distinction entre `info` et `tip` repose sur la profondeur du contenu :

- `info` répond principalement à « Qu'est-ce que je peux faire ou tester maintenant ? » ;
- `tip` répond principalement à « Quel principe ou retour d'expérience va m'aider à mieux décider ou comprendre ? ».

La longueur de l'encart ne suffit pas à déterminer son type. Un conseil très court peut être un Pro Tip s'il exprime un principe de maîtrise ; un encart plus développé reste `info` s'il décrit simplement une action pratique.

### 9.3.2 Palette étendue Full Front-End

Les autres types de callouts utilisent les couleurs déjà définies dans le système narratif FFE. Dans un document, ces couleurs signalent une fonction d'encart ; elles ne classent pas le document dans un territoire éditorial vidéo.

| Type canonique | Alias concernés | Fonction dans un document | Contour et titre | Fond |
| --- | --- | --- | --- | --- |
| `note` | — | Note neutre | `#5F5F5F` | `#F3F3F3` |
| `abstract` | `summary`, `tldr` | Synthèse ou structure | Cyan `#20C7D9` | Cyan clair `#E4F8FA` |
| `info` | — | Tips & Tricks, astuce pratique, action simple | Bleu FFE `#315FA8` | Bleu clair `#EAF1FB` |
| `todo` | — | Action à effectuer ou étape de workflow | Cyan `#20C7D9` | Cyan clair `#E4F8FA` |
| `tip` | `hint`, `important` | Pro Tip, conseil Full Front-End, principe ou maîtrise | Jaune FFE `#FFCC00` | Jaune clair `#FFF8D6` |
| `success` | `check`, `done` | Validation, réussite ou progression | Vert `#25B879` | Vert clair `#E5F7EF` |
| `question` | `help`, `faq` | Question, recul ou analyse | Violet `#7C55E8` | Violet clair `#EFEBFC` |
| `warning` | `caution`, `attention` | Vigilance ou point d'attention | Orange `#F2762E` | Orange clair `#FDEFE6` |
| `failure` | `fail`, `missing` | Échec ou élément manquant | Orange `#F2762E` avec titre `#161616` | Orange clair `#FDEFE6` |
| `danger` | `error` | Risque fort ou erreur bloquante | Noir FFE `#161616` avec accent orange `#F2762E` | Orange clair `#FDEFE6` |
| `bug` | — | Défaut technique | Gris secondaire `#333333` avec accent orange `#F2762E` | Gris clair `#F3F3F3` |
| `example` | — | Exemple neutre | Gris `#5F5F5F` | Gris clair `#F3F3F3` |
| `quote` | `cite` | Citation | Gris `#5F5F5F` | Gris clair `#F3F3F3` |

Les fonds cyan, vert, violet et orange sont des teintes claires dérivées des couleurs FFE afin de conserver du texte sombre lisible et une densité visuelle comparable aux encarts bleu et jaune existants.

Un callout doit rester identifiable sans dépendre uniquement de sa couleur. Le titre textuel, la bordure et, si elles sont retenues, les icônes doivent aussi différencier les rôles.

## 9.4 Syntaxe éditoriale Full Front-End

Cette syntaxe et ces titres sont configurés par le pack local Full Front-End, pas par le plugin générique.

Le plugin reconnaît uniquement les callouts écrits avec la syntaxe officielle Obsidian.

Les syntaxes Full Front-End recommandées sont :

```markdown
> [!info] Trucs et astuces — Titre
> Action ou information immédiatement applicable.
```

```markdown
> [!tip] Le regard de Fullfrontend — Titre
> Compréhension avancée ou principe issu de l'expérience.
```

Pour le branding Full Front-End, les titres implicites sont :

- `info` : « Trucs et astuces » ;
- `tip` : « Le regard de Fullfrontend ».

Un titre écrit explicitement après `[!info]` ou `[!tip]` reste prioritaire.

Les variantes de titres restent possibles sans modifier le type du callout :

```markdown
> [!info] Astuce pratique — Écoute sans regarder
> Contenu immédiatement applicable.
```

```markdown
> [!tip] Le conseil Full Front-End — Commence par être utile
> Principe ou retour d'expérience avancé.
```

Les brandings clients peuvent remplacer ces titres par défaut sans modifier leur sémantique.

Une blockquote Markdown ordinaire reste une citation ordinaire, quel que soit son premier texte. Le filtre ne recherche pas les libellés « Astuce pratique », « Le conseil Full Front-End », « Trucs et astuces », « Le regard de Fullfrontend » ou « Pro Tip » dans les blockquotes.

Il n'existe aucun mode legacy, aucune table d'alias de libellés historiques et aucune conversion implicite des anciennes blockquotes dans le plugin.

### 9.4.1 Migration du whitepaper de référence

Cette migration est exécutée uniquement après que le plugin et le nouveau filtre de callouts sont implémentés et testés sur des fixtures dédiées.

Dans `Le guide de ta première vidéo - De je n'ose pas à j'ai publié.md` :

- chaque encart `Astuce pratique` doit être converti vers `[!info] Astuce pratique — …` ;
- chaque encart `Le conseil Full Front-End` doit être converti vers `[!tip] Le conseil Full Front-End — …` ;
- le contenu des encarts ne doit pas être réécrit pendant cette migration ;
- les titres visibles existants sont conservés pendant la migration ;
- le rendu bleu des astuces et le rendu jaune des regards doivent rester visuellement identiques au PDF actuel.

La migration doit convertir tous les encarts concernés en une seule opération vérifiable. Après cette opération, aucune blockquote à libellé historique ne doit subsister dans le whitepaper.

Le whitepaper n'est utilisé comme test d'acceptation du nouveau plugin qu'après cette migration. Avant celle-ci, ses anciens encarts sont volontairement rendus comme de simples blockquotes.

## 9.5 Sortie LaTeX

Le filtre Lua produit des environnements LaTeX génériques et stables. Les environnements ne doivent pas contenir `ffe` dans leur nom.

Exemples indicatifs :

```latex
\begin{publishercallouttip}{Titre}
Contenu
\end{publishercallouttip}
```

```latex
\begin{publishercalloutwarning}{Titre}
Contenu
\end{publishercalloutwarning}
```

Le header du mode et le branding définissent l'apparence des environnements.

# 10. Frontmatter des notes

Le frontmatter reste destiné au contenu et à la sélection du mode ou du branding, pas à la description complète de la mise en page.

## 10.1 Métadonnées éditoriales conservées

Exemples :

```yaml
---
title: Titre du document
subtitle: Sous-titre
author:
  - Nom de l'auteur
date: 2026-07-21
subject: Sujet
keywords:
  - mot-clé
description: Description
---
```

## 10.2 Métadonnées du plugin

```yaml
---
pdf-mode: branded
branding: full-front-end
---
```

Dans le Vault d'un client, la même propriété utilise l'identifiant de son pack :

```yaml
---
pdf-mode: branded
branding: client-example
---
```

Valeurs :

- `pdf-mode` : `branded` ou `internal` ;
- `branding` : identifiant du branding global `generic` ou d'un pack local actif.

Si `pdf-mode` est absent, la commande choisie détermine le mode.

Si `branding` est absent pour un document brandé, le branding par défaut configuré dans le plugin est utilisé.

Si `branding` vaut `generic`, le branding global intégré est utilisé, même si un autre branding est sélectionné par défaut.

Si le branding demandé n'existe plus, le plugin arrête la publication et demande de choisir un branding au lieu de basculer silencieusement sur un autre.

Ordre de résolution pour un document brandé :

1. propriété frontmatter `branding` lorsqu'elle est présente ;
2. branding par défaut choisi dans les réglages ;
3. branding global `generic` si le réglage par défaut est vide ou devenu invalide.

Le fallback vers `generic` ne s'applique jamais lorsqu'un document demande explicitement un branding local absent : ce cas est une erreur.

## 10.3 Anciennes métadonnées visuelles

Les anciennes propriétés telles que `titlepage-color`, `titlepage-logo`, `logo-width`, `mainfont`, `header-right` ou `footer-left` ne constituent plus la source de vérité du branding.

Le plugin ne fournit aucun mode de compatibilité pour ces propriétés visuelles :

- elles sont exclues des métadonnées transmises au template lorsqu'un branding est appliqué ;
- les valeurs du branding sélectionné sont toujours utilisées ;
- une action « Créer un branding depuis cette note » peut être ajoutée ultérieurement comme outil de migration ponctuel, sans lecture legacy pendant la publication ;
- les nouvelles notes ne doivent plus dupliquer ces propriétés ;
- leur suppression des anciennes notes peut être réalisée séparément, sans bloquer la publication.

# 11. Commandes Obsidian

Le plugin enregistre les commandes suivantes :

## 11.1 Publier en PDF

Commande principale.

Résolution :

1. lit `pdf-mode` si présent ;
2. sinon propose `Document brandé` ou `Document interne` ;
3. pour un document brandé, lit `branding` ou utilise le branding par défaut ;
4. génère le PDF à côté de la note.

## 11.2 Publier en PDF — Document brandé

Publie directement avec le branding indiqué dans la note ou le branding par défaut.

## 11.3 Publier en PDF — Document interne

Publie avec la mise en page interne.

## 11.4 Publier en PDF avec un branding…

Affiche un sélecteur de brandings actifs, puis publie un document brandé sans modifier la note.

## 11.5 Tester la configuration PDF

Vérifie :

- le chemin de Pandoc ;
- la version de Pandoc ;
- le chemin et la version de XeLaTeX ;
- la présence d'Eisvogel ;
- la présence du header Unicode ;
- la présence des packages LaTeX requis ;
- l'intégrité du branding global `generic` ;
- l'existence des ressources du branding par défaut ;
- la génération d'un petit PDF temporaire avec Unicode et callout.

# 12. Réglages du plugin

## 12.1 Réglages globaux

- chemin de Pandoc ;
- chemin de XeLaTeX ;
- dossier local des brandings, relatif à la racine du Vault ;
- action pour créer, ouvrir et rescanner ce dossier ;
- branding par défaut, choisi dans une liste contenant `Generic — Global` et tous les packs locaux valides ;
- dossier de sortie : à côté de la note par défaut ;
- modèle de nom : `{{basename}}.pdf` par défaut ;
- remplacer automatiquement un PDF existant ;
- ouvrir le PDF après publication ;
- révéler le PDF dans le Finder ;
- conserver les fichiers LaTeX temporaires en cas d'erreur ;
- niveau de détail des logs.

Le choix d'un autre moteur PDF n'est pas exposé.

## 12.2 Réglages des modes

Les deux modes intégrés sont visibles pour consultation et réglages limités.

Options modifiables du mode interne :

- marges ;
- taille de police ;
- affichage du pied de page ;
- numérotation des pages ;
- densité des listes.

Les invariants du mode brandé — page de titre, table des matières et numérotation — ne doivent pas être désactivés dans la première version.

# 13. Prétraitement du Markdown Obsidian

Le plugin publie le Markdown source, et non le HTML rendu par Obsidian.

Avant Pandoc, il normalise au minimum :

- `[[Note]]` vers `Note` ;
- `[[Note|Libellé]]` vers `Libellé` ;
- `[[Note#Section|Libellé]]` vers `Libellé` ;
- `![[image.png]]` vers une image Markdown utilisant le chemin résolu dans le Vault ;
- les callouts Obsidian ;
- les checkboxes Markdown ;
- les caractères de contrôle ou variantes Unicode problématiques connues.

Les fichiers intégrés qui ne sont pas des images ne sont pas transclus dans la première version. Le plugin affiche un avertissement et conserve un libellé lisible.

Le prétraitement doit préserver :

- le frontmatter YAML ;
- le Markdown standard ;
- les blocs de code ;
- le contenu littéral des blocs de code, dans lesquels les wikilinks et callouts ne doivent pas être transformés ;
- les chemins d'images Markdown déjà valides.

# 14. Exécution Pandoc

## 14.1 Construction des arguments

Le plugin utilise `child_process.spawn` avec :

- `shell: false` ;
- un tableau d'arguments ;
- la racine du Vault comme répertoire de travail ;
- un environnement `PATH` contenant le dossier de XeLaTeX ;
- une entrée temporaire ou standard correctement encodée en UTF-8.

Il est interdit de construire une chaîne de commande puis de la découper sur les espaces.

Les chemins sont passés comme valeurs individuelles afin de prendre en charge les espaces, apostrophes, accents et caractères Unicode.

## 14.2 Chemins de ressources

Le `resource-path` contient au minimum :

1. le dossier de la note source ;
2. la racine du Vault ;
3. les éventuels dossiers de ressources du branding.

Le séparateur est calculé avec `path.delimiter` afin de rester compatible avec macOS, Linux et Windows, même si la première version est principalement validée sur macOS.

## 14.3 Fichier de sortie

Par défaut :

```text
Note.md
Note.pdf
```

Le plugin ne modifie jamais le Markdown source pendant la publication.

Si le PDF existe et que le remplacement automatique est activé, le fichier dérivé est remplacé. Sinon, le plugin demande confirmation ou propose un nouveau nom.

# 15. Retours utilisateur et erreurs

Au lancement :

```text
Publication de « Nom de la note » avec Nom du branding…
```

En cas de succès :

```text
PDF publié : Nom de la note.pdf
```

Actions proposées :

- ouvrir le PDF ;
- révéler dans le Finder ;
- copier la commande équivalente.

En cas d'échec, une fenêtre affiche :

- l'étape en échec ;
- un résumé compréhensible ;
- la sortie d'erreur Pandoc/XeLaTeX ;
- les chemins des ressources manquantes ;
- un bouton pour copier les détails ;
- le chemin du fichier `.tex` temporaire si sa conservation est activée.

Un export n'est considéré comme réussi que si le processus retourne le code `0` et que le PDF attendu existe.

# 16. Validation et critères d'acceptation

## 16.1 Environnement

- le test de configuration détecte Pandoc 3.10 sur l'installation actuelle ;
- XeLaTeX est effectivement le processus utilisé ;
- une configuration faisant référence à `pdflatex` est refusée ;
- le test Unicode rend correctement au minimum `✓`, les accents français, les apostrophes typographiques et les tirets longs.

## 16.2 Documents brandés

- un devis existant peut être publié avec le pack Full Front-End local sans recopier le branding dans son frontmatter ;
- un compte rendu existant peut être publié avec la même chaîne ;
- un whitepaper existant conserve son rendu, sa table des matières et ses encarts ;
- chaque document brandé possède une page de titre, une table des matières et des numéros de page ;
- le remplacement du branding modifie le logo, les couleurs, les polices, les en-têtes, les pieds de page et les callouts sans modifier la note ;
- un chemin de logo contenant des espaces et accents fonctionne.

## 16.2.1 Portabilité et isolation des packs

- le chemin du dossier de brandings est configurable et stocké relativement au Vault ;
- le branding global `generic` est disponible sur une installation neuve et permet un export brandé complet ;
- `generic` peut être sélectionné comme branding par défaut même lorsque des packs locaux sont installés ;
- un pack local valide peut être sélectionné comme branding par défaut ;
- `branding: generic` dans une note remplace correctement un branding local défini par défaut ;
- `branding: client-example` dans une note remplace correctement `generic` ou un autre branding défini par défaut ;
- copier un pack client autonome dans un autre Vault suffit pour le rendre détectable après configuration ou rescan ;
- toutes les ressources d'un pack restent résolues après déplacement du Vault ;
- une release GitHub fraîchement installée ne contient ni ne propose Full Front-End ;
- un Vault contenant uniquement le pack Client A ne peut pas sélectionner Full Front-End ou Client B ;
- demander un identifiant de branding absent dans le frontmatter bloque l'export avec une erreur explicite ;
- supprimer ou déplacer le pack actif invalide sa sélection au prochain scan ;
- le mode interne et les documents brandés génériques restent utilisables lorsqu'aucun pack local n'est installé.

## 16.3 Documents internes

- le plan de tournage de référence est publié sans page de titre ni table des matières ;
- ses checkboxes et symboles Unicode sont lisibles ;
- ses wikilinks deviennent des libellés propres ;
- le PDF reste compact et exploitable pendant un tournage.

## 16.4 Callouts

- tous les types et alias standards sont reconnus ;
- les titres implicites et personnalisés fonctionnent ;
- les callouts `+` et `-` contiennent toujours tout leur corps ;
- les callouts imbriqués ne cassent pas la compilation LaTeX ;
- un type inconnu utilise le style `note` ;
- un type personnalisé peut être associé à un rôle par branding ;
- une blockquote historique « Astuce pratique » ou « Le conseil Full Front-End » reste une citation ordinaire et n'est jamais transformée en callout ;
- `info` utilise le bleu FFE existant et le titre implicite « Trucs et astuces » ;
- `tip` utilise le jaune FFE existant et le titre implicite « Le regard de Fullfrontend » ;
- un callout `[!info]` intitulé « Astuce pratique » est classé comme Tips & Tricks ;
- un callout `[!tip]` intitulé « Le conseil Full Front-End » est classé comme Pro Tip ;
- les autres types respectent la palette étendue définie dans la section 9.3.2 ;
- les titres implicites `info` et `tip` peuvent être remplacés par chaque branding.

## 16.5 Robustesse

- une note ou ressource dont le chemin contient des espaces, accents ou apostrophes est publiée ;
- une image relative à la note est trouvée ;
- une image située ailleurs dans le Vault est trouvée après résolution d'un embed Obsidian ;
- une ressource manquante produit un message précis ;
- deux publications simultanées de la même note sont empêchées ;
- le Markdown source n'est jamais altéré.

# 17. Migration

La migration se déroule sans supprimer immédiatement les outils actuels.

1. Créer le socle commun XeLaTeX, Unicode et callouts.
2. Extraire de la configuration whitepaper uniquement les éléments génériques du mode `branded`.
3. Créer le pack local Full Front-End dans le Vault depuis les fichiers existants, sans l'ajouter au dépôt du plugin.
4. Créer le mode `internal` compact.
5. Étendre le filtre aux callouts Obsidian et à la palette FFE complète.
6. Tester le filtre sur des fixtures dédiées couvrant tous les types, alias, titres et imbrications.
7. Implémenter et valider le reste du plugin sur un devis ou compte rendu et un document interne.
8. Migrer en une seule opération tous les encarts du whitepaper de référence vers `info` et `tip`.
9. Vérifier qu'aucune blockquote historique d'encart ne subsiste dans le whitepaper.
10. Utiliser le whitepaper migré comme test d'acceptation du rendu brandé complet.
11. Installer et activer le nouveau plugin dans le Vault au moyen du symlink de développement.
12. Comparer les PDF avec les sorties de référence existantes et corriger les écarts.
13. Si l'interface en ligne de commande reste utile pendant la transition, l'adapter pour combiner les ressources génériques de l'installation du plugin et le pack local Full Front-End, sans maintenir une seconde copie des defaults, headers ou filtres.

# 18. Dépôt autonome et organisation des fichiers

## 18.1 Dépôt Git indépendant

Le plugin est développé dans un dépôt Git autonome, situé hors du Vault et hors du dépôt Git qui contient le Vault.

Chemin local canonique :

```text
/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/
```

Ce chemin est utilisé pour l'implémentation locale. Il ne doit jamais être un sous-dossier du Vault.

Le dépôt possède :

- son propre dossier `.git` ;
- son propre remote GitHub ;
- son propre versionnement sémantique ;
- ses issues, releases et changelog ;
- sa propre licence ;
- sa propre chaîne de tests et de publication.

Le dépôt du Vault ne contient ni la source du plugin ni une copie de son historique Git.

## 18.2 Arborescence du dépôt du plugin

```text
pandoc-publisher/
├── .github/
│   └── workflows/
│       ├── test.yml
│       └── release.yml
├── src/
│   ├── main.ts
│   ├── settings/
│   ├── publishing/
│   ├── brandings/
│   ├── markdown/
│   └── ui/
├── resources/
│   ├── common/
│   │   ├── base.yaml
│   │   ├── latex-unicode-symbols.tex
│   │   └── obsidian-callouts.lua
│   ├── modes/
│   │   ├── branded.yaml
│   │   ├── branded-header.tex
│   │   ├── internal.yaml
│   │   └── internal-header.tex
│   └── brandings/
│       └── generic/
│           ├── branding.yaml
│           ├── branding-header.tex
│           ├── defaults.yaml
│           ├── callouts.tex
│           └── assets/
├── tests/
│   ├── fixtures/
│   ├── unit/
│   └── integration/
├── docs/
│   ├── specification.md
│   ├── user-guide.md
│   └── branding-pack-guide.md
├── scripts/
│   └── link-vault.sh
├── manifest.json
├── package.json
├── tsconfig.json
├── esbuild.config.mjs
├── styles.css
├── README.md
├── CHANGELOG.md
└── LICENSE
```

Le dépôt autonome contient uniquement le moteur, les ressources communes, les modes génériques et le branding global `generic`. Il ne contient aucun logo, couleur, texte, header ou defaults Full Front-End exploitable.

Les éléments génériques extraits des fichiers existants du Vault servent de sources de migration pendant l'implémentation. Les éléments propres à Full Front-End restent exclusivement dans le pack local du Vault.

## 18.2.1 Chemins absolus de référence

| Rôle | Chemin absolu |
| --- | --- |
| Racine du Vault | `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End` |
| Racine du dépôt autonome du plugin | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher` |
| Emplacement chargé par Obsidian | `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/.obsidian/plugins/pandoc-publisher` |
| Spec de travail dans le Vault | `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Spécification - Plugin Obsidian Pandoc Publisher.md` |
| Copie versionnée de la spec dans le plugin | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/specification.md` |
| Guide utilisateur du plugin | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/user-guide.md` |
| Guide de création d'un pack | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/branding-pack-guide.md` |
| Branding global générique | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/resources/brandings/generic` |
| Réglages locaux en développement | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/data.json` |
| Manifest du plugin | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/manifest.json` |
| Build principal généré | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/main.js` |
| Dossier de brandings configuré pour le Vault FFE | `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings` |
| Pack local Full Front-End | `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings/Full Front-End` |

## 18.2.2 Copie initiale de la spécification

Avant toute implémentation :

1. créer le dépôt autonome ;
2. créer son dossier `docs/` ;
3. copier la présente spec vers `docs/specification.md` ;
4. vérifier que les deux fichiers ont le même contenu ;
5. commiter la spec dans le dépôt du plugin avant d'ajouter le code fonctionnel.

Le fichier versionné dans le plugin devient ensuite la source de vérité pour l'implémentation :

```text
/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/specification.md
```

La copie du Vault reste une trace de conception. Toute modification ultérieure de la spec pendant l'implémentation doit d'abord être effectuée ou répercutée dans `docs/specification.md`, puis synchronisée vers le Vault si elle concerne également cette documentation.

Le commit attendu pour l'import initial est :

```text
docs(spec): add plugin implementation specification
```

## 18.2.3 Migration des ressources existantes

| Source actuelle dans le Vault | Destination dans le dépôt du plugin |
| --- | --- |
| `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/latex-unicode-symbols.tex` | `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/resources/common/latex-unicode-symbols.tex` |
| `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/White Paper/ffe-callouts.lua` | Référence pour `/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/resources/common/obsidian-callouts.lua` ; le fichier n'est pas copié tel quel |
| `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/White Paper/ffe-white-paper.yaml` | Éléments génériques vers `resources/common/base.yaml` et `resources/modes/branded.yaml` ; éléments FFE vers `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings/Full Front-End/branding.yaml` |
| `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/White Paper/ffe-white-paper-header.tex` | Éléments génériques vers `resources/modes/branded-header.tex` ; éléments FFE vers `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/Pandoc/Brandings/Full Front-End/branding-header.tex` |
| `/Users/fullfrontend/OpenCloud/Personnel/Obsidian Vaults/Full Front-End/99 - TOOLS/bin/build-white-paper` | Conservé temporairement dans le Vault ; aucune copie dans le plugin sauf décision explicite d'ajouter une CLI générique |

Les chemins relatifs mentionnés dans la colonne de destination sont toujours résolus depuis :

```text
/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher
```

## 18.3 Symlink de développement

En développement, Obsidian charge directement le dépôt local grâce à un lien symbolique :

```text
<Vault>/.obsidian/plugins/pandoc-publisher
    → /Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher
```

Le lien cible la racine du dépôt, où le build local génère `main.js`. Le fichier `manifest.json` et `styles.css` se trouvent également à cette racine, conformément au format attendu par Obsidian.

Commande conceptuelle :

```bash
ln -s \
  "/chemin/vers/pandoc-publisher" \
  "/chemin/vers/le-vault/.obsidian/plugins/pandoc-publisher"
```

Le script `scripts/link-vault.sh` automatise cette opération :

- il reçoit le chemin du Vault en argument ou via une variable dédiée telle que `OBSIDIAN_VAULT_PATH` ;
- il vérifie que la cible est bien `.obsidian/plugins/pandoc-publisher` ;
- il refuse d'écraser un dossier ou un lien existant sans confirmation explicite ;
- il crée le dossier `.obsidian/plugins` s'il manque ;
- il affiche la cible finale du lien après création.

Le chemin local du Vault n'est jamais codé en dur dans les fichiers versionnés.

## 18.4 Séparation des données locales

Obsidian écrit les réglages du plugin dans `data.json`. En développement, le symlink implique que ce fichier apparaît à la racine du dépôt local du plugin.

Le dépôt du plugin doit ignorer au minimum :

```gitignore
node_modules/
main.js
data.json
*.log
.DS_Store
```

`data.json` ne doit jamais être publié sur GitHub, car il peut contenir :

- des chemins locaux ;
- le chemin du dossier local de brandings et l'identifiant du branding par défaut ;
- des références vers des logos ou ressources privées ;
- des préférences propres au Vault.

Les logos, noms, couleurs et ressources des brandings métier ou client ne sont jamais ajoutés au dépôt public. Ils restent dans les packs présents dans le Vault de leur propriétaire. `data.json` ne contient pas le contenu des packs. Seuls les assets volontairement neutres du branding `generic` sont publics.

Le branding Full Front-End n'est jamais intégré au dépôt, au build, aux fixtures publiées ou aux releases GitHub. Les tests publics utilisent uniquement `generic` ou une identité fictive sans actifs Full Front-End.

## 18.5 Règles pour le dépôt du Vault

Le chemin suivant doit être ignoré par le dépôt du Vault :

```gitignore
.obsidian/plugins/pandoc-publisher
```

Le symlink ne doit pas être commité, car sa cible est propre à la machine de développement.

Le Vault conserve uniquement :

- les notes et documents à publier ;
- les éventuelles ressources privées référencées par un branding local ;
- la présente spécification ;
- le script historique `build-white-paper` jusqu'à la fin de la migration.

## 18.6 Build et installation

### Développement local

1. installer les dépendances dans le dépôt du plugin ;
2. créer le symlink vers le Vault ;
3. lancer le build en mode watch ;
4. recharger le plugin dans Obsidian après chaque build significatif.

### Release GitHub

Une release contient au minimum :

- `main.js` ;
- `manifest.json` ;
- `styles.css` si nécessaire ;
- les ressources Pandoc, LaTeX et Lua requises par le plugin ;
- le branding global `generic` complet ;
- `README.md`, `docs/user-guide.md` et `docs/branding-pack-guide.md`.

Le workflow GitHub vérifie que la version de `manifest.json` correspond au tag de release et produit une archive installable sans le dépôt du Vault.

### Installation utilisateur

Le symlink est réservé au développement. Une installation normale utilise les fichiers d'une release GitHub, copiés dans `.obsidian/plugins/pandoc-publisher/`, ou un mécanisme de distribution compatible avec les plugins Obsidian.

## 18.7 Convention de commits

Tous les commits du dépôt du plugin respectent **Conventional Commits**.

Format obligatoire :

```text
<type>(<scope>): <description>
```

Pour un changement sans scope utile :

```text
<type>: <description>
```

Types autorisés :

| Type | Usage |
| --- | --- |
| `feat` | Nouvelle fonctionnalité visible ou nouvelle capacité |
| `fix` | Correction d'un comportement incorrect |
| `docs` | Documentation uniquement |
| `refactor` | Restructuration sans changement fonctionnel attendu |
| `test` | Ajout ou correction de tests |
| `build` | Build, dépendances ou packaging |
| `ci` | Workflows GitHub et automatisation continue |
| `chore` | Maintenance ne relevant pas des autres types |
| `perf` | Amélioration mesurable des performances |
| `style` | Formatage sans changement de comportement |
| `revert` | Annulation explicite d'un commit précédent |

Scopes recommandés :

- `spec` ;
- `plugin` ;
- `publishing` ;
- `pandoc` ;
- `latex` ;
- `unicode` ;
- `callouts` ;
- `branding` ;
- `internal` ;
- `markdown` ;
- `settings` ;
- `ui` ;
- `tests` ;
- `release` ;
- `repo`.

Exemples :

```text
chore(repo): initialize standalone plugin repository
docs(spec): add plugin implementation specification
feat(publishing): add branded PDF export command
feat(branding): add client branding editor
feat(callouts): render Obsidian callouts in LaTeX
fix(markdown): resolve embeds with accented paths
test(callouts): cover nested and foldable callouts
ci(release): package Obsidian plugin artifacts
```

Règles complémentaires :

- la description est courte, à l'impératif et sans point final ;
- les messages du dépôt sont rédigés en anglais pour rester cohérents avec un projet GitHub public ;
- un commit reste atomique et ne mélange pas plusieurs intentions indépendantes ;
- les tests et la documentation liés à une fonctionnalité peuvent faire partie du même commit fonctionnel lorsqu'ils décrivent exactement le même changement ;
- un changement incompatible utilise `!`, par exemple `feat(branding)!: replace branding schema` ;
- le corps ou le footer contient `BREAKING CHANGE:` lorsque le changement nécessite une migration ;
- les commits `fix` entraînent une version patch, les commits `feat` une version minor et les breaking changes une version major ;
- les commits `docs`, `test`, `refactor`, `build`, `ci`, `chore` et `style` ne déclenchent pas seuls une nouvelle version fonctionnelle, sauf configuration contraire du workflow de release ;
- les commits temporaires de type `WIP` ne sont pas conservés dans l'historique publié.

Le changelog et les notes de release peuvent être générés à partir de ces commits.

## 18.8 Documentation livrée

La documentation fait partie du produit et doit être complète avant la première release distribuée à un client.

### 18.8.1 README

`README.md` fournit une entrée courte :

- objectif du plugin ;
- prérequis Pandoc, XeLaTeX, Eisvogel et polices ;
- installation depuis une release GitHub ;
- activation dans Obsidian ;
- premier export avec le branding global `generic` ;
- configuration d'un dossier de packs locaux ;
- liens vers le guide utilisateur et le guide de création des packs ;
- compatibilité, licence et procédure de signalement d'un bug.

### 18.8.2 Guide utilisateur

Chemin :

```text
/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/user-guide.md
```

Le guide couvre au minimum :

1. installation du plugin ;
2. installation et test de Pandoc et XeLaTeX sur macOS, Windows et Linux lorsque pris en charge ;
3. réglages globaux ;
4. choix du dossier local des brandings ;
5. différences entre branding global et packs locaux ;
6. choix du branding par défaut ;
7. commandes de publication ;
8. modes `branded` et `internal` ;
9. propriété frontmatter `branding` avec exemples ;
10. propriété frontmatter `pdf-mode` ;
11. syntaxe des callouts Obsidian et rendu PDF ;
12. gestion des images, wikilinks, checkboxes et Unicode ;
13. emplacement et remplacement des PDF ;
14. installation d'un pack reçu ;
15. duplication du pack `generic` ;
16. erreurs courantes et diagnostic ;
17. mise à jour du plugin ;
18. confidentialité des packs et données locales.

Exemples obligatoires :

```yaml
---
pdf-mode: branded
branding: generic
---
```

```yaml
---
pdf-mode: branded
branding: client-example
---
```

### 18.8.3 Guide de création d'un pack de branding

Chemin :

```text
/Users/fullfrontend/Jobs/A - Full Front-End/tooling/pandoc-publisher/docs/branding-pack-guide.md
```

Le guide couvre au minimum :

1. structure complète d'un pack ;
2. schéma et version de `branding.yaml` ;
3. règles pour `id`, `name` et `version` ;
4. couleurs et formats acceptés ;
5. polices et vérification de leur disponibilité ;
6. logo, dimensions et formats d'images ;
7. en-têtes et pieds de page ;
8. page de titre, table des matières et pagination ;
9. configuration de tous les rôles de callouts ;
10. defaults Pandoc supplémentaires ;
11. headers LaTeX ;
12. filtres Lua et avertissement de sécurité ;
13. chemins relatifs et confinement dans le dossier du pack ;
14. validation depuis le plugin ;
15. test sur une note exemple ;
16. duplication du branding `generic` comme point de départ ;
17. préparation d'un pack autonome à transmettre ;
18. installation du pack dans un autre Vault ;
19. mise à jour et compatibilité de version d'un pack ;
20. résolution des erreurs de validation.

Le guide contient un exemple commenté complet correspondant au branding global `generic`. Chaque champ du manifeste est documenté avec son type, son caractère obligatoire ou facultatif, sa valeur par défaut et au moins un exemple.

### 18.8.4 Exigences de qualité documentaire

- les commandes et chemins sont copiables ;
- les captures ne montrent aucune donnée client ou ressource privée ;
- aucun exemple public n'utilise la charte Full Front-End ;
- les exemples sont testés dans l'intégration continue lorsque cela est automatisable ;
- les liens internes de la documentation sont vérifiés ;
- toute nouvelle option modifie la documentation dans le même commit ou dans un commit `docs` associé ;
- une release client n'est pas considérée terminée si les guides ne décrivent pas son comportement réel.

# 19. Décisions actées

- le plugin est dédié à la publication Markdown vers PDF ;
- XeLaTeX est le seul moteur PDF ;
- Unicode est pris en charge par tous les modes et brandings ;
- le plugin ne distribue aucun branding métier ou client ;
- le plugin distribue un branding global `generic`, complet, neutre et duplicable comme exemple ;
- le dossier local des packs de branding est configurable séparément dans chaque Vault ;
- le branding par défaut est sélectionnable entre `generic` et tous les packs locaux valides ;
- la propriété frontmatter `branding` remplace le branding par défaut pour la note concernée ;
- un branding explicitement demandé mais absent produit une erreur et ne déclenche aucun fallback ;
- Full Front-End est le branding par défaut uniquement dans le Vault Full Front-End, parce que son pack y est installé localement ;
- les clients peuvent avoir leur propre branding ;
- un client reçoit le plugin générique et uniquement son pack copiable dans son Vault ;
- un branding absent du dossier local ne peut ni être affiché ni être utilisé ;
- devis, comptes rendus, offres et whitepapers utilisent la même base brandée ;
- tous les documents brandés ont une page de titre, une table des matières et une numérotation des pages ;
- les callouts Obsidian sont pris en charge dans tous les documents ;
- dans le branding FFE, `info` signifie « Trucs et astuces » et conserve l'encart bleu existant ;
- dans le branding FFE, `tip` signifie « Le regard de Fullfrontend » et conserve l'encart jaune existant ;
- les autres callouts utilisent la palette étendue issue du système de couleur narratif FFE ;
- le plugin ne reconnaît aucun ancien encart fondé sur un libellé de blockquote ;
- le whitepaper de référence est migré vers la syntaxe Obsidian uniquement après l'implémentation et les tests du plugin ;
- les documents internes utilisent une mise en page compacte séparée ;
- le frontmatter contient principalement le contenu et la sélection du mode ou du branding ;
- la configuration partagée est la source de vérité du rendu ;
- le plugin possède un dépôt Git et un remote GitHub indépendants du Vault ;
- la source, les tests et les ressources génériques intégrées du plugin vivent hors du Vault, tandis que les packs de branding privés vivent dans leur Vault respectif ;
- le Vault charge le dépôt local au moyen d'un symlink non versionné pendant le développement ;
- les releases GitHub sont autonomes et ne dépendent d'aucun fichier du Vault ;
- les réglages locaux et brandings clients ne sont jamais publiés dans le dépôt du plugin ;
- la spec est copiée dans `docs/specification.md` et commitée avant l'implémentation fonctionnelle ;
- le dépôt contient un guide utilisateur complet et un guide complet de création et distribution des packs ;
- tous les commits du plugin respectent Conventional Commits et utilisent des descriptions en anglais.

# 20. Questions laissées pour l'implémentation

Ces points ne bloquent pas l'architecture, mais devront être tranchés pendant le prototype :

- faut-il afficher le numéro sur la première page de contenu après la couverture ou commencer visuellement à la page suivante ?
- le mode interne doit-il activer sa numérotation des pages par défaut ou seulement à partir de deux pages ?
- faut-il ouvrir automatiquement le PDF après publication ou uniquement afficher une action ?
- faut-il intégrer une bibliothèque d'icônes dans les callouts PDF ou privilégier des titres textuels plus robustes ?

# 21. Références

- Documentation officielle des callouts Obsidian : <https://obsidian.md/help/callouts>
- Manuel Pandoc : <https://pandoc.org/MANUAL.html>
- Template Eisvogel : installation actuelle dans `~/.pandoc/templates/eisvogel.latex`
- Script existant : `99 - TOOLS/bin/build-white-paper`
- Système de couleur narratif : `01 - Projets/04 - Production de Contenu/95 - Assets Communs/01 - Gouvernance éditoriale/Système de couleur narratif - FFE.md`
- Charte graphique : `02 - Aires/Pilotage Interne Business/01 - Branding/00 - Références/FFE Charte Graphique.md`
- Whitepaper de référence : `01 - Projets/04 - Production de Contenu/05 - White Papers/01 - Commence simple/Le guide de ta première vidéo - De je n'ose pas à j'ai publié.md`
