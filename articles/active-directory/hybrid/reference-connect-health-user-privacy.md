---
title: Azure AD Connect Health et confidentialité des utilisateurs | Microsoft Docs
description: Ce document décrit la confidentialité des utilisateurs avec Azure AD Connect Health.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 04/26/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 58ecdfefa79716bcc0a69063293d5a1067693a78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89182358"
---
# <a name="user-privacy-and-azure-ad-connect-health"></a>Confidentialité des utilisateurs et Azure AD Connect Health 

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

>[!NOTE] 
>Cet article porte sur Azure AD Connect Health et la confidentialité des utilisateurs.  Pour plus d’informations sur Azure AD Connect et la confidentialité des utilisateurs, consultez [cet article](reference-connect-user-privacy.md).

## <a name="user-privacy-classification"></a>Classification de la confidentialité des utilisateurs
Azure AD Connect Health est classé dans la catégorie **Processeur de données** de la classification catégorie RGPD. En tant que pipeline de processeur de données, le service fournit un traitement des données pour les partenaires et les clients. Azure AD Connect Health ne génère pas de données utilisateur et n’a aucun contrôle indépendant sur les données personnelles qui sont collectées, ni sur la façon dont elles sont utilisées. Dans Azure AD Connect Health, l’extraction, l’agrégation et l’analyse de données, ainsi que la création de rapports contenant ces données, sont basées sur des données locales existantes. 

## <a name="data-retention-policy"></a>Stratégie de conservation des données
Azure AD Connect Health ne génère pas de rapports, n’effectue pas d’analyses et ne fournit pas d’insights au-delà de 30 jours. Par conséquent, Azure AD Connect Health ne stocke pas, ne traite pas et ne conserve pas de données au-delà de 30 jours. Cette conception est conforme aux réglementations RGPD, aux réglementations de Microsoft concernant la conformité, ainsi qu’aux stratégies de conservation des données Azure AD. 

Les serveurs sur lesquels il existe des alertes d'**erreur** **Les données du Service de contrôle d'intégrité ne sont pas à jour** pendant plus de 30 jours consécutifs suggèrent qu'aucune donnée n'a atteint Connect Health pendant cet intervalle de temps. Ces serveurs sont désactivés et n’apparaissent pas dans le portail Connect Health. Pour les réactiver, vous devez désinstaller et [réinstaller l’agent d’intégrité](how-to-connect-health-agent-install.md). Notez que cette désactivation ne s’applique pas aux **avertissements** avec le même type d’alerte. Les avertissements indiquent qu’il manque une partie des données en provenance du serveur pour lequel vous voulez être alerté. 
 
## <a name="disable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>Désactiver la collecte et la surveillance des données dans Azure AD Connect Health
Azure AD Connect Health vous permet d’arrêter la collecte des données pour chaque serveur surveillé ou pour une instance d’un service surveillé. Par exemple, vous pouvez arrêter la collecte des données pour chaque serveur ADFS qui est surveillé à l’aide d’Azure AD Connect Health. Vous pouvez également arrêter la collecte de données pour l’intégralité d’une instance ADFS qui est surveillée par Azure AD Connect Health. Dans ce cas, les serveurs correspondants sont supprimés du portail Azure AD Connect Health, après l’arrêt de la collecte de données. 

>[!IMPORTANT]
> Pour supprimer les serveurs surveillés d’Azure AD Connect Health, vous devez disposer de privilèges d’administrateur général Azure AD ou du rôle Contributeur dans Azure RBAC.
>
> La suppression d’un serveur ou d’une instance de service d’Azure AD Connect Health est une action irréversible. 

### <a name="what-to-expect"></a>À quoi s’attendre ?
Si vous arrêtez la collecte et la surveillance des données pour un serveur surveillé ou une instance de service surveillé, prenez en compte les points suivants :

- Lorsque vous supprimez une instance d’un service surveillé, l’instance est supprimée de la liste du service de surveillance d’Azure AD Connect Health dans le portail. 
- Lorsque vous supprimez un serveur surveillé ou une instance d’un service surveillé, Health Agent n’est pas désinstallé ni supprimé de vos serveurs. Health Agent est configuré pour ne pas envoyer de données vers Azure AD Connect Health. Vous devez désinstaller manuellement Health Agent sur les serveurs précédemment surveillés.
- Si vous n’avez pas désinstallé l’agent Health avant d’exécuter cette étape, il est possible que des événements d’erreur s’affichent sur le(s) serveur(s) associé(s) à l’agent.
- Toutes les données appartenant à l’instance du service surveillé sont supprimées, conformément à la stratégie de conservation des données de Microsoft Azure.

