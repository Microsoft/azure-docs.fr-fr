---
title: Impossible d’ajouter un domaine personnalisé à l’aide du certificat Key Vault
titleSuffix: Azure API Management
description: Découvrez comment résoudre le problème qui empêche d’ajouter un domaine personnalisé dans Gestion des API Azure à l’aide d’un certificat de coffre de clés.
services: api-management
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/19/2019
ms.author: tehnoonr
ms.openlocfilehash: a09c15466a4a9f62b2696b087cb7ab23cc767379
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75430576"
---
# <a name="failed-to-update-api-management-service-hostnames"></a>Échec de la mise à jour des noms d’hôte du service Gestion des API

Cet article aborde l’erreur « Échec de la mise à jour des noms d’hôte du service Gestion des API », que vous pouvez rencontrer lorsque vous ajoutez un domaine personnalisé dans le service Gestion des API Azure. Cet article fournit les étapes nécessaires à la résolution de ce problème.

## <a name="symptoms"></a>Symptômes

Lorsque vous tentez d’ajouter un domaine personnalisé pour votre service Gestion des API à l’aide d’un certificat Azure Key Vault, le message d’erreur suivant s’affiche :

- Échec de la mise à jour des noms d’hôte du service Gestion des API. La requête envoyée à la ressource « https://vaultname.vault.azure.net/secrets/secretname/?api-version=7.0  » a échoué avec le code d’état : Forbidden pour RequestId : Message d’exception : L’opération a retourné un code d’état non valide : « Forbidden ».

## <a name="cause"></a>Cause

Le service Gestion des API n’est pas autorisé à accéder au coffre de clés que vous tentez d’utiliser pour le domaine personnalisé.

## <a name="solution"></a>Solution

Pour résoudre ce problème, effectuez les étapes suivantes :

1. Accédez au [portail Azure](Https://portal.azure.com), sélectionnez votre instance Gestion des API, puis sélectionnez **Identités managées**. Vérifiez que l’option **Inscription auprès d’Azure Active Directory** est bien définie sur **Oui**. 
    ![Inscription auprès d’Azure Active Directory](./media/api-management-troubleshoot-cannot-add-custom-domain/register-with-aad.png)
1. Dans le portail Azure, ouvrez le service **Coffres de clés**, puis sélectionnez le coffre de clés que vous souhaitez utiliser pour le domaine personnalisé.
1. Sélectionnez **Stratégies d’accès**, puis vérifiez qu’il existe un principal de service dont le nom correspond à celui de l’instance du service Gestion des API. Si c’est le cas, sélectionnez le principal de service, et vérifiez que l’autorisation **GET** figure bien sous **Autorisations du secret**.  
    ![Ajout d’une stratégie d’accès pour le principal de service](./media/api-management-troubleshoot-cannot-add-custom-domain/access-policy.png)
1. Si le service Gestion des API ne figure pas dans la liste, sélectionnez **Ajouter une stratégie d’accès**, puis créez la stratégie d’accès suivante :
    - **Configurer à partir du modèle** : None
    - **Sélectionner le principal** : recherchez le nom du service Gestion des API, puis sélectionnez-le dans la liste.
    - **Autorisations de clé** : None
    - **Autorisations du secret** : Obtenir
    - **Autorisations de certificat** : None
1. Sélectionnez **OK** pour créer la stratégie d’accès.
1. Sélectionnez **Enregistrer** pour enregistrer les modifications.

Vérifiez si le problème est résolu. Pour ce faire, essayez de créer le domaine personnalisé dans le service Gestion des API à l’aide du certificat du coffre de clés.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur le service Gestion des API :

- Découvrez plus de [vidéos](https://azure.microsoft.com/documentation/videos/index/?services=api-management) sur Gestion des API.
* Pour les autres méthodes permettant de sécuriser votre service principal, consultez [Authentification mutuelle des certificats](api-management-howto-mutual-certificates.md).

* [Création d’une instance du service Gestion des API](get-started-create-service-instance.md).
* [Gérer votre première API](import-and-publish.md).