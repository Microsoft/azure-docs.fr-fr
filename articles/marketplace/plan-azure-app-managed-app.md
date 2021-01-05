---
title: Planifier une application managée Azure pour une offre d’application Azure
description: Découvrez ce qui est nécessaire pour créer un plan d’application managée pour une nouvelle offre d’application Azure à l’aide du portail de la place de marché commerciale dans l’Espace partenaires Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 694f501efc565ed498c1c8d8e2e38326277e8605
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621414"
---
# <a name="plan-an-azure-managed-application-for-an-azure-application-offer"></a>Planifier une application managée Azure pour une offre d’application Azure

Un plan d’_application managée_ Azure est un moyen de publier une offre d’application Azure dans la Place de marché Azure. Si vous ne l’avez pas encore fait, lisez [Planifier une offre d’application Azure pour la place de marché commerciale](plan-azure-application-offer.md).

Les applications managées sont des offres de transaction qui sont déployées et facturées via Place de marché Azure. L’option de référencement qu’un utilisateur voit est Obtenir maintenant.

Utiliser une application Azure : Plan d’application managée lorsque les conditions suivantes sont requises :

- Vous allez déployer une solution basée sur un abonnement pour votre client à l’aide d’une machine virtuelle (VM) ou d’une solution basée sur une infrastructure as a service (IaaS) complète.
- Vous ou votre client exigez que la solution soit managée par un partenaire. Par exemple, un partenaire peut être un intégrateur système ou un fournisseur de services managés (MSP).

## <a name="managed-application-offer-requirements"></a>Exigences relatives à une offre d’application managée

| Spécifications | Détails |
| ------------ | ------------- |
| Abonnement Azure | Les applications gérées doivent être déployées dans l’abonnement d’un client, mais elles peuvent être gérées par un tiers. |
| Facturation et mesure | Les ressources sont fournies dans l’abonnement Azure d’un client. Les machines virtuelles qui utilisent le modèle de paiement à l’utilisation font l’objet de transactions avec le client par le biais de Microsoft et sont facturées dans le cadre de l’abonnement Azure du client. <br><br> Pour les machines virtuelles BYOL (apportez votre propre licence), Microsoft facture tous les frais d’infrastructure engagés dans l’abonnement client, mais vous effectuez la transaction de vos frais de licence logicielle directement avec le client. |
| Disque dur virtuel compatible avec Azure | Les machines virtuelles doivent être basées sur Windows ou Linux. Pour plus d'informations, consultez les pages suivantes :<br> • [Créer une ressource technique de machine virtuelle Azure](./azure-vm-create-certification-faq.md#address-a-vulnerability-or-an-exploit-in-a-vm-offer) (pour les disques durs virtuels Windows).<br> • [Distributions Linux approuvées sur Azure](../virtual-machines/linux/endorsed-distros.md) (pour les disques durs virtuel Linux). |
| Attribution de l’utilisation client | Toutes les nouvelles offres d’applications Azure doivent également inclure un GUID d’[attribution de l’utilisation de client partenaire Azure](azure-partner-customer-usage-attribution.md). Pour plus d’informations sur l’attribution de l’utilisation de client et sur son activation, consultez [Attribution de l’utilisation de client partenaire Azure](azure-partner-customer-usage-attribution.md). |
| Package de déploiement | Vous aurez besoin d’un package de déploiement qui permettra aux clients de déployer votre plan. Si vous créez plusieurs plans nécessitant la même configuration technique, vous pouvez utiliser le même package. Pour plus d’informations, consultez la section suivante : Package de déploiement. |
|||

> [!NOTE]
> Les applications managées doivent pouvoir être déployées via Place de marché. Si la communication client est un problème, contactez les clients intéressés une fois que vous avez activé le partage des prospects.

## <a name="deployment-package"></a>Package de déploiement

Le package de déploiement regroupe tous les fichiers de modèle requis pour ce plan, ainsi que toutes les ressources supplémentaires, empaquetés dans un fichier .zip.

Toutes les applications Azure doivent inclure les deux fichiers suivants dans le dossier racine d’une archive .zip :

