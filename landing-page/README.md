# LOSIAM — Landing page (homepage Ghost)

Ce dossier contient le code source de la page d'accueil de [losiam.com](https://www.losiam.com), utilisée comme bloc "HTML card" dans Ghost. `losiam-home.html` est la version **v3**, refactorée pour que les mises à jour courantes (nouveaux articles, produits phares) ne demandent plus de toucher au HTML/CSS.

## Ce qui a changé par rapport à la version précédente

| Avant | Maintenant |
|---|---|
| Chaque produit phare = un bloc HTML complet à copier/coller et modifier à la main | Un tableau `PRODUCTS` en bas du fichier, un objet = un produit |
| Les 3 derniers articles étaient recopiés à la main à chaque publication | Récupérés automatiquement depuis Ghost via la Content API (à configurer une fois, voir plus bas) |
| Le bandeau défilant (marquee) était une liste `<span>` dupliquée à la main | Un tableau `MARQUEE_ITEMS` |
| ~150 lignes de CSS mort (un premier bloc `.ls-hero` jamais utilisé par le HTML réel) | Supprimé |

Tout le contenu à mettre à jour se trouve dans **un seul endroit** : le bloc `<script>` tout en bas du fichier, sous le commentaire `CONTENU À METTRE À JOUR`. Le reste du fichier (styles, structure des sections) n'a normalement plus besoin d'être touché pour publier du nouveau contenu.

## Comment mettre à jour

### Ajouter / modifier un produit phare

Dans `losiam-home.html`, cherche le tableau `PRODUCTS` et ajoute un objet :

```js
{
  tag: 'New',                 // badge affiché en haut à droite de la photo (facultatif, laisse '' pour aucun badge)
  category: 'Fresh Leaf Absolute',
  name: 'Nom du produit',
  img: 'https://www.losiam.com/content/images/...',
  alt: 'Texte alternatif de l\'image',
  desc: 'Description courte, 1-2 phrases.',
  notes: 'Note 1 · Note 2 · Note 3',
}
```

Pour récupérer l'URL d'une photo : upload-la dans un post brouillon Ghost, clique dessus > "Copy image URL", colle l'URL dans `img`, puis supprime le brouillon (la photo reste hébergée).

### Ajouter / retirer un botanical dans le bandeau défilant

Cherche `MARQUEE_ITEMS` et ajoute/retire une ligne du tableau (une chaîne de texte par botanical).

### Faire apparaître un article dans "From the laboratory journal"

C'est automatique **une fois la configuration ci-dessous faite** : il suffit de publier l'article dans Ghost avec le tag configuré (voir `ARTICLES_TAG_SLUG`). Pas besoin de revenir modifier ce fichier.

### Ajouter un badge sur un article (ex. "New", "Selection Chiang Mai Design Week 2026")

Ajoute au post, dans Ghost, un second tag interne au format `#badge:<texte du badge>` — par exemple `#badge:New`. Le badge apparaît automatiquement en haut à droite de la carte de cet article. Pas de tag `#badge:...` sur le post → pas de badge affiché. C'est le même principe que le tag de sélection (`#homepage-feature`) : un tag commençant par `#` est interne à Ghost, invisible pour les lecteurs, il ne sert qu'à piloter l'affichage de la homepage.

Pour les 3 articles de secours (`ARTICLES_FALLBACK`, utilisés uniquement si l'API n'est pas configurée), il n'y a pas de tag à créer : remplis directement le champ `badge` de l'objet correspondant dans le fichier.

## Configuration à faire une seule fois (récupération auto des articles)

Sans cette configuration, la page affiche 3 articles de secours codés en dur (`ARTICLES_FALLBACK`) pour ne jamais casser visuellement la page — mais ils ne se mettront jamais à jour tout seuls.

1. Dans Ghost admin : **Settings → Integrations → Add custom integration** (ou réutilise une intégration existante), copie la **Content API Key**.
2. Dans `losiam-home.html`, renseigne :
   ```js
   const GHOST_API_URL = 'https://www.losiam.com';
   const GHOST_CONTENT_API_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxxxx';
   ```
