---
title: Résoudre les problèmes de connexion liés à l’agent Serveurs activés par Azure Arc
description: Cet article explique comment résoudre les problèmes liés à l’agent Connected Machine qui surviennent avec des Serveurs activés par Azure Arc (préversion) lors de la tentative de connexion au service.
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 36feb6a65ec52d99dfd664ae54cb099ea6a7e239
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90900675"
---
# <a name="troubleshoot-the-connected-machine-agent-connection-issues"></a>Résoudre les problèmes de connexion liés à l'agent Connected Machine

Cet article fournit des informations sur la résolution des problèmes qui peuvent survenir lors de la tentative de configuration de l’agent Connected Machine de Serveurs activés par Azure Arc pour Windows ou Linux. Les méthodes d'installation interactive et d'installation à grande échelle utilisables lors de la configuration de la connexion au service sont toutes deux incluses. Pour obtenir des informations d’ordre général, consultez [Présentation de Serveurs avec Arc](./overview.md).

## <a name="agent-verbose-log"></a>Journal d'activité détaillé de l'agent

Avant de suivre les étapes de résolution des problèmes décrites plus loin dans cet article, vous devez au minimum disposer du journal d'activité détaillé. Celui-ci contient la sortie des commandes de l'outil **azcmagent**, lorsque l'argument détaillé (-v) est utilisé. Les fichiers journaux sont enregistrés sous `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` pour Windows, et sous `/var/opt/azcmagent/log/azcmagent.log` pour Linux.

### <a name="windows"></a>Windows

L'exemple de commande suivant permet d'activer la journalisation détaillée avec l'agent Connected Machine pour Windows lors d'une installation interactive.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

L'exemple de commande suivant permet d'activer la journalisation détaillée avec l'agent Connected Machine pour Windows lors d'une installation à grande échelle à l'aide d'un principal de service.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
  --verbose
```

### <a name="linux"></a>Linux

L'exemple de commande suivant permet d'activer la journalisation détaillée avec l'agent Connected Machine pour Linux lors d'une installation interactive.

>[!NOTE]
>Vous devez disposer des autorisations d’accès *racine* sur les ordinateurs Linux pour exécuter **azcmagent**.

```bash
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

L'exemple de commande suivant permet d'activer la journalisation détaillée avec l'agent Connected Machine pour Linux lors d'une installation à grande échelle à l'aide d'un principal de service.

```bash
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
  --verbose
```

## <a name="agent-connection-issues-to-service"></a>Problèmes rencontrés par l'agent pour se connecter au service

Le tableau suivant répertorie certaines erreurs connues ainsi que des suggestions de résolution de celles-ci.

|Message |Error |Cause probable |Solution |
|--------|------|---------------|---------|
|Échec de l'acquisition du flux de périphérique du jeton d'autorisation |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` |Impossible de joindre le point de terminaison `login.windows.net` | Vérifiez la connectivité au point de terminaison. |
|Échec de l'acquisition du flux de périphérique du jeton d'autorisation |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. |Le proxy ou le pare-feu bloque l'accès au point de terminaison `login.windows.net`. | Vérifiez que la connectivité au point de terminaison n'est pas bloquée par un pare-feu ou par un serveur proxy. |
|Échec de l'acquisition du flux de périphérique du jeton d'autorisation  |`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp lookup login.windows.net: no such host`. | L’objet de stratégie de groupe (GPO) *Configuration ordinateur\Modèles d’administration\Système\Profils utilisateur\Supprimer les profils utilisateur datant de plus d’un certain nombre de jours lors du redémarrage du système* est activé. | Vérifiez que le GPO est activé et cible la machine concernée. Pour plus d’informations, consultez la note de bas de page <sup>[1](#footnote1)</sup>. |
|Échec de l'acquisition du jeton d'autorisation à partir du nom du principal de service |`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` |Le proxy ou le pare-feu bloque l'accès au point de terminaison `login.windows.net`. |Vérifiez que la connectivité au point de terminaison n'est pas bloquée par un pare-feu ou par un serveur proxy. |
|Échec de l'acquisition du jeton d'autorisation à partir du nom du principal de service |`Invalid client secret is provided` |Secret du principal de service incorrect ou non valide. |Vérifiez le secret du principal de service. |
| Échec de l'acquisition du jeton d'autorisation à partir du nom du principal de service |`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' was not found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` |Principal du service et/ou ID de locataire incorrects. |Vérifiez le principal de service et/ou l'ID du locataire.|
|Obtenir une réponse des ressources ARM |`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` |Informations d'identification et/ou autorisations incorrectes |Vérifiez que vous ou le principal de service êtes membre du rôle **Intégration d'Azure Connected Machine**. |
|Échec de la ressource ARM AzcmagentConnect |`The subscription is not registered to use namespace 'Microsoft.HybridCompute'` |Les fournisseurs de ressources Azure ne sont pas inscrits. |Inscrivez les [fournisseurs de ressources](./agent-overview.md#register-azure-resource-providers). |
|Échec de la ressource ARM AzcmagentConnect |`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` |Le serveur proxy ou le pare-feu bloque l'accès au point de terminaison `management.azure.com`. |Vérifiez que la connectivité au point de terminaison n'est pas bloquée par un pare-feu ou par un serveur proxy. |

<a name="footnote1"></a><sup>1</sup> Si ce GPO est activé et s’applique aux machines dotées de l’agent Connected Machine, il supprime le profil utilisateur associé au compte intégré spécifié pour le service *himds*. Par conséquent, il supprime également le certificat d’authentification utilisé pour communiquer avec le service mis en cache dans le magasin de certificats local pendant 30 jours. Avant la délai de 30 jours, une tentative de renouvellement du certificat est effectuée. Pour résoudre ce problème, suivez les étapes visant à [désinscrire la machine](manage-agent.md#unregister-machine), puis inscrivez-la de nouveau auprès du service qui exécute `azcmagent connect`.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas ici ou que vous ne pouvez pas le résoudre, utilisez un des canaux suivants pour obtenir une aide supplémentaire :

* Obtenez des réponses d'experts Azure par le biais de [Microsoft Q&A](/answers/topics/azure-arc.html).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Le Support Azure fournit à la communauté Azure des réponses, un support technique et des experts.

* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.
