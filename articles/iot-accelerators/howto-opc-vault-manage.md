---
title: Guide pratique pour gérer le service de certificats OPC Vault - Azure | Microsoft Docs
description: Gérez les certificats d’autorité de certification racine OPC Vault et les autorisations utilisateur.
author: mregen
ms.author: mregen
ms.date: 8/16/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 0829d4b3fca068ddb0db2df53dd635ab7ad80bed
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91281913"
---
# <a name="manage-the-opc-vault-certificate-service"></a>Gérer le service de certificats OPC Vault

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article décrit les tâches d’administration liées au service de gestion de certificats OPC Vault dans Azure. Il indique notamment comment renouveler les certificats de l’autorité de certification émettrice, comment renouveler la liste de révocation de certificats et comment accorder et révoquer un accès utilisateur.

## <a name="create-or-renew-the-root-ca-certificate"></a>Créer ou renouveler le certificat d’autorité de certification racine

Après avoir déployé OPC Vault, vous devez créer le certificat d’autorité de certification racine. Sans un certificat d’autorité de certification émettrice valide, vous ne pouvez pas signer ou émettre de certificats d’application. Consultez [Certificats](howto-opc-vault-secure-ca.md#certificates) pour gérer vos certificats avec des durées de vie raisonnables et garanties. Renouvelez un certificat d’autorité de certification émettrice une fois qu’il a dépassé la moitié de sa durée de vie. À cette occasion, tenez aussi compte du fait que la durée de vie configurée d’un certificat d’application nouvellement signé ne doit pas dépasser la durée de vie du certificat de l’autorité de certification émettrice.
> [!IMPORTANT]
> Le rôle Administrateur est nécessaire pour créer ou renouveler le certificat de l’autorité de certification émettrice.

1. Ouvrez votre service de certificats à l’adresse `https://myResourceGroup-app.azurewebsites.net`, puis connectez-vous.
2. Accédez à **Certificate Groups** (Groupes de certificats).
3. Un groupe de certificats par défaut est listé. Sélectionnez **Modifier**.
4. Dans **Edit Certificate Group Details** (Modifier les détails du groupe de certificats), vous pouvez modifier le nom du sujet et la durée de vie de vos certificats d’application et d’autorité de certification. Le sujet et les durées de vie ne doivent être définis qu’une seule fois avant l’émission du premier certificat d’autorité de certification. Les changements de durée de vie pendant les opérations peuvent entraîner des discordances entre les durées de vie des certificats émis et les listes de révocation de certificats.
5. Entrez un sujet valide (par exemple, `CN=My CA Root, O=MyCompany, OU=MyDepartment`).<br>
   > [!IMPORTANT]
   > Si vous modifiez le sujet, vous devez renouveler le certificat émetteur, sans quoi le service ne pourra pas signer les certificats d’application. Le sujet de la configuration est comparé au sujet du certificat émetteur actif. Si les sujets ne correspondent pas, la signature du certificat est refusée.
6. Sélectionnez **Enregistrer**.
7. Si, à ce stade, vous rencontrez une erreur qui indique une interdiction, cela signifie que vos informations d’identification utilisateur ne disposent pas de l’autorisation d’administrateur permettant de modifier ou de créer un certificat racine. Par défaut, l’utilisateur qui a déployé le service possède les rôles d’administrateur et de signature pour le service. Les autres utilisateurs doivent être ajoutés aux rôles Approbateur, Rédacteur ou Administrateur (selon le cas) pendant l’inscription d’application Azure Active Directory (Azure AD).
8. Sélectionnez **Details** (Détails). Des informations mises à jour s’affichent à l’écran.
9. Sélectionnez **Renew CA Certificate** (Renouveler le certificat d’autorité de certification) pour émettre le premier certificat d’autorité de certification émettrice ou pour renouveler le certificat émetteur. Sélectionnez ensuite **OK**.
10. Au bout de quelques secondes, **Certificate Details** (Détails du certificat) s’affiche. Pour télécharger le certificat d’autorité de certification et la liste de révocation de certificats les plus récents en vue de les distribuer à vos applications OPC UA, sélectionnez **Issuer** (Émetteur) ou **Crl** (Liste de révocation de certificats).

Le service de gestion de certificats OPC UA est maintenant prêt à émettre des certificats pour les applications OPC UA.

## <a name="renew-the-crl"></a>Renouveler la liste de révocation de certificats (CRL)

Le renouvellement de la liste de révocation de certificats est une mise à jour qui doit être distribuée aux applications à intervalles réguliers. Les appareils OPC UA, qui prennent en charge l’extension Point de distribution de liste de révocation de certificats x509, peuvent mettre à jour la liste de révocation de certificats directement à partir du point de terminaison du microservice. Les autres appareils OPC UA peuvent nécessiter des mises à jour manuelles ou peuvent être mis à jour à l’aide d’extensions de transmission (de type push) des messages par le serveur GDS (*) pour mettre à jour les listes de confiance avec les certificats et les listes de révocation de certificats.

Dans le workflow suivant, toutes les demandes de certificats à l’état supprimé sont révoquées dans les listes de révocation de certificats, ce qui correspond au certificat d’autorité de certification émettrice pour lequel elles ont été émises. Le numéro de version de la liste de révocation de certificats est incrémenté de 1. <br>
> [!NOTE]
> Toutes les listes de révocation de certificats émises sont valides jusqu’à l’expiration du certificat de l’autorité de certification émettrice. Cela est dû au fait que la spécification OPC UA n’exige pas de modèle de distribution déterministe obligatoire pour la liste de révocation de certificats.

> [!IMPORTANT]
> Le rôle Administrateur est nécessaire pour renouveler la liste de révocation de certificats émettrice.

1. Ouvrez votre service de certificats à l’adresse `https://myResourceGroup.azurewebsites.net`, puis connectez-vous.
2. Accédez à la page **Certificate Groups** (Groupes de certificats).
3. Sélectionnez **Details** (Détails). Les informations sur le certificat et la liste de révocation de certificats actuels s’affichent.
4. Sélectionnez **Update CRL Revocation List (CRL)** (Mettre à jour la liste de révocation de certificats) pour émettre une liste de révocation de certificats mise à jour pour tous les certificats émetteurs actifs dans le stockage OPC Vault.
5. Au bout de quelques secondes, **Certificate Details** (Détails du certificat) s’affiche. Pour télécharger le certificat d’autorité de certification et la liste de révocation de certificats les plus récents en vue de les distribuer à vos applications OPC UA, sélectionnez **Issuer** (Émetteur) ou **Crl** (Liste de révocation de certificats).

## <a name="manage-user-roles"></a>Gérer les rôles d’utilisateur

Vous pouvez gérer les rôles d’utilisateur pour le microservice OPC Vault dans l’application d’entreprise Azure AD. Pour obtenir une description détaillée des définitions de rôles, consultez [Rôles](howto-opc-vault-secure-ca.md#roles).

Par défaut, un utilisateur authentifié dans le locataire peut se connecter dans le service en tant que Lecteur. Les rôles dotés de privilèges supérieurs doivent être gérés manuellement sur le portail Azure ou à l’aide de PowerShell.

### <a name="add-user"></a>Ajouter un utilisateur

1. Ouvrez le portail Azure.
2. Accédez à **Azure Active Directory** > **Enterprise applications (Applications d’entreprise)** .
3. Choisissez l’inscription du microservice OPC Vault (par défaut, votre `resourceGroupName-service`).
4. Accédez à **Users and Groups** (Utilisateurs et Groupes).
5. Sélectionnez **Add User** (Ajouter un utilisateur).
6. Sélectionnez ou invitez l’utilisateur afin de l’attribuer à un rôle spécifique.
7. Sélectionnez le rôle pour les utilisateurs.
8. Sélectionnez **Attribuer**.
9. Pour les utilisateurs possédant le rôle Administrateur ou Approbateur, continuez à ajouter des stratégies d’accès Azure Key Vault.

### <a name="remove-user"></a>Supprimer l’utilisateur

1. Ouvrez le portail Azure.
2. Accédez à **Azure Active Directory** > **Enterprise applications (Applications d’entreprise)** .
3. Choisissez l’inscription du microservice OPC Vault (par défaut, votre `resourceGroupName-service`).
4. Accédez à **Users and Groups** (Utilisateurs et Groupes).
5. Sélectionnez un utilisateur avec un rôle à supprimer, puis sélectionnez **Remove** (Supprimer).
6. Pour les utilisateurs retirés du rôle Administrateur ou Approbateur, retirez-les aussi des stratégies Azure Key Vault.

### <a name="add-user-access-policy-to-azure-key-vault"></a>Ajouter une stratégie d’accès utilisateur à Azure Key Vault

Des stratégies d’accès supplémentaires sont nécessaires pour les Approbateurs et Administrateurs.

Par défaut, l’identité du service dispose uniquement d’autorisations d’accès limitées à Key Vault. Cela vise à empêcher les opérations avec élévation de privilèges ou que des changements se produisent sans emprunt d’identité d’utilisateur. Les autorisations de service de base sont Get (Obtenir) et List (Lister), aussi bien pour les secrets que pour les certificats. Pour les secrets, il n’existe qu’une seule exception : le service peut supprimer une clé privée du magasin des secrets après qu’elle a été acceptée par un utilisateur. Toutes les autres opérations exigent des autorisations d’utilisateur dont l’identité a été empruntée.

#### <a name="for-an-approver-role-the-following-permissions-must-be-added-to-key-vault"></a>Pour un rôle Approbateur, les autorisations suivantes doivent être ajoutées à Key Vault

1. Ouvrez le portail Azure.
2. Accédez à votre `resourceGroupName` OPC Vault utilisé pendant le déploiement.
3. Accédez au `resourceGroupName-xxxxx` Key Vault.
4. Accédez à **Access Policies** (Stratégies d’accès).
5. Sélectionnez **Ajouter**.
6. Ignorez le modèle. Il n’existe aucun modèle conforme aux exigences.
7. Choisissez **Select Principal** (Sélectionner le principal), puis sélectionnez l’utilisateur à ajouter ou invitez un nouvel utilisateur dans le locataire.
8. Sélectionnez les autorisations de clé (**Key permissions**à suivantes : **Get** (Obtenir), **List** (Lister) et **Sign** (Signer).
9. Sélectionnez les autorisations de secret (**Secret permissions**) suivantes : **Get** (Obtenir), **List** (Lister), **Set** (Définir) et **Delete** (Supprimer).
10. Sélectionnez les autorisations de certificat (**Certificate permissions**) suivantes : **Get** (Obtenir) et **List** (Lister).
11. Sélectionnez **OK**, puis sélectionnez **Save** (Enregistrer).

#### <a name="for-an-administrator-role-the-following-permissions-must-be-added-to-key-vault"></a>Pour un rôle Administrateur, les autorisations suivantes doivent être ajoutées à Key Vault

1. Ouvrez le portail Azure.
2. Accédez à votre `resourceGroupName` OPC Vault utilisé pendant le déploiement.
3. Accédez au `resourceGroupName-xxxxx` Key Vault.
4. Accédez à **Access Policies** (Stratégies d’accès).
5. Sélectionnez **Ajouter**.
6. Ignorez le modèle. Il n’existe aucun modèle conforme aux exigences.
7. Choisissez **Select Principal** (Sélectionner le principal), puis sélectionnez l’utilisateur à ajouter ou invitez un nouvel utilisateur dans le locataire.
8. Sélectionnez les autorisations de clé (**Key permissions**à suivantes : **Get** (Obtenir), **List** (Lister) et **Sign** (Signer).
9. Sélectionnez les autorisations de secret (**Secret permissions**) suivantes : **Get** (Obtenir), **List** (Lister), **Set** (Définir) et **Delete** (Supprimer).
10. Sélectionnez les autorisations de certificat (**Certificate permissions**) suivantes : **Get** (Obtenir), **List** (Lister), **Update** (Mettre à jour), **Create** (Créer) et **Import** (Importer).
11. Sélectionnez **OK**, puis sélectionnez **Save** (Enregistrer).

### <a name="remove-user-access-policy-from-azure-key-vault"></a>Supprimer une stratégie d’accès utilisateur d’Azure Key Vault

1. Ouvrez le portail Azure.
2. Accédez à votre `resourceGroupName` OPC Vault utilisé pendant le déploiement.
3. Accédez au `resourceGroupName-xxxxx` Key Vault.
4. Accédez à **Access Policies** (Stratégies d’accès).
5. Recherchez l’utilisateur à supprimer, puis sélectionnez **Delete** (Supprimer).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment gérer les utilisateurs et les certificats OPC Vault, vous pouvez :

> [!div class="nextstepaction"]
> [Sécuriser la communication des appareils OPC](howto-opc-vault-secure.md)