3. Crée un tag Ghost dédié à la sélection homepage, par exemple `#homepage-feature` (un tag commençant par `#` est un tag *interne*, invisible pour les lecteurs du blog — il sert uniquement à curer ce qui apparaît sur la homepage).
4. Renseigne son slug dans le fichier — un tag `#homepage-feature` devient `hash-homepage-feature` dans l'API Ghost :
   ```js
   const ARTICLES_TAG_SLUG = 'hash-homepage-feature';
   ```
5. Ajoute ce tag aux 3 articles que tu veux voir sur la homepage. Pour en changer, retire/ajoute simplement le tag sur les articles concernés — plus aucune édition de code nécessaire.

La clé Content API est conçue pour être exposée côté client (lecture seule, contenu déjà public) — ce n'est pas un secret à protéger comme une clé Admin API.

## Étape 2 : migration vers un vrai template de thème (`page-landingpage.hbs`)

La solution ci-dessus reste un unique bloc "HTML card" Ghost : pratique et rapide à mettre à jour, mais toujours un fichier monolithique édité à la main dans l'admin Ghost, sans preview ni diff propre, et les produits/articles restent pilotés en JS plutôt que d'être du vrai contenu Ghost.

Le fichier **`theme/page-landingpage.hbs`** de ce dossier est la version pérenne : un vrai template de thème qui remplace la carte HTML. D'après le brief GA4 du 5 octobre 2026, le site tourne sur le thème officiel **Headline** (Ghost 6.x) — ce fichier a été construit et vérifié contre le code source public de ce thème ([TryGhost/Headline](https://github.com/TryGhost/Headline)), qui définit bien `home.hbs`/`page.hbs` et les helpers (`{{#get}}`, `{{img_url}}`, `{{date}}`, `{{#foreach}}`) utilisés ici.

**Pourquoi `page-landingpage.hbs` et pas `home.hbs`** : Ghost sert automatiquement ce fichier pour la page dont le slug est `landingpage` (convention de nommage `page-<slug>.hbs`), exactement la page déjà utilisée aujourd'hui comme accueil via `routes.yaml`. Aucune modification de `routes.yaml` n'est donc nécessaire — contrairement à un `home.hbs`, qui aurait demandé de changer le routage.

**Ce que ce template remplace** : les produits phares et les 3 derniers articles ne sont plus des tableaux JS ni un fetch côté client — ce sont de vrais **posts Ghost**, injectés nativement avec `{{#get "posts"}}`. Publier un produit ou sélectionner un article pour la home se fait depuis l'éditeur Ghost normal. Le bandeau défilant (marquee) reste piloté par un petit tableau JS en bas du fichier, car ce n'est pas de l'éditorial.

### Convention de tags à respecter

Le Handlebars de Ghost ne peut pas parser un texte comme `#badge:New` (ça demanderait un helper JS personnalisé que les thèmes Ghost ne permettent pas) : la lecture se fait donc **par position dans la liste des tags du post**, pas par préfixe de texte. Tague chaque post dans cet ordre exact :

**Produits phares** (sélectionnés par le tag interne `#product`) :
1. un tag public = catégorie (ex. `Fresh Flowers Absolute`) → affiché au-dessus du nom
2. *(optionnel)* un tag public = badge/ruban (ex. `Ultra-rare`) → affiché sur la photo
3. un tag public par note olfactive (ex. `Floral`, `Delicate`, `Songkran heritage`) → affichés en bas, séparés par " · "
4. **en dernier**, le tag interne `#product` (slug Ghost : `hash-product`) — c'est lui qui sélectionne le post pour cette grille

Champs du post : titre = nom du produit, image de une = photo du produit, extrait (excerpt) = description courte.

**Articles "From the laboratory journal"** (sélectionnés par le tag interne `#homepage-feature`) :
1. un tag public = catégorie (ex. `Material`)
2. *(optionnel)* un tag public = badge (ex. `New`, `Selection Chiang Mai Design Week 2026`)
3. **en dernier**, le tag interne `#homepage-feature` (slug Ghost : `hash-homepage-feature`)

