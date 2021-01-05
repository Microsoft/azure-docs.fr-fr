---
title: Développer un pool d’hôtes Windows Virtual Desktop (classique) existant avec de nouveaux hôtes de session - Azure
description: Comment développer d’un pool d’hôtes existant avec de nouveaux hôtes de session dans Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 61cf28b0f1ebee6a0312ec3f23f22b01c6c4919e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88009169"
---
# <a name="expand-an-existing-host-pool-with-new-session-hosts-in-windows-virtual-desktop-classic"></a>Développer d’un pool d’hôtes existant avec de nouveaux hôtes de session dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../expand-existing-host-pool.md).

À mesure que vous intensifiez l’utilisation au sein de votre pool d’hôtes, il se peut que vous deviez développer votre pool d’hôtes existant avec de nouveaux hôtes de session pour gérer la nouvelle charge.

Cet article explique comment développer un pool d’hôtes existant avec de nouveaux hôtes de session.

## <a name="what-you-need-to-expand-the-host-pool"></a>Ce dont vous avez besoin pour développer le pool d’hôtes

Avant de commencer, assurez-vous que vous avez créé un pool d’hôtes et des machines virtuelles hôtes de session à l’aide de l’une des méthodes suivantes :

- [Offres de la Place de marché Microsoft Azure](create-host-pools-azure-marketplace-2019.md)
- [Modèle Azure Resource Manager du GitHub](create-host-pools-arm-template.md)
- [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell-2019.md)

Vous aurez également besoin des informations suivantes recueillies lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session :

- Taille de machine virtuelle, image et préfixe de nom
- Jonction de domaine et informations d’identification d’administrateur client de Windows Virtual Desktop
- Nom du réseau virtuel et nom du sous-réseau

Les trois sections suivantes sont trois méthodes que vous pouvez utiliser pour développer le pool d’hôtes. Vous pouvez vous servir de l’outil de déploiement avec lequel êtes les plus à l’aise.

>[!NOTE]
>Pendant la phase de déploiement, des messages d’erreur s’affichent pour les ressources de machine virtuelle hôte de session précédentes si elles sont arrêtées. Ces erreurs se produisent parce qu’Azure ne peut pas exécuter l’extension de DSC PowerShell pour vérifier que les machines virtuelles hôtes de session sont correctement inscrites auprès de votre pool d’hôtes existant. Vous pouvez ignorer ces erreurs en toute sécurité, ou éviter les erreurs en démarrant toutes les machines virtuelles hôtes de session dans le pool d’hôtes existant avant de commencer le processus de déploiement.

## <a name="redeploy-from-azure"></a>Redéployer à partir d’Azure

Si vous avez déjà créé un pool d’hôtes et des machines virtuelles hôtes de session à l’aide de l’[offre de la Place de marché Microsoft Azure](create-host-pools-azure-marketplace-2019.md) ou du [modèle Azure Resource Manager du GitHub](create-host-pools-arm-template.md), vous pouvez redéployer le même modèle à partir du portail Azure. Le redéploiement du modèle réentre automatiquement toutes les informations que vous avez entrées dans le modèle d’origine, à l’exception des mots de passe.

Voici comment redéployer le modèle Azure Resource Manager pour développer un pool d’hôtes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la barre de recherche en haut du portail Azure, recherchez **Groupes de ressources** et sélectionnez l’élément sous **Services**.
3. Recherchez et sélectionnez le groupe de ressources que vous avez créé lorsque vous avez constitué le pool d’hôtes.
4. Dans le volet gauche du navigateur, sélectionnez **Déploiements**.
5. Sélectionnez le déploiement approprié pour le processus de création de votre pool d’hôtes :
     - Si vous avez créé le pool d’hôtes d’origine avec l’offre de la Place de marché Microsoft Azure, sélectionnez le déploiement commençant par **rds.wvd-provision-host-pool**.
     - Si vous avez créé le pool d’hôtes d’origine avec le modèle Azure Resource Manager du GitHub, sélectionnez le déploiement nommé **Microsoft.Template**.
6. Sélectionnez **Redéployer**.

     >[!NOTE]
     >Si le modèle ne se redéploie pas automatiquement lorsque vous sélectionnez **Redéployer**, sélectionnez **Modèle** dans le volet de gauche de votre navigateur, puis choisissez **Déployer**.

