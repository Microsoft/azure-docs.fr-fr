---
title: Configurer la conversion de modèle
description: Description de tous les paramètres de conversion de modèle
author: florianborn71
ms.author: flborn
ms.date: 03/06/2020
ms.topic: how-to
ms.openlocfilehash: 1cb5312e164bac09930497c377f1590b6a77ca05
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92205317"
---
# <a name="configure-the-model-conversion"></a>Configurer la conversion de modèle

Ce chapitre décrit les options de conversion de modèle.

## <a name="settings-file"></a>Fichier de paramètres

Si un fichier appelé `<modelName>.ConversionSettings.json` se trouve dans le conteneur d’entrée en regard du modèle d’entrée `<modelName>.<ext>`, il sera utilisé pour fournir une configuration supplémentaire au processus de conversion de modèle.
Par exemple, `box.ConversionSettings.json` pourrait être utilisé lors de la conversion de `box.gltf`.

Le contenu du fichier doit respecter le schéma JSON suivant :

```json
{
    "$schema" : "http://json-schema.org/schema#",
    "description" : "ARR ConversionSettings Schema",
    "type" : "object",
    "properties" :
    {
        "scaling" : { "type" : "number", "exclusiveMinimum" : 0, "default" : 1.0 },
        "recenterToOrigin" : { "type" : "boolean", "default" : false },
        "opaqueMaterialDefaultSidedness" : { "type" : "string", "enum" : [ "SingleSided", "DoubleSided" ], "default" : "DoubleSided" },
        "gammaToLinearMaterial" : { "type" : "boolean", "default" : false },
        "gammaToLinearVertex" : { "type" : "boolean", "default" : false },
        "sceneGraphMode": { "type" : "string", "enum" : [ "none", "static", "dynamic" ], "default" : "dynamic" },
        "generateCollisionMesh" : { "type" : "boolean", "default" : true },
        "unlitMaterials" : { "type" : "boolean", "default" : false },
        "fbxAssumeMetallic" : { "type" : "boolean", "default" : true },
        "deduplicateMaterials" : { "type" : "boolean", "default" : true },
        "axis" : {
            "type" : "array",
            "items" : {
                "type" : "string",
                "enum" : ["default", "+x", "-x", "+y", "-y", "+z", "-z"]
            },
            "minItems": 3,
            "maxItems": 3
        },
        "metadataKeys": {
            "type": "array",
            "items": {
                "type": "string"
            }
        }
    },
    "additionalProperties" : false
}
```

Voici un exemple de fichier `box.ConversionSettings.json` :

```json
{
    "scaling" : 0.01,
    "recenterToOrigin" : true,
    "material-override" : "box_materials_override.json"
}
```

### <a name="geometry-parameters"></a>Paramètres Geometry

* `scaling` : ce paramètre met à l’échelle un modèle uniformément. La mise à l’échelle peut être utilisée pour augmenter ou réduire un modèle, par exemple pour afficher un modèle de construction sur une table.
La mise à l’échelle est également importante quand un modèle est défini dans des unités différentes du mètre, car le moteur de rendu attend des mètres.
Par exemple, si un modèle est défini en centimètres, l’application d’une échelle de 0,01 doit afficher le modèle à la taille correcte.
Certains formats de données sources (par exemple .fbx) fournissent un indicateur de mise à l’échelle des unités, auquel cas la conversion met à l’échelle implicitement le modèle en mètres. La mise à l’échelle implicite fournie par le format source est appliquée en plus du paramètre de mise à l’échelle.
Le facteur de mise à l’échelle final est appliqué aux vertex géométriques et aux transformations locales des nœuds de graphique de scène. La mise à l’échelle de la transformation de l’entité racine reste inchangée.

* `recenterToOrigin` : indique qu’un modèle doit être converti pour que son cadre englobant soit centré à l’origine.
Si un modèle source est éloigné de l’origine, les problèmes de précision à virgule flottante peuvent entraîner des artefacts de rendu.
Le centrage du modèle peut aider dans cette situation.

