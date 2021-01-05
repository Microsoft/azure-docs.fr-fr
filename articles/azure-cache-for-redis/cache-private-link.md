---
title: Azure Cache pour Redis avec Azure Private Link (préversion)
description: Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à Azure Cache pour Redis optimisé par Azure Private Link. Cet article explique comment créer un cache Azure, un réseau virtuel Azure et un point de terminaison privé à l’aide du portail Azure.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 22bdf93e7236ae5220a6bb7c6ead898628bb51a1
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007583"
---
# <a name="azure-cache-for-redis-with-azure-private-link-public-preview"></a>Azure Cache pour Redis avec Azure Private Link (Préversion publique)
Dans cet article, vous allez apprendre à créer un réseau virtuel et une instance Azure Cache pour Redis avec un point de terminaison privé à l’aide du portail Azure. Vous apprendrez également à ajouter un point de terminaison privé à une instance Azure Cache pour Redis existante.

Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à Azure Cache pour Redis optimisé par Azure Private Link. 

## <a name="prerequisites"></a>Prérequis
* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)

> [!IMPORTANT]
> Pour utiliser des points de terminaison privés, votre instance Azure Cache pour Redis doit avoir été créée après le 28 juillet 2020.
> Actuellement, la géoréplication, les règles de pare-feu, la prise en charge de la console du portail, la multiplicité des points de terminaison par cache en cluster, la persistance au pare-feu et les caches injectés de réseau virtuel ne sont pas pris en charge. 
>
>

## <a name="create-a-private-endpoint-with-a-new-azure-cache-for-redis-instance"></a>Créer un point de terminaison privé avec une nouvelle instance Azure Cache pour Redis 

Dans cette section, vous allez créer une instance Azure Cache pour Redis avec un point de terminaison privé.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel 

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Créer une ressource**.

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Sélectionnez Créer une ressource.":::

2. Dans la page **Nouvelle**, sélectionnez **Mise en réseau**, puis sélectionnez **Réseau virtuel**.

3. Sélectionnez **Ajouter** pour créer un réseau virtuel.

4. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement dans lequel créer ce réseau virtuel. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre réseau virtuel et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Nom** | Entrez un nom de réseau virtuel. | Le nom doit commencer par une lettre ou un chiffre et se terminer par une lettre, un chiffre ou un trait de soulignement, et il ne peut contenir que des lettres, des chiffres, des traits de soulignement, des points ou des traits d’union. | 
   | **Région** | Dans la liste déroulante, sélectionnez une région. | Choisissez une [région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre réseau virtuel. |

5. Sélectionnez l’onglet **Adresses IP** ou cliquez sur le bouton **Suivant : Adresses IP** au bas de la page.

6. Dans l’onglet **Adresses IP**, spécifiez l’**espace d’adressage IPv4** comme un ou plusieurs préfixes d’adresse en notation CIDR (par exemple, 192.168.1.0/24).

7. Sous **Nom du sous-réseau**, cliquez sur **Par défaut** pour modifier les propriétés du sous-réseau.

8. Dans le volet **Modifier le sous-réseau**, spécifiez un **nom de sous-réseau** ainsi que la **plage d’adresses du sous-réseau**. La plage d’adresses du sous-réseau doit être en notation CIDR (par exemple, 192.168.1.0/24). Elle doit être contenue dans l’espace d’adressage du réseau virtuel.

9. Sélectionnez **Enregistrer**.

10. Sélectionnez l’onglet **Vérifier + créer** cliquez sur le bouton **Vérifier + créer**.

11. Vérifiez que toutes les informations sont correctes, puis cliquez sur **Créer** pour approvisionner le réseau virtuel.

### <a name="create-an-azure-cache-for-redis-instance-with-a-private-endpoint"></a>Créer une instance Azure Cache pour Redis avec un point de terminaison privé
Pour créer une instance de cache, procédez comme suit.

1. Revenez à la page d’accueil du portail Azure ou ouvrez le menu latéral, puis sélectionnez **Créer une ressource**. 
   
1. Dans la page **Nouvelle**, sélectionnez **Bases de données**, puis **Azure Cache pour Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Sélectionnez Azure Cache pour Redis.":::
   
1. Dans la page **Nouveau cache Redis**, configurez les paramètres du nouveau cache.
   
   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net*. | 
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Lieu** | Dans la liste déroulante, sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
   | **Niveau tarifaire** | Sélectionnez un [Niveau tarifaire](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |

1. Sélectionnez l’onglet **Réseau** ou cliquez sur le bouton **Réseau** au bas de la page.

1. Sous l’onglet **Réseau**, sélectionnez **Point de terminaison privé** comme méthode de connectivité.

1. Cliquez sur le bouton **Ajouter** pour créer votre point de terminaison privé.

    :::image type="content" source="media/cache-private-link/3-add-private-endpoint.png" alt-text="Dans Réseau, ajouter un point de terminaison privé.":::

1. Dans la page **Créer un point de terminaison privé**, configurez les paramètres de votre point de terminaison privé avec le réseau virtuel et le sous-réseau que vous avez créés dans la dernière section, puis sélectionnez **OK**. 

1. Sélectionnez le bouton **Suivant : Avancé** ou cliquez sur le bouton **Suivant : Avancé** en bas de la page.

1. Sous l’onglet **Avancé** d’une instance de cache de base ou standard, sélectionnez Activer/désactiver si vous souhaitez activer un port non-TLS.

1. Sous l’onglet **Avancé** d’une instance de cache premium, configurez les paramètres pour le port non-TLS, le clustering et la persistance des données.

1. Sélectionnez le bouton **Suivant : Étiquettes** ou cliquez sur le bouton **Suivant : Étiquettes** au bas de la page.

1. Si vous le voulez, sous l’onglet **Étiquettes**, entrez le nom et la valeur si vous souhaitez catégoriser la ressource. 

1. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers l’onglet Vérifier + créer où Azure valide votre configuration.

1. Une fois que le message vert Validation réussie s’affiche, sélectionnez **Créer**.

La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution**, le cache est prêt pour utilisation. 
    
> [!IMPORTANT]
> 
> Il existe un indicateur `publicNetworkAccess` qui est `Disabled` par défaut. 
> Cet indicateur est destiné à vous permettre d’autoriser l’accès de points de terminaison public et privé au cache s’il est défini sur `Enabled`. Si la valeur est `Disabled`, il autorise uniquement l’accès de point de terminaison privé. Vous pouvez définir la valeur sur `Disabled` ou `Enabled` avec la requête PATCH suivante. Modifiez la valeur pour refléter l’indicateur que vous souhaitez appliquer à votre cache.
> ```http
> PATCH  https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resourcegroup}/providers/Microsoft.Cache/Redis/{cache}?api-version=2020-06-01
> {    "properties": {
>        "publicNetworkAccess":"Disabled"
>    }
> }
> ```
>

> [!IMPORTANT]
> 
> Pour vous connecter à un cache en cluster, `publicNetworkAccess` doit être défini sur `Disabled`, et il ne peut y avoir qu’une seule connexion de point de terminaison privé. 
>

## <a name="create-a-private-endpoint-with-an-existing-azure-cache-for-redis-instance"></a>Créer un point de terminaison privé avec une instance Azure Cache pour Redis existante 

Dans cette section, vous allez ajouter un point de terminaison privé à une instance Azure Cache pour Redis existante. 

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel 
Pour créer un réseau virtuel, procédez comme suit.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Créer une ressource**.

2. Dans la page **Nouvelle**, sélectionnez **Mise en réseau**, puis sélectionnez **Réseau virtuel**.

3. Sélectionnez **Ajouter** pour créer un réseau virtuel.

4. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement dans lequel créer ce réseau virtuel. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre réseau virtuel et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Nom** | Entrez un nom de réseau virtuel. | Le nom doit commencer par une lettre ou un chiffre et se terminer par une lettre, un chiffre ou un trait de soulignement, et il ne peut contenir que des lettres, des chiffres, des traits de soulignement, des points ou des traits d’union. | 
   | **Région** | Dans la liste déroulante, sélectionnez une région. | Choisissez une [région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre réseau virtuel. |

5. Sélectionnez l’onglet **Adresses IP** ou cliquez sur le bouton **Suivant : Adresses IP** au bas de la page.

6. Dans l’onglet **Adresses IP**, spécifiez l’**espace d’adressage IPv4** comme un ou plusieurs préfixes d’adresse en notation CIDR (par exemple, 192.168.1.0/24).

7. Sous **Nom du sous-réseau**, cliquez sur **Par défaut** pour modifier les propriétés du sous-réseau.

8. Dans le volet **Modifier le sous-réseau**, spécifiez un **nom de sous-réseau** ainsi que la **plage d’adresses du sous-réseau**. La plage d’adresses du sous-réseau doit être en notation CIDR (par exemple, 192.168.1.0/24). Elle doit être contenue dans l’espace d’adressage du réseau virtuel.

9. Sélectionnez **Enregistrer**.

10. Sélectionnez l’onglet **Vérifier + créer** cliquez sur le bouton **Vérifier + créer**.

11. Vérifiez que toutes les informations sont correctes, puis cliquez sur **Créer** pour approvisionner le réseau virtuel.

### <a name="create-a-private-endpoint"></a>Créer un Private Endpoint 

Pour créer un point de terminaison privé, procédez comme suit.

1. Dans le portail Azure, recherchez **Azure Cache pour Redis** et appuyez sur Entrée ou sélectionnez l’élément correspondant dans les suggestions de recherche.

    :::image type="content" source="media/cache-private-link/4-search-for-cache.png" alt-text="Rechercher Azure Cache pour Redis.":::

2. Sélectionnez l’instance de cache à laquelle vous souhaitez ajouter un point de terminaison privé.

3. Sur le côté gauche de l’écran, sélectionnez **(PRÉVERSION) Point de terminaison privé**.

4. Cliquez sur le bouton **Point de terminaison privé** pour créer votre point de terminaison privé.

    :::image type="content" source="media/cache-private-link/5-add-private-endpoint.png" alt-text="Ajouter un point de terminaison privé.":::

5. Dans la page **Créer un point de terminaison privé**, configurez les paramètres de votre point de terminaison privé.

   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement dans lequel créer ce point de terminaison privé. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre point de terminaison privé et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Nom** | Entrez un nom de point de terminaison privé. | Le nom doit commencer par une lettre ou un chiffre et se terminer par une lettre, un chiffre ou un trait de soulignement, et il ne peut contenir que des lettres, des chiffres, des traits de soulignement, des points ou des traits d’union. | 
   | **Région** | Dans la liste déroulante, sélectionnez une région. | Choisissez une [région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre point de terminaison privé. |

6. Cliquez sur le bouton **Next: Ressource** en bas de la page.

7. Sous l’onglet **Ressources**, sélectionnez votre abonnement, choisissez le type de ressource `Microsoft.Cache/Redis`, puis sélectionnez le cache auquel vous souhaitez connecter le point de terminaison privé.

8. Cliquez sur le bouton **Next: Configuration** en bas de la page.

9. Sous l’onglet **Configuration**, sélectionnez le réseau virtuel et le sous-réseau que vous avez créés dans la section précédente.

10. Cliquez sur le bouton **Next: Étiquettes** au bas de la page.

11. Si vous le voulez, sous l’onglet **Étiquettes**, entrez le nom et la valeur si vous souhaitez catégoriser la ressource.

12. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers l’onglet **Vérifier + créer** où Azure valide votre configuration.

13. Une fois que le message vert **Validation réussie** s’affiche, sélectionnez **Créer**.

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="why-cant-i-connect-to-a-private-endpoint"></a>Pourquoi ne puis-je pas me connecter à un point de terminaison privé ?
Si votre cache est déjà un cache injecté sur un réseau virtuel, les points de terminaison privés ne peuvent pas être utilisés avec votre instance de cache. Si votre instance de cache utilise une fonctionnalité non prise en charge (voir la liste ci-dessous), vous ne pourrez pas vous connecter à votre instance de point de terminaison privé. En outre, les instances de cache doivent être créées après le 27 juillet pour utiliser des points de terminaison privés.

### <a name="what-features-are-not-supported-with-private-endpoints"></a>Quelles fonctionnalités ne sont pas prises en charge avec les points de terminaison privés ?
La géoréplication, les règles de pare-feu, la prise en charge de la console du portail, la multiplicité des points de terminaison par cache en cluster, la persistance aux règles de pare-feu et la redondance de zone. 

### <a name="how-can-i-change-my-private-endpoint-to-be-disabled-or-enabled-from-public-network-access"></a>Comment puis-je modifier mon point de terminaison privé pour qu’il soit désactivé ou activé à partir de l’accès au réseau public ?
Il existe un indicateur `publicNetworkAccess` qui est `Disabled` par défaut. Cet indicateur est destiné à vous permettre d’autoriser l’accès de points de terminaison public et privé au cache s’il est défini sur `Enabled`. Si la valeur est `Disabled`, il autorise uniquement l’accès de point de terminaison privé. Vous pouvez définir la valeur sur `Disabled` ou `Enabled` avec la requête PATCH suivante. Modifiez la valeur pour refléter l’indicateur que vous souhaitez appliquer à votre cache.

```http
PATCH  https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resourcegroup}/providers/Microsoft.Cache/Redis/{cache}?api-version=2020-06-01
{    "properties": {
       "publicNetworkAccess":"Disabled"
   }
}
```

### <a name="are-network-security-groups-nsg-enabled-for-private-endpoints"></a>Les groupes de sécurité réseau (NSG) sont-ils activés pour les points de terminaison privés ?
Non, ils sont désactivés pour les points de terminaison privés. Si les sous-réseaux contenant Private Endpoint peuvent être associés à un groupe de sécurité réseau, les règles ne sont pas effectives sur le trafic traité par Private Endpoint. Vous devez [désactiver l’application des stratégies réseau](../private-link/disable-private-endpoint-network-policy.md) pour déployer Private Endpoint dans un sous-réseau. Le groupe de sécurité réseau est toujours appliqué sur les autres charges de travail hébergées sur le même sous-réseau. Les itinéraires sur un sous-réseau client, quel qu’il soit, utiliseront un préfixe /32. La modification du comportement de routage par défaut nécessite une UDR similaire. 

Contrôlez le trafic à l’aide de règles de groupe de sécurité réseau pour le trafic sortant sur les clients source. Déployez des routes individuelles avec le préfixe /32 pour remplacer les routes de points de terminaison privés. Les journaux de trafic NSG et les informations de supervision pour les connexions sortantes sont toujours pris en charge et peuvent être utilisés.

### <a name="can-i-use-firewall-rules-with-private-endpoints"></a>Puis-je utiliser des règles de pare-feu avec des points de terminaison privés ?
Non, il s’agit d’une limitation actuelle des points de terminaison privés. Le point de terminaison privé ne fonctionnera pas correctement si des règles de pare-feu sont configurées sur le cache.

### <a name="how-can-i-connect-to-a-clustered-cache"></a>Comment puis-je me connecter à un cache en cluster ?
`publicNetworkAccess` doit être défini sur `Disabled`, et il ne peut y avoir qu’une seule connexion de point de terminaison privé.

### <a name="since-my-private-endpoint-instance-is-not-in-my-vnet-how-is-it-associated-with-my-vnet"></a>Mon instance de point de terminaison privé ne figure pas dans mon réseau virtuel. Comment est-elle associée à mon réseau virtuel ?
Elle est uniquement liée à votre réseau virtuel. Comme elle ne se trouve pas dans votre réseau virtuel, il n’est pas nécessaire de modifier les règles NSG pour les points de terminaison dépendants.

### <a name="how-can-i-migrate-my-vnet-injected-cache-to-a-private-endpoint-cache"></a>Comment migrer mon cache injecté sur un réseau virtuel vers un cache de point de terminaison privé ?
Vous devrez supprimer votre cache injecté sur un réseau virtuel et créer une nouvelle instance de cache avec un point de terminaison privé.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur Azure Private Link, consultez la [documentation d’Azure Private Link](../private-link/private-link-overview.md).
* Pour comparer les différentes options d’isolement réseau de votre instance de cache, consultez la [documentation sur les options d’isolement réseau du cache Azure pour Redis](cache-network-isolation.md).