7. Sélectionnez le groupe de ressources contenant les machines virtuelles hôtes de session actuelles dans le pool d’hôtes existant.

     >[!NOTE]
     >Si vous voyez une erreur qui vous indique de sélectionner un autre groupe de ressources, même si celui que vous avez entré est correct, sélectionnez un autre groupe de ressources, puis le groupe de ressources d’origine.

8. Entrez l’URL suivante pour *_artifactsLocation* : `https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/`
9. Entrez le nouveau nombre total d’hôtes de session souhaité dans *Rdsh Number Of Instances*. Par exemple, si vous développez votre pool d’hôtes de cinq hôtes de session à huit, entrez **8**.
10. Entrez le mot de passe de domaine existant que vous avez utilisé pour l’UPN du domaine existant. Ne modifiez pas le nom d’utilisateur, car cela provoquera une erreur lors de l’exécution du modèle.
11. Entrez le mot de passe d’administrateur de locataire utilisé pour l’ID d’utilisateur ou d’application que vous avez entré pour *Tenant Admin Upn Or Application Id*. Une fois encore, ne modifiez pas le nom d’utilisateur.
12. Effectuez l’envoi pour développer votre pool d’hôtes.

## <a name="run-the-azure-marketplace-offering"></a>Exécuter l’offre de la Place de marché Microsoft Azure

