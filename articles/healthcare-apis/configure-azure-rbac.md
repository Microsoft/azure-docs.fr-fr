---
title: Configurer un contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour l’API Azure pour FHIR
description: Cet article explique comment configurer un RBAC Azure pour le plan de données de l’API Azure pour FHIR
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 03/15/2020
ms.author: matjazl
ms.reviewer: dseven
ms.openlocfilehash: 5cadfad445c76726b1b825b131de4016a57979fa
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93391838"
---
# <a name="configure-azure-rbac-for-fhir"></a>Configurer un RBAC Azure pour FHIR 

Cet article explique comment utiliser un [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/index.yml) pour attribuer l’accès au plan de données de l’API Azure pour FHIR. Un RBAC Azure est la méthode recommandée pour l’attribution de l’accès au plan de données lorsque les utilisateurs du plan de données sont gérés dans le locataire Azure Active Directory associé à votre abonnement Azure. Si vous utilisez un locataire Azure Active Directory externe, consultez la [référence sur l’attribution d’un RBAC local](configure-local-rbac.md).

## <a name="confirm-azure-rbac-mode"></a>Confirmer le mode RBAC Azure

Pour utiliser un RBAC Azure, votre API Azure pour FHIR doit être configurée pour utiliser votre locataire d’abonnement Azure pour le plan de données, et aucun ID d’objet d’identité ne doit être attribué. Vous pouvez vérifier vos paramètres en examinant le panneau **Authentification** de votre API Azure pour FHIR :

:::image type="content" source="media/rbac/confirm-azure-rbac-mode.png" alt-text="Confirmer le mode RBAC Azure":::

L’ **Autorité** définie doit être le locataire Azure Active Directory associé à votre abonnement et aucun GUID ne doit figurer dans la zone **ID d’objet autorisés**. Vous remarquerez également que la zone est désactivée et qu’une étiquette indique qu’un RBAC Azure doit être utilisé pour attribuer des rôles de plan de données.

## <a name="assign-roles"></a>Attribuer des rôles

Pour accorder à des utilisateurs, à des principaux de service ou à des groupes l’accès au plan de données FHIR, cliquez sur **Contrôle d’accès (IAM)** , sur **Attributions de rôles** , puis sur **+ Ajouter**  :

:::image type="content" source="media/rbac/add-azure-rbac-role-assignment.png" alt-text="Ajouter une attribution de rôle Azure":::

Dans la sélection **Rôle** , recherchez l’un des rôles intégrés pour le plan de données FHIR :

:::image type="content" source="media/rbac/built-in-fhir-data-roles.png" alt-text="Rôles de données FHIR intégrés":::

Vous pouvez choisir entre les rôles suivants :

* Lecteur de données FHIR : peut lire (et rechercher) des données FHIR.
* Enregistreur de données FHIR : peut lire, écrire et supprimer de manière réversible des données FHIR.
* Exportateur de données FHIR : peut lire et exporter des données (opérateur `$export`).
* Contributeur aux données FHIR : peut effectuer toutes les opérations de plan de données.

Si ces rôles ne suffisent pour combler vos besoins, vous pouvez également [créer des rôles personnalisés](../role-based-access-control/tutorial-custom-role-powershell.md).

Dans la zone **Sélectionner** , recherchez un utilisateur, un principal de service ou un groupe auquel vous souhaitez attribuer le rôle.

## <a name="caching-behavior"></a>Comportement de mise en cache

L’API Azure pour FHIR met en cache les décisions pendant jusqu’à 5 minutes. Si vous accordez à un utilisateur l’accès au serveur FHIR en l’ajoutant à la liste des ID d’objet autorisés, ou le supprimez de la liste, jusqu’à cinq minutes peuvent être nécessaires pour que les modifications apportées aux autorisations soient propagées.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a expliqué comment attribuer des rôles Azure pour le plan de données FHIR. Pour en savoir plus sur les paramètres supplémentaires pour l’API Azure pour FHIR :
 
>[!div class="nextstepaction"]
>[Paramètres supplémentaires de l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)