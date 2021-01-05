---
title: Prise en charge de plusieurs origines par point de terminaison Azure CDN (préversion)
description: Prise en main de plusieurs origines par point de terminaison Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.topic: how-to
ms.date: 9/06/2020
ms.author: allensu
ms.openlocfilehash: f9293206526778f8c3de8a368a1916a2cb3f88c2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89504671"
---
# <a name="azure-cdn-endpoint-multi-origin-preview"></a>Prise en charge de plusieurs origines par point de terminaison Azure CDN (préversion)

La prise en charge de plusieurs origines élimine les temps d’arrêt et établit une redondance globale. 

En choisissant plusieurs origines au sein d’un point de terminaison Azure CDN, la redondance fournie réagit au risque en sondant l’intégrité de chaque origine et en basculant si nécessaire.

Configurez un ou plusieurs groupes d’origines et choisissez un groupe d’origines par défaut. Chaque groupe d'origines est un ensemble d'une ou plusieurs origines qui peuvent prendre des charges de travail similaires.

> [!NOTE]
> Actuellement, cette fonctionnalité est uniquement disponible à partir d'Azure CDN de Microsoft. 

> [!IMPORTANT]
> Le point de terminaison Azure CDN à plusieurs origines est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-the-origin-group"></a>Créer le groupe d'origines