Suivez les instructions de la section [Créer un pool d’hôtes en utilisant la Place de marché Azure](create-host-pools-azure-marketplace-2019.md) jusqu’à atteindre la section [Exécuter l’offre de la Place de marché Azure pour provisionner un nouveau pool d’hôtes](create-host-pools-azure-marketplace-2019.md#run-the-azure-marketplace-offering-to-provision-a-new-host-pool). Quand vous parvenez à ce stade, vous devez entrer les informations suivantes pour chaque onglet :

### <a name="basics"></a>Concepts de base

Toutes les valeurs dans cette section doivent correspondre à celles que vous avez fournies lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session, à l’exception des *utilisateurs de bureau par défaut* :

1.    Pour *Abonnement*, sélectionnez l’abonnement dans lequel vous avez créé initialement le pool d’hôtes.
2.    Pour *Groupe de ressources*, sélectionnez le groupe de ressources dans lequel se trouvent les machines virtuelles hôtes de session du pool d’hôtes existant.
3.    Pour *Région*, sélectionnez la région dans laquelle se trouvent les machines virtuelles hôtes de session du pool d’hôtes existant.
4.    Pour *Nom du pool d’hôtes*, entrez le nom du pool d’hôtes existant.
5.    Pour *Type de bureau*, sélectionnez le type de bureau correspondant au pool d’hôtes existant.
6.    Pour *Utilisateurs de bureau par défaut*, entrez une liste séparée par des virgules des utilisateurs supplémentaires dont vous voulez qu’il se connectent aux clients Windows Virtual Desktop et accèdent à un ordinateur de bureau une fois l’offre de la Place de marché Microsoft Azure terminée. Par exemple, si vous voulez accorder l’accès à user3@contoso.com et user4@contoso.com, entrez user3@contoso.com,user4@contoso.com.
7.    Sélectionnez **Suivant : Configurer la machine virtuelle**.

>[!NOTE]
>À l’exception des *Utilisateurs de bureau par défaut*, tous les champs doivent correspondre exactement à ce qui a été configuré dans le pool d’hôtes existant. S’il existe une discordance qui entraîne un nouveau pool d’hôtes.

### <a name="configure-virtual-machines"></a>Configurer des machines virtuelles

Toutes les valeurs de paramètre dans cette section doivent correspondre à celles que vous avez fournies lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session, à l’exception du nombre total de machines virtuelles. Le nombre de machines virtuelles que vous entrez est le nombre de machines virtuelles dans votre pool d’hôtes développé :

1. Sélectionnez la taille de machine virtuelle correspondant aux machines virtuelles hôtes de session existantes.

    >[!NOTE]
    >Si la taille de machine virtuelle que vous recherchez n’apparaît pas dans le sélecteur de taille de machine virtuelle, c’est parce que nous ne l’avons pas encore intégrée à l’outil Place de marché Azure. Pour demander une taille de machine virtuelle, créez une demande ou votez pour une demande existante dans le [forum UserVoice sur Windows Virtual Desktop](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).

2. Personnalisez les paramètres *Profil d’utilisation*, *Nombre total d’utilisateurs* et *Nombre de machines virtuelles* pour sélectionner le nombre total d’hôtes de session que vous souhaitez avoir dans votre pool d’hôtes. Par exemple, si vous développez votre pool d’hôtes en passant de cinq hôtes de session à huit, configurez ces options pour atteindre 8 machines virtuelles.
3. Entrez un préfixe pour le nom des machines virtuelles. Par exemple, si vous entrez le nom « préfixe », les machines virtuelles s’appelleront « préfixe-0 », « préfixe-1 », etc.
4. Sélectionnez **Suivant : Paramètres de la machine virtuelle**.

### <a name="virtual-machine-settings"></a>Paramètres de la machine virtuelle

Toutes les valeurs de paramètre dans cette section doivent correspondre à celles que vous avez fournies lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session :

1. Pour les paramètres *Source d’image* et *Version du système d’exploitation de l’image*, entrez les informations que vous avez fournies lors de la création initiale du pool d’hôtes.
2. Pour *UPN de jonction de domaine Active Directory* et les mots de passe associés, entrez les informations que vous avez fournies lors de la création initiale du pool d’hôtes pour joindre les machines virtuelles au domaine Active Directory. Ces informations d’identification seront utilisées pour créer un compte local sur vos machines virtuelles. Vous pouvez réinitialiser ces comptes locaux pour modifier leurs informations d’identification ultérieurement.
3. Pour les informations sur le réseau virtuel, sélectionnez les mêmes réseau virtuel et sous-réseau pour l’emplacement où se trouvent vos machines virtuelles hôtes de session de pool d’hôtes.
4. Sélectionnez **Suivant : configurer les Informations sur Windows Virtual Desktop**.

### <a name="windows-virtual-desktop-information"></a>Informations sur Windows Virtual Desktop

Toutes les valeurs de paramètre dans cette section doivent correspondre à celles que vous avez fournies lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session :

1. Dans *Nom du groupe de locataires Windows Virtual Desktop*, entrez le nom du groupe de locataires qui contient votre locataire. Conservez la valeur par défaut, sauf si un nom de groupe de locataires spécifique vous a été fourni.
2. Dans *Nom du locataire Windows Virtual Desktop*, entrez le nom du locataire que vous allez créer dans ce pool d’hôtes.
3. Spécifiez les informations d’identification que vous avez utilisées lors de la création initiale du pool d’hôtes et des machines virtuelles hôtes de session. Si vous utilisez un principal de service, entrez l’ID de l’instance Azure Active Directory dans laquelle celui-ci se trouve.
4. Sélectionnez **Suivant : Vérifier + créer**.

## <a name="run-the-github-azure-resource-manager-template"></a>Exécuter le modèle Azure Resource Manager du GitHub

Suivez les instructions de la section [Exécuter le modèle Azure Resource Manager pour l’approvisionnement du nouveau pool d’hôtes](create-host-pools-arm-template.md#run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool) et fournissez les mêmes valeurs de paramètres, à l’exception du paramètre *Rdsh Number Of Instances*. Entrez le nombre de machines virtuelles hôtes de session que vous souhaitez dans le pool d’hôtes après l’exécution du modèle. Par exemple, si vous développez votre pool d’hôtes de cinq hôtes de session à huit, entrez **8**.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez développé votre pool d’hôtes existant, vous pouvez vous connecter à un client Windows Virtual Desktop pour les tester dans le cadre d’une session utilisateur. Vous pouvez vous connecter à une session avec l’un des clients suivants :

- [Se connecter avec le client Windows Desktop](connect-windows-7-10-2019.md)
- [Se connecter avec le client web](connect-web-2019.md)
- [Se connecter avec le client Android](connect-android-2019.md)
- [Se connecter avec le client macOS](connect-macos-2019.md)
- [Se connecter avec le client iOS](connect-ios-2019.md)
