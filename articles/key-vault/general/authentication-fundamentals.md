---
title: Concepts de base de l’authentification Azure Key Vault
description: En savoir plus sur le fonctionnement du modèle d’authentification du coffre de clés
author: ShaneBala-keyvault
ms.author: sudbalas
ms.date: 09/25/2020
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.openlocfilehash: 6de0fc52ae265a47ca7f52d46e5f44b74c1277aa
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530916"
---
# <a name="key-vault-authentication-fundamentals"></a>Concepts de base de l’authentification Key Vault

Azure Key Vault vous permet de stocker et de gérer en toute sécurité les informations d’identification de l’application, telles que les secrets, les clés et les certificats, dans un référentiel cloud central et sécurisé. Grâce à Key Vault, il n’est plus nécessaire de stocker les informations d’identification dans vos applications. Vos applications peuvent s’authentifier auprès de Key Vault au moment de l’exécution pour récupérer les informations d’identification.

En tant qu’administrateur, vous pouvez contrôler étroitement les utilisateurs et les applications qui peuvent accéder à votre coffre de clés, et vous pouvez restreindre et auditer les opérations qu’ils effectuent. Ce document explique les concepts fondamentaux du modèle d’accès au coffre de clés. Il vous permettra d’acquérir un premier niveau de connaissances et vous montrera du début à la fin de quelle manière vous pouvez authentifier un utilisateur ou une application auprès du coffre de clés.

## <a name="required-knowledge"></a>Connaissances requises

Ce document suppose que vous connaissez les concepts suivants. Si vous n’êtes pas familiarisé avec ces concepts, suivez les liens d’aide avant de continuer.

