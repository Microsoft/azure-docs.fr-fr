---
title: Accès, alimentation et mode de connectivité de l'appareil Azure Data Box Gateway
description: Explique comment gérer l’accès, la puissance et le mode de connectivité de l’appareil Azure Data Box Gateway qui aide à transférer des données sur Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: 994e5a11cd3b42de339ed11a9d2517c81856e672
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580865"
---
# <a name="manage-access-power-and-connectivity-mode-for-your-azure-data-box-gateway"></a>Gérer l’accès, l’alimentation et le mode de connectivité de votre appareil Azure Data Box Gateway

Cet article explique comment gérer l’accès, la puissance et le mode de connectivité d’Azure Data Box Gateway. Ces opérations sont effectuées sur l’interface utilisateur web locale ou le Portail Azure. 

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Gérer l’accès aux appareils
> * Gérer le mode de connectivité
> * Gérer la puissance

## <a name="manage-device-access"></a>Gérer l’accès aux appareils

L’accès à votre appareil Data Box Gateway est contrôlé par l’utilisation d’un mot de passe d’appareil. Vous pouvez changer ce mot de passe à l’aide de l’interface utilisateur web locale. Vous pouvez également le réinitialiser sur le portail Azure.

### <a name="change-device-password"></a>Modifier le mot de passe d’un appareil

Suivez ces étapes sur l’interface utilisateur locale pour changer le mot de passe de l’appareil.

1. Sur l’interface utilisateur web locale, accédez à **Maintenance > Modification du mot de passe**.
2. Entrez le mot de passe actuel, puis le nouveau. Le mot de passe indiqué doit comporter entre 8 et 16 caractères, d’au moins trois types : majuscule, minuscule, chiffre et caractère spécial. Confirmez le nouveau mot de passe.

    ![Modifier le mot de passe](media/data-box-gateway-manage-access-power-connectivity-mode/change-password-1.png)

3. Cliquez sur **Modifier le mot de passe**.
 
### <a name="reset-device-password"></a>Réinitialiser le mot de passe de l’appareil

Le workflow de réinitialisation n’implique pas de se rappeler l’ancien mot de passe, ce qui est utile en cas de perte. Il s’effectue sur le Portail Azure.

1. Sur le Portail Azure, accédez à **Vue d’ensemble > Réinitialiser le mot de passe d’administrateur**.

    ![Réinitialiser le mot de passe](media/data-box-gateway-manage-access-power-connectivity-mode/reset-password-1.png)

 
2. Entrez le nouveau mot de passe, puis confirmez-le. Le mot de passe indiqué doit comporter entre 8 et 16 caractères, d’au moins trois types : majuscule, minuscule, chiffre et caractère spécial. Cliquez sur **Réinitialiser**.

    ![Réinitialiser le mot de passe 2](media/data-box-gateway-manage-access-power-connectivity-mode/reset-password-2.png)

## <a name="manage-resource-access"></a>Gérer l’accès aux ressources

Pour créer vos ressources Azure Data Box Gateway, IoT Hub et Stockage Azure, vous devez disposer d'autorisations de contributeur (ou plus élevées) au niveau du groupe de ressources. Les fournisseurs de ressources correspondants doivent également être inscrits. Pour toutes les opérations qui impliquent des informations d’identification et une clé d’activation, des autorisations sur l’API Azure Active Directory Graph sont également exigées. Elles sont décrites dans les sections suivantes.

### <a name="manage-microsoft-graph-api-permissions"></a>Gérer les autorisations pour l’API Microsoft Graph

Lorsque vous générez la clé d'activation de l'appareil ou que vous effectuez des opérations qui nécessitent des informations d'identification, vous devez disposer d'autorisations pour l'API Microsoft Graph. Les opérations nécessitant des informations d’identification peuvent être les suivantes :

-  Création d’un partage avec un compte de stockage associé.
-  Création d’un utilisateur qui peut accéder aux partages se trouvant sur l’appareil.

Vous devez disposer d'un accès `User` sur le locataire Active Directory afin de pouvoir `Read all directory objects`. Un utilisateur invité n'est pas autorisé à `Read all directory objects`. Si vous êtes un invité, les opérations telles que la génération d'une clé d'activation, la création d'un partage sur votre appareil et la création d'un utilisateur échoueront.