Pourquoi "en dernier" : dans l'éditeur Ghost, un tag qu'on ajoute s'insère à la fin de la liste — il suffit donc d'ajouter le tag de sélection (`#product` / `#homepage-feature`) **après** avoir renseigné catégorie/badge/notes, et l'ordre est automatiquement correct.

⚠️ Ceci change la convention des articles par rapport à l'étape 1 : le badge n'est plus un tag au format texte `#badge:...` mais le 2ᵉ tag public de la liste. Si tu actives ce template, re-tague les 3 articles déjà sélectionnés en conséquence.

### Déployer ce template

1. Ghost admin → **Settings → Design → Edit theme code → Download** (récupère le zip du thème actif).
2. Ajoute `theme/page-landingpage.hbs` (ce fichier) à la racine du zip.
3. Re-zippe, puis dans le même écran : **Upload → Save & activate**. Tu peux d'abord l'uploader sous un autre nom de thème pour le tester sans toucher au thème actif en production.
4. Crée les 2 tags internes `#product` et `#homepage-feature` dans Ghost s'ils n'existent pas déjà, et tague au moins 3 produits + 3 articles **avant** d'activer — sinon les grilles s'afficheront vides (pas de contenu de secours sur cette version, puisque les produits sont maintenant du vrai contenu Ghost).
5. Une fois actif, le contenu de la page Ghost "landingpage" (carte HTML) n'est plus utilisé : ce template n'appelle jamais `{{content}}`. Tu peux le laisser tel quel ou le vider.

### Limite connue

Ce fichier a été construit à partir du code source **public** du thème Headline, pas d'un export du thème réellement actif sur losiam.com — si ce thème a été personnalisé au-delà des réglages standards de Ghost (Settings → Design), il peut y avoir de petits conflits CSS à ajuster après l'upload. Teste d'abord sous un nom de thème différent pour vérifier le rendu avant d'activer en production.

## Correctif : deux balises H1 sur la homepage

Constaté le 12 octobre 2026 : une fois `page-landingpage.hbs` actif, la page contient deux `<h1>` — le logo du header ("Le Laboratoire Olfactif du Siam - LOSIAM") et le vrai titre de la page ("Rare Thai botanicals, captured in premium natural absolutes"). Une seule balise H1 par page est la bonne pratique SEO ; le logo ne doit pas en être une.

**Cause** : `default.hbs` (le header, partagé par toutes les pages du thème) wrappe le logo dans un `<h1>` uniquement sur la page d'accueil — un pattern standard dans les thèmes Ghost, mais qui entre en conflit avec le vrai H1 défini dans `page-landingpage.hbs`.

**Fichier** : `theme/default.hbs` dans ce dossier — le `default.hbs` public du thème Headline avec un patch de 3 lignes (détaillé en commentaire en tête du fichier, retiré automatiquement par Ghost au rendu) : la classe `gh-head-logo` est appliquée directement au lien `<a>` au lieu d'être portée par un `<h1>` conditionnel. Aucun changement visuel attendu (le style ne dépendait pas de la balise elle-même).

**Déploiement** : même procédure que pour `page-landingpage.hbs` — ajouter ce fichier à la racine du zip du thème (il remplace le `default.hbs` existant), re-zipper, uploader, tester, puis activer. ⚠️ Si ton `default.hbs` actuel a été personnalisé au-delà du logo (nav, footer, etc.), ne le remplace pas en bloc : applique seulement les 3 lignes indiquées dans le commentaire du fichier à TON fichier réel.

**Vérification après déploiement** :
1. Affiche le code source de `https://www.losiam.com/` et confirme qu'il n'y a plus qu'un seul `<h1>`, celui du hero ("Rare Thai botanicals...").
2. Vérifie la balise `modified_time` pour confirmer que le cache Fastly sert bien la nouvelle version avant de déclarer le correctif effectif.
3. Vérifie que le logo du header a toujours le même rendu visuel qu'avant sur toutes les pages (pas seulement l'accueil).
