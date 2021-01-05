---
author: billmath
ms.service: active-directory
ms.subservice: governance
ms.topic: include
ms.date: 10/16/2019
ms.author: billmath
ms.openlocfilehash: be33e492f44f6926f2ae2d133cf112245fba15f1
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93134950"
---
## <a name="cloud-hr-application-to-azure-active-directory-user-provisioning"></a>Application RH cloud pour l’attribution d’utilisateurs Azure Active Directory

Historiquement, le personnel informatique s’appuyait sur des méthodes manuelles pour créer, mettre à jour et supprimer des employés. Ils ont utilisé des méthodes telles que le téléchargement de fichiers CSV ou de scripts personnalisés pour synchroniser les données des employés. Ces processus d’approvisionnement sont sujets aux erreurs, non sécurisés et difficiles à gérer.

Pour gérer les cycles de vie des identités des employés, des fournisseurs ou des travailleurs occasionnels, le [service d’approvisionnement d’utilisateurs Azure Active Directory (Azure AD)](../articles/active-directory/app-provisioning/user-provisioning.md) offre une intégration avec des applications de ressources humaines (RH) informatiques basées sur le cloud. Exemples d’applications : Workday ou SuccessFactors.

Azure AD utilise cette intégration pour activer les workflows suivants d’applications RH cloud :

- **Approvisionnement des utilisateurs vers Active Directory :** approvisionnez les ensembles d’utilisateurs sélectionnés dans une application RH cloud dans un ou plusieurs domaines Active Directory (AD).
- **Approvisionnement des utilisateurs du cloud uniquement vers Azure AD :** dans les scénarios où Active Directory n’est pas utilisé, approvisionnez les utilisateurs directement à partir de l’application RH du cloud vers Azure AD.
- **Mise à jour de l’application RH cloud**  : mettez à jour les adresses e-mail et les attributs de nom d’utilisateur à partir d’Azure AD sur l’application RH cloud.


## <a name="enabled-hr-scenarios"></a>Scénarios RH activés

Le service d’approvisionnement d’utilisateurs Azure AD permet d’automatiser les scénarios suivants de gestion du cycle de vie des identités basée sur les RH :

- **Embauche de nouveaux employés :** lorsqu’un nouvel employé est ajouté à l’application RH cloud, un compte d’utilisateur est automatiquement créé dans Active Directory et Azure AD avec la possibilité de mettre à jour l’adresse e-mail et les attributs du nom d’utilisateur dans l’application RH cloud.
- **Mises à jour des profils et des attributs des employés :** lorsqu’un enregistrement d’employé, tel qu’un nom, un titre ou un responsable, est mis à jour dans l’application RH cloud, son compte d’utilisateur est automatiquement mis à jour dans Active Directory et Azure AD.
- **Licenciement des employés :** lorsqu’un employé est licencié dans l’application RH cloud, son compte d’utilisateur est automatiquement désactivé dans Active Directory et Azure AD.
- **Réembauche d’employés :** lorsqu’un employé est réembauché dans l’application RH cloud, son ancien compte peut être automatiquement réactivé ou réapprovisionné dans Active Directory et Azure AD.

## <a name="who-is-this-integration-best-suited-for"></a>À qui cette intégration convient-elle le mieux ?

L’intégration de l’application RH cloud avec l’approvisionnement d’utilisateurs Azure AD est idéale pour les organisations qui :

- veulent une solution informatique prédéfinie basée sur le cloud pour l’approvisionnement d’utilisateurs RH cloud ;
- nécessitent un approvisionnement d’utilisateurs direct de l’application RH cloud vers Active Directory ou Azure AD ;
- exigent que les utilisateurs soient approvisionnés à l’aide des données obtenues à partir de l’application RH cloud ;
- ont besoin de joindre, de déplacer et de laisser les utilisateurs à synchroniser avec une ou plusieurs unités d’organisation, domaines et forêts Active Directory seulement sur la base d’informations de modifications détectées dans l’application RH cloud ;
- utilisent Office 365 pour les e-mails.


### <a name="key-benefits"></a>Principaux avantages

Cette fonctionnalité d’approvisionnement informatique axé sur les ressources humaines offre les avantages significatifs suivants pour l’entreprise :

- **Augmenter la productivité :** vous pouvez désormais automatiser l’attribution de comptes d’utilisateur et de licences Office 365 et fournir un accès aux groupes de clés. L’automatisation des attributions offre aux nouveaux employés un accès immédiat à leurs outils de travail et augmente la productivité.
- **Gérer le risque :** vous pouvez augmenter la sécurité en automatisant les changements, en fonction du statut des employés ou des appartenances aux groupes grâce aux données transmises à partir de l’application RH cloud. L’automatisation des modifications permet de s’assurer que les identités des utilisateurs et l’accès aux applications clés sont automatiquement mis à jour lorsque les utilisateurs sont transférés ou quittent l’organisation.
- **Conformité et gouvernance des adresses :** Azure AD prend en charge les journaux d’audit natifs pour les demandes d’approvisionnement d’utilisateurs effectuées par les applications des systèmes source et cible. Lors d’audit, vous pouvez effectuer le suivi des personnes qui ont accès aux applications à partir d’un seul écran.
- **Gérer les coûts :** l’approvisionnement automatique réduit les coûts tout en évitant l’inefficacité et les erreurs humaines qui sont inhérents à l’approvisionnement manuel. Cela réduit le besoin de solutions d’approvisionnement d’utilisateurs personnalisées créées au fil du temps à l’aide de plateformes héritées et obsolètes.