### <a name="disable-data-collection-and-monitoring-for-an-instance-of-a-monitored-service"></a>Désactiver la collecte et la surveillance des données pour une instance d’un service surveillé
Regardez [comment supprimer une instance de service d’Azure AD Connect Health](how-to-connect-health-operations.md#delete-a-service-instance-from-azure-ad-connect-health-service).

### <a name="disable-data-collection-and-monitoring-for-a-monitored-server"></a>Désactiver la collecte et la surveillance des données pour un serveur surveillé
Regardez [comment supprimer un serveur d’Azure AD Connect Health](how-to-connect-health-operations.md#delete-a-server-from-the-azure-ad-connect-health-service).

### <a name="disable-data-collection-and-monitoring-for-all-monitored-services-in-azure-ad-connect-health"></a>Désactiver la collecte de données et la surveillance pour tous les services surveillés dans Azure AD Connect Health
Azure AD Connect Health fournit également la possibilité d’arrêter la collecte de données de **tous** les services inscrits dans le locataire. Nous vous recommandons d’être très prudent et de bénéficier de l’assentiment complet de tous les administrateurs généraux avant de prendre des mesures. Une fois le processus démarré, le service Connect Health cessera de recevoir, de traiter et de créer des rapports sur les données pour tous vos services. Les données existantes dans le service Connect Health seront conservées pendant 30 jours maximum.
Si vous souhaitez arrêter la collecte de données pour un serveur spécifique, suivez les étapes de suppression de serveurs spécifiques. Pour arrêter la collecte de données pour un locataire, suivez les étapes qui décrivent comment arrêter la collecte de données et supprimez tous les services du locataire.

1. Cliquez sur **Paramètres généraux** sous Configuration dans le panneau principal. 
2. Cliquez sur le bouton **Arrêter la collecte de données** en haut du panneau. Les autres options des paramètres de configuration du locataire seront désactivées une fois le processus démarré.  
 
   ![Arrêter la collecte de données](./media/reference-connect-health-user-privacy/gdpr4.png)
  
3. Vérifiez la liste des services intégrés qui sont affectés par l’arrêt de la collecte de données. 
4. Entrez le nom exact du locataire pour activer le bouton **Supprimer**.
5. Cliquez sur **Supprimer** pour déclencher la suppression de tous les services. Connect Health cessera de recevoir, de traiter et de créer des rapports pour toutes les données envoyées à partir de vos services intégrés. L’ensemble du processus peut prendre jusqu’à 24 heures. Notez que cette étape n’est pas réversible. 
6. Une fois le processus terminé, vous ne verrez plus aucun service inscrit dans Connect Health. 

   ![Après l’arrêt de la collecte de données](./media/reference-connect-health-user-privacy/gdpr5.png)

## <a name="re-enable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>Réactiver la collecte et la surveillance des données dans Azure AD Connect Health
Si vous souhaitez réactiver la surveillance dans Azure AD Connect Health pour un service surveillé ayant été supprimé, vous devez désinstaller puis [réinstaller Health Agent](how-to-connect-health-agent-install.md) sur tous les serveurs.

### <a name="re-enable-data-collection-and-monitoring-for-all-monitored-services"></a>Réactiver la collecte et la surveillance des données pour tous les services surveillés

Vous pouvez reprendre la collecte des données pour un locataire dans Azure AD Connect Health. Nous vous recommandons d’être très prudent et de bénéficier de l’assentiment complet de tous les administrateurs généraux avant de prendre des mesures.

>[!IMPORTANT]
> Les étapes suivantes seront disponibles 24 heures après l’exécution de l’action de désactivation.
> Après l’activation de la collecte de données, les données de surveillance présentées dans Connect Health n’afficheront aucune donnée héritée collectée auparavant. 

1. Cliquez sur **Paramètres généraux** sous Configuration dans le panneau principal. 
2. Cliquez sur le bouton **Activer la collecte de données** en haut du panneau. 
 
   ![Activer la collecte des données](./media/reference-connect-health-user-privacy/gdpr6.png)
 
3. Entrez le nom exact du locataire pour activer le bouton **Activer**.
4. Cliquez sur le bouton **Activer** pour accorder l’autorisation de collecte de données dans le service Connect Health. La modification sera appliquée quelques instants plus tard. 
5. Suivez le [processus d’installation](how-to-connect-health-agent-install.md) pour réinstaller l’agent sur les serveurs à surveiller, et les services seront présents dans le portail.  


## <a name="next-steps"></a>Étapes suivantes
* [Lire la politique de confidentialité Microsoft sur le Centre de confidentialité](https://www.microsoft.com/trustcenter)
* [Azure AD Connect et confidentialité des utilisateurs](reference-connect-user-privacy.md)

