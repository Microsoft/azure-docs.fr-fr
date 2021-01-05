---
title: Gérer les stratégies de laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Apprenez à définir des stratégies de laboratoire telles que les tailles de machine virtuelle, le nombre maximal de machines virtuelles par utilisateur et l’arrêt automatique.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a7ada980b2a251b8164f2e1a20f65da54c89ba72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90530350"
---
# <a name="manage-all-policies-for-a-lab-in-azure-devtest-labs"></a>Gérer toutes les stratégies d’un laboratoire dans Azure DevTest Labs

Azure DevTest Labs vous permet de contrôler les coûts et réduit le gaspillage dans vos laboratoires en gérant les stratégies (paramètres) de chacun d’entre eux. Cet article décrit étape par étape comment définir chaque stratégie.  

## <a name="set-allowed-virtual-machine-sizes"></a>Définir les tailles de machine virtuelle autorisées
La stratégie pour définir les tailles de machine virtuelle autorisées vous permet de spécifier les tailles de machine virtuelle autorisées dans le laboratoire et contribue ainsi à réduire les pertes de laboratoire. Si cette stratégie est activée, seules les tailles de machine virtuelle de cette liste peuvent être utilisées pour créer des machines virtuelles.

1. Dans le [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040), sélectionnez un lab, puis **Configuration and policies** (Configuration et stratégies).

    ![Accéder à la configuration et aux stratégies du lab](./media/devtest-lab-set-lab-policy/policies-menu.png)

1. Dans le volet **Configuration et stratégies** du lab, sélectionnez **Tailles de machine virtuelle autorisées**.
   
    ![Tailles de machine virtuelle autorisées](./media/devtest-lab-set-lab-policy/allowed-vm-sizes.png)

1. Sélectionnez **Activer** ou **Désactiver** pour activer ou désactiver cette stratégie.

1. Si vous activez cette stratégie, sélectionnez une ou plusieurs tailles de machine virtuelle pouvant être créées dans votre laboratoire.

1. Sélectionnez **Enregistrer**.

## <a name="set-virtual-machines-per-user"></a>Définir les machines virtuelles par utilisateur
La stratégie **Machines virtuelles par utilisateur** vous permet de spécifier le nombre de machines virtuelles pouvant être créées par un utilisateur individuel. Si un utilisateur essaie de créer ou de revendiquer une machine virtuelle alors que le nombre limite par utilisateur est atteint, un message d’erreur indique que la machine virtuelle ne peut pas être créée/revendiquée. 

1. Dans le volet **Configuration et stratégies** du lab, sélectionnez **Machines virtuelles par utilisateur**.
   
    ![Machines virtuelles par utilisateur](./media/devtest-lab-set-lab-policy/max-vms-per-user.png)

1. Sélectionnez **Oui** pour limiter le nombre de machines virtuelles par utilisateur. Si vous ne souhaitez pas limiter le nombre de machines virtuelles par utilisateur, sélectionnez **Non**. Si vous sélectionnez **Oui**, entrez une valeur numérique indiquant le nombre de machines virtuelles qu’un utilisateur peut créer ou revendiquer. 

1. Sélectionnez **Oui** pour limiter le nombre de machines virtuelles pouvant utiliser un disque SSD. Si vous ne souhaitez pas limiter le nombre de machines virtuelles pouvant utiliser un disque SSD, sélectionnez **Non**. Si vous sélectionnez **Oui**, entrez une valeur indiquant le nombre de machines virtuelles qu’un utilisateur peut créer à l’aide d’un disque SSD. 

1. Sélectionnez **Enregistrer**.

## <a name="set-virtual-machines-per-lab"></a>Définir les machines virtuelles par laboratoire
La stratégie **Machines virtuelles par lab** vous permet de spécifier le nombre de machines virtuelles pouvant être créées pour le lab actuel. Si un utilisateur essaie de créer une machine virtuelle alors que le nombre limite par laboratoire est atteint, un message d’erreur indique que la machine virtuelle ne peut pas être créée. 

1. Dans le volet **Configuration et stratégies** du lab, sélectionnez **Machines virtuelles par lab**.
   
    ![Machines virtuelles par laboratoire](./media/devtest-lab-set-lab-policy/max-vms-per-lab.png)

