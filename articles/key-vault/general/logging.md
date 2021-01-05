---
title: Journalisation d’Azure Key Vault | Microsoft Docs
description: Découvrez comment superviser l’accès à vos coffres de clés en activant la journalisation pour Azure Key Vault, ce qui enregistre les informations dans un compte de stockage Azure que vous fournissez.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/12/2019
ms.author: mbaldwin
ms.openlocfilehash: eef4f6b8ee5821e54b5b7709eee7f8dad8749e63
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94488534"
---
# <a name="azure-key-vault-logging"></a>Journalisation d’Azure Key Vault

Une fois que vous avez créé un ou plusieurs coffres de clés, vous voulez probablement contrôler qui accède à ces derniers, par quel moyen et quand. Pour ce faire, vous pouvez activer la journalisation d’Azure Key Vault, ce qui enregistre les informations dans un compte de stockage Azure que vous fournissez. Pour obtenir des instructions pas à pas sur la configuration de cette option, consultez [Guide pratique pour activer la journalisation de Key Vault](howto-logging.md).

Vous pouvez accéder aux informations de journalisation 10 minutes (au maximum) après l’opération sur le coffre de clés. Dans la plupart des cas, ce sera plus rapide.  C’est à vous de gérer vos journaux d’activité dans votre compte de stockage :

* Utilisez les méthodes de contrôle d’accès Azure standard pour assurer la sécurité de vos journaux d’activité en limitant l’accès à ces derniers.
* Supprimez les journaux d’activité que vous ne souhaitez plus conserver dans votre compte de stockage.

