---
title: Gérer les contraintes de contexte de sécurité dans Azure Red Hat OpenShift | Microsoft Docs
description: Contraintes de contexte de sécurité pour les administrateurs de cluster Azure Red Hat OpenShift
services: container-service
author: troy0820
ms.author: b-trconn
ms.service: container-service
ms.topic: article
ms.date: 09/25/2019
ms.openlocfilehash: eb5c568f056a99187a0e7a78a6f89b206f2d8dec
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220242"
---
# <a name="manage-security-context-constraints-in-azure-red-hat-openshift"></a>Gérer les contraintes de contexte de sécurité dans Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Les contraintes de contexte de sécurité permettent aux administrateurs de cluster de contrôler les autorisations pour les pods. Pour en savoir plus sur ce type d’API, consultez la [documentation sur l’architecture des contraintes de contexte de sécurité](https://docs.openshift.com/container-platform/3.11/architecture/additional_concepts/authorization.html). Vous pouvez gérer les contraintes de contexte de sécurité dans votre instance comme des objets d’API normaux en utilisant l’interface de ligne de commande.

## <a name="list-security-context-constraints"></a>Lister les contraintes de contexte de sécurité

Pour obtenir la liste actuelle des contraintes de contexte de sécurité, utilisez cette commande : 

```bash
$ oc get scc

NAME               PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
hostaccess         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim secret]
hostmount-anyuid   false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim secret]
hostnetwork        false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
nonroot            false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
privileged         true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
```

## <a name="examine-an-object-for-security-context-constraints"></a>Examiner les contraintes de contexte de sécurité dans un objet

Pour examiner une contrainte de contexte de sécurité particulière, utilisez `oc get`, `oc describe` ou `oc edit`.  Par exemple, pour examiner la contrainte de contexte de sécurité **restricted** , utilisez cette commande :
```bash
$ oc describe scc restricted
Name:                    restricted
Priority:                <none>
Access:
  Users:                <none>
  Groups:                system:authenticated
Settings:
  Allow Privileged:            false
  Default Add Capabilities:        <none>
  Required Drop Capabilities:        KILL,MKNOD,SYS_CHROOT,SETUID,SETGID
  Allowed Capabilities:            <none>
  Allowed Seccomp Profiles:        <none>
  Allowed Volume Types:            configMap,downwardAPI,emptyDir,persistentVolumeClaim,projected,secret
  Allow Host Network:            false
  Allow Host Ports:            false
  Allow Host PID:            false
  Allow Host IPC:            false
  Read Only Root Filesystem:        false
  Run As User Strategy: MustRunAsRange
    UID:                <none>
    UID Range Min:            <none>
    UID Range Max:            <none>
  SELinux Context Strategy: MustRunAs
    User:                <none>
    Role:                <none>
    Type:                <none>
    Level:                <none>
  FSGroup Strategy: MustRunAs
    Ranges:                <none>
  Supplemental Groups Strategy: RunAsAny
    Ranges:                <none>
```
## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md) 