1. Sélectionnez **Oui** pour limiter le nombre de machines virtuelles par laboratoire. Si vous ne souhaitez pas limiter le nombre de machines virtuelles par laboratoire, sélectionnez **Non**. Si vous sélectionnez **Oui**, entrez une valeur numérique indiquant le nombre de machines virtuelles qu’un utilisateur peut créer ou revendiquer. 

1. Sélectionnez **Oui** pour limiter le nombre de machines virtuelles pouvant utiliser un disque SSD. Si vous ne souhaitez pas limiter le nombre de machines virtuelles pouvant utiliser un disque SSD, sélectionnez **Non**. Si vous sélectionnez **Oui**, entrez une valeur indiquant le nombre de machines virtuelles qu’un utilisateur peut créer à l’aide d’un disque SSD. 

1. Sélectionnez **Enregistrer**.

## <a name="set-auto-shutdown"></a>Définir l'arrêt automatique
La stratégie d’arrêt automatique vous permet d’indiquer l’heure à laquelle les machines virtuelles du lab doivent s’arrêter et contribue ainsi à réduire les pertes du lab.

1. Dans le volet **Configuration et stratégies** du lab, sélectionnez **Arrêt automatique**.
   
    ![Arrêt automatique](./media/devtest-lab-set-lab-policy/auto-shutdown.png)

1. Sélectionnez **Activer** ou **Désactiver** pour activer ou désactiver cette stratégie.

1. Si vous activez cette stratégie, spécifiez l’heure (et le fuseau horaire) de l’arrêt pour toutes les machines virtuelles du laboratoire actif.

1. Spécifiez **Oui** ou **Non** pour l’option d’envoi de notification 15 minutes avant l’heure d’arrêt automatique spécifiée. Si vous choisissez **Oui**, saisissez un point de terminaison de l’URL de Webhook ou une adresse e-mail spécifiant où vous désirez publier ou envoyer la notification. L’utilisateur reçoit une notification et peut différer l’arrêt.

   Pour plus d’informations sur les webhooks, consultez [Créer une fonction Azure d’API ou de webhook](../azure-functions/functions-bindings-http-webhook.md). 

1. Sélectionnez **Enregistrer**.

Par défaut, une fois activée, cette stratégie s’applique à toutes les machines virtuelles dans le laboratoire en cours. Pour supprimer ce paramètre sur une machine virtuelle spécifique, ouvrez le volet de gestion de la machine virtuelle et modifiez son paramètre **Arrêt automatique**.

## <a name="set-auto-shutdown-policy"></a>Définir la stratégie d'arrêt automatique
En tant que propriétaire d’un laboratoire, vous pouvez configurer une planification d’arrêt pour toutes les machines virtuelles de votre laboratoire. Ceci vous permet de réaliser des économies en stoppant l’exécution de machines inutilisées (inactives). Vous pouvez appliquer une stratégie d’arrêt centralisée à toutes les machines virtuelles de votre laboratoire et, également, épargner aux utilisateurs de votre laboratoire l’établissement d’une planification pour leurs machines individuelles. Cette fonctionnalité vous permet de définir la stratégie de planification de votre laboratoire, du contrôle inexistant au contrôle total, pour les utilisateurs de votre laboratoire. En tant que propriétaire de laboratoire, vous pouvez configurer cette stratégie en procédant comme suit :