* `opaqueMaterialDefaultSidedness` : le moteur de rendu part du principe que les matériaux opaques sont recto verso.
Si cette hypothèse ne s’applique pas pour un modèle particulier, le paramètre doit être défini sur « SingleSided ». Pour plus d’informations, consultez [Rendu :::no-loc text="single sided":::](../../overview/features/single-sided-rendering.md).

### <a name="material-de-duplication"></a>Déduplication de matériaux

* `deduplicateMaterials` : ce paramètre active ou désactive la déduplication automatique des matériaux qui partagent les mêmes propriétés et textures. La déduplication se produit après le traitement des remplacements de matériaux. Il est activé par défaut.

* Si, même après la déduplication, un modèle contient plus de 65 535 matériaux, le service tente de fusionner les matériaux présentant des propriétés similaires. En dernier recours, les matériaux dépassant la limite seront remplacés par un matériau d’erreur rouge.

![Image montre deux cubes de 68 921 triangles de couleur.](media/mat-dedup.png?raw=true)

Deux cubes 68 921 triangles de couleur. Gauche : Avant la déduplication avec 68 921 matériaux de couleur. Droite : Après la déduplication avec 64 000 matériaux de couleur. La limite est de 65 535 matériaux. (Voir [limites](../../reference/limits.md).)

### <a name="color-space-parameters"></a>Paramètres d’espace colorimétrique

Le moteur de rendu s’attend à ce que les valeurs de couleur soient dans l’espace linéaire.
Si un modèle est défini à l’aide de l’espace gamma, ces options doivent être définies sur true.

* `gammaToLinearMaterial` : convertir les couleurs de matériau de l’espace gamma en espace linéaire
* `gammaToLinearVertex` : convertir les couleurs :::no-loc text="vertex"::: de l’espace gamma en espace linéaire

> [!NOTE]
> Pour les fichiers FBX, ces paramètres sont définis sur `true` par défaut. Pour tous les autres types de fichiers, la valeur par défaut est `false`.

### <a name="scene-parameters"></a>Paramètres de scène

* `sceneGraphMode` : définit la manière dont le graphique de scène dans le fichier source est converti :
  * `dynamic` (par défaut) : Tous les objets du fichier sont exposés en tant [qu’entités](../../concepts/entities.md) dans l’API et peuvent être transformés indépendamment. La hiérarchie de nœuds au moment de l’exécution est identique à la structure dans le fichier source.
  * `static`: Tous les objets sont exposés dans l’API, mais ils ne peuvent pas être transformés indépendamment.
  * `none`: Le graphique de scène est réduit en un seul objet.

Chaque mode a des performances d’exécution différentes. En mode `dynamic`, le coût des performances est mis à l’échelle de manière linéaire avec le nombre [d’entités](../../concepts/entities.md) dans le graphique, même si aucune partie n’est déplacée. Utilisez le mode `dynamic` uniquement lorsqu’il est nécessaire de déplacer des parties individuellement, par exemple pour une animation « vue en éclaté ».

Le mode `static` exporte le graphique complet de la scène, mais les parties à l’intérieur de ce graphique ont une transformation constante par rapport à la partie racine. Toutefois, le nœud racine de l’objet peut toujours être déplacé, pivoté ou mis à l’échelle sans coût de performance significatif. En outre, les [requêtes spatiales](../../overview/features/spatial-queries.md) retournent des parties individuelles et chaque composant peut être modifié par le biais de [remplacements d’état](../../overview/features/override-hierarchical-state.md). Avec ce mode, la charge d’exécution par objet est négligeable. Il est idéal pour les grandes scènes où vous avez toujours besoin d’une inspection par objet, mais sans transformation par objet.

Le mode `none` a la surcharge d’exécution la plus basse et des temps de chargement légèrement meilleurs. L’inspection ou la transformation d’objets uniques n’est pas possible dans ce mode. Les cas d’usage sont, par exemple, des modèles de photogrammétrie qui n’ont pas de graphique de scène significatif en premier lieu.

