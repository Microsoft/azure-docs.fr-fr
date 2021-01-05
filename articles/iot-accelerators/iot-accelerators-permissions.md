---
title: Utiliser le site des solutions Azure IoT - Azure | Microsoft Docs
description: Explique comment utiliser le site web AzureIoTSolutions.com pour déployer votre accélérateur de solution.
author: dominicbetts
manager: philmea
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/13/2018
ms.author: dobett
ms.openlocfilehash: 5a088cb07fa74b65e497de54be8c97bd51d31c16
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387834"
---
# <a name="use-the-azureiotsolutionscom-site-to-deploy-your-solution-accelerator"></a>Utiliser le site azureiotsolutions.com pour déployer votre accélérateur de solution

Vous pouvez déployer des accélérateurs de solution Azure IoT sur votre abonnement Azure à partir du site [AzureIoTSolutions.com](https://www.azureiotsolutions.com/Accelerators). AzureIoTSolutions.com héberge à la fois les accélérateurs de solution open source Microsoft et de partenaire. Ces accélérateurs de solutions sont adaptés à [l’architecture de référence Azure IoT](/azure/architecture/reference-architectures/iot). Vous pouvez utiliser le site pour déployer rapidement un accélérateur de solution comme environnement de démonstration ou de production.

:::image type="content" source="media/iot-accelerators-permissions/iotsolutionscom.png" alt-text="Page d’accueil des solutions IoT":::

> [!TIP]
> Si vous avez besoin de davantage de contrôle sur le processus de déploiement, vous pouvez utiliser l’interface CLI pour déployer un accélérateur de solution.

Vous pouvez déployer les accélérateurs de solution dans les configurations suivantes :

* **Standard** : un déploiement étendu de l’infrastructure pour le développement d’un environnement de production. Azure Container Service déploie les microservices sur plusieurs machines virtuelles Azure. Kubernetes orchestre les conteneurs Docker qui hébergent les microservices individuels.
* **De base** : une version à coût réduit pour une démonstration ou pour tester un déploiement. Tous les microservices se déploient sur une seule machine virtuelle Azure.
* **Local** : un déploiement de machine local pour les tests et le développement. Cette approche déploie les microservices vers un conteneur Docker local et se connecte aux services IoT Hub, Azure Cosmos DB et Stockage Azure dans le cloud.

Chacun des accélérateurs de solution utilise une combinaison différente des services Azure comme IoT Hub, Azure Stream Analytics et Cosmos DB. Pour plus d’informations, visitez [AzureIoTSolutions.com](https://www.azureiotsolutions.com/Accelerators) et sélectionnez un accélérateur de solution.

## <a name="sign-in-at-azureiotsolutionscom"></a>Se connecter à azureiotsolutions.com

Pour pouvoir déployer un accélérateur de solution, vous devez vous connecter à AzureIoTSolutions.com à l’aide des informations d’identification associées à un abonnement Azure. Si votre compte est associé à plusieurs locataires Microsoft Azure Active Directory (AD), vous pouvez utiliser la **liste déroulante de sélection de compte** pour choisir l’annuaire à utiliser.

Vos autorisations pour déployer des accélérateurs de solutions, gérer les utilisateurs et gérer les services Azure dépendent de votre rôle dans l’annuaire sélectionné. Les rôles Azure AD courants associés aux accélérateurs de solutions sont les suivants :

* **Administrateur général** : il peut y avoir de nombreux [administrateurs généraux](../active-directory/roles/permissions-reference.md) pour chaque locataire Azure AD :

  * Quand vous créez un locataire Azure AD, vous en êtes l’administrateur général par défaut.
  * L’administrateur général peut déployer des accélérateurs de solution de base et standard.

* **Utilisateur de domaine** : il peut y avoir de nombreux utilisateurs de domaine pour chaque locataire Azure AD. Un utilisateur de domaine peut déployer un accélérateur de solution de base.

* **Utilisateur invité** : il peut y avoir de nombreux utilisateurs invités pour chaque locataire Azure AD. Les utilisateurs invités ne peuvent pas déployer un accélérateur de solution dans le locataire Azure AD.

Pour plus d’informations sur les utilisateurs et les rôles dans Azure AD, consultez les ressources suivantes :

* [Créer des utilisateurs dans Azure Active Directory](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)
* [Affecter des utilisateurs aux applications](../active-directory/manage-apps/assign-user-or-group-access-portal.md)

## <a name="choose-your-device"></a>Choisir votre appareil

Le site AzureIoTSolutions.com propose un lien vers le [catalogue d’appareils Azure Certified pour IoT](https://catalog.azureiotsolutions.com/).

Ce catalogue liste des centaines d’appareils matériels IoT certifiés que vous pouvez connecter à vos accélérateurs de solution pour commencer à générer votre solution IoT.

![Catalogue d’appareils](media/iot-accelerators-permissions/devicecatalog.png)

Si vous êtes fabricant de matériel, cliquez sur **Devenir un partenaire** pour en savoir plus sur la façon de développer un partenariat avec Microsoft sur le programme Certifié pour IoT.

## <a name="next-steps"></a>Étapes suivantes

Pour essayer l’un des accélérateurs de solution IoT, consultez le démarrage rapide : [Essayer une solution de fabrique connectée](quickstart-connected-factory-deploy.md).
