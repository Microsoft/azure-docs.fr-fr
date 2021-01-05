---
title: Configurer des environnements intermédiaires
description: Découvrez comment déployer des applications à un emplacement hors production et comment les faire passer automatiquement en production. Améliorez la fiabilité et éliminez les temps d’arrêt des applications à partir des déploiements.
ms.assetid: e224fc4f-800d-469a-8d6a-72bcde612450
ms.topic: article
ms.date: 04/30/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: ef90603e8c8cdd66d43b9f88f6d128d8a472fd8a
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150336"
---
# <a name="set-up-staging-environments-in-azure-app-service"></a>Configurer des environnements intermédiaires dans Azure App Service
<a name="Overview"></a>

Lorsque vous déployez votre application web, votre application web Linux, votre backend mobile ou votre API dans [Azure App Service](./overview.md), vous pouvez utiliser un autre emplacement de déploiement que l’emplacement de production par défaut lorsque vous exécutez le niveau de plan **Standard**, **Premium** ou **Isolé** d’App Service. Les emplacements de déploiement sont des applications en production pourvues de leur propre nom d’hôte. Les éléments de contenu et de configuration des applications peuvent être échangés entre deux emplacements de déploiement, y compris l’emplacement de production. 

Le déploiement de votre application sur un emplacement hors production présente les avantages suivants :

