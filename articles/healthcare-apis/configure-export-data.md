---
title: Configurer les paramètres d’exportation dans API Azure pour FHIR
description: Cet article décrit comment configurer les paramètres d’exportation dans API Azure pour FHIR
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 3/5/2020
ms.author: matjazl
ms.openlocfilehash: e4adceea5c2cd2a36d7a867ca9b9d2ad7c33c155
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91529981"
---
# <a name="configure-export-setting-and-set-up-the-storage-account"></a>Configurer le paramètre d’exportation et le compte de stockage

API Azure pour FHIR prend en charge la commande $export qui vous permet d’exporter les données d’un compte API Azure pour FHIR vers un compte de stockage.

Trois étapes sont nécessaires à la configuration de l’exportation dans API Azure pour FHIR :

1. Activer l’identité managée sur le service API Azure pour FHIR
2. Créer un compte de stockage Azure (si vous ne l’avez pas déjà fait) et attribuer au compte de stockage une autorisation d’accès à API Azure pour FHIR
3. Sélectionner le compte de stockage dans API Azure pour FHIR en tant que compte de stockage d’exportation

## <a name="enabling-managed-identity-on-azure-api-for-fhir"></a>Activation de l’identité managée sur API Azure pour FHIR

La première étape de la configuration d’API Azure pour FHIR en vue de l’exportation consiste à activer l’identité managée à l’échelle du système sur le service. Vous pouvez lire toutes les informations relatives aux identités managées dans Azure [ici](../active-directory/managed-identities-azure-resources/overview.md).

Pour ce faire, accédez au service API Azure pour FHIR et sélectionnez le panneau Identité. La modification de l’état sur activé permettra d’activer l’identité managée dans le service API Azure pour FHIR.

![Activer une identité managée](media/export-data/fhir-mi-enabled.png)

Nous pouvons maintenant passer à l’étape suivante et créer un compte de stockage et attribuer une autorisation à notre service.

## <a name="adding-permission-to-storage-account"></a>Ajout d’autorisations au compte de stockage

L’étape suivante de l’exportation consiste à attribuer au service API Azure pour FHIR l’autorisation d’écrire dans le compte de stockage.

Après avoir créé un compte de stockage, accédez au panneau Access Control (IAM) dans Compte de stockage et sélectionnez Ajouter des attributions de rôles.

![Exporter une attribution de rôle](media/export-data/fhir-export-role-assignment.png)

Ici, nous ajoutons ensuite le rôle Contributeur aux données blob du stockage à notre nom de service.

![Ajout de rôle](media/export-data/fhir-export-role-add.png)

Nous sommes maintenant prêts pour l’étape suivante, dans laquelle nous pouvons sélectionner le compte de stockage dans API Azure pour FHIR comme compte de stockage par défaut pour $export.

## <a name="selecting-the-storage-account-for-export"></a>Sélection du compte de stockage pour $export

La dernière étape consiste à attribuer le compte Stockage Azure vers lequel API Azure pour FHIR exportera les données. Pour ce faire, accédez au panneau Intégration du service API Azure pour FHIR dans le portail Azure et sélectionnez le compte de stockage.

![Stockage d’exportation FHIR](media/export-data/fhir-export-storage.png)

Après cela, nous sommes prêts à exporter les données à l’aide de la commande $export.

>[!div class="nextstepaction"]
>[Paramètres supplémentaires](azure-api-for-fhir-additional-settings.md)
