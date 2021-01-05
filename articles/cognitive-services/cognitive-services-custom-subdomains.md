---
title: Créer des sous-domaines
titleSuffix: Azure Cognitive Services
description: Les noms de sous-domaines personnalisés de chaque ressource Cognitive Service sont créés via le portail Azure, Azure Cloud Shell ou Azure CLI.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: erhopf
ms.openlocfilehash: 4746aad2d7cd62cb309a1823f8c50487e6f7e87c
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032964"
---
# <a name="custom-subdomain-names-for-cognitive-services"></a>Sous-domaines personnalisés pour Cognitive Services

Azure Cognitive Services utilise des noms de sous-domaines personnalisés pour chaque ressource créée à l’aide du [portail Azure](https://portal.azure.com), [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) ou [Azure CLI](/cli/azure/install-azure-cli). Contrairement aux points de terminaison régionaux, qui étaient communs à tous les clients dans une région Azure spécifique, les noms de sous-domaines personnalisés sont uniques à la ressource. Des noms de sous-domaines personnalisés sont requis pour activer des fonctionnalités telles que Azure Active Directory (Azure AD) pour l’authentification.

## <a name="how-does-this-impact-existing-resources"></a>Comment cela affecte-t-il les ressources existantes?

Les ressources Cognitive Services créées avant le 1er juillet 2019 utiliseront les points de terminaison régionaux pour le service associé. Ces points de terminaison fonctionnent avec les ressources existantes et nouvelles.

Si vous souhaitez migrer une ressource existante pour tirer parti des noms de sous-domaines personnalisés, afin de pouvoir activer des fonctionnalités telles que Azure AD, suivez ces instructions :

1. Connectez-vous au Portail Azure et recherchez la ressource Cognitive Services à laquelle vous souhaitez ajouter un nom de sous-domaine personnalisé.
2. Dans le panneau **Vue d’ensemble**, recherchez et sélectionnez **Générer un nom de domaine personnalisé**.
3. Cela ouvre un panneau contenant des instructions pour créer un sous-domaine personnalisé unique pour votre ressource.
   > [!WARNING]
   > Une fois que vous avez créé un nom de sous-domaine personnalisé, il ne **peut pas** être modifié.

## <a name="do-i-need-to-update-my-existing-resources"></a>Dois-je mettre à jour mes ressources existantes ?

Non. Le point de terminaison régional continue de fonctionner pour les ressources Cognitive Services nouvelles et existantes, et le nom de sous-domaine personnalisé est facultatif. Même si un nom de sous-domaine personnalisé est ajouté, le point de terminaison régional continue à fonctionner avec la ressource.

## <a name="what-if-an-sdk-asks-me-for-the-region-for-a-resource"></a>Que se passe-t-il si un kit de développement logiciel (SDK) me demande la région d’une ressource ?

> [!WARNING]
> Les Speech Services utilisent des sous-domaines personnalisés avec de [points de terminaison privés](Speech-Service/speech-services-private-link.md) **uniquement**. Dans tous les autres cas, utilisez des **points de terminaison régionaux** avec les Speech Services et les kits de développement logiciel associés.

Les points de terminaison régionaux et les noms de sous-domaines personnalisés sont pris en charge et peuvent être utilisés indifféremment. Toutefois, le point de terminaison complet est requis.

Les informations de région sont disponibles dans le panneau **Vue d’ensemble** de votre ressource dans le [portail Azure](https://portal.azure.com). Pour obtenir la liste complète des points de terminaison régionaux, consultez [Existe-t-il une liste de points de terminaison régionaux ?](#is-there-a-list-of-regional-endpoints)

## <a name="are-custom-subdomain-names-regional"></a>Les noms de sous-domaines personnalisés sont-ils régionaux ?

Oui. L’utilisation d’un nom de sous-domaine personnalisé ne modifie pas les aspects régionaux de votre ressource Cognitive Services.

## <a name="what-are-the-requirements-for-a-custom-subdomain-name"></a>Quelles sont les conditions requises pour un nom de sous-domaine personnalisé ?

Un nom de sous-domaine personnalisé est propre à votre ressource. Le nom ne peut inclure que des caractères alphanumériques et le caractère `-` ; il doit comprendre entre 2 et 64 caractères, et ne peut pas se terminer par un `-`.

## <a name="can-i-change-a-custom-domain-name"></a>Puis-je modifier un nom de domaine personnalisé ?

Non. Une fois qu’un nom de sous-domaine personnalisé a été créé et associé à une ressource, il ne peut pas être modifié.

## <a name="can-i-reuse-a-custom-domain-name"></a>Puis-je réutiliser un nom de domaine personnalisé ?

Chaque nom de sous-domaine personnalisé étant unique, pour pouvoir réutiliser un nom de sous-domaine personnalisé que vous avez affecté à une ressource Cognitive Services, vous devez supprimer la ressource existante. Une fois que la ressource a été supprimée, vous pouvez réutiliser le nom de sous-domaine personnalisé.

## <a name="is-there-a-list-of-regional-endpoints"></a>Existe-t-il une liste des points de terminaison régionaux ?

Oui. Il s’agit d’une liste de points de terminaison régionaux que vous pouvez utiliser avec les ressources Azure Cognitive Services.

> [!NOTE]
> Les API Translator et Bing Search utilisent des points de terminaison globaux.

| Type de point de terminaison | Région | Point de terminaison |
|---------------|--------|----------|
| Public | Global (Translator et Bing) | `https://api.cognitive.microsoft.com` |
| | Australie Est | `https://australiaeast.api.cognitive.microsoft.com` |
| | Brésil Sud | `https://brazilsouth.api.cognitive.microsoft.com` |
| | Centre du Canada | `https://canadacentral.api.cognitive.microsoft.com` |
| | USA Centre | `https://centralus.api.cognitive.microsoft.com` |
| | Asie Est | `https://eastasia.api.cognitive.microsoft.com` |
| | USA Est | `https://eastus.api.cognitive.microsoft.com` |
| | USA Est 2 | `https://eastus2.api.cognitive.microsoft.com` |
| | France Centre | `https://francecentral.api.cognitive.microsoft.com` |
| | Inde Centre | `https://centralindia.api.cognitive.microsoft.com` |
| | Japon Est | `https://japaneast.api.cognitive.microsoft.com` |
| | Centre de la Corée | `https://koreacentral.api.cognitive.microsoft.com` |
| | Centre-Nord des États-Unis | `https://northcentralus.api.cognitive.microsoft.com` |
| | Europe Nord | `https://northeurope.api.cognitive.microsoft.com` |
| | Afrique du Sud Nord | `https://southafricanorth.api.cognitive.microsoft.com` |
| | États-Unis - partie centrale méridionale | `https://southcentralus.api.cognitive.microsoft.com` |
| | Asie Sud-Est | `https://southeastasia.api.cognitive.microsoft.com` |
| | Sud du Royaume-Uni | `https://uksouth.api.cognitive.microsoft.com` |
| | Centre-USA Ouest | `https://westcentralus.api.cognitive.microsoft.com` |
| | Europe Ouest | `https://westeurope.api.cognitive.microsoft.com` |
| | USA Ouest | `https://westus.api.cognitive.microsoft.com` |
| | USA Ouest 2 | `https://westus2.api.cognitive.microsoft.com` |
| Gouvernement des États-Unis | Gouvernement américain - Virginie | `https://virginia.api.cognitive.microsoft.us` |
| Chine | Chine orientale 2 | `https://chinaeast2.api.cognitive.azure.cn` |
| | Chine du Nord | `https://chinanorth.api.cognitive.azure.cn` |

## <a name="see-also"></a>Voir aussi

* [Présentation de Cognitive Services](./what-are-cognitive-services.md)
* [Authentification](authentication.md)