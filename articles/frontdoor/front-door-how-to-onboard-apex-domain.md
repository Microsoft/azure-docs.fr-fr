---
title: Intégrer un domaine racine ou apex à un Front Door existant – Portail Azure
description: Découvrez comment intégrer un domaine racine ou apex à un Front Door existant à l’aide du portail Azure.
services: front-door
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 11/13/2020
ms.author: duau
ms.openlocfilehash: 55eefe7a7490df050aa7ebc2bb41fbadcc8d8279
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94646336"
---
# <a name="onboard-a-root-or-apex-domain-on-your-front-door"></a>Intégrer un domaine racine ou apex à votre Front Door
Azure Front Door utilise des enregistrements CNAME pour valider la propriété du domaine en vue de l’intégration de domaines personnalisés. Front Door n’expose pas l’adresse IP du serveur frontal associée à votre profil Front Door. Vous ne pouvez donc pas mapper votre domaine apex à une adresse IP si vous avez l’intention de l’intégrer à Azure Front Door.

Le protocole DNS empêche l'attribution d'enregistrements CNAME à l'extrémité de la zone. Par exemple, si votre domaine est `contoso.com`, vous pouvez créer des enregistrements CNAME pour `somelabel.contoso.com` ; mais vous ne pouvez pas en créer pour `contoso.com` lui-même. Cette restriction pose un problème aux propriétaires d’applications qui disposent d’applications à charge équilibrée derrière Azure Front Door. Dans la mesure où l’utilisation d’un profil Front Door nécessite la création d’un enregistrement CNAME, il est impossible de pointer vers le profil Front Door à partir de l’apex de zone.

Ce problème peut être résolu à l’aide d’enregistrements d’alias dans Azure DNS. Contrairement aux enregistrements CNAME, les enregistrements d’alias sont créés à l’apex de la zone. Les propriétaires d’applications peuvent l’utiliser pour pointer leur enregistrement d’apex de zone vers un profil Front Door disposant de points de terminaison publics. Les propriétaires d’applications peuvent pointer vers le même profil Front Door que celui utilisé pour tout autre domaine de leur zone DNS. Par exemple, `contoso.com` et `www.contoso.com` peuvent pointer vers le même profil Front Door. 

Le mappage de votre apex ou domaine racine à votre profil Front Door nécessite une mise à plat CNAME ou une recherche DNS. Un mécanisme dans lequel le fournisseur DNS résout de manière récursive l’entrée CNAME jusqu’à ce qu’elle atteigne une adresse IP. Cette fonctionnalité est prise en charge par Azure DNS pour les points de terminaison Front Door. 

> [!NOTE]
> D’autres fournisseurs de DNS prennent également en charge la mise à plat de CNAME ou la recherche de DNS. Azure Front Door recommande toutefois d’utiliser Azure DNS pour ses clients en vue de l’hébergement de leurs domaines.

Vous pouvez utiliser le portail Azure pour intégrer un domaine apex à votre Front Door et activer le protocole HTTPS sur celui-ci en l’associant à un certificat pour point de terminaison TLS. Les domaines apex sont également appelés des domaines racine ou nus.

## <a name="create-an-alias-record-for-zone-apex"></a>Créer un enregistrement d’alias d’apex de zone

1. Ouvrez la configuration **Azure DNS** du domaine à intégrer.

1. Créez ou modifiez l’enregistrement d’apex de zone.

1. Sélectionnez le **type** d’enregistrement *A*, puis sélectionnez *Oui* pour **Jeu d’enregistrements d’alias**. **Type d’alias** doit être défini sur *Ressource Azure*.

1. Sélectionnez l’abonnement Azure sur lequel le profil Front Door est hébergé. Sélectionnez ensuite la ressource Front Door dans la liste déroulante des **ressources Azure**.

1. Sélectionnez **OK** pour envoyer vos modifications.

    :::image type="content" source="./media/front-door-apex-domain/front-door-apex-alias-record.png" alt-text="Enregistrement d’alias d’apex de zone":::

1. L’étape ci-dessus crée un enregistrement d’apex de zone qui pointe vers votre ressource Front Door, ainsi qu’un mappage d’enregistrement CNAME « afdverify » (exemple : `afdverify.contosonews.com`) vers  qui sera utilisé pour l’intégration du domaine à votre profil Front Door.

## <a name="onboard-the-custom-domain-on-your-front-door"></a>Intégrer le domaine personnalisé à votre Front Door

1. Dans l’onglet du concepteur de Front Door, sélectionnez l’icône « + » dans la section des hôtes frontend pour ajouter un nouveau domaine personnalisé.

1. Entrez le nom de domaine racine ou apex dans le champ du nom d’hôte personnalisé, par exemple `contosonews.com`.

1. Une fois le mappage CNAME du domaine à votre Front Door validé, sélectionnez **Ajouter** pour ajouter le domaine personnalisé.

1. Sélectionnez **Enregistrer** pour envoyer les modifications.

   :::image type="content" source="./media/front-door-apex-domain/front-door-onboard-apex-domain.png" alt-text="Menu Domaines personnalisés":::

## <a name="enable-https-on-your-custom-domain"></a>Activer HTTPS sur votre domaine personnalisé

1. Sélectionnez le domaine personnalisé qui a été ajouté puis, sous la section **HTTPS sur un domaine personnalisé**, modifiez l’état sur **Activé**.

1. Sélectionnez le **type de gestion des certificats** *« Utiliser mon propre certificat »* .

   :::image type="content" source="./media/front-door-apex-domain/front-door-onboard-apex-custom-domain.png" alt-text="Paramètres HTTPS d’un domaine personnalisé":::    

   > [!WARNING]
   > Le type de gestion des certificats managés par Front Door n’est actuellement pas pris en charge pour les domaines apex ou racine. La seule option d’activation HTTPS sur un domaine apex ou racine disponible pour Front Door consiste à utiliser votre propre certificat TLS/SSL personnalisé hébergé sur Azure Key Vault.

1. Vérifiez que vous avez configuré les autorisations Front Door appropriées pour accéder à votre coffre de clés, comme indiqué dans l’interface utilisateur, avant de passer à l’étape suivante.

1. Choisissez un **compte Key Vault** dans votre abonnement actuel, puis sélectionnez le **Secret** et la **Version du secret** appropriés à mapper au bon certificat.

1. Sélectionnez **Mettre à jour** pour enregistrer la sélection, puis sélectionnez **Enregistrer**.

1. Après quelques minutes, sélectionnez **Actualiser**, puis sélectionnez une nouvelle fois le domaine personnalisé pour voir la progression de l’approvisionnement de certificats. 

> [!WARNING]
> Vérifiez que vous avez créé des règles d’acheminement appropriées pour votre domaine apex ou ajouté le domaine aux règles d’acheminement existantes.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