* [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md)
* [Principaux de sécurité](./authentication.md#app-identity-and-security-principals)

## <a name="key-vault-configuration-steps-summary"></a>Résumé des étapes de configuration de Key Vault

1. Inscrivez votre utilisateur ou votre application dans Azure Active Directory en tant que principal de sécurité.
1. Configurez une attribution de rôle pour votre principal de sécurité dans Azure Active Directory.
1. Configurez les stratégies d’accès au coffre de clés pour votre principal de sécurité.
1. Configurez l’accès du pare-feu Key Vault à votre coffre de clés (facultatif).
1. Testez la capacité de votre principal de sécurité à accéder au coffre de clés.

## <a name="register-a-user-or-application-in-azure-active-directory-as-a-security-principal"></a>Inscrire un utilisateur ou une application dans Azure Active Directory en tant que principal de sécurité

Lorsqu’un utilisateur ou une application effectue une requête auprès du coffre de clés, la requête doit d’abord être authentifiée par Azure Active Directory. Pour que cela fonctionne, l’utilisateur ou l’application doit être inscrit dans Azure Active Directory en tant que principal de sécurité.

Suivez les liens de la documentation ci-dessous pour comprendre comment inscrire un utilisateur ou une application dans Azure Active Directory.
**Veillez à créer un mot de passe pour l’inscription de l’utilisateur et une clé secrète client ou un certificat client pour les applications.**

* [Inscription d’un utilisateur dans Azure Active Directory](../../active-directory/fundamentals/add-users-azure-active-directory.md)
* [Inscription d’une application dans Azure Active Directory](../../active-directory/develop/quickstart-register-app.md)

## <a name="assign-your-security-principal-a-role"></a>Attribuer un rôle à votre principal de sécurité

Vous pouvez utiliser le contrôle d’accès Azure en fonction du rôle (Azure RBAC) pour accorder des autorisations à des principaux de sécurité. Ces autorisations sont appelées attributions de rôles.

Dans le contexte du coffre de clés, ces attributions de rôles déterminent le niveau d’accès d’un principal de sécurité au plan de gestion (également appelé « plan de contrôle ») du coffre de clés. Ces attributions de rôles ne permettent pas d’accéder directement aux secrets du plan de données, mais elles permettent de gérer les propriétés du coffre de clés. Par exemple, un utilisateur ou une application affectés à un **rôle Lecteur** ne sera pas autorisé à apporter des modifications aux paramètres du pare-feu du coffre de clés, alors qu’un utilisateur ou une application affectés à un **rôle Contributeur** peut apporter des modifications. Aucun des deux rôles n’aura un accès direct pour effectuer des opérations sur les secrets, les clés et les certificats, comme la création ou la récupération de leur valeur, tant qu’un accès au plan de données du coffre de clés ne leur sera pas attribué. Cette question est traitée dans l’étape suivante.

>[!IMPORTANT]
> Bien que les utilisateurs ayant le rôle Contributeur ou Propriétaire ne puissent pas effectuer par défaut d’opérations sur les secrets stockés dans le coffre de clés, les rôles Contributeur et Propriétaire disposent des autorisations nécessaires pour ajouter des stratégies d’accès aux secrets stockés dans le coffre de clés ou en supprimer. Par conséquent, un utilisateur disposant de ces attributions de rôles peut s’autoriser lui-même à accéder aux secrets du coffre de clés. Pour cette raison, il est recommandé que seuls les administrateurs aient accès aux rôles Contributeur ou Propriétaire. Les utilisateurs et les applications qui n’ont besoin que de récupérer des secrets dans le coffre de clés doivent se voir attribuer le rôle Lecteur. **Pour plus d’informations, consultez la section suivante.**

>[!NOTE]
> Lorsque vous attribuez un rôle à un utilisateur au niveau du locataire Azure Active Directory, cet ensemble d’autorisations s’étend à tous les abonnements, groupes de ressources et ressources au sein de l’étendue de l’attribution. Pour suivre le principe du moindre privilège, vous pouvez effectuer cette attribution de rôle dans une étendue plus granulaire. Par exemple, vous pouvez attribuer à un utilisateur un rôle Lecteur au niveau de l’abonnement et un rôle Propriétaire pour un coffre de clés unique. Accédez aux paramètres de gestion des identités et des accès (IAM) d’un abonnement, d’un groupe de ressources ou d’un coffre de clés pour effectuer une attribution de rôle dans une étendue plus granulaire.

* Pour en savoir plus sur les rôles Azure [lien](../../role-based-access-control/built-in-roles.md)
* [En savoir plus sur l’attribution ou la suppression des rôles](../../role-based-access-control/role-assignments-portal.md)

## <a name="configure-key-vault-access-policies-for-your-security-principal"></a>Configurer les stratégies d’accès au coffre de clés pour votre principal de sécurité

Avant d’accorder à vos utilisateurs et applications l’accès au coffre de clés, il est important de comprendre les différents types d’opérations qui peuvent être effectuées sur un coffre de clés. Il existe deux principaux types d’opérations sur les coffres de clés, les opérations de plan de gestion (également appelé « plan de contrôle ») et les opérations de plan de données.

Ce tableau présente plusieurs exemples des différentes opérations contrôlées par le plan de gestion par rapport au plan de données. Les opérations qui modifient les propriétés du coffre de clés sont des opérations de plan de gestion. Les opérations qui modifient ou récupèrent la valeur des secrets stockés dans le coffre de clés sont des opérations de plan de données.

|Opérations de plan de gestion (exemples)|Opérations de plan de données (exemples)|
| --- | --- |
| Création d'un coffre de clés | Créer une clé, un secret, un certificat
| Supprimer un coffre de clés | Supprimer une clé, un secret, un certificat
| Ajouter ou supprimer des attributions de rôles pour le coffre de clés | Répertorier et récupérer les valeurs des clés, des secrets et des certificats
| Ajouter ou supprimer des stratégies d’accès au coffre de clés | Sauvegarder et restaurer des clés, des secrets et des certificats
| Modifier les paramètres du pare-feu Key Vault | Renouveler les clés, les secrets et les certificats
| Modifier les paramètres de récupération du coffre de clés | Vider ou récupérer des clés, des secrets et des certificats supprimés de manière réversible
| Modifier les paramètres des journaux de diagnostic du coffre de clés

### <a name="management-plane-access--azure-active-directory-role-assignments"></a>Accès au plan de gestion et attributions de rôles Azure Active Directory

Les attributions de rôles Azure Active Directory permettent d’effectuer des opérations de plan de gestion sur un coffre de clés. Cet accès est généralement accordé aux utilisateurs, et non aux applications. Vous pouvez restreindre les opérations de plan de gestion qu’un utilisateur peut effectuer en modifiant l’attribution de rôle d’un utilisateur.

Par exemple, l’attribution du rôle Lecteur de coffre de clés à un utilisateur lui permet de voir les propriétés de votre coffre de clés, telles que les stratégies d’accès, mais ne lui permet pas d’y apporter des modifications. L’attribution du rôle Propriétaire à un utilisateur lui accorde un accès complet aux paramètres du plan de gestion du coffre de clés et lui permet de les modifier.

Les attributions de rôles sont contrôlées dans le panneau Contrôle d’accès (IAM) du coffre de clés. Si vous souhaitez qu’un utilisateur particulier puisse être lecteur ou administrateur de plusieurs ressources de coffre de clés, vous pouvez créer une attribution de rôle au niveau du coffre, du groupe de ressources ou de l’abonnement, et l’attribution de rôle sera ajoutée à toutes les ressources dans l’étendue de l’attribution.

L’accès au plan de données, ou l’accès aux opérations sur les clés, les secrets et les certificats stockés dans le coffre de clés, peut être ajouté de deux manières.

### <a name="data-plane-access-option-1-classic-key-vault-access-policies"></a>Option no 1 d’accès au plan de données : stratégies classiques d’accès au coffre de clés

Les stratégies d’accès au coffre de clés permettent aux utilisateurs et aux applications d’effectuer des opérations de plan de données sur un coffre de clés.

> [!NOTE]
> Ce modèle d’accès n’est pas compatible avec le contrôle d’accès Azure en fonction du rôle (RBAC) pour Key Vault (option 2) documenté ci-dessous. Vous devez choisir l’un ou l’autre. Vous aurez la possibilité d’effectuer cette sélection lorsque vous cliquerez sur l’onglet Stratégie d’accès de votre coffre de clés.

Les stratégies d’accès classiques sont granulaires, ce qui signifie que vous pouvez autoriser chaque utilisateur ou application à effectuer des opérations individuelles au sein d’un coffre de clés ou les en empêcher. Voici quelques exemples :

* Le principal de sécurité 1 peut effectuer n’importe quelle opération sur les clés, mais n’est pas autorisé à effectuer des opérations sur les secrets ou les certificats.
* Le principal de sécurité 2 peut répertorier et lire toutes les clés et tous les secrets et certificats, mais ne peut effectuer aucune opération de création, de suppression ou de renouvellement.
* Le principal de sécurité 3 peut sauvegarder et restaurer tous les secrets, mais ne peut pas lire la valeur des secrets eux-mêmes.

Toutefois, les stratégies d’accès classiques n’accordent pas d’autorisations au niveau de chaque objet, et les autorisations attribuées sont appliquées à l’étendue d’un coffre de clés individuel. Par exemple, si vous accordez l’autorisation de stratégie d’accès « Récupérer un secret » à un principal de sécurité dans un coffre de clés particulier, le principal de sécurité a la possibilité de récupérer tous les secrets dans ce coffre de clés particulier. Toutefois, cette autorisation « Récupérer un secret » ne s’étendra pas automatiquement à d’autres coffres de clés et doit être attribuée explicitement.

> [!IMPORTANT]
> Les stratégies classiques d’accès au coffre de clés et les attributions de rôles Azure Active Directory sont indépendantes les unes des autres. L’attribution du rôle « Contributeur » à un principal de sécurité au niveau d’un abonnement ne lui permet pas automatiquement d’effectuer des opérations de plan de données sur chaque coffre de clés dans l’étendue de l’abonnement. Le principal de sécurité doit toujours recevoir, ou s’octroyer, des autorisations de stratégie d’accès pour effectuer des opérations de plan de données.

### <a name="data-plane-access-option-2--azure-rbac-for-key-vault-preview"></a>Option no 2 d’accès au plan de données :  Contrôle d’accès Azure en fonction du rôle (RBAC) pour Key Vault (préversion)

Une nouvelle façon d’accorder l’accès au plan de données du coffre de clés est d’utiliser le contrôle d’accès Azure en fonction du rôle (Azure RBAC) pour le coffre de clés.

> [!NOTE]
> Ce modèle d’accès n’est pas compatible avec les stratégies classiques d’accès au coffre de clés illustrées ci-dessus. Vous devez choisir l’un ou l’autre. Vous aurez la possibilité d’effectuer cette sélection lorsque vous cliquerez sur l’onglet Stratégie d’accès de votre coffre de clés.

Les attributions de rôles Azure Key Vault sont un ensemble de rôles intégrés Azure qui englobent des ensembles communs d’autorisations utilisés pour accéder aux clés, aux secrets ou aux certificats. Ce modèle d’autorisation active également des capacités supplémentaires qui ne sont pas disponibles dans le modèle des stratégies classiques d’accès au coffre de clés.

* Les autorisations Azure RBAC peuvent être gérées à grande échelle en permettant aux utilisateurs de se voir attribuer ces rôles au niveau d’un abonnement, d’un groupe de ressources ou d’un coffre de clés. Un utilisateur disposera des autorisations de plan de données pour tous les coffres de clés dans l’étendue de l’attribution Azure RBAC. Il n’est donc plus nécessaire d’attribuer des autorisations individuelles de stratégie d’accès par utilisateur/application par coffre de clés.

* Les autorisations Azure RBAC sont compatibles avec Privileged Identity Management (PIM). Cela vous permet de configurer des contrôles d’accès juste-à-temps pour les rôles privilégiés, comme Administrateur de coffre de clés. Il s’agit d’une pratique de sécurité optimale qui suit le principe du moindre privilège en éliminant l’accès permanent à vos coffres de clés.

Pour en savoir plus sur Azure RBAC pour Key Vault, consultez les documents suivants :

* Azure RBAC pour Key Vault [lien](./secure-your-key-vault.md#management-plane-and-azure-rbac)
* Rôles Azure RBAC pour Key Vault (préversion) [lien](../../role-based-access-control/built-in-roles.md#key-vault-administrator-preview)

## <a name="configure-key-vault-firewall"></a>Configurer le pare-feu Key Vault

Par défaut, le coffre de clés permet au trafic en provenance de l’Internet public d’atteindre votre coffre de clés par le biais d’un point de terminaison public. Pour une couche de sécurité supplémentaire, vous pouvez configurer le pare-feu Azure Key Vault pour limiter l’accès au point de terminaison public du coffre de clés.

Pour activer le pare-feu du coffre de clés, cliquez sur l’onglet Mise en réseau dans le portail du coffre de clés, puis sélectionnez la case d’option Autoriser l’accès depuis : « Point de terminaison privé et réseaux sélectionnés ». Si vous choisissez d’activer le pare-feu de coffre de clés, vous pouvez autoriser le trafic via le pare-feu de coffre de clés des façons suivantes :

* Ajoutez des adresses IPv4 à la liste d’autorisation du pare-feu de coffre de clés. Cette option fonctionne mieux pour les applications qui ont des adresses IP statiques.

* Ajoutez un réseau virtuel au pare-feu de coffre de clés. Cette option fonctionne mieux pour les ressources Azure qui ont des adresses IP dynamiques comme les machines virtuelles. Vous pouvez ajouter des ressources Azure à un réseau virtuel, puis ajouter le réseau virtuel à la liste d’autorisation du pare-feu de coffre de clés. Cette option utilise un point de terminaison de service qui est une adresse IP privée au sein du réseau virtuel. Cela constitue une couche de protection supplémentaire, de sorte qu’aucun trafic entre le coffre de clés et votre réseau virtuel n’est acheminé via l’Internet public. Pour en savoir plus sur le point de terminaison de service, consultez la documentation accessible [ici](./network-security.md).

* Ajoutez une connexion de liaison privée au coffre de clés. Cette option connecte votre réseau virtuel directement à une instance particulière du coffre de clés, ce qui permet d’intégrer efficacement votre coffre de clés à votre réseau virtuel. Pour en savoir plus sur la configuration d’une connexion de point de terminaison privé au coffre de clés, suivez [ce lien](./private-link-service.md).

## <a name="test-your-service-principals-ability-to-access-key-vault"></a>Tester la capacité de votre principal de service à accéder au coffre de clés

Une fois que vous avez suivi toutes les étapes ci-dessus, vous pourrez définir et récupérer des secrets à partir de votre coffre de clés.

### <a name="authentication-process-for-users-examples"></a>Processus d’authentification des utilisateurs (exemples)

* Les utilisateurs peuvent se connecter au portail Azure pour utiliser le coffre de clés. [Démarrage rapide Key Vault via le portail](./quick-create-portal.md)

* L’utilisateur peut utiliser Azure CLI pour utiliser le coffre de clés. [Démarrage rapide Key Vault via Azure CLI](./quick-create-cli.md)

* L’utilisateur peut utiliser Azure PowerShell pour utiliser le coffre de clés. [Démarrage rapide Key Vault via Azure PowerShell](./quick-create-powershell.md)

### <a name="azure-active-directory-authentication-process-for-applications-or-services-examples"></a>Processus d’authentification Azure Active Directory des applications ou des services (exemples)

* Une application fournit une clé secrète client et un ID client dans une fonction afin d’obtenir un jeton Azure Active Directory. 

* Une application fournit un certificat afin d’obtenir un jeton Azure Active Directory. 

* Une ressource Azure utilise l’authentification MSI pour recevoir un jeton Azure Active Directory. 

* En savoir plus sur l’[authentification MSI](../../active-directory/managed-identities-azure-resources/overview.md).

### <a name="authentication-process-for-application-python-example"></a>Processus d’authentification des applications (exemple Python)

Utilisez l’exemple de code suivant pour tester si votre application peut récupérer un secret à partir de votre coffre de clés en utilisant le principal de service que vous avez configuré.

>[!NOTE]
>Cet exemple est fourni à des fins de démonstration et de test uniquement. **N’UTILISEZ PAS L’AUTHENTIFICATION PAR CLÉ SECRÈTE CLIENT EN PRODUCTION.** Il ne s’agit pas d’une pratique de conception sécurisée. Vous devez utiliser un certificat client ou l’authentification MSI comme meilleure pratique.

```python
from azure.identity import ClientSecretCredential
from azure.keyvault.secrets import SecretClient

tenant_id = "{ENTER YOUR TENANT ID HERE}"                          ##ENTER AZURE TENANT ID
vault_url = "https://{ENTER YOUR VAULT NAME}.vault.azure.net/"     ##ENTER THE URL OF YOUR KEY VAULT
client_id = "{ENTER YOUR CLIENT ID HERE}"                          ##ENTER THE CLIENT ID OF YOUR SERVICE PRINCIPAL
cert_path = "{ENTER YOUR CLIENT SECRET HERE}"                      ##ENTER THE CLIENT SECRET OF YOUR SERVICE PRINCIPAL

def main():

    #AUTHENTICATION TO Azure Active Directory USING CLIENT ID AND CLIENT CERTIFICATE (GET Azure Active Directory TOKEN)
    token = ClientSecretCredential(tenant_id=tenant_id, client_id=client_id, client_secret=client_secret)

    #AUTHENTICATION TO KEY VAULT PRESENTING Azure Active Directory TOKEN
    client = SecretClient(vault_url=vault_url, credential=token)

    #CALL TO KEY VAULT TO GET SECRET
    #ENTER NAME OF A SECRET STORED IN KEY VAULT
    secret = client.get_secret('{SECRET_NAME}')

    #GET PLAINTEXT OF SECRET
    print(secret.value)

#CALL MAIN()
if __name__ == "__main__":
    main()
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’authentification du coffre de clés, consultez le document suivant : [Authentification du coffre de clés](./authentication.md)
