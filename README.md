# WLED LED Library

Catalogue partagé des produits LED utilisés avec [WLED Fleet](https://github.com/Tensegrity-Lighting-Service/WLED-Fleet).

Un **produit** décrit ce qui est branché sur une sortie : type de LED, ordre des
couleurs, échange du blanc, mA par LED, LEDs sautées, off refresh, LEDs par
mètre, et une ou plusieurs longueurs types.

Ce qui relève de l'**installation** — sens de parcours, index de départ,
univers, adresse DMX — n'y est délibérément pas. C'est ce qui permet
d'appliquer un produit à une sortie sans jamais casser un patch existant.

## Organisation

```
products/<uuid>.json     un fichier par produit
```

Le nom du fichier est l'**uuid** du produit, pas son nom commercial. Deux
personnes hors ligne peuvent donc créer des produits en même temps sans se
disputer un nom de fichier, et renommer un produit ne déplace jamais son
fichier.

## Format d'un produit

```jsonc
{ "format": "wled-led-product",
  "formatVersion": 3,

  "uid": "d8972dbb-1383-4f3c-b0c0-c1ea6a2dc378",  // clé, à vie
  "rev": 4,                    // monte quand les RÉGLAGES changent
  "basedOnRev": 3,             // révision sur laquelle celle-ci s'appuie
  "slug": "ledpro-flex60",     // lisible, figé à la création
  "legacyId": null,            // ancien identifiant court, s'il a existé

  "ref": { "brand": "LEDpro", "model": "Flex60", "sku": "LP-F60-2815",
           "internal": "TLS-RUB-012", "note": "bobine 5 m, IP67, 24 V" },

  "led": { "type": 22,         // type WLED (hw.led.ins[].type)
           "order": 1,         // ordre des couleurs — quartet BAS de `order`
           "wswap": 0,         // échange du blanc — quartet HAUT
           "ledma": 55,        // mA par LED à pleine luminosité
           "skip": 0,          // LEDs sautées en tête de câble
           "offRefresh": false,
           "perM": 60 },       // LEDs par mètre, pour nommer les longueurs

  "presets": [ { "label": "2 m", "px": 120, "default": true },
               { "label": "5 m", "px": 300 } ],

  "retired": false,            // retiré du catalogue, mais jamais effacé
  "updatedAt": 1757337600000,
  "updatedBy": "" }
```

`led` reprend exactement le vocabulaire de `hw.led.ins[]` dans la configuration
WLED : appliquer un produit est une copie de champs, sans traduction.

## Identifiants et révisions

**Un `uid` n'est jamais réattribué.** Les nodes portent ce marqueur dans leur
`/fleet.json` ; le recycler ferait pointer d'anciennes installations vers un
autre produit. Un produit qu'on ne veut plus proposer est marqué
`"retired": true`, il n'est pas supprimé.

**`rev` monte quand `led` ou `presets` changent, pas quand `ref` change.**
Corriger une faute de frappe dans un nom ne doit pas signaler « en retard » à
tous les nodes déjà patchés avec ce produit.

Un node mémorise la révision qui a servi à le régler. Comparer les deux répond à
une question que le nom seul ne permet pas de poser : ce node porte-t-il
vraiment les réglages actuels, ou ceux d'une version antérieure ?

## Écritures concurrentes

Plusieurs postes alimentent ce dépôt, souvent sans se voir. Les écritures
passent par l'API contents de GitHub avec le `sha` de la version qu'on croit
remplacer : si le fichier a changé entre-temps, GitHub refuse et n'écrit rien.

Sur refus, WLED Fleet ne rend pas la main : il reprend la version en ligne comme
base, y réapplique ses propres réglages, et repart à `rev = celle d'en ligne + 1`
en notant `basedOnRev`. Le travail des deux côtés survit, la version remplacée
reste dans l'historique git, et **aucun numéro de révision ne désigne jamais deux
contenus différents** — ce dont dépend tout le reste, puisque c'est ce numéro qui
dit aux nodes s'ils sont à jour.

## Modifier le catalogue

Depuis WLED Fleet, onglet **Bibliothèque** : éditer un produit, puis
« ↑ Publier ». « ↓ Rafraîchir » tire le dépôt sans jamais écraser un produit
modifié localement et pas encore publié.

Une modification à la main dans ce dépôt est possible — c'est du JSON — mais
elle doit respecter les deux règles ci-dessus : ne pas réutiliser un `uid`, et
faire monter `rev` dès que `led` ou `presets` changent.

## Voir aussi

- [WLED Fleet](https://github.com/Tensegrity-Lighting-Service/WLED-Fleet)
- [`docs/fixture-mapping.md`](https://github.com/Tensegrity-Lighting-Service/WLED-Fleet/blob/main/docs/fixture-mapping.md)
  — ce que Fleet écrit sur les nodes, et comment un outil tiers le relit
- [`docs/api.md`](https://github.com/Tensegrity-Lighting-Service/WLED-Fleet/blob/main/docs/api.md)
  — l'API de Fleet, générée depuis son code
