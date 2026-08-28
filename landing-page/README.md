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

## Aller plus loin : migrer vers un vrai template de thème (`home.hbs`)

La solution actuelle reste un unique bloc "HTML card" Ghost : pratique et rapide à mettre à jour, mais toujours un seul fichier monolithique édité à la main dans l'admin Ghost, sans preview ni diff propre.

La solution la plus pérenne est de migrer ce contenu dans le thème Ghost actif, sous forme de template `home.hbs`. L'avantage : les produits phares peuvent eux aussi devenir de simples **posts Ghost** (avec un tag dédié, ex. `#product`), et Ghost les injecte nativement — plus aucun tableau JS à maintenir, tout se fait depuis l'admin Ghost comme un article de blog classique.

Exemple de structure (à adapter selon le thème actif) :

```handlebars
{{!< default}}

{{! ... hero, services, process, story, contact : contenu statique, identique au HTML actuel ... }}

<div class="ls-catalog__grid">
  {{#get "posts" filter="tag:hash-product" limit="3" as |products|}}
    {{#foreach products}}
      <article class="ls-product">
        <div class="ls-product__visual">
          <img src="{{feature_image}}" alt="{{title}}">
        </div>
        <div class="ls-product__body">
          <h3 class="ls-product__name">{{title}}</h3>
          <p class="ls-product__desc">{{excerpt}}</p>
        </div>
      </article>
    {{/foreach}}
  {{/get}}
</div>

<div class="ls-blog__grid">
  {{#get "posts" filter="tag:hash-homepage-feature" limit="3" as |articles|}}
    {{#foreach articles}}
      {{> "article-card"}}
    {{/foreach}}
  {{/get}}
</div>
```

Étapes pour migrer :
1. Récupérer le thème Ghost actif (Admin → Settings → Design → Edit theme code, ou en zip).
2. Créer `home.hbs` à la racine du thème avec `{{!< default}}` en première ligne.
3. Y transposer le CSS de `losiam-home.html` (dans `<style>` ou dans le fichier CSS du thème).
4. Remplacer les sections "produits" et "articles" par les helpers `{{#get "posts"}}` ci-dessus.
5. Re-uploader le thème, l'activer, puis vérifier que la page d'accueil (`routes.yaml` ou réglage "homepage") pointe bien vers ce template.

Cette migration demande un peu plus de rigueur (accès au thème, test en environnement de dev si possible) — c'est pour ça qu'on la fait dans un second temps, une fois que la v3 (ce dossier) aura prouvé son confort d'usage au quotidien.