- Un fichier de modèle Resource Manager nommé [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). Ce modèle qui définit les ressources à déployer dans l’abonnement Azure du client. Pour obtenir des exemples de modèles Resource Manager, consultez la [Galerie de modèles de démarrage rapide Microsoft Azure](https://azure.microsoft.com/documentation/templates/) ou le dépôt [GitHub : Modèles de démarrage rapide Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondant.
- Une définition d’interface utilisateur pour l’expérience de création d’applications Azure nommée [createUiDefinition.json](../azure-resource-manager/managed-applications/create-uidefinition-overview.md). Dans l’interface utilisateur, vous spécifiez les éléments qui permettent aux consommateurs de fournir des valeurs de paramètre.

Tailles maximales de fichiers prises en charge :

- jusqu’à 1 Go dans la taille totale de l’archive .zip compressée ;
- jusqu’à 1 Go pour tous les fichiers non compressés individuels au sein de l’archive .zip.

Toutes les nouvelles offres d’applications Azure doivent également inclure un GUID d’[attribution de l’utilisation de client partenaire Azure](azure-partner-customer-usage-attribution.md).

## <a name="azure-regions"></a>Régions Azure

Vous pouvez publier votre plan dans la région publique Azure, la région Azure Government, ou les deux. Avant de le publier dans [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md), testez et validez votre plan dans l’environnement, car certains points de terminaison peuvent être différents. Pour configurer et tester votre plan, demandez un compte d’essai depuis la page [Microsoft Azure Government Trial](https://azure.microsoft.com/global-infrastructure/government/request/).

En tant qu’éditeur, il vous incombe de mettre en place l’ensemble des contrôles de conformité, des mesures de sécurité et des bonnes pratiques nécessaires. Azure Government utilise des réseaux et des centres de données qui sont physiquement isolés (situés aux États-Unis uniquement).

Pour obtenir la liste des pays et régions que dessert la Place de marché commerciale, consultez [Prise en charge de la disponibilité géographique et des devises](marketplace-geo-availability-currencies.md).

Les services Azure Government gèrent des données soumises à certaines réglementations et exigences gouvernementales. Par exemple, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 et CJIS. Afin de renseigner les clients sur vos certifications pour ces programmes, vous pouvez fournir jusqu’à 100 liens décrivant ces certifications. Ces liens peuvent être des liens vers votre liste de plans directement sur le programme, ou des liens vers des descriptions de votre conformité avec ces certifications sur vos propres sites web. Ces liens sont visibles uniquement par les clients Azure Government.

## <a name="choose-who-can-see-your-plan"></a>Choisir qui peut voir votre plan

Vous pouvez configurer chaque plan de sorte qu’il soit visible de tous (public) ou d’un public spécifique (privé). Vous pouvez créer jusqu’à 100 plans dont jusqu’à 45 peuvent être privés. Vous pouvez créer un plan privé pour offrir différentes options de tarification ou configurations techniques à des clients spécifiques.

Vous accordez l’accès à un plan privé à l’aide d’ID d’abonnement Azure, avec la possibilité d’inclure une description de chaque ID d’abonnement que vous attribuez. Vous pouvez ajouter jusqu’à 10 ID d’abonnement manuellement ou jusqu’à 10 000 ID d’abonnement à l’aide d’un fichier .CSV. Les ID d’abonnements Azure sont représentés par des GUID, où les lettres doivent être en minuscules.

Les plans privés ne sont pas pris en charge avec les abonnements Azure souscrits via un revendeur participant au programme Fournisseurs de solutions cloud (CSP). Pour plus d’informations, consultez [Offres privées dans la Place de marché commerciale Microsoft](private-offers.md).

> [!NOTE]
> Si vous publiez un plan privé, vous pouvez modifier sa visibilité sur public ultérieurement. Toutefois, une fois que vous avez publié un plan public, vous ne pouvez pas changer sa visibilité sur privé.

## <a name="define-pricing"></a>Définir la tarification

Vous devez indiquer le tarif mensuel de chaque plan. Ce tarif s’ajoute aux coûts logiciels de paiement à l’utilisation et aux coûts de l’infrastructure Azure qui sont engendrés par les ressources déployées par cette solution.

En plus du tarif mensuel, vous pouvez fixer des prix pour l’utilisation d’unités non standard assortie d’une [facturation basée sur des mesures](partner-center-portal/azure-app-metered-billing.md). Si vous le souhaitez, vous pouvez définir le tarif mensuel sur zéro et facturer exclusivement via une facturation basée sur des mesures.

Les prix fixés en USD (USD = dollar américain) sont convertis dans la devise locale de tous les marchés sélectionnés, selon les taux de change en vigueur disponibles lors de l’enregistrement. Toutefois, vous pouvez choisir de définir les prix des clients pour chaque marché.

## <a name="just-in-time-jit-access"></a>Accès juste-à-temps (JIT)

L'accès JIT vous permet de demander un accès élevé aux ressources d'une application managée à des fins de résolution des problèmes ou de maintenance. Vous bénéficiez toujours d'un accès en lecture seule aux ressources, mais pour une durée spécifique, votre accès est étendu. Pour plus d’informations, consultez [Activer et demander l’accès juste-à-temps pour les applications managées Azure](../azure-resource-manager/managed-applications/request-just-in-time-access.md).

> [!NOTE]
> N’oubliez pas de mettre à jour votre fichier `createUiDefinition.json` pour la prise en charge de cette fonctionnalité.

## <a name="deployment-mode"></a>Mode de déploiement

Vous pouvez configurer un plan d’application managée pour qu’il utilise le mode de déploiement **Complet** ou **Incrémentiel**. En mode Complet, un redéploiement de l’application par le client entraîne la suppression, dans le groupe de ressources managées, des ressources non définies dans le fichier [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). En mode incrémentiel, un redéploiement de l’application laisse les ressources existantes inchangées. Pour en savoir plus, consultez [Modes de déploiement d’Azure Resource Manager](../azure-resource-manager/templates/deployment-modes.md).

## <a name="notification-endpoint-url"></a>URL de point de terminaison de notification

En option, vous pouvez également spécifier un point de terminaison Webhook HTTPS pour recevoir des notifications de toutes les opérations CRUD effectuées sur les instances d’application managée d’un plan.

## <a name="customize-allowed-customer-actions-optional"></a>Personnaliser les actions des clients autorisées (facultatif)

En option, vous pouvez spécifier les actions que les clients peuvent effectuer sur les ressources managées en plus des actions `*/read` autorisées par défaut.

Si vous choisissez cette option, vous devez fournir les actions de contrôle, les actions de données autorisées ou les deux. Pour plus d’informations, consultez [Comprendre les affectations de refus relatives aux ressources Azure](../role-based-access-control/deny-assignments.md). Pour obtenir la liste des actions disponibles, consultez [Opérations du fournisseur de ressources Azure Resource Manager](../role-based-access-control/resource-provider-operations.md). Par exemple, pour permettre aux consommateurs de redémarrer des machines virtuelles, ajoutez `Microsoft.Compute/virtualMachines/restart/action` aux actions autorisées.

## <a name="choose-who-can-manage-the-application"></a>Choisir qui peut gérer l’application

Vous devez indiquer qui peut gérer une application managée dans chacun des clouds sélectionnés : _Cloud Azure public_ et _cloud Azure Government_. Collectez les informations suivantes :

- **ID de locataire Azure Active Directory** : ID du locataire Azure AD (également appelé ID d’annuaire) contenant les identités des utilisateurs, des groupes ou des applications auxquels vous souhaitez accorder des autorisations. Cet ID est indiqué dans les [propriétés d’Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties) sur le portail Azure.
- **Autorisations** : ajoutez l’ID d’objet Azure Active Directory de chaque utilisateur, groupe ou application que vous souhaitez autoriser à accéder au groupe de ressources managées. Identifiez l’utilisateur par son ID de principal, que vous trouverez dans le [panneau des utilisateurs Azure Active Directory sur le portail Azure](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers).

Pour chaque ID principal, vous allez associer l’un des rôles intégrés Azure AD (Propriétaire ou Contributeur). Le rôle sélectionné détermine les autorisations que le principal aura sur les ressources dans l’abonnement client. Pour plus d’informations, voir [Rôles intégrés Azure](../role-based-access-control/built-in-roles.md). Pour plus d’informations sur le contrôle d’accès en fonction du rôle (RBAC), consultez [Bien démarrer avec le contrôle d’accès en fonction du rôle (RBAC) dans le portail Azure](../role-based-access-control/overview.md).

> [!NOTE]
> Vous avez la possibilité d'ajouter jusqu'à 100 autorisations par région Azure. Toutefois, il est généralement plus simple de créer un groupe d'utilisateurs Active Directory et de spécifier son ID dans le champ « ID du principal ». Cela vous permet d’ajouter des utilisateurs au groupe d’administration après le déploiement du plan, et a pour effet de réduire le besoin de mise à jour du plan à l’ajout de quelques autorisations.

## <a name="policy-settings"></a>Paramètres de stratégie

Vous pouvez appliquer des [stratégies Azure](../governance/policy/index.yml) à votre application managée afin de spécifier des exigences de conformité pour la solution déployée. Pour les définitions de stratégie et le format des valeurs de paramètre, consultez [Exemples Azure Policy](../governance/policy/samples/index.md).

Vous pouvez configurer un maximum de cinq stratégies, et une seule instance de chaque type de stratégie. Certains types de stratégies requièrent des paramètres supplémentaires.

| Type de stratégie | Paramètres de stratégie requis |
| ------------ | ------------- |
| Chiffrement d’Azure SQL Database | Non |
| Paramètres d’audit d’Azure SQL Server | Oui |
| Chiffrement d’Azure Data Lake Store | Non |
| Auditer le paramètre de diagnostic | Oui |
| Auditer la conformité de l’emplacement des ressources | Non |
|||

Pour chaque type de stratégie que vous ajoutez, vous devez associer une référence SKU de stratégie standard ou gratuite. La référence SKU Standard est requise pour les stratégies d’audit. Les noms de requête sont limités à 50 caractères.

## <a name="next-steps"></a>Étapes suivantes

- [Comment créer une offre d’application Azure dans la Place de marché commerciale](create-new-azure-apps-offer.md)
