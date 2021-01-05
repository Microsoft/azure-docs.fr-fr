---
title: Intégrer un domaine racine ou apex à un point de terminaison Azure CDN existant – Portail Azure
description: Découvrez comment intégrer un domaine racine ou apex à un point de terminaison Azure CDN existant à l’aide du portail Azure.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 11/07/2020
ms.author: allensu
ms.openlocfilehash: 8ab4f698c7149d8d57f790e221ccbe35ec090fe6
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369772"
---
# <a name="onboard-a-root-or-apex-domain-to-an-existing-azure-cdn-endpoint"></a>Intégrer un domaine racine ou apex à un point de terminaison Azure CDN existant

Azure CDN utilise des enregistrements CNAME pour valider la propriété du domaine en vue de l’intégration de domaines personnalisés. CDN n’expose pas l’adresse IP du serveur frontal associée à votre profil CDN. Vous ne pouvez pas mapper votre domaine apex à une adresse IP si vous avez l’intention de l’intégrer à Azure CDN.

Le protocole DNS empêche l'attribution d'enregistrements CNAME à l'extrémité de la zone. Par exemple, si votre domaine est `contoso.com`, vous pouvez créer des enregistrements CNAME pour `somelabel.contoso.com` ; mais vous ne pouvez pas en créer pour `contoso.com` lui-même. Cette restriction pose un problème aux propriétaires d’applications qui disposent d’applications à charge équilibrée derrière Azure CDN. Dans la mesure où l’utilisation d’un profil CDN nécessite la création d’un enregistrement CNAME, il est impossible de pointer vers le profil CDN à partir de l’apex de zone.

Ce problème peut être résolu à l’aide d’enregistrements d’alias dans Azure DNS. Contrairement aux enregistrements CNAME, les enregistrements d’alias sont créés à l’apex de la zone. Les propriétaires d’applications peuvent l’utiliser pour pointer leur enregistrement d’apex de zone vers un profil CDN disposant de points de terminaison publics. Les propriétaires d’applications peuvent pointer vers le même profil CDN que celui utilisé pour tout autre domaine de leur zone DNS. Par exemple, `contoso.com` et `www.contoso.com` peuvent pointer vers le même profil CDN. 

Le mappage de votre apex ou domaine racine à votre profil CDN nécessite une mise à plat de CNAME ou une recherche de DNS. Un mécanisme dans lequel le fournisseur DNS résout de manière récursive l’entrée CNAME jusqu’à ce qu’elle atteigne une adresse IP. Cette fonctionnalité est prise en charge par Azure DNS pour les points de terminaison CDN. 

> [!NOTE]
> D’autres fournisseurs de DNS prennent également en charge la mise à plat de CNAME ou la recherche de DNS. Azure CDN recommande toutefois d’utiliser Azure DNS pour ses clients en vue de l’hébergement de leurs domaines.

Vous pouvez utiliser le portail Azure pour intégrer un domaine apex à votre CDN et activer le protocole HTTPS sur celui-ci en l’associant à un certificat pour point de terminaison TLS. Les domaines apex sont également appelés des domaines racine ou nus.

## <a name="create-an-alias-record-for-zone-apex"></a>Créer un enregistrement d’alias d’apex de zone

1. Ouvrez la configuration **Azure DNS** du domaine à intégrer.

2. Sélectionnez **+ Jeu d’enregistrements**.

3. Dans **Ajouter un jeu d’enregistrements**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ------|
    | Nom | Entrez **@** . |
    | Type | Sélectionnez **A**. |
    | Jeu d’enregistrements d’alias | Sélectionnez **Oui**. |
    | Type d’alias | Sélectionnez **Ressource Azure**. |
    | Sélectionnez un abonnement | Sélectionnez votre abonnement. |
    | Ressource Azure | Sélectionnez votre point de terminaison CDN. |

4. Entrez votre valeur pour **TTL**.

5. Sélectionnez **OK** pour envoyer vos modifications.

    :::image type="content" source="./media/onboard-apex-domain/cdn-apex-alias-record.png" alt-text="Enregistrement d’alias d’apex de zone":::

6. L’étape ci-dessus crée un enregistrement d’apex de zone pointant vers votre ressource CDN. Un mappage d’enregistrement CNAMe **cdnverify** est utilisé pour l’intégration du domaine à votre profil CDN.
    1. Par exemple, **cdnverify.contoso.com**.
    

## <a name="onboard-the-custom-domain-on-your-cdn"></a>Intégrer le domaine personnalisé à votre CDN

Une fois que vous avez enregistré votre domaine personnalisé, vous pouvez l’ajouter à votre point de terminaison CDN. 

1. Connectez-vous au [Portail Azure](https://portal.azure.com/), puis accédez au profil CDN contenant le point de terminaison que vous souhaitez mapper à un domaine personnalisé.
    
2. Sur la page **Profil CDN**, sélectionnez le point de terminaison CDN à associer avec le domaine personnalisé.

    :::image type="content" source="media/onboard-apex-domain/cdn-endpoint-selection.png" alt-text="Sélection de point de terminaison CDN" border="true":::
    
3. Sélectionnez **+ Domaine personnalisé**. 

   :::image type="content" source="media/onboard-apex-domain/cdn-custom-domain-button.png" alt-text="Bouton Ajouter un domaine personnalisé" border="true":::

4. Dans **Ajouter un domaine personnalisé**, le **Nom d’hôte du point de terminaison** pré-rempli est dérivé de votre URL de point de terminaison CDN : **\<endpoint-hostname>** .zureedge.net. Cette valeur n’est pas modifiable.

5. Pour **Nom d’hôte personnalisé**, entrez votre domaine racine ou apex personnalisé à utiliser comme domaine source de votre enregistrement CNAME. 
    1. Par exemple, **contoso.com**. **N’utilisez pas le nom de sous-domaine cdnverify**.

    :::image type="content" source="media/onboard-apex-domain/cdn-add-custom-domain.png" alt-text="Ajouter un domaine personnalisé" border="true":::

6. Sélectionnez **Ajouter**.

   Azure vérifie que l’enregistrement CNAME existe pour le nom de domaine personnalisé que vous avez entré. Si l'enregistrement CNAME est correct, votre domaine personnalisé sera validé. 

   La propagation des nouveaux paramètres de domaine personnalisé sur tous les nœuds de périphérie CDN peut prendre un certain temps : 
    - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
    - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
    - Dans le cas des profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 10 minutes.   

## <a name="enable-https-on-your-custom-domain"></a>Activer HTTPS sur votre domaine personnalisé

Pour plus d’informations sur l’activation de HTTPS sur votre domaine personnalisé pour Azure CDN, consultez [Tutoriel : Configurer HTTPS sur un domaine personnalisé Azure CDN](cdn-custom-ssl.md)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un point de terminaison CDN](cdn-create-new-endpoint.md).