* Vous pouvez valider les modifications d’une application dans un emplacement de déploiement intermédiaire avant de l’échanger avec l’emplacement de production.
* Déployer d’abord une application dans un emplacement et la basculer ensuite en production permet de vous assurer que toutes les instances de l’emplacement sont initialisées avant d’être basculées en production. Cela permet d’éliminer les temps d’arrêt lors du déploiement de l’application. La redirection du trafic est transparente et aucune requête n’est abandonnée du fait d’opérations de permutation. Vous pouvez automatiser l’intégralité de ce workflow en configurant l’[échange automatique](#Auto-Swap) lorsqu’aucune validation n’est nécessaire avant l’échange.
* Après basculement, la précédente application de production se retrouve dans l’emplacement de l’application précédemment intermédiaire. Si les modifications permutées en production ne vous conviennent pas, vous pouvez effectuer la même permutation afin de récupérer immédiatement le contenu du précédent site qui vous plaisait.

Chaque niveau de plan App Service prend en charge un nombre différent d’emplacements de déploiement. L’utilisation de ces emplacements de déploiement n’entraîne aucun coût supplémentaire. Pour connaître le nombre d’emplacements pris en charge par le plan de votre application, consultez [Limites App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits). 

Pour mettre votre application à l’échelle vers un autre niveau, vérifiez que le niveau cible peut prendre en charge le nombre d’emplacements déjà utilisés par votre application. Par exemple, si votre application comprend plus de cinq emplacements, vous ne pouvez pas effectuer de scale-down vers le niveau **Standard**, car le niveau **Standard** ne prend en charge que cinq emplacements de déploiement. 

<a name="Add"></a>

## <a name="add-a-slot"></a>Ajouter un emplacement
Pour que vous puissiez activer plusieurs emplacements de déploiement, l’application doit s’exécuter au niveau **Standard**, **Premium** ou **Isolé**.


1. dans le [portail Azure](https://portal.azure.com/), recherchez et sélectionnez **App Services**, puis sélectionnez votre application. 
   
    ![Rechercher App Services](./media/web-sites-staged-publishing/search-for-app-services.png)
   

2. Dans le volet gauche, sélectionnez **Emplacements de déploiement** > **Ajouter un emplacement**.
   
    ![Add a new deployment slot](./media/web-sites-staged-publishing/QGAddNewDeploymentSlot.png)
   
   > [!NOTE]
   > Si l’application ne s’exécute pas au niveau **Standard**, **Premium** ou **Isolé**, vous recevez un message précisant les niveaux pris en charge pour la publication intermédiaire. À ce stade, vous pouvez sélectionner **Mettre à niveau** et accéder à l’onglet **Mettre à l’échelle** de votre application avant de continuer.
   > 

3. Dans la boîte de dialogue **Ajouter un emplacement**, nommez l’emplacement, puis indiquez si vous souhaitez cloner une configuration d’application à partir d’un autre emplacement de déploiement. Sélectionnez **Ajouter** pour continuer.
   
    ![Source de configuration](./media/web-sites-staged-publishing/ConfigurationSource1.png)
   
    Vous pouvez cloner une configuration à partir d’un emplacement existant. Parmi les paramètres pouvant être clonés figurent des paramètres de l’application, des chaînes de connexion, des versions du framework de langage, des sockets web, la version HTTP et le nombre de bits de la plateforme.

4. Une fois l’emplacement ajouté, sélectionnez **Fermer** pour fermer la boîte de dialogue. Le nouvel emplacement est désormais affiché dans la page **Emplacements de déploiement**. Par défaut, l’option **% de trafic** est définie sur 0 pour le nouvel emplacement, avec tout le trafic client acheminé vers l’emplacement de production.

5. Sélectionnez le nouvel emplacement de déploiement pour ouvrir la page des ressources de cet emplacement.
   
    ![Titre de l’emplacement de déploiement](./media/web-sites-staged-publishing/StagingTitle.png)

    L’emplacement intermédiaire dispose d’une page de gestion, comme n’importe quelle application App Service. Vous pouvez modifier la configuration de l’emplacement. Pour vous rappeler que vous voyez l’emplacement de déploiement, le nom de l’application apparaît sous la forme **\<app-name>/\<slot-name>** , et le type d’application est **App Service (Emplacement)** . Vous pouvez également afficher l’emplacement sous la forme d’une application distincte dans votre groupe de ressources, avec les mêmes désignations.

6. Sélectionnez l’URL de l’application dans la page des ressources de l’emplacement. L’emplacement de déploiement est une application en production et il a son propre nom d’hôte. Pour limiter l’accès public à l’emplacement de déploiement, consultez [Restrictions d’adresse IP avec Azure App Service](app-service-ip-restrictions.md).

Le nouvel emplacement de déploiement n’a aucun contenu, même si vous clonez les paramètres à partir d’un autre emplacement. Par exemple, vous pouvez [publier sur cet emplacement avec Git](./deploy-local-git.md). Vous pouvez effectuer un déploiement sur l’emplacement à partir d’une autre branche du dépôt, ou d’un dépôt différent.

<a name="AboutConfiguration"></a>

## <a name="what-happens-during-a-swap"></a>Que se passe-t-il pendant un échange ?

### <a name="swap-operation-steps"></a>Étapes qui composent une opération d’échange

Lorsque vous échangez deux emplacements (en général, vous passez d’un emplacement de préproduction à un emplacement de production), App Service effectue ce qui suit pour éviter que l’emplacement cible ne subisse un temps d’arrêt :

1. Applique les paramètres suivants de l’emplacement cible (par exemple, l’emplacement de production) à toutes les instances de l’emplacement source : 
    - Les paramètres d’application et les chaînes de connexion [propres aux emplacements](#which-settings-are-swapped) (si nécessaire)
    - Les paramètres de [déploiement continu](deploy-continuous-deployment.md) (si activés)
    - Les paramètres relatifs à l’[authentification App Service](overview-authentication-authorization.md) (si activés)
    
    Quel que soit le cas, cela déclenche le redémarrage de toutes les instances dans l’emplacement source. Au cours d’un [échange avec aperçu](#Multi-Phase), cela marque la fin de la première phase. L’opération d’échange est suspendue, ce qui vous permet de vérifier que l’emplacement source fonctionne correctement avec les paramètres de l’emplacement cible.

1. Attendez que chaque instance de l’emplacement source ait terminé son redémarrage. Si l’une des instances ne parvient pas à redémarrer, l’opération d’échange rétablit toutes les modifications apportées à l’emplacement source, puis cesse d’être exécutée.

1. Si le [cache local](overview-local-cache.md) est activé, déclenchez son initialisation en envoyant une requête HTTP à la racine de l’application (« / »), sur chaque instance de l’emplacement source. Attendez que chaque instance retourne une réponse HTTP. L’initialisation du cache local provoque un autre redémarrage sur chaque instance.

1. Si l’[échange automatique](#Auto-Swap) est activé avec l’[initialisation personnalisée](#Warm-up), déclenchez l’[initialisation de l’application](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) en envoyant une requête HTTP à la racine de l’application (« / ») sur chaque instance de l’emplacement source.

    Si `applicationInitialization` n’est pas spécifié, déclenchez une requête HTTP à la racine de l’application de l’emplacement source sur chaque instance. 
    
    Si une instance retourne une réponse HTTP, elle est considérée comme initialisée.

1. Si toutes les instances de l’emplacement source sont correctement initialisées, échangez les deux emplacements en échangeant leurs règles de routage. Après cette étape, l’emplacement cible (par exemple, l’emplacement de production) dispose de l’application déjà initialisée dans l’emplacement source.

1. Maintenant que l’emplacement source dispose de l’application de l’emplacement cible avant échange, effectuez la même opération en appliquant tous les paramètres et en redémarrant les instances.

Lors d’une opération d’échange, l’intégralité du processus d’initialisation des applications échangées se déroule dans l’emplacement source. L’emplacement cible reste en ligne pendant la préparation et l’initialisation de l’emplacement source, que l’échange ait réussi ou échoué. Pour échanger un emplacement de préproduction avec un emplacement de production, vérifiez que l’emplacement de production est toujours l’emplacement cible. De cette façon, l’opération d’échange n’affectera pas votre application de production.

### <a name="which-settings-are-swapped"></a>Quels sont les paramètres échangés ?

[!INCLUDE [app-service-deployment-slots-settings](../../includes/app-service-deployment-slots-settings.md)]

Si vous souhaitez configurer un paramètre d’application ou une chaîne de connexion pour qu’ils restent dans leur emplacement d’origine (et ne puissent donc pas être échangés), accédez à la page **Configuration** de l’emplacement en question. Ajoutez ou modifiez un paramètre, puis sélectionnez **Paramètre d’emplacement de déploiement**. Le fait de cocher cette case indique à App Service que le paramètre n’est pas échangeable. 

![Paramètre de l’emplacement](./media/web-sites-staged-publishing/SlotSetting.png)

<a name="Swap"></a>

## <a name="swap-two-slots"></a>Permuter deux emplacements 
Vous pouvez échanger des emplacements de déploiement dans la page **Emplacements de déploiement** et la page **Vue d’ensemble** de votre application. Pour obtenir des détails techniques sur l’échange des emplacements, consultez [Que se passe-t-il pendant un échange ?](#AboutConfiguration).

> [!IMPORTANT]
> Avant de basculer une application de l’emplacement de déploiement vers l’emplacement de production, vérifiez que l’emplacement de production est bien votre emplacement cible, et que tous les paramètres de l’emplacement source sont configurés comme vous le souhaitez dans l’emplacement de production.
> 
> 

Pour échanger des emplacements de déploiement :

1. Accédez à la page **Emplacements de déploiement** de votre application, puis sélectionnez **Échanger**.
   
    ![Bouton Échanger](./media/web-sites-staged-publishing/SwapButtonBar.png)

    La boîte de dialogue **Échanger** regroupe les paramètres des emplacements source et cible sélectionnés qui vont être échangés.

2. Sélectionnez les emplacements **Source** et **Cible** souhaités. En général, la cible correspond à l’emplacement de production. Sélectionnez également les onglets **Modifications sources** et **Modifications cibles** pour vérifier que les changements de configuration à opérer correspondent à ce que vous attendez. Lorsque vous avez terminé, vous pouvez échanger les emplacements immédiatement en sélectionnant **Échanger**.

    ![Échange effectué](./media/web-sites-staged-publishing/SwapImmediately.png)

    Pour voir comment votre emplacement cible s’exécuterait avec les nouveaux paramètres avant que l’échange ne soit réellement effectué, ne sélectionnez pas **Échanger**, mais suivez les instructions fournies dans [Échange avec aperçu](#Multi-Phase).

3. Lorsque vous avez terminé, fermez la boîte de dialogue en sélectionnant **Fermer**.

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés aux échanges](#troubleshoot-swaps).

<a name="Multi-Phase"></a>

### <a name="swap-with-preview-multi-phase-swap"></a>Échange avec aperçu (échange multiphase)

Avant de passer à l’emplacement de production (emplacement cible), contrôlez l’exécution de l’application avec les paramètres échangés. L’emplacement source est également initialisé avant la fin de l’échange, ce qui est souhaitable pour les applications stratégiques.

Lorsque vous effectuez un échange avec aperçu, App Service effectue la même [opération d’échange](#AboutConfiguration), mais il fait une pause après la première étape. Vous pouvez donc vérifier le résultat sur l’emplacement de préproduction avant la fin de l’échange. 

Si vous annulez l’échange, App Service réapplique les éléments de configuration à l’emplacement source.

Pour effectuer un échange avec aperçu :

1. Procédez comme indiqué dans [Échanger des emplacements de déploiement](#Swap), mais sélectionnez **Effectuer l’échange avec aperçu**.

    ![Échange avec aperçu](./media/web-sites-staged-publishing/SwapWithPreview.png)

    La boîte de dialogue montre comment la configuration de l’emplacement source est modifiée dans la phase 1, et comment les emplacements source et cible sont modifiés dans la phase 2.

2. Lorsque vous êtes prêt à démarrer l’échange, sélectionnez **Démarrer l’échange**.

    Lorsque la phase 1 est terminée, vous en êtes averti dans la boîte de dialogue. Affichez l’aperçu de l’échange dans l’emplacement source en accédant à `https://<app_name>-<source-slot-name>.azurewebsites.net`. 

3. Lorsque vous êtes prêt à effectuer l’échange en attente, sélectionnez **Terminer l’échange** dans **Action d’échange**, puis sélectionnez **Terminer l’échange**.

    Pour annuler un échange en attente, sélectionnez plutôt **Annuler l’échange**.

4. Lorsque vous avez terminé, fermez la boîte de dialogue en sélectionnant **Fermer**.

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés aux échanges](#troubleshoot-swaps).

Pour automatiser un échange multiphase, consultez [Automatiser avec PowerShell](#automate-with-powershell).

<a name="Rollback"></a>

## <a name="roll-back-a-swap"></a>Restaurer un échange
Si des erreurs se produisent dans l’emplacement cible (par exemple, l’emplacement de production) après une permutation d’emplacements, rétablissez ces deux emplacements comme ils étaient avant l’opération, en les intervertissant immédiatement.

<a name="Auto-Swap"></a>

## <a name="configure-auto-swap"></a>Configurer l’échange automatique

> [!NOTE]
> L’échange automatique n’est pas pris en charge dans les applications web sur Linux.

L’échange automatique simplifie les scénarios Azure DevOps impliquant un déploiement de l’application en continu, sans démarrage à froid ni temps d’arrêt pour les utilisateurs de l’application. Lorsque vous activez l’échange automatique d’un emplacement vers l’emplacement de production, chaque fois que vous envoyez (push) des modifications de votre code à cet emplacement, App Service [fait basculer automatiquement l’application vers la production](#swap-operation-steps) après son initialisation dans l’emplacement source.

   > [!NOTE]
   > Avant de configurer l’échange automatique pour l’emplacement de production, il est conseillé de tester l’échange automatique sur un emplacement cible hors production.
   > 

Pour configurer l’échange automatique :

1. Accédez à la page des ressources de votre application. Sélectionnez **Emplacements de déploiement** >  *\<desired source slot>*  > **Configuration** > **Paramètres généraux**.
   
2. Pour **Échange automatique activé**, sélectionnez **Activé**. Ensuite, sélectionnez l’emplacement cible souhaité pour **Échanger automatiquement l’emplacement de déploiement**, puis sélectionnez **Enregistrer** dans la barre de commandes. 
   
    ![Sélections pour la configuration de l’échange automatique](./media/web-sites-staged-publishing/AutoSwap02.png)

3. Exécutez un push de code sur l’emplacement source. L’échange automatique se produit peu après, et la mise à jour est appliquée dans l’URL de votre emplacement cible.

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés aux échanges](#troubleshoot-swaps).

<a name="Warm-up"></a>

## <a name="specify-custom-warm-up"></a>Spécifier l’initialisation personnalisée

Certaines applications peuvent nécessiter quelques actions préparatoires personnalisées avant l’échange. L’élément de configuration `applicationInitialization` du fichier web.config vous permet de spécifier les actions d’initialisation personnalisées à exécuter. L’[opération d’échange](#AboutConfiguration) attend la fin de l’initialisation personnalisée pour procéder à l’échange avec l’emplacement cible. Voici un exemple de fragment web.config.

```xml
<system.webServer>
    <applicationInitialization>
        <add initializationPage="/" hostName="[app hostname]" />
        <add initializationPage="/Home/About" hostName="[app hostname]" />
    </applicationInitialization>
</system.webServer>
```

Pour plus d’informations sur la personnalisation de l’élément `applicationInitialization`, consultez [Most common deployment slot swap failures and how to fix them](https://ruslany.net/2017/11/most-common-deployment-slot-swap-failures-and-how-to-fix-them/).

Vous pouvez également personnaliser le comportement d’initialisation en utilisant l’un des deux [paramètres d’application](configure-common.md) suivants (ou les deux) :

- `WEBSITE_SWAP_WARMUP_PING_PATH`: chemin permettant d’effectuer un test ping afin d’initialiser votre site. Ajoutez ce paramètre d’application en spécifiant un chemin d’accès personnalisé qui commence par une barre oblique comme valeur. par exemple `/statuscheck`. La valeur par défaut est `/`. 
- `WEBSITE_SWAP_WARMUP_PING_STATUSES`: Codes de réponse HTTP valides pour l’opération d'initialisation. Ajoutez ce paramètre d’application avec une liste séparée par des virgules de codes HTTP. Par exemple `200,202`. Si le code d’état retourné ne figure pas dans la liste, les opérations d’initialisation et d’échange sont arrêtées. Par défaut, tous les codes de réponse sont valides.

> [!NOTE]
> L’élément de configuration `<applicationInitialization>` fait partie de chaque démarrage d’application, tandis que les deux paramètres d’application de comportement de préchauffage s’appliquent uniquement aux échanges d’emplacements.

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés aux échanges](#troubleshoot-swaps).

## <a name="monitor-a-swap"></a>Superviser un échange

Si l’exécution de l’[opération d’échange](#AboutConfiguration) prend beaucoup de temps, vous pouvez obtenir des informations au sujet de cette opération dans le [journal d’activité](../azure-monitor/platform/platform-logs-overview.md).

Sur le portail, dans la page des ressources de votre application, sélectionnez **Journal d’activité** dans le volet de gauche.

Une opération d’échange s’affiche dans la requête de journal en tant que `Swap Web App Slots`. Vous pouvez la développer et sélectionner l’une des sous-opérations ou erreurs afin d’afficher le contenu en détail.

## <a name="route-traffic"></a>Acheminer le trafic

Par défaut, toutes les requêtes clientes vers les URL de production de l’application (`http://<app_name>.azurewebsites.net`) sont acheminées vers l’emplacement de production. Vous pouvez acheminer une partie du trafic vers un autre emplacement. Cette fonctionnalité est utile si vous avez besoin d’un retour d’expérience utilisateur pour une nouvelle mise à jour, mais que vous n’êtes pas prêt à la publier en production.

### <a name="route-production-traffic-automatically"></a>Acheminer le trafic de production automatiquement

Pour router le trafic de production automatiquement :

1. Accédez à la page des ressources de votre application et sélectionnez **Emplacements de déploiement**.

2. Dans la colonne **% de trafic** de l’emplacement vers lequel vous souhaitez acheminer le trafic, spécifiez un pourcentage (compris entre 0 et 100) pour représenter la quantité totale de trafic à diriger. Sélectionnez **Enregistrer**.

    ![Définition d’un pourcentage de trafic](./media/web-sites-staged-publishing/RouteTraffic.png)

Une fois le paramètre enregistré, le pourcentage de clients spécifié est routé de manière aléatoire vers l’emplacement hors production. 

Lorsqu’un client est automatiquement routé vers un emplacement particulier, il est « épinglé » à cet emplacement pendant toute la durée de cette session cliente. Dans le navigateur client, vous pouvez voir à quel emplacement votre session est épinglée en examinant le cookie `x-ms-routing-name` dans les en-têtes HTTP. Une requête qui est acheminée vers l’emplacement « intermédiaire » contient le cookie `x-ms-routing-name=staging`. Une requête qui est acheminée vers l’emplacement de production a le cookie `x-ms-routing-name=self`.

   > [!NOTE]
   > À côté du portail Azure, vous pouvez également utiliser la commande [`az webapp traffic-routing set`](/cli/azure/webapp/traffic-routing#az-webapp-traffic-routing-set) dans Azure CLI pour définir les pourcentages de routage à partir d’outils CI/CD comme les pipelines DevOps ou d’autres systèmes d’automatisation.
   > 

### <a name="route-production-traffic-manually"></a>Acheminer le trafic de production manuellement

Parallèlement au routage automatique du trafic, App Service peut acheminer les requêtes vers un emplacement particulier. Cela s’avère utile si vous souhaitez que vos utilisateurs puissent choisir d’accepter ou de refuser votre application bêta. Pour router le trafic de production manuellement, vous utilisez le paramètre de requête `x-ms-routing-name`.

Par exemple, pour permettre aux utilisateurs de refuser votre application bêta, vous pouvez placer ce lien dans votre page web :

```html
<a href="<webappname>.azurewebsites.net/?x-ms-routing-name=self">Go back to production app</a>
```

La chaîne `x-ms-routing-name=self` spécifie l’emplacement de production. Lorsque le navigateur client accède au lien, il est redirigé vers l’emplacement de production. Chaque requête ultérieure comprendra le cookie `x-ms-routing-name=self` qui épinglera la session à l’emplacement de production.

Pour permettre aux utilisateurs d’accepter votre application bêta, définissez le même paramètre de requête pour le nom de l’emplacement hors production. Voici un exemple :

```
<webappname>.azurewebsites.net/?x-ms-routing-name=staging
```

Par défaut, les nouveaux emplacements se voient attribuer une règle de routage de `0%` (indiquée en gris). Lorsque vous définissez cette valeur explicitement sur `0%` (indiquée en noir), vos utilisateurs peuvent accéder à l’emplacement de préproduction manuellement, à l’aide du paramètre de requête `x-ms-routing-name`. Toutefois, ils ne seront pas routés vers l’emplacement automatiquement, car le pourcentage de routage est défini sur 0. Il s’agit d’un scénario avancé où vous pouvez « cacher » votre emplacement de préproduction du public, tout en permettant aux équipes internes de tester les modifications sur l’emplacement.

<a name="Delete"></a>

## <a name="delete-a-slot"></a>Supprimer un emplacement

Recherchez et sélectionnez votre application. Sélectionnez **Emplacements de déploiement** >  *\<slot to delete>*  > **Vue d’ensemble**. Le type d’application est indiqué comme **App Service (Emplacement)** pour vous rappeler que vous consultez un emplacement de déploiement. Sélectionnez **Supprimer** dans la barre de commandes.  

![Supprimer un emplacement de déploiement](./media/web-sites-staged-publishing/DeleteStagingSiteButton.png)

<!-- ======== AZURE POWERSHELL CMDLETS =========== -->

<a name="PowerShell"></a>

## <a name="automate-with-powershell"></a>Automatiser avec PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Azure PowerShell est un module qui fournit des applets de commande pour gérer Azure via Windows PowerShell, notamment la prise en charge de la gestion des emplacements de déploiement des applications dans Azure App Service.

Pour plus d’informations sur l’installation et la configuration d’Azure PowerShell et sur l’authentification d’Azure PowerShell avec votre abonnement Azure, consultez la page [Installation et configuration d’Azure PowerShell](/powershell/azure/).  

---
### <a name="create-a-web-app"></a>Créer une application web
```powershell
New-AzWebApp -ResourceGroupName [resource group name] -Name [app name] -Location [location] -AppServicePlan [app service plan name]
```

---
### <a name="create-a-slot"></a>Créer un emplacement
```powershell
New-AzWebAppSlot -ResourceGroupName [resource group name] -Name [app name] -Slot [deployment slot name] -AppServicePlan [app service plan name]
```

---
### <a name="initiate-a-swap-with-a-preview-multi-phase-swap-and-apply-destination-slot-configuration-to-the-source-slot"></a>Démarrer un échange avec aperçu (échange multiphase) et appliquer la configuration de l’emplacement de destination à l’emplacement source
```powershell
$ParametersObject = @{targetSlot  = "[slot name – e.g. "production"]"}
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action applySlotConfig -Parameters $ParametersObject -ApiVersion 2015-07-01
```

---
### <a name="cancel-a-pending-swap-swap-with-review-and-restore-the-source-slot-configuration"></a>Annuler un échange en attente (échange avec aperçu) et restaurer la configuration de l’emplacement source
```powershell
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action resetSlotConfig -ApiVersion 2015-07-01
```

---
### <a name="swap-deployment-slots"></a>Échanger des emplacements de déploiement
```powershell
$ParametersObject = @{targetSlot  = "[slot name – e.g. "production"]"}
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action slotsswap -Parameters $ParametersObject -ApiVersion 2015-07-01
```

### <a name="monitor-swap-events-in-the-activity-log"></a>Superviser les événements d’échange dans le journal d’activité
```powershell
Get-AzLog -ResourceGroup [resource group name] -StartTime 2018-03-07 -Caller SlotSwapJobProcessor  
```

---
### <a name="delete-a-slot"></a>Supprimer un emplacement
```powershell
Remove-AzResource -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots –Name [app name]/[slot name] -ApiVersion 2015-07-01
```

## <a name="automate-with-resource-manager-templates"></a>Automatiser avec des modèles Resource Manager

Les [modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md) sont des fichiers JSON déclaratifs utilisés pour automatiser le déploiement et la configuration de ressources Azure. Pour échanger des emplacements à l’aide de modèles Resource Manager, vous devez définir deux propriétés sur les ressources *Microsoft.Web/sites/slots* et *Microsoft.Web/sites* :

- `buildVersion` : propriété de type chaîne qui représente la version actuelle de l’application déployée dans l’emplacement. Par exemple : « v1 », « 1.0.0.1 » ou « 2019-09-20T11:53:25.2887393-07:00 ».
- `targetBuildVersion` : propriété de type chaîne spécifiant la valeur `buildVersion` que l’emplacement doit avoir. Si la valeur targetBuildVersion n’est pas la valeur `buildVersion` actuelle, cela déclenche l’opération d’échange en recherchant l’emplacement qui a la valeur `buildVersion` spécifiée.

### <a name="example-resource-manager-template"></a>Exemple de modèle Resource Manager

Le modèle Resource Manager suivant met à jour la valeur `buildVersion` de l’emplacement de préproduction et définit la valeur `targetBuildVersion` sur l’emplacement de production. Cela a pour effet d’échanger les deux emplacements. Le modèle suppose que vous avez déjà créé un application web avec un emplacement nommé « préproduction ».

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "my_site_name": {
            "defaultValue": "SwapAPIDemo",
            "type": "String"
        },
        "sites_buildVersion": {
            "defaultValue": "v1",
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Web/sites/slots",
            "apiVersion": "2018-02-01",
            "name": "[concat(parameters('my_site_name'), '/staging')]",
            "location": "East US",
            "kind": "app",
            "properties": {
                "buildVersion": "[parameters('sites_buildVersion')]"
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2018-02-01",
            "name": "[parameters('my_site_name')]",
            "location": "East US",
            "kind": "app",
            "dependsOn": [
                "[resourceId('Microsoft.Web/sites/slots', parameters('my_site_name'), 'staging')]"
            ],
            "properties": {
                "targetBuildVersion": "[parameters('sites_buildVersion')]"
            }
        }        
    ]
}
```

Ce modèle Resource Manager est idempotent, ce qui signifie qu’il peut être exécuté à plusieurs reprises et produire le même état des emplacements. Après la première exécution, la valeur `targetBuildVersion` correspond à la valeur `buildVersion` actuelle, de sorte qu’aucun échange n’est déclenché.

<!-- ======== Azure CLI =========== -->

<a name="CLI"></a>

## <a name="automate-with-the-cli"></a>Automatiser avec l’interface CLI

Pour connaître les commandes de l’interface [Azure CLI](https://github.com/Azure/azure-cli) pour les emplacements de déploiement, consultez [az webapp deployment slot](/cli/azure/webapp/deployment/slot).

## <a name="troubleshoot-swaps"></a>Résoudre les problèmes liés aux échanges

Si une erreur se produit pendant un [échange d’emplacement](#AboutConfiguration), celle-ci est journalisée dans *D:\home\LogFiles\eventlog.xml*. Elle est également journalisée dans le journal des erreurs propres à l’application.

Voici quelques erreurs courantes liées aux échanges :

- Une requête HTTP envoyée à la racine de l’application a expiré. L’opération d’échange attend 90 secondes pour chaque requête HTTP, et retente jusqu’à 5 fois. Si toutes les nouvelles tentatives expirent elles aussi, l’opération d’échange est arrêtée.

- L’initialisation du cache local peut échouer si le contenu de l’application dépasse le quota de disque local spécifié. Pour plus d’informations, consultez [Vue d’ensemble du cache local](overview-local-cache.md).

- Pendant l’[initialisation personnalisée](#Warm-up), les requêtes HTTP sont effectuées en interne (sans passer par l’URL externe). Elles peuvent échouer avec certaines règles de réécriture d’URL dans *Web.config*. Par exemple, les règles de redirection des noms de domaine ou d’application du protocole HTTPS peuvent empêcher les requêtes d’initialisation d’atteindre le code de l’application. Pour contourner ce problème, modifiez vos règles de réécriture en ajoutant les deux conditions suivantes :

    ```xml
    <conditions>
      <add input="{WARMUP_REQUEST}" pattern="1" negate="true" />
      <add input="{REMOTE_ADDR}" pattern="^100?\." negate="true" />
      ...
    </conditions>
    ```
- Sans une initialisation personnalisée, les règles de réécriture d’URL peuvent encore bloquer les requêtes HTTP. Pour contourner ce problème, modifiez vos règles de réécriture en ajoutant la condition suivante :

    ```xml
    <conditions>
      <add input="{REMOTE_ADDR}" pattern="^100?\." negate="true" />
      ...
    </conditions>
    ```

- Après des échanges d’emplacements, l’application peut rencontrer des redémarrages inattendus. En effet, après un échange, la configuration de la liaison du nom d’hôte se désynchronise, ce qui n’entraîne pas de redémarrages. En revanche, certains événements de stockage sous-jacents (comme des basculements de volume de stockage) peuvent détecter ces différences et forcer le redémarrage de tous les processus Worker. Pour minimiser ces types de redémarrages, définissez le [paramètre d’application`WEBSITE_ADD_SITENAME_BINDINGS_IN_APPHOST_CONFIG=1`](https://github.com/projectkudu/kudu/wiki/Configurable-settings#disable-the-generation-of-bindings-in-applicationhostconfig) sur *tous les emplacements*. En revanche, ce paramètre d’application ne fonctionne *pas* avec des applications Windows Communication Foundation (WCF).

## <a name="next-steps"></a>Étapes suivantes
[Bloquer l’accès à des emplacements hors production](app-service-ip-restrictions.md)