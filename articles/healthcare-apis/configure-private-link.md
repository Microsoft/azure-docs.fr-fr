---
title: Liaison privée pour I’API Azure pour FHIR
description: Cet article explique comment configurer un point de terminaison privé pour les services de l’API Azure pour FHIR
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 10/12/2020
ms.author: matjazl
ms.openlocfilehash: bfbdb98e691312db5665261743f8ce698541d4cc
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398213"
---
# <a name="configure-private-link"></a>Configurer une liaison privée

> [!IMPORTANT]
> Cette fonctionnalité en préversion publique est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Une liaison privée vous permet d’accéder à l’API Azure pour FHIR via un point de terminaison privé, une interface réseau qui vous connecte de façon confidentielle et sécurisée en utilisant une adresse IP privée de votre réseau virtuel. Avec une liaison privée, vous pouvez accéder à nos services en toute sécurité à partir de votre réseau virtuel en tant que service interne sans avoir à passer par un DNS public. Cet article explique comment créer, tester et gérer votre point de terminaison privé pour l’API Azure pour FHIR.

## <a name="prerequisites"></a>Prérequis

Avant de créer un point de terminaison privé, vous devez créer certaines ressources Azure :

- Groupe de ressources : groupe de ressources Azure qui contiendra le réseau virtuel et le point de terminaison privé.
- API Azure pour FHIR : ressource FHIR que vous souhaitez placer derrière un point de terminaison privé.
- Réseau virtuel : réseau virtuel auquel vos services client et le point de terminaison privé seront connectés.

Pour plus d’informations, consultez la [Documentation sur Liaison privée](../private-link/index.yml).

## <a name="disable-public-network-access"></a>Désactiver l’accès au réseau public

La création d’un point de terminaison privé pour votre ressource FHIR ne désactive pas automatiquement le trafic public vers celui-ci. Pour ce faire, vous devez mettre à jour votre ressource FHIR pour définir une nouvelle propriété « Accès public » « Activé » sur « Désactivé ». Soyez prudent lorsque vous désactivez l’accès public au réseau, car toutes les demandes adressées à votre service FHIR, qui ne proviennent pas d’un point de terminaison privé correctement configuré sont refusées. Seul le trafic provenant de vos points de terminaison privés est autorisé.

![Désactiver l’accès public au réseau](media/private-link/private-link-disable.png)

## <a name="create-private-endpoint"></a>Créer un point de terminaison privé

Pour créer un point de terminaison privé, un développeur disposant d’autorisations de RBAC sur la ressource FHIR peut utiliser le portail Azure, [Azure PowerShell](../private-link/create-private-endpoint-powershell.md) ou [Azure CLI](../private-link/create-private-endpoint-cli.md). Cet article vous guide dans les étapes d’utilisation du portail Azure. L’utilisation du portail Azure est recommandée, car elle automatise la création et la configuration de la zone DNS privé. Pour plus d’informations, consultez les [Guides de démarrage rapide de liaison privée](../private-link/create-private-endpoint-portal.md).

Vous pouvez créer un point de terminaison privé de deux façons. Le flux d’approbation automatique permet à un utilisateur disposant d’autorisations de RBAC sur la ressource FHIR de créer un point de terminaison privé sans avoir besoin d’approbation. Le flux d’approbation manuelle permet à un utilisateur dépourvu d’autorisation sur la ressource FHIR de demander l’approbation d’un point de terminaison privé par les propriétaires de la ressource FHIR.

### <a name="auto-approval"></a>Approbation automatique

Assurez-vous que la région du nouveau point de terminaison privé est identique à la région de votre réseau virtuel. La région de votre ressource FHIR peut être différente.

![Onglet Options de base du portail Azure](media/private-link/private-link-portal2.png)

Pour Type de ressource, recherchez et sélectionnez « Microsoft.HealthcareApis/services ». Pour Ressource, sélectionnez la ressource FHIR. Pour Sous-ressource cible, sélectionnez « fhir ».

![Onglet Ressource du portail Azure](media/private-link/private-link-portal1.png)

Si vous ne disposez pas d’une zone DNS privé configurée, sélectionnez « (New)privatelink.azurehealthcareapis.com ». Si vous disposez d’une zone DNS privé configurée, vous pouvez la sélectionner dans la liste. Elle doit être au format « privatelink.azurehealthcareapis.com ».

![Onglet Configuration du portail Azure](media/private-link/private-link-portal3.png)

Une fois le déploiement terminé, vous pouvez revenir à l’onglet « Connexions de points de terminaison privés » sous lequel l’état de la connexion est « Approuvée ».

### <a name="manual-approval"></a>Approbation manuelle

Pour une approbation manuelle, sous Ressource, sélectionnez la deuxième option, « Se connecter à une ressource Azure par ID de ressource ou alias ». Pour Sous-ressource cible, dans Approbation automatique, entrez « fhir ».

![Approbation manuelle](media/private-link/private-link-manual.png)

Une fois le déploiement terminé, vous pouvez revenir à l’onglet « Connexions de points de terminaison privés », sous lequel vous pouvez approuver, rejeter ou supprimer votre connexion.

![Options](media/private-link/private-link-options.png)

## <a name="test-private-endpoint"></a>Tester un point de terminaison privé

Pour vous assurer que votre serveur FHIR ne reçoive pas de trafic public après la désactivation de l’accès public au réseau, essayez d’atteindre le point de terminaison /metadata de votre serveur à partir de votre ordinateur. Le message 403 Interdit devrait d’afficher. Notez que, suite à la mise à jour de l’indicateur d’accès public au réseau, jusqu’à 5 minutes peuvent s’écouler avant que le trafic public soit bloqué.

Pour vous assurer que votre point de terminaison privé peut acheminer le trafic vers votre serveur :

1. Créez une machine virtuelle connectée au réseau virtuel et au sous-réseau sur lequel votre point de terminaison privé est configuré. Pour vous assurer que votre trafic en provenance de la machine virtuelle emprunte uniquement le réseau privé, vous pouvez désactiver le trafic Internet sortant via une règle de groupe de sécurité réseau (NSG).
2. RDP dans la machine virtuelle.
3. Essayez d’atteindre le point de terminaison /metadata de votre serveur FHIR à partir de la machine virtuelle. Vous devriez recevoir la déclaration de capacité en réponse.

## <a name="manage-private-endpoint"></a>Point de terminaison privé managé

### <a name="view"></a>Affichage

Les points de terminaison privés et la carte réseau associée sont visibles dans le portail Azure à partir du groupe de ressources dans lequel ils ont été créés.

![Affichage dans les ressources](media/private-link/private-link-view.png)

### <a name="delete"></a>Supprimer

Vous ne pouvez supprimer des points de terminaison du portail Azure que via le panneau Vue d’ensemble (comme ci-dessous) ou l’option Supprimer sous l’onglet « Connexions de points de terminaison privés » de Mise en réseau (préversion). Un clic sur le bouton Supprimer pour effet de supprimer le point de terminaison privé et la carte réseau associée. Si vous supprimez tous les points de terminaison privés de la ressource FHIR et que l’accès public au réseau est désactivé, aucune demande n’est adressée à votre serveur FHIR. Avant de pouvoir supprimer ou déplacer la ressource FHIR, tous les points de terminaison privés doivent être supprimés de la ressource FHIR.

![Supprimer un point de terminaison privé](media/private-link/private-link-delete.png)