> [!TIP]
> De nombreuses applications chargent plusieurs modèles. Vous devez optimiser les paramètres de conversion pour chaque modèle en fonction de la façon dont il sera utilisé. Par exemple, si vous souhaitez afficher le modèle d’une voiture que l’utilisateur doit démonter et examiner en détail, vous devez le convertir en mode `dynamic`. Toutefois, si vous souhaitez également placer la voiture dans un environnement d’exposition, ce modèle peut être converti avec `sceneGraphMode` défini sur `static`, ou même `none`.

### <a name="physics-parameters"></a>Paramètres physiques

* `generateCollisionMesh` : si vous avez besoin de prendre en charge les [requêtes spatiales](../../overview/features/spatial-queries.md) sur un modèle, cette option doit être activée. Dans le pire des cas, la création d’une maille de collision peut doubler le temps de conversion. Le chargement des modèles avec des maillages de collisions prend plus de temps et, lors de l’utilisation d’un graphique de scène `dynamic`, ils ont également une surcharge de performances d’exécution supérieure. Pour des performances optimales globales, vous devez désactiver cette option sur tous les modèles sur lesquels vous n’avez pas besoin de requêtes spatiales.

### <a name="unlit-materials"></a>Matériaux non éclairés

* `unlitMaterials` : par défaut, la conversion préfère créer des [matériaux PBR](../../overview/features/pbr-materials.md). Cette option indique au convertisseur de traiter tous les matériaux comme des [matériaux couleur](../../overview/features/color-materials.md) à la place. Si vous disposez de données qui incorporent déjà de l’éclairage, comme des modèles créés par photogrammétrie, cette option vous permet d’appliquer rapidement la bonne conversion pour tous les matériaux, sans avoir à [remplacer chaque matériau](override-materials.md) individuellement.

### <a name="converting-from-older-fbx-formats-with-a-phong-material-model"></a>Conversion à partir de formats FBX plus anciens, avec un modèle de matériau Phong

* `fbxAssumeMetallic` : les versions antérieures du format FBX définissent leurs documents à l’aide d’un modèle de matériau Phong. Le processus de conversion doit déduire la manière dont ces matériaux sont mappés au [modèle PBR](../../overview/features/pbr-materials.md) du convertisseur. Cela fonctionne généralement bien, mais une ambiguïté peut exister lorsqu’un matériau n’a pas de texture, de valeurs spéculaires élevées et une couleur Albedo non grisée. Dans ce cas, la conversion doit choisir entre prioriser les valeurs spéculaires élevées, définir un matériau très réfléchissant, un matériau métallique où la couleur Albedo est dissoute, ou prioriser la couleur Albedo, en définissant quelque chose comme un plastique à couleur brillante. Par défaut, le processus de conversion suppose que les valeurs hautement spéculaires impliquent un matériau métallique dans les cas où l’ambiguïté s’applique. Ce paramètre peut être défini sur `false` pour basculer vers l’option inverse.

### <a name="coordinate-system-overriding"></a>Substitution du système de coordonnées

* `axis` : pour remplacer les vecteurs d’unités du système de coordonnées. Les valeurs par défaut sont `["+x", "+y", "+z"]`. En théorie, le format FBX contient un en-tête dans lequel ces vecteurs sont définis, et la conversion utilise ces informations pour transformer la scène. Le format glTF définit également un système de coordonnées fixe. Dans la pratique, certaines ressources contiennent des informations incorrectes dans leur en-tête ou ont été enregistrées avec une convention de système de coordonnées différente. Cette option vous permet de remplacer le système de coordonnées pour compenser cela. Par exemple : `"axis" : ["+x", "+z", "-y"]` va échanger l’axe Z et l’axe Y et maintenir l’orientation du système de coordonnées en inversant le sens de l’axe Y.

### <a name="node-meta-data"></a>Métadonnées de nœud

