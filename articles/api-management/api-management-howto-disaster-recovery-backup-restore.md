---
title: Implémenter une reprise d’activité à l’aide d’une sauvegarde et d’une restauration dans Gestion des API
titleSuffix: Azure API Management
description: Apprenez à utiliser la sauvegarde et la restauration pour effectuer une récupération d'urgence dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/05/2020
ms.author: apimpm
ms.openlocfilehash: 25356e7101293fc27d4107b3a618cfc481aee969
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779581"
---
# <a name="how-to-implement-disaster-recovery-using-service-backup-and-restore-in-azure-api-management"></a>Comment implémenter une récupération d'urgence à l'aide d'une sauvegarde de service et la récupérer dans Gestion des API Azure

En publiant et en gérant vos API via Gestion des API Azure, vous bénéficiez de fonctionnalités de tolérance de panne et d’infrastructure que vous n’avez plus à concevoir, implémenter ou gérer manuellement. La plateforme Azure permet de limiter une grande partie des risques de défaillance à moindres frais.

Pour effectuer une récupération à la suite de problèmes de disponibilité affectant la région où vous hébergez votre service Gestion des API, préparez-vous à reconstituer votre service dans une autre région à tout moment. Selon votre objectif de délai de récupération, vous souhaiterez peut-être conserver un service en attente dans une ou plusieurs régions. Vous pouvez aussi essayer de synchroniser leur configuration et leur contenu avec le service actif en fonction de l’objectif de délai de récupération. La fonctionnalité de sauvegarde et de restauration de service fournit les blocs de construction nécessaires pour implémenter votre stratégie de récupération d’urgence.

Vous pouvez également recourir à des opérations de sauvegarde et restauration pour répliquer la configuration du service Gestion des API entre les environnements d’exploitation (par exemple, développement et mise en lots). Notez que les données de runtime telles que les informations utilisateur et les abonnements seront également copiées, ce qui n’est pas toujours souhaitable.

Ce guide montre comment automatiser les opérations de sauvegarde et de restauration et garantir l’authentification des requêtes de sauvegarde et de restauration par Azure Resource Manager.

> [!IMPORTANT]
> Une opération de restauration ne modifie pas la configuration du nom d’hôte du service cible. Nous vous recommandons d’utiliser les mêmes certificats TLS et nom d’hôte personnalisés pour les services actifs et en veille, afin que le trafic puisse être redirigé vers l’instance en veille via un simple changement de CNAME DNS une fois l’opération de restauration terminée.
>
> Une opération de sauvegarde ne capture pas les données de journal préagrégées qui sont utilisées dans les rapports affichés dans le panneau Analytics du portail Microsoft Azure.

> [!WARNING]
> Chaque sauvegarde expire au bout de 30 jours. Si vous essayez de restaurer une sauvegarde après l’expiration de la période de 30 jours, la restauration échoue avec un message `Cannot restore: backup expired`.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="authenticating-azure-resource-manager-requests"></a>Demandes d'authentification Azure Resource Manager

> [!IMPORTANT]
> L'API REST pour la sauvegarde et la restauration utilise Azure Resource Manager et dispose d'un autre mécanisme d'authentification que pour les API REST pour la gestion de vos entités de gestion des API. Les étapes de cette section décrivent comment authentifier les requêtes Azure Resource Manager. Pour plus d’informations, consultez [Demandes d'authentification Azure Resource Manager](/rest/api/index).

Toutes les tâches que vous effectuez sur les ressources à l’aide d’Azure Resource Manager doivent être authentifiées avec Azure Active Directory en effectuant les étapes suivantes :

-   Ajoutez une application au locataire Azure Active Directory.
-   Définissez les autorisations pour l'application que vous avez ajoutée.
-   Obtenez le jeton d'authentification des demandes pour Azure Resource Manager.

### <a name="create-an-azure-active-directory-application"></a>Créer une application Azure Active Directory

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. En utilisant l’abonnement qui contient votre instance du service Gestion des API, accédez à l’onglet **Inscriptions des applications** dans **Azure Active Directory** (Azure Active Directory > Gérer/Inscriptions des applications).

    > [!NOTE]
    > Si le répertoire par défaut Azure Active Directory n’est pas visible dans votre compte, contactez l’administrateur de l’abonnement Azure pour accorder les autorisations nécessaires à votre compte.