Pour plus d’informations sur la façon de fournir aux utilisateurs un accès à l’API Microsoft Graph, consultez [Informations de référence sur les autorisations Microsoft Graph](https://docs.microsoft.com/graph/permissions-reference).

### <a name="register-resource-providers"></a>Inscrire des fournisseurs de ressources

Pour provisionner une ressource dans Azure (dans le modèle Azure Resource Manager), vous avez besoin d’un fournisseur de ressources qui prend en charge la création de cette ressource. Par exemple, pour approvisionner une machine virtuelle, vous devez avoir un fournisseur de ressources « Microsoft.Compute » disponible dans l’abonnement.
 
Les fournisseurs de ressources sont inscrits sur le niveau de l’abonnement. Par défaut, tout nouvel abonnement Azure est préinscrit auprès d’une liste de fournisseurs de ressources couramment utilisés. Le fournisseur de ressources pour « Microsoft.DataBoxEdge » n’est pas inclus dans cette liste.

Vous n'avez pas besoin d'accorder des autorisations d'accès au niveau de l'abonnement pour que les utilisateurs puissent créer des ressources comme « Microsoft.DataBoxEdge » dans des groupes de ressources sur lesquels ils disposent de droits de propriétaire, à condition que les fournisseurs de ces ressources soient déjà inscrits.

Avant d'essayer de créer une ressource, vérifiez que le fournisseur de ressources est inscrit dans le cadre de l'abonnement. Si le fournisseur de ressources n'est pas inscrit, vous devez vérifier que l'utilisateur qui crée la ressource dispose de droits suffisants pour inscrire le fournisseur de ressources nécessaire au niveau de l'abonnement. Si vous ne l’avez pas fait également, l’erreur suivante s’affiche :

*L’abonnement \<Subscription name> ne dispose pas des autorisations nécessaires pour inscrire le fournisseur de ressources : Microsoft.DataBoxEdge.*


Pour obtenir une liste des fournisseurs de ressources inscrits dans l’abonnement actuel, exécutez la commande suivante :

```PowerShell
Get-AzResourceProvider -ListAvailable |where {$_.Registrationstate -eq "Registered"}
```

Pour un appareil Data Box Gateway, `Microsoft.DataBoxEdge` doit être inscrit. Pour inscrire `Microsoft.DataBoxEdge`, l'administrateur de l'abonnement doit exécuter la commande suivante :

```PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.DataBoxEdge
```

Pour plus d’informations sur la façon d’inscrire un fournisseur de ressources, consultez [Résoudre les erreurs d’inscription de fournisseurs de ressources](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-register-provider-errors).

## <a name="manage-connectivity-mode"></a>Gérer le mode de connectivité

Mis à part le mode normal par défaut, l'appareil peut s'exécuter en mode déconnecté ou partiellement déconnecté :

- **Partiellement déconnecté** : dans ce mode, l'appareil ne peut pas charger de données sur les partages. Il est en revanche possible de le gérer sur le portail Azure.

    Ce mode est généralement utilisé dans l'objectif de réduire la consommation de bande passante sur un réseau satellite limité. Une consommation réseau minimale peut subsister pour les opérations de monitoring des appareils.

- **Déconnecté** : dans ce mode, l'appareil est entièrement déconnecté du cloud ; les chargements et téléchargements cloud sont désactivés. L’appareil n’est gérable que sur l’interface utilisateur web locale.

    Ce mode est généralement utilisé pour mettre l’appareil hors connexion.

Pour modifier le mode de l'appareil, procédez comme suit :

1. Sur l’interface utilisateur web locale de votre appareil, accédez à **Configuration > Paramètres cloud**.
2. Désactivez **Chargement et téléchargement cloud**.
3. Pour exécuter l’appareil en mode partiellement déconnecté, activez **Gestion sur le Portail Azure**.

    ![Mode de connectivité](media/data-box-gateway-manage-access-power-connectivity-mode/connectivity-mode-1.png)
 
4. Pour exécuter l’appareil en mode déconnecté, désactivez **Gestion sur le Portail Azure**. L’appareil n’est plus gérable que sur l’interface utilisateur web locale.

    ![Mode de connectivité 2](media/data-box-gateway-manage-access-power-connectivity-mode/connectivity-mode-2.png)

## <a name="manage-power"></a>Gérer la puissance

Vous pouvez arrêter ou redémarrer votre appareil virtuel à l'aide de l'interface utilisateur web locale. Avant de redémarrer l'appareil, nous vous recommandons de mettre les partages hors connexion sur l'ordinateur hôte, puis sur l'appareil. Cette action réduit les risques d’altération des données.

1. Sur l’interface utilisateur web locale, accédez à **Maintenance > Paramètres d'alimentation**.
2. Cliquez sur **Arrêter** ou **Redémarrer** selon ce que vous voulez faire.

    ![Paramètres d’alimentation](media/data-box-gateway-manage-access-power-connectivity-mode/shut-down-restart-1.png)

3. Lorsque la boîte de dialogue de confirmation apparaît, cliquez sur **Oui** pour continuer.

> [!NOTE]
> Si vous arrêtez l’appareil virtuel, vous devrez le démarrer par le biais de la gestion de l’hyperviseur.
