---
title: Configurer Lockbox pour Azure Data Box
description: Découvrez comment utiliser Customer Lockbox avec Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.topic: how-to
ms.date: 07/10/2020
ms.author: alkohli
ms.subservice: pod
ms.openlocfilehash: 178ad169c1b576458f38b440ca79f4bb9eb012f5
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92124909"
---
# <a name="use-customer-lockbox-for-azure-data-box-preview"></a>Utiliser Customer Lockbox pour Azure Data Box (préversion)

Azure Data Box est utilisé pour transférer des données client vers et à partir d’Azure. Il existe des situations où le Support Microsoft peut avoir besoin d’accéder aux données client lors d’une demande de support (Support request). Vous pouvez utiliser Customer Lockbox en tant qu’interface pour examiner et approuver ou rejeter ces demandes d’accès aux données. 

Cet article décrit comment les demandes Customer Lockbox sont initiées et suivies pour les commandes d’importation et d’exportation de Data Box. L’article s’applique aux appareils Azure Data Box et Azure Data Box Heavy. 

## <a name="devops-workflow-for-data-access"></a>Flux de travail DevOps pour l’accès aux données

En général, l’équipe de Support et de Fonctionnement Data Box de Microsoft n’accède pas aux données client. Ils essaient de résoudre les problèmes à l’aide d’outils standard et de la télémétrie. <!--The only scenarios where there is a need to access customer data is when there is an issue with the data that needs to be fixed. For example, if the data is copied to a wrong folder or is in an incorrect format and is likely to result in an upload or download failure, then Microsoft will try to access your data in the Azure datacenter.--> 

Si les problèmes ne peuvent pas être résolus et nécessitent l’intervention du Support Microsoft pour examiner ou réparer des données,l’équipe demande un accès élevé via le portail Just in Time (JIT). Le portail JIP valide le niveau d’autorisation, fournit une authentification multifacteur et inclut également une approbation des approbateurs internes de Microsoft. Par exemple, l’approbateur pourrait être DevOps Manager. 

Une fois que votre demande d’accès avec privilèges élevés est approuvée via le portail JIT, si vous avez activé Lockbox, Microsoft aura également besoin de votre autorisation explicite pour accéder aux données. L’accès est demandé et suivi via le service Customer Lockbox dans le portail. 

Si vous n’avez pas activé Lockbox, votre autorisation n’est pas nécessaire pour accéder aux données.


## <a name="prerequisites-for-access-request"></a>Prérequis pour la requête d’accès

Avant de commencer, vérifiez les points suivants :

1. Vous avez créé une commande Azure Data Box conformément aux instructions dans :
    1. [Tutoriel : Commande d’Azure Data Box](data-box-deploy-ordered.md) pour les commandes d’importation.
    1. [Tutoriel : Commande d’Azure Data Box](data-box-deploy-export-ordered.md) pour les commandes dexmportation.

2. Vous avez configuré Customer Lockbox pour Data Box. Il s’agit d’un service d’opt-in. 

    1. Customer Lockbox est actuellement en préversion pour le service Data Box. Pour activer Customer Lockbox pour Data Box pour votre organisation, inscrivez-vous à [Customer Lockbox pour la préversion publique d’Azure](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_Kwz02N6XVCoKNpxIpqE_hUNzlTUUNYVkozOVlFNVRSWDVHRkkwTFQyViQlQCN0PWcu).
    2. Customer Lockbox est automatiquement disponible pour tous les clients qui ont un plan de support Azure avec un niveau minimal de Développeur. <!--How do you enable Lockbox? change this for Azure Data Box, perhaps you need a different support plan When you have an eligible support plan, no action is required by you to enable Customer Lockbox. Customer Lockbox requests are initiated by a Microsoft engineer if this action is needed to progress a support ticket that is filed from somebody in your organization.-->

3. Une demande de service ou un ticket de support est déjà ouvert pour ce problème. Pour plus d’informations sur les ticket de support, consultez [Demande de service pour Data Box](data-box-disk-contact-microsoft-support.md).


## <a name="track-approve-request-via-lockbox"></a>Suivre, approuver une demande via Lockbox

Pour suivre et approuver une demande d’accès aux données client, procédez comme suit :

1. Microsoft détecte un problème pendant le chargement ou le téléchargement des données dans le centre de données Azure. Par exemple, la commande de Data Box est interrompue pendant la **phase de copie des données**. 

    L’ingénieur du support technique se connecte à Data Box via la session de support et tente de résoudre le problème à l’aide d’outils standard et de la télémétrie. Si les disques Data Box sont verrouillés et que les partages ne sont pas accessibles, l’ingénieur du support crée une demande de Lockbox. 
 
2. Lorsque la demande est créée, la notification est généralement envoyée à l’administrateur de l’abonnement, mais vous pouvez également configurer un groupe pour les notifications. 

3. Vous pouvez voir la demande de Lockbox dans le Portail Azure pour votre approbation. 

    ![Demande dans le portail Azure](./media/data-box-customer-lockbox/3-lockbox-request-azure-portal.png)

    Pour approuver la demande de Lockbox à partir du portail, vous pouvez sélectionner **Approuver**.

    ![Approuver la demande](./media/data-box-customer-lockbox/4-lockbox-request-details-azure-portal.png)


    Une fois la demande approuvée, les disques de l’appareil sont déverrouillés et les partages sont accessibles dans la session de support.

4. L’ingénieur du support technique résout le problème de chargement, puis désactive la session de support.

Une fois le problème résolu, la tâche de copie de données continuera jusqu’à arriver à son terme.


## <a name="next-steps"></a>Étapes suivantes

- [Customer Lockbox pour Microsoft Azure](../security/fundamentals/customer-lockbox-overview.md)

<!--- [Approve, audit support access requests to VMs using Customer Lockbox for Azure](https://azure.microsoft.com/blog/approve-audit-support-access-requests-to-vms-using-customer-lockbox-for-azure/)-->