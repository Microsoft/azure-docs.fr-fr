---
title: Déployer OpenShift Container Platform 4.x dans Azure
description: Déployez OpenShift Container Platform 4.x dans Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: f2fb1657c26fce3e1fdd67f36b0c6511b78dd216
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87373536"
---
# <a name="deploy-openshift-container-platform-4x-in-azure"></a>Déployer OpenShift Container Platform 4.x dans Azure

Le déploiement d’OpenShift Container Platform (OCP) 4.2 est maintenant pris en charge dans Azure via le modèle IPI (infrastructure provisionnée par le programme d’installation).  La page d’accueil pour essayer OpenShift 4 est [try.openshift.com](https://try.openshift.com/). Pour installer OCP 4.2 dans Azure, visitez la page [Red Hat OpenShift Cluster Manager](https://cloud.redhat.com/openshift/install/azure/installer-provisioned).  Des informations d’identification Red Hat sont requises pour accéder à ce site.


## <a name="notes"></a>Notes 

 - Un principal de service Azure Active Directory (AAD) est requis pour installer et exécuter OCP 4.x dans Azure
     - Le principal de service doit disposer de l’autorisation d’API **Application.ReadWrite.OwnedBy** pour Azure Active Directory Graph
     - Un administrateur de locataires AAD doit accorder son consentement administrateur pour que cette autorisation d’API prenne effet
     - Le principal de service doit se voir octroyer les rôles **Collaborateur** et **Administrateur de l’accès utilisateur** pour l’abonnement
 - Le modèle d’installation pour OCP 4.x est différent de celui pour 3.x et aucun modèle Azure Resource Manager n’est disponible pour le déploiement d’OCP 4.x dans Azure
 - Si des problèmes surviennent au cours du processus d’installation, contactez l’entreprise appropriée (Microsoft ou Red Hat)

| Description du problème | Point de contact |
|-------------------|---------------|
| Problèmes spécifiques à Azure (AAD, principal de service, abonnement Azure, etc.)                              | Microsoft |
| Problèmes spécifiques à OpenShift (échecs/erreurs d’installation, abonnement Red Hat, etc.) |  Red Hat  |




## <a name="next-steps"></a>Étapes suivantes

- [Prise en main d’OpenShift Container Platform](https://docs.openshift.com)
