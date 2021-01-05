---
title: Journaux d’audit Azure Attestation
description: Décrit les journaux d’audit complets qui sont collectés pour Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 11/23/2020
ms.author: mbaldwin
ms.openlocfilehash: 1fa4a458a4e3e1df1d84c343a32e3a41a4a25e75
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95758672"
---
# <a name="audit-logs-for-azure-attestation"></a>Journaux d’audit pour Azure Attestation

Les journaux d’audit sont des enregistrements sécurisés, immuables et horodatés d’événements discrets qui se sont produits au fil du temps. Ces journaux capturent des événements importants susceptibles d’affecter les fonctionnalités de votre instance d’attestation.

Azure Attestation gère les instances d’attestation et les stratégies qui leur sont associées. Les actions associées à la gestion des instances et aux changements de stratégie sont auditées et journalisées.

Cet article contient des informations sur les événements journalisés, les informations collectées et l’emplacement de ces journaux.

## <a name="about-audit-logs"></a>À propos des journaux d’audit

Azure Attestation utilise du code pour générer des journaux d’audit pour les événements qui affectent la façon dont l’attestation est effectuée. Cela se résume généralement à la façon et au moment où des modifications de stratégie sont apportées à votre instance d’attestation, ainsi qu’à certaines actions d’administration.

### <a name="auditable-events"></a>Événements pouvant être audités
Voici quelques-uns des journaux d’audit que nous collectons :

|     Événement/API                              |     Description de l’événement                                                                         |
|--------------------------------------------|-----------------------------------------------------------------------------------------------|
|     Créer une instance                        |     Crée une nouvelle instance d’un service d’attestation. |
|     Détruire l’instance                       |     Détruit une instance d’un service d’attestation. |
|     Ajouter un certificat de stratégie                 |     Ajoute un certificat à l’ensemble actuel de certificats de gestion de stratégie. |
|     Supprimer un certificat de stratégie              |     Supprime un certificat de l’ensemble actuel de certificats de gestion de stratégie. |
|     Définir la stratégie actuelle                     |     Définit la stratégie d’attestation pour un type TEE donné. |
|     Réinitialiser la stratégie d’attestation               |     Réinitialise la stratégie d’attestation pour un type TEE donné. |
|     Préparer la mise à jour de la stratégie               |     Prépare la mise à jour de la stratégie d’attestation pour un type TEE donné. |
|     Réalimenter les locataires après un sinistre       |     Scelle de nouveau tous les locataires d’attestation sur cette instance du service d’attestation. Cette opération ne peut être effectuée que par les administrateurs du service d’attestation. |

### <a name="collected--information"></a>Informations collectées
Pour chacun de ces événements, Azure Attestation collecte les informations suivantes :

- Nom d’opération
- Réussite de l’opération
- Appelant de l’opération, qui peut être l’un des éléments suivants :
    - Nom d’utilisateur principal Azure AD
    - ID de l'objet
    - Certificat
    - ID de locataire Azure AD
- Cible de l’opération, qui peut être l’un des éléments suivants :
    - Environnement
    - Région du service
    - Rôle du service
    - Instance de rôle du service
    - ID de ressource
    - Région de ressource

### <a name="sample-audit-log"></a>Exemple de journal d’audit

Les journaux d’audit sont fournis au format JSON. Voici un exemple de ce à quoi un journal d’audit peut ressembler.

```json
{"operationName":"SetCurrentPolicy","resultType":"Success","resultDescription":null,"auditEventCategory":["ApplicationManagement"],"nCloud":null,"requestId":null,"callerIpAddress":null,"callerDisplayName":null,"callerIdentities":[{"callerIdentityType":"ObjectID","callerIdentity":"<some object ID>"},{"callerIdentityType":"TenantId","callerIdentity":"<some tenant ID>"}],"targetResources":[{"targetResourceType":"Environment","targetResourceName":"PublicCloud"},{"targetResourceType":"ServiceRegion","targetResourceName":"EastUS2"},{"targetResourceType":"ServiceRole","targetResourceName":"AttestationRpType"},{"targetResourceType":"ServiceRoleInstance","targetResourceName":"<some service role instance>"},{"targetResourceType":"ResourceId","targetResourceName":"/subscriptions/<some subscription ID>/resourceGroups/<some resource group name>/providers/Microsoft.Attestation/attestationProviders/<some instance name>"},{"targetResourceType":"ResourceRegion","targetResourceName":"EastUS2"}],"ifxAuditFormat":"Json","env_ver":"2.1","env_name":"#Ifx.AuditSchema","env_time":"2020-11-23T18:23:29.9427158Z","env_epoch":"MKZ6G","env_seqNum":1277,"env_popSample":0.0,"env_iKey":null,"env_flags":257,"env_cv":"##00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000","env_os":null,"env_osVer":null,"env_appId":null,"env_appVer":null,"env_cloud_ver":"1.0","env_cloud_name":null,"env_cloud_role":null,"env_cloud_roleVer":null,"env_cloud_roleInstance":null,"env_cloud_environment":null,"env_cloud_location":null,"env_cloud_deploymentUnit":null}
```

## <a name="access-audit-logs"></a>Journaux d’audit d’accès

Ces journaux sont stockés dans Azure et ne sont pas accessibles directement. Si vous avez besoin d’accéder à ces journaux, remplissez un ticket de support. Pour plus d'informations, consultez [Contacter le support technique de Microsoft](https://azure.microsoft.com/support/options/). 

Une fois le ticket de support rempli, Microsoft télécharge et vous transmet l’accès à ces journaux.