1. Connectez-vous au [portail Azure](https://portal.azure.com)

2. Sélectionnez votre profil Azure CDN, puis choisissez le point de terminaison à configurer avec plusieurs origines.

3. Sélectionnez **Origine** sous **Paramètres** dans la configuration du point de terminaison :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-1.png" alt-text="Point de terminaison CDN" border="true":::

4. Pour activer plusieurs origines, au moins un groupe d'origines est nécessaire. Sélectionnez **Créer un groupe d'origines** :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-2.png" alt-text="Point de terminaison CDN" border="true":::

5. Dans la configuration **Ajouter un groupe d'origines**, entrez ou sélectionnez les informations suivantes :

   | Paramètre           | Valeur                                                                 |
   |-------------------|-----------------------------------------------------------------------|
   | Nom du groupe d'origines | Entrez un nom pour votre groupe d'origines.                                   |
   | État de la sonde d’intégrité      | Sélectionnez **Enabled**. </br> Azure CDN exécutera des sondes d’intégrité à partir de différents points dans le monde pour déterminer l’intégrité de l’origine. Ne pas activer si le groupe d’origines actuel n’est pas actif pour éviter des coûts supplémentaires.
   | Chemin d’analyse        | Chemin d'accès à l'origine utilisé pour déterminer l'intégrité. |
   | Intervalle de sondage    | Sélectionnez un intervalle de sondage de 1, 2 ou 4 minutes.                        |
   | Protocole de sondage    | Sélectionnez **HTTP** ou **HTTPS**.                                         |
   | Méthode de sondage      | Sélectionnez **Head** ou **Get**.                                           |
   | Groupe d'origines par défaut | Cochez la case pour définir comme groupe d’origines par défaut.
    
   :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-3.png" alt-text="Point de terminaison CDN" border="true":::

6. Sélectionnez **Ajouter**.

## <a name="add-multiple-origins"></a>Ajouter plusieurs origines

1. Dans les paramètres d'origine de votre point de terminaison, sélectionnez **+ Créer une origine** :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-5.png" alt-text="Point de terminaison CDN" border="true":::

2. Entrez ou sélectionnez les informations suivantes dans la configuration **Ajouter une origine** :

   | Paramètre           | Valeur                                                                 |
   |-------------------|-----------------------------------------------------------------------|
   | Nom        | Entrez le nom de l’origine.        |
   | Type d'origine | Sélectionnez **Stockage**, **Service cloud**, **Application web** ou **Origine personnalisée**.                                   |
   | Nom d’hôte de l’origine        | Sélectionnez ou entrez le nom d'hôte de votre origine.  La liste déroulante répertorie toutes les origines disponibles pour le type que vous avez spécifié au paramètre précédent. Si vous avez sélectionné **Origine personnalisée** comme type d'origine, entrez le domaine du serveur d'origine de votre client. |
   | En-tête de l’hôte d’origine    | Entrez l'en-tête de l'hôte qu'Azure CDN doit envoyer avec chaque requête, ou conservez la valeur par défaut.                        |
   | Port HTTP   | Entrez le port HTTP.                                         |
   | Port HTTPS     | Entrez le port HTTPS.                                           |
   | Priorité    | Entrez un chiffre compris entre 1 et 5.       |
   | Poids      | Entrez un nombre compris entre 1 et 1000.   |

    > [!NOTE]
    > Lorsqu'une origine est créée au sein d'un groupe d'origines, une priorité et un poids doivent lui être attribués. Si un groupe ne comprend qu'une seule origine, la priorité et le poids par défaut sont définis sur 1. Le trafic est acheminé vers l'origine à la priorité la plus élevée si elle est saine. Si une origine n'est pas jugée saine, les connexions sont détournées vers une autre origine, en respectant l'ordre de priorité. Si deux origines ont la même priorité, le trafic est distribué selon le poids spécifié pour l'origine. 

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-6.png" alt-text="Point de terminaison CDN" border="true":::

3. Sélectionnez **Ajouter**.

4. Sélectionnez **Configurer l'origine** pour définir le chemin d'accès de toutes les origines :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-7.png" alt-text="Point de terminaison CDN" border="true":::

5. Sélectionnez **OK**.

## <a name="configure-origins-and-origin-group-settings"></a>Configurer les paramètres des origines et groupes d'origines

Lorsque vous disposez de plusieurs origines et d'un groupe d'origines, vous pouvez ajouter ou supprimer des origines dans différents groupes. Les origines d'un même groupe doivent servir des charges de travail similaires. Le trafic sera réparti entre ces origines en fonction de leur état d'intégrité, de leur priorité et de leur poids. 

1. Dans les paramètres de l'origine du point de terminaison Azure CDN, sélectionnez le nom du groupe d’origines que vous souhaitez configurer :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-8.png" alt-text="Point de terminaison CDN" border="true":::

2. Dans **Mettre à jour le groupe d'origines**, sélectionnez **+ Sélectionner l’origine** :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-9.png" alt-text="Point de terminaison CDN" border="true":::

4. Sélectionnez l'origine à ajouter au groupe dans la liste déroulante, puis sélectionnez **OK**.

5. Vérifiez que l'origine a été ajoutée au groupe, puis sélectionnez **Enregistrer** :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-10.png" alt-text="Point de terminaison CDN" border="true":::

## <a name="remove-origin-from-origin-group"></a>Supprimer une origine du groupe d'origines

1. Dans les paramètres de l'origine du point de terminaison Azure CDN, sélectionnez le nom du groupe d'origines :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-8.png" alt-text="Point de terminaison CDN" border="true":::

2. Pour supprimer une origine du groupe d'origines, sélectionnez l'icône Corbeille en regard de l'origine, puis sélectionnez **Enregistrer** :

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-11.png" alt-text="Point de terminaison CDN" border="true":::

## <a name="override-origin-group-with-rules-engine"></a>Remplacer le groupe d’origines par le moteur de règles

Personnaliser la façon dont le trafic est distribué à différents groupes d’origines à l’aide du moteur de règles standard.

Répartissez le trafic vers un autre groupe en fonction de l’URL de la requête.

1. Dans votre point de terminaison CDN, sélectionnez **Moteur de règles** sous **Paramètres** :

:::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-12.png" alt-text="Point de terminaison CDN" border="true":::

2. Sélectionnez **+ Ajouter une règle**.

3. Entrez le nom de la règle dans **Nom**.

4. Sélectionnez **+ Condition**, puis sélectionnez **Chemin de l’URL**.

5. Dans la liste déroulante **opérateur**, sélectionnez **Contient**.

6. Dans **Valeur**, entrez **/images**.

7. Sélectionnez **+ Ajouter une action**, puis sélectionnez **Remplacement du groupe d’origines**.

8. Dans **Groupe d’origines**, sélectionnez le groupe d'origines dans la liste déroulante.

:::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-13.png" alt-text="Point de terminaison CDN" border="true":::

Pour toutes les requêtes entrantes si le chemin d’accès de l’URL contient **/images**, la requête est alors attribuée au groupe d’origines dans la section d’action **(myorigingroup)** . 

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez activé plusieurs origines par point de terminaison Azure CDN.

Pour plus d’informations sur Azure CDN et les autres services Azure mentionnés dans cet article, consultez :

* [Azure CDN](./cdn-overview.md)
* [Comparer les caractéristiques du produit CDN Azure](./cdn-features.md)