3. Cliquez sur **Nouvelle inscription d’application**.

    La fenêtre **Créer** s’affiche sur la droite. C’est là que vous entrez les informations appropriées de l’application AAD.

4. Entrez un nom pour l’application.
5. Pour le type d’application, sélectionnez **Native**.
6. Entrez une URL d’espace réservé comme `http://resources` pour l’**URI de redirection**, puisqu’il s’agit d’un champ obligatoire, mais la valeur n’est pas utilisée par la suite. Cliquez sur la case à cocher pour enregistrer l'application.
7. Cliquez sur **Créer**.

### <a name="add-an-application"></a>Ajouter une application

1. Une fois l’application créée, cliquez sur **Autorisations des API**.
2. Cliquez sur **+ Ajouter une autorisation**.
4. Appuyez sur **Sélectionner des API Microsoft**.
5. Choisissez **Gestion des services Azure**.
6. Appuyez sur **Sélectionner**.

    ![Ajout d’autorisations](./media/api-management-howto-disaster-recovery-backup-restore/add-app.png)

7. Cliquez sur **Autorisations déléguées** à côté de l’application ajoutée, puis cochez la case **Accès à la gestion des services Azure (aperçu)** .
8. Appuyez sur **Sélectionner**.
9. Cliquez sur **Accorder des autorisations**.

### <a name="configuring-your-app"></a>Configuration de votre application

Avant d’appeler les API qui génèrent la sauvegarde, puis la restaure, vous devez obtenir un jeton. L’exemple suivant utilise le package NuGet [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) pour récupérer le jeton.

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System;

namespace GetTokenResourceManagerRequests
{
    class Program
    {
        static void Main(string[] args)
        {
            var authenticationContext = new AuthenticationContext("https://login.microsoftonline.com/{tenant id}");
            var result = authenticationContext.AcquireTokenAsync("https://management.azure.com/", "{application id}", new Uri("{redirect uri}"), new PlatformParameters(PromptBehavior.Auto)).Result;

            if (result == null) {
                throw new InvalidOperationException("Failed to obtain the JWT token");
            }

            Console.WriteLine(result.AccessToken);

            Console.ReadLine();
        }
    }
}
```

Remplacez `{tenant id}`, `{application id}` et `{redirect uri}` en suivant les instructions suivantes :

1. Remplacez `{tenant id}` par l’ID de locataire de l’application Azure Active Directory que vous avez créée. Vous pouvez accéder à l’ID en cliquant sur **Inscriptions d’applications** -> **Points de terminaison**.

    ![Points de terminaison][api-management-endpoint]

2. Remplacez `{application id}` par la valeur que vous avez obtenue en accédant à la page **Paramètres**.
3. Remplacez `{redirect uri}` par la valeur provenant de l’onglet **URI de redirection** de votre application Azure Active Directory.

    Une fois que les valeurs sont spécifiées, l’exemple de code doit renvoyer un jeton similaire à l’exemple suivant :

    ![par jeton][api-management-arm-token]

    > [!NOTE]
    > Le jeton peut expirer après un certain temps. Réexécutez l’exemple de code pour générer un nouveau jeton.

## <a name="calling-the-backup-and-restore-operations"></a>Appel d’opérations de sauvegarde et de restauration

Les API REST sont [Service Gestion des API - Sauvegarde](/rest/api/apimanagement/2019-12-01/apimanagementservice/backup) et [Service Gestion des API - Restauration](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore).

Avant d’appeler les opérations de « sauvegarde et de restauration » décrites dans les sections suivantes, définissez l’en-tête de demande d’autorisation de votre appel REST.

```csharp
request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + token);
```

### <a name="back-up-an-api-management-service"></a><a name="step1"> </a>Sauvegarde d’un service Gestion des API

Pour sauvegarder un service Gestion des API, envoyez la demande HTTP suivante :

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/backup?api-version={api-version}
```

où :

-   `subscriptionId` : ID de l’abonnement qui contient le service Gestion des API que vous tentez de sauvegarder
-   `resourceGroupName` : nom du groupe de ressources de votre service Gestion des API Azure
-   `serviceName` : Nom du service Gestion des API que vous sauvegardez, spécifié au moment de sa création
-   `api-version` : à remplacer par `2019-12-01`