Pour obtenir des informations générales sur Key Vault, consultez l’article [Qu’est-ce qu’Azure Key Vault ?](overview.md). Pour plus d’informations sur les endroits où Key Vault est disponible, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/key-vault/). Découvrez plus d’informations sur l’utilisation d’[Azure Monitor pour Key Vault](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="interpret-your-key-vault-logs"></a>Interpréter vos journaux d’activité Key Vault

Quand vous activez la journalisation, un nouveau conteneur appelé **insights-logs-auditevent** est automatiquement créé pour le compte de stockage que vous avez spécifié. Vous pouvez utiliser ce même compte de stockage pour collecter les journaux de plusieurs coffres de clés.

Les objets blob individuels sont stockés sous forme de texte en tant qu’objet blob JSON. Examinons un exemple d’entrée du journal. 

```json
    {
        "records":
        [
            {
                "time": "2016-01-05T01:32:01.2691226Z",
                "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
                "operationName": "VaultGet",
                "operationVersion": "2015-06-01",
                "category": "AuditEvent",
                "resultType": "Success",
                "resultSignature": "OK",
                "resultDescription": "",
                "durationMs": "78",
                "callerIpAddress": "104.40.82.76",
                "correlationId": "",
                "identity": {"claim":{"http://schemas.microsoft.com/identity/claims/objectidentifier":"d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn":"live.com#username@outlook.com","appid":"1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},
                "properties": {"clientInfo":"azure-resource-manager/2.0","requestUri":"https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id":"https://contosokeyvault.vault.azure.net/","httpStatusCode":200}
            }
        ]
    }
```

Le tableau ci-après répertorie les noms de champ et leurs descriptions :

| Nom du champ | Description |
| --- | --- |
| **time** |Date et heure en temps universel coordonné (UTC). |
| **resourceId** |ID de ressource Azure Resource Manager. Pour les journaux d’activité de coffre de clés, il s’agit toujours de l’ID de ressource du coffre de clés. |
| **operationName** |Nom de l’opération, comme indiqué dans le tableau suivant. |
| **operationVersion** |Version d’API REST demandée par le client. |
| **category** |Type de résultat. Pour les journaux Key Vault, `AuditEvent` est la seule valeur disponible. |
| **resultType** |Résultat de la requête d’API REST. |
| **resultSignature** |État HTTP |
| **resultDescription** |Description supplémentaire du résultat le cas échéant. |
| **durationMs** |Délai nécessaire pour répondre à la demande API REST, en millisecondes. La latence du réseau n’est pas incluse dans ce chiffre et donc, le temps mesuré côté client peut ne pas correspondre à cette durée. |
| **callerIpAddress** |Adresse IP du client qui a effectué la requête. |
| **correlationId** |GUID facultatif que le client peut transférer pour mettre en corrélation les journaux d’activité côté client avec les journaux d’activité côté service (Key Vault). |
| **identity** |Identité issue du jeton qui a été présenté dans la requête d’API REST. Il s’agit généralement d’un « utilisateur », d’un « principal de service » ou de la « utilisateur + appId », comme dans le cas d’une requête résultant d’une cmdlet Azure PowerShell. |
| **properties** |Informations variables en fonction de l’opération (**operationName**). Dans la plupart des cas, ce champ contient des informations sur le client (chaîne d’agent utilisateur transmise par le client), l’URI de requête d’API REST exacte et le code d’état HTTP. En outre, quand un objet est retourné suite à une requête (par exemple, **KeyCreate** ou **VaultGet**), ce champ contient également l’URI de la clé (sous la forme `id`), l’URI du coffre ou l’URI du secret. |

Les valeurs du champ **operationName** sont indiquées au format *ObjectVerb*. Par exemple :

* Toutes les opérations de coffre de clés présentent le format `Vault<action>`, comme `VaultGet` et `VaultCreate`.
* Toutes les opérations de clé présentent le format `Key<action>`, comme `KeySign` et `KeyList`.
* Toutes les opérations de secret présentent le format `Secret<action>`, comme `SecretGet` et `SecretListVersions`.

Le tableau ci-après répertorie les valeurs **operationName** et les commandes API REST correspondantes :

### <a name="operation-names-table"></a>Table des noms d’opération

| operationName | Commande API REST |
| --- | --- |
| **Authentification** |Authentification par le biais du point de terminaison Azure Active Directory |
| **VaultGet** |[Obtention des informations sur un coffre de clés](/rest/api/keyvault/vaults) |
| **VaultPut** |[Création ou mise à jour d’un coffre de clés](/rest/api/keyvault/vaults) |
| **VaultDelete** |[Suppression d’un coffre de clés](/rest/api/keyvault/vaults) |
| **VaultPatch** |[Mise à jour d’un coffre de clés](/rest/api/keyvault/vaults) |
| **VaultList** |[Liste de l’ensemble des coffres de clés dans un groupe de ressources](/rest/api/keyvault/vaults) |
| **VaultPurge** |[Vider le coffre supprimé](/rest/api/keyvault/vaults/purgedeleted) |
| **VaultRecover** |Récupérer un coffre supprimé|
| **VaultGetDeleted** |[Obtenir un coffre supprimé](/rest/api/keyvault/vaults/getdeleted) |
| **VaultListDeleted** |[Lister les coffres supprimés](/rest/api/keyvault/vaults/listdeleted) |
| **KeyCreate** |[Création d’une clé](/rest/api/keyvault/createkey) |
| **KeyGet** |[Obtention des informations sur une clé](/rest/api/keyvault/getkey) |
| **KeyImport** |[Importation d’une clé dans un coffre](/rest/api/keyvault/vaults) |
| **KeyDelete** |[Suppression d’une clé](/rest/api/keyvault/deletekey) |
| **KeySign** |[Signature avec une clé](/rest/api/keyvault/sign) |
| **KeyVerify** |[Vérification à l’aide d’une clé](/rest/api/keyvault/vaults) |
| **KeyWrap** |[Encapsulage d’une clé](/rest/api/keyvault/wrapkey) |
| **KeyUnwrap** |[Désencapsulage d’une clé](/rest/api/keyvault/unwrapkey) |
| **KeyEncrypt** |[Chiffrement avec une clé](/rest/api/keyvault/encrypt) |
| **KeyDecrypt** |[Déchiffrement avec une clé](/rest/api/keyvault/decrypt) |
| **KeyUpdate** |[Mise à jour d’une clé](/rest/api/keyvault/updatekey) |
| **KeyList** |[Liste des clés dans un coffre](/rest/api/keyvault/getkeys) |
| **KeyListVersions** |[Liste des versions d’une clé](/rest/api/keyvault/getkeyversions) |
| **KeyPurge** |[Vider une clé](/rest/api/keyvault/purgedeletedkey) |
| **KeyBackup** |[Sauvegarder une clé](/rest/api/keyvault/backupkey) |
| **KeyRestore** |[Restauration d’une clé](/rest/api/keyvault/restorekey) |
| **KeyRecover** |[Récupérer une clé](/rest/api/keyvault/recoverdeletedkey) |
| **KeyGetDeleted** |[Obtenir une clé supprimée](/rest/api/keyvault/getdeletedkey) |
| **KeyListDeleted** |[Lister les clés supprimées dans un coffre](/rest/api/keyvault/getdeletedkeys) |
| **CertificateGet** |[Obtenir des informations sur un certificat](/rest/api/keyvault/getcertificate) |
| **CertificateCreate** |[Créer un certificat](/rest/api/keyvault/createcertificate) |
| **CertificateImport** |[Importer un certificat dans un coffre](/rest/api/keyvault/importcertificate) |
| **CertificateUpdate** |[Mettre à jour un certificat](/rest/api/keyvault/updatecertificate) |
| **CertificateList** |[Lister les certificats d’un coffre](/rest/api/keyvault/getcertificates) |
| **CertificateListVersions** |[Lister les versions d’un certificat](/rest/api/keyvault/getcertificateversions) |
| **CertificateDelete** |[Supprimer un certificat](/rest/api/keyvault/deletecertificate) |
| **CertificatePurge** |[Vider un certificat](/rest/api/keyvault/purgedeletedcertificate) |
| **CertificateBackup** |[Sauvegarder un certificat](/rest/api/keyvault/backupcertificate) |
| **CertificateRestore** |[Restaurer un certificat](/rest/api/keyvault/restorecertificate) |
| **CertificateRecover** |[Récupérer un certificat](/rest/api/keyvault/recoverdeletedcertificate) |
| **CertificateGetDeleted** |[Obtenir un certificat supprimé](/rest/api/keyvault/getdeletedcertificate) |
| **CertificateListDeleted** |[Lister les certificats supprimés d’un coffre](/rest/api/keyvault/getdeletedcertificates) |
| **CertificatePolicyGet** |[Obtenir une stratégie de certificat](/rest/api/keyvault/getcertificatepolicy) |
| **CertificatePolicyUpdate** |[Mettre à jour une stratégie de certificat](/rest/api/keyvault/updatecertificatepolicy) |
| **CertificatePolicySet** |[Créer une stratégie de certificat](/rest/api/keyvault/createcertificate) |
| **CertificateContactsGet** |[Obtenir des contacts de certificat](/rest/api/keyvault/getcertificatecontacts) |
| **CertificateContactsSet** |[Définir des contacts de certificat](/rest/api/keyvault/setcertificatecontacts) |
| **CertificateContactsDelete** |[Supprimer des contacts de certificat](/rest/api/keyvault/deletecertificatecontacts) |
| **CertificateIssuerGet** |[Obtenir un émetteur de certificat](/rest/api/keyvault/getcertificateissuer) |
| **CertificateIssuerSet** |[Définir un émetteur de certificat](/rest/api/keyvault/setcertificateissuer) |
| **CertificateIssuerUpdate** |[Mettre à jour un émetteur de certificat](/rest/api/keyvault/updatecertificateissuer) |
| **CertificateIssuerDelete** |[Supprimer un émetteur de certificat](/rest/api/keyvault/deletecertificateissuer) |
| **CertificateIssuersList** |[Lister les émetteurs de certificats](/rest/api/keyvault/getcertificateissuers) |
| **CertificateEnroll** |Inscrire un certificat |
| **CertificateRenew** |Renouveler un certificat |
| **CertificatePendingGet** |Récupérer le certificat en attente |
| **CertificatePendingMerge** |En attente d’une fusion de certificats |
| **CertificatePendingUpdate** |En attente d’une mise à jour de certificat |
| **CertificatePendingDelete** |Supprimer un certificat en attente |
| **SecretSet** |[Création d’une clé secrète](/rest/api/keyvault/updatecertificate) |
| **SecretGet** |[Obtention d’un secret](/rest/api/keyvault/getsecret) |
| **SecretUpdate** |[Mise à jour d’une clé secrète](/rest/api/keyvault/updatesecret) |
| **SecretDelete** |[Suppression d’une clé secrète](/rest/api/keyvault/deletesecret) |
| **SecretList** |[Liste des clés secrètes d’un coffre](/rest/api/keyvault/getsecrets) |
| **SecretListVersions** |[Liste des versions d’une clé secrète](/rest/api/keyvault/getsecretversions) |
| **SecretPurge** |[Vider un secret](/rest/api/keyvault/purgedeletedsecret) |
| **SecretBackup** |[Sauvegarder un secret](/rest/api/keyvault/backupsecret) |
| **SecretRestore** |[Restaurer un secret](/rest/api/keyvault/restoresecret) |
| **SecretRecover** |[Récupérer un secret](/rest/api/keyvault/recoverdeletedsecret) |
| **SecretGetDeleted** |[Obtenir un secret supprimé](/rest/api/keyvault/getdeletedsecret) |
| **SecretListDeleted** |[Lister les secrets supprimés dans un coffre](/rest/api/keyvault/getdeletedsecrets) |
| **VaultAccessPolicyChangedEventGridNotification** | Événement lié à la modification de la stratégie d’accès au coffre publié |
| **SecretNearExpiryEventGridNotification** |Événement lié à un secret proche de l’expiration publié |
| **SecretExpiredEventGridNotification** |Événement lié à un secret expiré publié |
| **KeyNearExpiryEventGridNotification** |Événement lié à une clé proche de l’expiration publié |
| **KeyExpiredEventGridNotification** |Événement lié à une clé expirée publié |
| **CertificateNearExpiryEventGridNotification** |Événement lié à un certificat proche de l’expiration publié |
| **CertificateExpiredEventGridNotification** |Événement lié à un certificat expiré publié |

## <a name="use-azure-monitor-logs"></a>Utiliser les journaux d’activité Azure Monitor

Vous pouvez utiliser la solution Key Vault dans les journaux Azure Monitor pour consulter les journaux `AuditEvent` de Key Vault. Dans les journaux Azure Monitor, vous utilisez des requêtes de journaux pour analyser les données et obtenir les informations dont vous avez besoin. 

Pour plus d’informations, notamment sur la procédure de configuration correspondante, consultez [Azure Key Vault dans Azure Monitor](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour activer la journalisation de Key Vault](howto-logging.md)
- Pour accéder à un tutoriel utilisant Azure Key Vault dans une application web .NET, consultez l’article [Utilisation d’Azure Key Vault à partir d’une application web](tutorial-net-create-vault-azure-web-app.md).
- Pour les références de programmation, consultez le [guide du développeur de coffre de clés Azure](developers-guide.md).
- Pour obtenir la liste des cmdlets Azure PowerShell 1.0 concernant Azure Key Vault, consultez l’article [Azure Key Vault Cmdlets](/powershell/module/az.keyvault/?view=azps-1.2.0#key_vault) (Cmdlets Azure Key Vault).