1. Sur la page d’accueil de votre laboratoire, sélectionnez **Configuration et stratégies**.
2. Sélectionnez **Stratégie d'arrêt automatique** dans la section **Planifications** du menu de gauche.
3. Sélectionnez l'une des options. Les sections suivantes vous donnent plus de détails sur ces options : la stratégie définie s’applique uniquement aux nouvelles machines virtuelles créées dans le laboratoire et non aux machines virtuelles déjà existantes. 

    ![Options de stratégie d'arrêt automatique](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-options.png)

### <a name="user-sets-a-schedule-and-can-opt-out"></a>L'utilisateur définit une planification qu'il peut refuser
Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire peuvent remplacer ou refuser la planification du laboratoire. Cette option accorde aux utilisateurs du laboratoire un contrôle total sur la planification d’arrêt automatique de leurs machines virtuelles. Les utilisateurs du laboratoire ne voient aucune modification de leur page de planification d’arrêt automatique des machine virtuelles.

![Option de stratégie d'arrêt automatique - 1](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-1.png)

### <a name="user-sets-a-schedule-and-cannot-opt-out"></a>L'utilisateur définit une planification qu'il ne peut pas refuser
Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire peuvent remplacer la planification du laboratoire. Ils ne peuvent cependant pas renoncer à la stratégie d’arrêt automatique. Cette option permet de s’assurer que chaque machine de votre laboratoire fait l’objet d’une planification d’arrêt automatique. Les utilisateurs du laboratoire peuvent mettre à jour la planification d’arrêt automatique de leurs machines virtuelles et définir des notifications d’arrêt.

![Option de stratégie d'arrêt automatique - 2](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-2.png)

### <a name="user-has-no-control-over-the-schedule-set-by-lab-admin"></a>L'utilisateur n'a aucun contrôle sur la planification définie par l'administrateur du laboratoire
Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire ne peuvent ni remplacer ni refuser la planification du laboratoire. Cette option offre à l’administrateur du laboratoire le contrôle total sur la planification de chaque machine du laboratoire. Les utilisateurs du laboratoire ne peuvent configurer de notifications d’arrêt automatique que pour leurs machines virtuelles.

![Option de stratégie d'arrêt automatique - 3](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-3.png)

## <a name="set-autostart"></a>Définir le démarrage automatique
La stratégie de démarrage automatique vous permet de spécifier quand les machines virtuelles du laboratoire actuel doivent être démarrées.  

1. Dans le volet **Configuration et stratégies** du laboratoire, sélectionnez **Démarrage automatique**.
   
    ![La capture d’écran montre les options de démarrage automatique pour une machine virtuelle.](./media/devtest-lab-set-lab-policy/auto-start.png)

2. Sélectionnez **Activer** ou **Désactiver** pour activer ou désactiver cette stratégie.

3. Si vous activez cette stratégie, spécifiez l’heure de démarrage programmée, le fuseau horaire et les jours de la semaine auxquels cette heure s’applique. 

4. Sélectionnez **Enregistrer**.

Une fois activée, cette stratégie n’est pas automatiquement appliquée à toutes les machines virtuelles dans le laboratoire en cours. Pour appliquer ce paramètre à une machine virtuelle spécifique, ouvrez le volet de gestion de la machine virtuelle et modifiez son paramètre **Démarrage automatique**.

## <a name="set-expiration-date"></a>Définir la date d’expiration
Lorsque vous [créez la machine virtuelle](devtest-lab-add-vm.md), vous pouvez définir une date d’expiration. Dans **Paramètres avancés**, choisissez l’icône de calendrier pour spécifier la date à laquelle la machine virtuelle est automatiquement supprimée. Par défaut, la machine virtuelle n’arrive jamais à expiration.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez défini et appliqué les différents paramètres de stratégies des machines virtuelles pour votre laboratoire, voici ce que vous pouvez essayer de faire :

* [Comprendre les adresses IP partagées](devtest-lab-shared-ip.md) : explique comment les adresses IP partagées sont utilisées dans DevTest Labs pour réduire le nombre d’adresses IP publiques requises pour se connecter aux machines virtuelles de votre laboratoire.
* [Configurer la gestion des coûts](devtest-lab-configure-cost-management.md) : montre comment utiliser le graphique **Tendance des coûts mensuels estimés**  
  pour afficher le coût estimé à ce jour pour le mois en cours et le coût projeté pour la fin du mois.
* [Créer une image personnalisée](devtest-lab-create-template.md) : quand vous créez une machine virtuelle, vous spécifiez une base, qui peut être soit une image personnalisée, soit une image Marketplace. Cet article explique comment créer une image personnalisée à partir d’un fichier VHD.
* [Configurer des images Marketplace](devtest-lab-configure-marketplace-images.md) : Azure DevTest Labs prend en charge la création de machines virtuelles basées sur des images Azure Marketplace. Cet article explique comment spécifier, le cas échéant, les images Azure Marketplace pouvant être utilisées lors de la création de machines virtuelles dans un laboratoire.
* [Créer une machine virtuelle dans un laboratoire](devtest-lab-add-vm.md) : montre comment créer une machine virtuelle à partir d’une image de base (personnalisée ou Place de marché) et comment utiliser des artefacts dans votre machine virtuelle.