Dans le corps de la demande, spécifiez le nom du compte de stockage Azure cible, la clé d’accès, le nom du conteneur d’objets blob et le nom de la sauvegarde :

```json
{
    "storageAccount": "{storage account name for the backup}",
    "accessKey": "{access key for the account}",
    "containerName": "{backup container name}",
    "backupName": "{backup blob name}"
}
```

Définissez la valeur de l’en-tête de la demande `Content-Type` sur `application/json`.

La sauvegarde est une opération longue qui peut prendre plusieurs minutes. Si la demande a réussi et que le processus de sauvegarde a commencé, vous recevez le code d’état de réponse `202 Accepted` avec un en-tête `Location`. Envoyez des demandes « GET » à l’URL dans l’en-tête `Location` pour connaître l’état de l’opération. Quand la sauvegarde est en cours, vous continuez à recevoir le code d’état « 202 Accepted ». Un code de réponse `200 OK` indique que l’opération de sauvegarde a réussi.

### <a name="restore-an-api-management-service"></a><a name="step2"> </a>Restauration d’un service Gestion des API

Pour restaurer un service Gestion des API à partir d’une sauvegarde créée précédemment, envoyez la demande HTTP suivante :

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/restore?api-version={api-version}
```

où :

-   `subscriptionId` : ID de l’abonnement qui contient le service Gestion des API dans lequel vous restaurez une sauvegarde
-   `resourceGroupName` : Nom du groupe de ressources qui contient le service Gestion des API Azure dans lequel vous restaurez une sauvegarde
-   `serviceName` : Nom du service Gestion des API en cours de restauration, spécifié au moment de sa création
-   `api-version` : à remplacer par `api-version=2019-12-01`

Dans le corps de la demande, spécifiez l’emplacement du fichier de sauvegarde. C’est-à-dire, ajoutez le nom du compte de stockage Azure, la clé d’accès, le nom du conteneur d’objets blob et le nom de la sauvegarde :

```json
{
    "storageAccount": "{storage account name for the backup}",
    "accessKey": "{access key for the account}",
    "containerName": "{backup container name}",
    "backupName": "{backup blob name}"
}
```

Définissez la valeur de l’en-tête de la demande `Content-Type` sur `application/json`.

La récupération est une opération de longue durée qui peut prendre jusqu'à 30 minutes, voire plus. Si la demande a réussi et que le processus de restauration a commencé, vous recevez le code d’état de réponse `202 Accepted` avec un en-tête `Location`. Envoyez des demandes « GET » à l’URL dans l’en-tête `Location` pour connaître l’état de l’opération. Quand la récupération est en cours, vous continuez à recevoir le code d’état « 202 Accepted ». Un code de réponse `200 OK` indique que l’opération de restauration a réussi.

> [!IMPORTANT]
> Le **SKU** du service à restaurer **doit correspondre** à celui du service sauvegardé utilisé pour la restauration.
>
> Les **modifications** de configuration du service (par exemple, API, stratégies, apparence du portail des développeurs) pendant qu’une opération de restauration est en cours **peuvent être écrasées**.

<!-- Dummy comment added to suppress markdown lint warning -->

> [!NOTE]
> Vous pouvez aussi effectuer les opérations de sauvegarde et de restauration avec les commandes PowerShell [_Backup-AzApiManagement_](/powershell/module/az.apimanagement/backup-azapimanagement) et [_Restore-AzApiManagement_](/powershell/module/az.apimanagement/restore-azapimanagement), respectivement.

## <a name="constraints-when-making-backup-or-restore-request"></a>Contraintes lors de la création d’une requête de sauvegarde ou de restauration

-   Le **conteneur** spécifié dans le corps de la demande **doit exister**.
-   Pendant la sauvegarde, **évitez toutes les modifications de gestion dans le service**, comme mettre à niveau une référence SKU ou la passer à une version antérieure, changer un nom de domaine, etc.
-   La restauration d’une **sauvegarde n’est garantie que pendant 30 jours** à partir du moment de sa création.
-   Les **changements** de configuration du service (par exemple, les API, les stratégies et l’apparence du portail des développeurs) pendant une opération de sauvegarde **peuvent être exclus de la sauvegarde et être perdus**.
-   Si le [pare-feu][azure-storage-ip-firewall] est activé sur le compte de stockage Azure, le client doit **autoriser** l’ensemble des [adresses IP du plan de contrôle de Gestion des API Azure][control-plane-ip-address] sur son compte de stockage à des fins de sauvegarde ou de restauration. Le compte de stockage Azure peut se trouver dans n’importe quelle région Azure, à l’exception de celle où se situe le service Gestion des API. Par exemple, si le service Gestion des API se trouve dans la région USA Ouest 2, le compte de stockage Azure peut être dans la région USA Ouest 2 et le client doit ouvrir l’adresse IP 13.64.39.16 du plan de contrôle (adresse IP du plan de contrôle de Gestion des API de la région USA Ouest 2) sur le pare-feu. Cela est dû au fait que les demandes adressées à Stockage Azure ne font pas l’objet d’une traduction d’adresse réseau source en IP publique dans Compute (Plan de contrôle de Gestion des API Azure) dans la même région Azure. La demande de stockage inter-région fera l’objet d’une traduction d’adresse réseau source en IP publique.
-   [Le partage des ressources Cross-Origin (CORS)](/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) ne doit **pas** être activé sur le service BLOB dans le compte de stockage Azure.
-   Le **SKU** du service à restaurer **doit correspondre** à celui du service sauvegardé utilisé pour la restauration.

## <a name="what-is-not-backed-up"></a>Éléments non sauvegardés
-   Les **données d’utilisation** servant à la création des rapports analytiques **ne sont pas incluses** dans la sauvegarde. Utilisez l'[API REST de Gestion des API Azure][azure api management rest api] pour récupérer régulièrement les rapports d'analyse et les conserver en toute sécurité.
-   Certificats [TLS/SSL de domaine personnalisé](configure-custom-domain.md)
-   [Certificat d’autorité de certification personnalisé](api-management-howto-ca-certificates.md) qui inclut des certificats intermédiaires ou racines chargés par le client
-   Paramètres d’intégration du [réseau virtuel](api-management-using-with-vnet.md).
-   Configuration d’[identité managée](api-management-howto-use-managed-service-identity.md).
-   Configuration des [diagnostics Azure Monitor](api-management-howto-use-azure-monitor.md).
-   Paramètres [de chiffrement et des protocoles](api-management-howto-manage-protocols-ciphers.md).
-   Contenu du [portail des développeurs](api-management-howto-developer-portal.md#is-the-portals-content-saved-with-the-backuprestore-functionality-in-api-management).

La fréquence à laquelle vous effectuez les sauvegardes du service affecte votre objectif de point de récupération. Pour la réduire, nous vous conseillons d’implémenter des sauvegardes régulières et d’effectuer des sauvegardes à la demande quand vous apportez des changements à votre service Gestion des API.

## <a name="next-steps"></a>Étapes suivantes

Consultez les ressources suivantes pour accéder à différentes procédures pas à pas relatives au processus de sauvegarde et de restauration.

-   [Répliquer des comptes de gestion des API Azure](https://www.returngis.net/en/2015/06/replicate-azure-api-management-accounts/)
-   [Automating API Management Backup and Restore with Logic Apps](https://github.com/Azure/api-management-samples/tree/master/tutorials/automating-apim-backup-restore-with-logic-apps) (Automatiser la sauvegarde et la restauration avec Gestion des API avec Logic Apps)
-   [Gestion des API Azure : sauvegarde et restauration de la configuration](/archive/blogs/stuartleeks/azure-api-management-backing-up-and-restoring-configuration)
    _L’approche détaillée par Stuart ne correspond pas à l’aide officielle mais est intéressante._

[backup an api management service]: #step1
[restore an api management service]: #step2
[azure api management rest api]: /rest/api/apimanagement/apimanagementrest/api-management-rest
[api-management-add-aad-application]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-add-aad-application.png
[api-management-aad-permissions]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-permissions.png
[api-management-aad-permissions-add]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-permissions-add.png
[api-management-aad-delegated-permissions]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-delegated-permissions.png
[api-management-aad-default-directory]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-default-directory.png
[api-management-aad-resources]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-aad-resources.png
[api-management-arm-token]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-arm-token.png
[api-management-endpoint]: ./media/api-management-howto-disaster-recovery-backup-restore/api-management-endpoint.png
[control-plane-ip-address]: api-management-using-with-vnet.md#control-plane-ips
[azure-storage-ip-firewall]: ../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range