* `metadataKeys` : vous permet de spécifier les clés des propriétés de métadonnées de nœud que vous souhaitez conserver dans le résultat de la conversion. Vous pouvez spécifier des clés exactes ou des clés génériques . Les clés génériques sont au format « ABC* » et correspondent à n’importe quelle clé commençant par « ABC ». Les types de valeurs de métadonnées pris en charge sont `bool`, `int`, `float` et `string`.

    Pour les fichiers GLTF, ces données proviennent de l’[objet extras sur les nœuds](https://github.com/KhronosGroup/glTF/tree/master/specification/2.0#nodeextras). Pour les fichiers FBX, ces données proviennent des données `Properties70` sur `Model nodes`. Pour plus d’informations, consultez la documentation de votre outil de ressource 3D.

### <a name="no-loc-textvertex-format"></a>Format :::no-loc text="Vertex":::

Il est possible d’ajuster le format :::no-loc text="vertex"::: d’un maillage, pour faire des économies de mémoire au détriment de la précision. Un encombrement mémoire plus faible vous permet de charger des modèles plus volumineux ou d’obtenir de meilleures performances. Toutefois, en fonction de vos données, le mauvais format peut avoir un impact significatif sur la qualité du rendu.

> [!CAUTION]
> La modification du format :::no-loc text="vertex"::: ne doit envisagée qu’en dernier recours en cas d’insuffisance de mémoire pour les modèles ou d’optimisation des performances. Les modifications peuvent facilement introduire des artefacts de rendu, qui peuvent être aussi bien évidents que subtils. À moins de savoir ce à quoi vous devez prêter attention, il est déconseillé de modifier la valeur par défaut.

Les ajustements suivants sont possibles :

* Des flux de données spécifiques peuvent être explicitement inclus ou exclus.
* La précision des flux de données peut être diminuée pour réduire l’encombrement mémoire.

La section `vertex` suivante dans le fichier `.json` est facultative. Pour chaque partie qui n’est pas explicitement spécifiée, le service de conversion revient à sa valeur par défaut.

```json
{
    ...
    "vertex" : {
        "position"  : "32_32_32_FLOAT",
        "color0"    : "NONE",
        "color1"    : "NONE",
        "normal"    : "NONE",
        "tangent"   : "NONE",
        "binormal"  : "NONE",
        "texcoord0" : "32_32_FLOAT",
        "texcoord1" : "NONE"
    },
    ...
```

En forçant un composant sur `NONE`, il est garanti que le maillage de sortie n’a pas le flux respectif.

#### <a name="component-formats-per-no-loc-textvertex-stream"></a>Format des composants par flux :::no-loc text="vertex":::

Ces formats sont autorisés pour les composants respectifs :

| Composant :::no-loc text="Vertex"::: | Formats pris en charge (gras = par défaut) |
|:-----------------|:------------------|
|position| **32_32_32_FLOAT**, 16_16_16_16_FLOAT |
|color0| **8_8_8_8_UNSIGNED_NORMALIZED**, NONE |
|color1| 8_8_8_8_UNSIGNED_NORMALIZED, **NONE**|
|normal| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|tangent| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|binormal| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|texcoord0| **32_32_FLOAT**, 16_16_FLOAT, NONE |
|texcoord1| **32_32_FLOAT**, 16_16_FLOAT, NONE |

#### <a name="supported-component-formats"></a>Formats de composant pris en charge

Les empreintes mémoire des formats sont les suivantes :

| Format | Description | Octets par :::no-loc text="vertex"::: |
|:-------|:------------|:---------------|
|32_32_FLOAT|précision à virgule flottante complète à deux composants|8
|16_16_FLOAT|précision à virgule flottante partielle à deux composants|4
|32_32_32_FLOAT|précision à virgule flottante complète à trois composants|12
|16_16_16_16_FLOAT|précision à virgule flottante partielle à quatre composants|8
|8_8_8_8_UNSIGNED_NORMALIZED|octet à quatre composants, normalisé à la plage `[0; 1]`|4
|8_8_8_8_SIGNED_NORMALIZED|octet à quatre composants, normalisé à la plage `[-1; 1]`|4

#### <a name="best-practices-for-component-format-changes"></a>Meilleures pratiques pour les modifications de format de composant

* `position`: Il est rare que la précision réduite soit suffisante. **16_16_16_16_FLOAT** présente des artefacts de quantification perceptibles, même pour les petits modèles.
* `normal`, `tangent`, `binormal` : En général, ces valeurs sont modifiées ensemble. À moins qu’il y ait des artefacts d’éclairage perceptibles découlant d’une quantification normale, il n’y a aucune raison d’augmenter leur précision. Dans certains cas, toutefois, ces composants peuvent être définis sur **NONE** :
  * `normal`, `tangent` et `binormal` sont nécessaires uniquement lorsqu’au moins un matériau du modèle doit être éclairé. Dans ARR, c’est le cas lorsqu’un matériel [PBR](../../overview/features/pbr-materials.md) est utilisé sur le modèle à tout moment.
  * `tangent` et `binormal` sont nécessaires uniquement lorsque l’un des matériaux éclairés utilise une texture cartographique normale.
* `texcoord0`, `texcoord1` : Les coordonnées de texture peuvent utiliser une précision réduite (**16_16_FLOAT**) lorsque leurs valeurs restent dans la plage de `[0; 1]` et que les textures adressées ont une taille maximale de 2048 x 2048 pixels. Si ces limites sont dépassées, la qualité du mappage de texture en pâtira.

#### <a name="example"></a>Exemple

Supposons que vous avez un modèle de photogrammétrie qui a une lumière intégrée dans les textures. Les positions :::no-loc text="vertex"::: et les coordonnées de texture suffisent pour restituer le modèle.

Par défaut, le convertisseur doit supposer que vous pouvez utiliser des matériaux PBR sur un modèle à un moment donné, et il génère donc des données `normal`, `tangent` et `binormal` pour vous. Par conséquent, l’utilisation de la mémoire par vertex est de `position` (12 octets) + `texcoord0` (8 octets) + `normal` (4 octets) + `tangent` (4 octets) + `binormal` (4 octets) = 32 octets. Les grands modèles de ce type peuvent facilement comporter plusieurs millions de :::no-loc text="vertices":::, et ainsi occuper plusieurs gigaoctets de mémoire. Ces grandes quantités de données affectent les performances et vous pourriez même manquer de mémoire.

Sachant que vous n’avez jamais besoin d’éclairage dynamique sur le modèle et que toutes les coordonnées de texture se trouvent dans la plage `[0; 1]`, vous pouvez définir `normal`, `tangent` et `binormal` sur `NONE` et `texcoord0` sur la demi-précision (`16_16_FLOAT`), ce qui ne fait que 16 octets par :::no-loc text="vertex":::. La réduction de moitié des données de maillage vous permet de charger des modèles plus volumineux et d’améliorer potentiellement les performances.

## <a name="memory-optimizations"></a>Optimisations de mémoire

La consommation de mémoire du contenu chargé peut devenir un goulot d’étranglement sur le système de rendu. Une charge utile trop importante de la mémoire risque de compromettre les performances de rendu ou d’empêcher le modèle de se charger entièrement. Ce paragraphe présente quelques stratégies importantes pour réduire l’empreinte mémoire.

### <a name="instancing"></a>Instanciation

L’instanciation est un concept selon lequel les maillages sont réutilisés de sorte que les différentes parties correspondent à des transformations spatiales distinctes, par opposition à la configuration selon laquelle chaque partie référence sa propre géométrie. L’instanciation a un impact significatif sur l’empreinte mémoire.
Parmi ses cas d’usage, citons par exemple les vis d’un modèle de moteur ou les chaises dans un modèle architectural.

> [!NOTE]
> L’instanciation peut améliorer considérablement la consommation de mémoire (et donc les temps de chargement), mais les améliorations obtenues du côté des performances de rendu ne sont pas significatives.

Le service de conversion respecte l’instanciation si les parties concernées sont marquées en conséquence dans le fichier source. Toutefois, la conversion n’effectue pas d’analyse approfondie supplémentaire des données de maillage pour identifier les parties réutilisables. L’outil de création de contenu et son pipeline d’exportation sont donc les critères déterminants d’une configuration d’instanciation appropriée.

Un moyen simple de tester si les informations d’instanciation sont conservées au cours de la conversion consiste à examiner les [statistiques de sortie](get-information.md#example-info-file), et notamment le membre `numMeshPartsInstanced`. Si la valeur de `numMeshPartsInstanced` est supérieure à zéro, cela signifie que les maillages sont partagés entre les instances.

#### <a name="example-instancing-setup-in-3ds-max"></a>Exemple : Configuration de l’instanciation dans 3ds Max

[Autodesk 3ds Max](https://www.autodesk.de/products/3ds-max) possède des modes de clonage d’objets distincts appelés **`Copy`** , **`Instance`** et **`Reference`** qui se comportent différemment en ce qui concerne l’instanciation dans le fichier `.fbx` exporté.

![Clonage dans 3ds Max](./media/3dsmax-clone-object.png)

* **`Copy`**  : Dans ce mode, le maillage est cloné. De ce fait, aucune instanciation n’est utilisée (`numMeshPartsInstanced` = 0).
* **`Instance`**  : Les deux objets partagent le même maillage. L’instanciation est donc utilisée (`numMeshPartsInstanced` = 1).
* **`Reference`**  : des modificateurs distincts peuvent être appliqués aux géométries, de sorte que l’exportateur choisit une approche conservatrice et n’utilise pas l’instanciation (`numMeshPartsInstanced` = 0).


### <a name="depth-based-composition-mode"></a>Mode de composition basé sur la profondeur

Si la mémoire pose problème, configurez le convertisseur avec le [mode de composition à base de profondeur](../../concepts/rendering-modes.md#depthbasedcomposition-mode). Dans ce mode, la charge utile GPU est distribuée sur plusieurs GPU.

### <a name="decrease-vertex-size"></a>Réduction de la taille de vertex

Comme nous l’avons vu dans la section [Meilleures pratiques pour les modifications de format de composant](configure-model-conversion.md#best-practices-for-component-format-changes), la modification du format de vertex peut réduire l’empreinte mémoire. Toutefois, cette option ne doit être envisagée qu’en dernier recours.

### <a name="texture-sizes"></a>Taille des textures

Selon le type de scénario, la quantité de données de texture peut dépasser la mémoire utilisée pour les données de maillage. Les modèles de photogrammétrie sont des candidats.
La configuration de la conversion n’offre aucun moyen de réduire automatiquement les textures. Si nécessaire, la diminution des textures doit être effectuée dans une étape de prétraitement côté client. Toutefois, l’étape de conversion sélectionne un [format de compression de texture](/windows/win32/direct3d11/texture-block-compression-in-direct3d-11) adapté :

* `BC1` pour les textures de couleur opaque
* `BC7` pour les textures de couleur source avec canal alpha

Étant donné que le format `BC7` présente deux fois plus d’empreinte mémoire que `BC1`, il est important de vérifier que les textures d’entrée ne fournissent pas inutilement un canal alpha.

## <a name="typical-use-cases"></a>Études de cas classiques

La recherche de bons paramètres d’importation pour un cas d’usage donné peut être un processus fastidieux. En revanche, les paramètres de conversion peuvent avoir un impact significatif sur les performances d’exécution.

Certaines classes de cas d’usage sont éligibles à des optimisations spécifiques. Voici quelques exemples.

### <a name="use-case-architectural-visualization--large-outdoor-maps"></a>Cas d’usage Visualisation architecturale/grandes cartes d’extérieur

* Ces types de scènes ont tendance à être statiques, ce qui signifie qu’ils n’ont pas besoin de pièces mobiles. En conséquence, vous pouvez définir `sceneGraphMode` sur `static` ou même `none`, ce qui améliore les performances d’exécution. Avec le mode `static`, le nœud racine de la scène peut toujours être déplacé, pivoté et mis à l’échelle, par exemple pour basculer dynamiquement entre l’échelle 1:1 (pour la vue à la première personne) et une vue de dessus de table.

* Lorsque vous devez déplacer des parties, cela signifie également que vous avez besoin d’une prise en charge des raycasts ou d’autres [requêtes spatiales](../../overview/features/spatial-queries.md), vous pouvez donc choisir ces parties en premier lieu. En revanche, si vous n’envisagez pas de déplacer une partie, il y a de fortes chances que vous n’ayez pas non plus besoin de la faire participer à des requêtes spatiales. Par conséquent, vous pouvez désactiver l’indicateur `generateCollisionMesh`. Ce commutateur a un impact significatif sur les temps de conversion, les temps de chargement, ainsi que les coûts de mise à jour par image.

* Si l’application n’utilise pas de [plans de coupe](../../overview/features/cut-planes.md), l’indicateur `opaqueMaterialDefaultSidedness` doit être désactivé. Le gain de performances est généralement de 20 à 30 %. Les plans de coupe peuvent toujours être utilisés, mais il n’y a pas de face arrière lorsque vous observez les parties internes des objets, ce qui peut donner un aspect contre-intuitif. Pour plus d’informations, consultez [Rendu :::no-loc text="single sided":::](../../overview/features/single-sided-rendering.md).

### <a name="use-case-photogrammetry-models"></a>Cas d’usage Modèles de photogrammétrie

Lors du rendu des modèles de photogrammétrie, il n’est généralement pas nécessaire d’utiliser un graphique de scène. Vous pouvez donc définir `sceneGraphMode` sur `none`. Étant donné que ces modèles contiennent rarement un graphique de scène complexe en premier lieu, l’impact de cette option devrait cependant être minime.

Étant donné que l’éclairage est déjà intégré aux textures, aucun éclairage dynamique n’est nécessaire. Par conséquent :

* Définissez l’indicateur `unlitMaterials` sur `true` pour transformer tous les matériaux en [matériaux de couleur](../../overview/features/color-materials.md) non éclairés.
* Supprimez les données inutiles du format de vertex. Voir [l’exemple](#example) ci-dessus.

### <a name="use-case-visualization-of-compact-machines-etc"></a>Cas d’usage Visualisation de machines compactées, etc.

Dans de tels cas d’utilisation, les modèles présentent souvent des détails très importants dans un petit volume. Le convertisseur est fortement optimisé pour gérer de tels cas. Toutefois, la plupart des optimisations mentionnées dans le cas d’usage précédent ne s’appliquent pas ici :

* Les pièces individuelles doivent être sélectionnables et mobiles, et l’indicateur `sceneGraphMode` doit donc être laissé sur `dynamic`.
* Les raycastings font généralement partie intégrante de l’application. Les maillages de collision doivent donc être générés.
* Les plans de coupe présentent un meilleur aspect avec l’indicateur `opaqueMaterialDefaultSidedness` activé.

## <a name="deprecated-features"></a>Fonctionnalités dépréciées

La possibilité de fournir des paramètres à l’aide du nom de fichier non spécifique au modèle `conversionSettings.json` est toujours prise en charge, mais elle est dépréciée.
Utilisez plutôt le nom de fichier propre au modèle `<modelName>.ConversionSettings.json`.

L’utilisation d’un paramètre `material-override` pour identifier un [fichier Remplacement de matériel](override-materials.md) dans le fichier des paramètres de conversion est toujours prise en charge mais déconseillée. Utilisez plutôt le nom de fichier propre au modèle `<modelName>.MaterialOverrides.json`.

## <a name="next-steps"></a>Étapes suivantes

* [Conversion de modèle](model-conversion.md)
* [Matériaux couleur](../../overview/features/color-materials.md)
* [Matériaux PBR](../../overview/features/pbr-materials.md)
* [Remplacer des matériaux pendant la conversion d’un modèle](override-materials.md)