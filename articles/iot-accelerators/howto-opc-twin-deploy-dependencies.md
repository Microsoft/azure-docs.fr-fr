---
title: Guide pratique pour déployer des dépendances cloud OPC Twin dans Azure | Microsoft Docs
description: Cet article explique comment déployer les dépendances Azure OPC Twin nécessaires au développement et au débogage local.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 42024fc506de7befed7c44ebcc410756b6f43a35
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078984"
---
# <a name="deploying-dependencies-for-local-development"></a>Déploiement de dépendances pour un développement local

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article explique comment déployer seulement les services de plateforme Azure nécessaires pour le développement et le débogage local.   À la fin, vous aurez un groupe de ressources déployé qui contient tout ce dont vous avez besoin pour le développement et le débogage local.

## <a name="deploy-azure-platform-services"></a>Déployer des services de plateforme Azure

1. Vérifiez que PowerShell et les extensions [AzureRM PowerShell](/powershell/azure/azurerm/install-azurerm-ps) sont installés.  Ouvrez une invite de commandes ou un terminal, et exécutez :

   ```bash
   git clone https://github.com/Azure/azure-iiot-components
   cd azure-iiot-components
   ```

   ```bash
   deploy -type local
   ```

2. Suivez les invites pour affecter un nom au groupe de ressources pour votre déploiement.  Le script déploie seulement les dépendances sur ce groupe de ressources dans votre abonnement Azure, mais pas les microservices.  Le script inscrit également une application dans Azure Active Directory.  Ceci est nécessaire pour la prise en charge de l’authentification OAUTH.  Le déploiement peut prendre plusieurs minutes.

3. Une fois le script terminé, vous pouvez choisir d’enregistrer le fichier .env.  Le fichier d’environnement .env est le fichier de configuration de tous les services et outils que vous voulez exécuter sur votre machine de développement.  

## <a name="troubleshooting-deployment-failures"></a>Résoudre les problèmes de déploiement

### <a name="resource-group-name"></a>Nom de groupe ressources

Veillez à utiliser un nom de groupe de ressources court et simple.  Le nom est également utilisé pour nommer les ressources : il doit donc être conforme aux spécifications de nommage des ressources.  

### <a name="azure-active-directory-ad-registration"></a>Inscription dans Azure Active Directory (AD)

Le script de déploiement essaie d’inscrire les applications Azure AD dans Azure AD.  En fonction de vos droits sur le locataire Azure AD sélectionné, cette opération peut échouer. Vous disposez de trois options :

1. Si vous avez choisi un locataire Azure AD dans une liste de locataires, redémarrez le script et choisissez un autre locataire dans la liste.
2. Vous pouvez aussi déployer un locataire Azure AD privé, redémarrer le script et choisir de l’utiliser.
3. Continuer sans authentification.  Comme vous exécutez vos microservices localement, ceci est acceptable, mais ne reproduit pas les environnements de production.  

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez déployé des services OPC Twin sur un projet existant, voici l’étape suivante suggérée :

> [!div class="nextstepaction"]
> [Découvrir comment déployer des modules OPC Twin](howto-opc-twin-deploy-modules.md)