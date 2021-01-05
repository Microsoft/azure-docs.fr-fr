---
title: Écrire du code pour effectuer le suivi des requêtes avec Azure Application Insights | Microsoft Docs
description: Écrivez du code pour effectuer le suivi des requêtes avec Application Insights afin d’obtenir des profils pour vos requêtes.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.custom: devx-track-csharp
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: aaa1d6df9faa20b1a561bfccdfea682af7645c18
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88930245"
---
# <a name="write-code-to-track-requests-with-application-insights"></a>Écrire du code pour effectuer le suivi des requêtes avec Application Insights

Pour afficher les profils pour votre application dans la page Performances, Azure Application Insights doit effectuer le suivi des requêtes pour votre application. Application Insights peut effectuer automatiquement le suivi des requêtes pour les applications qui reposent sur des frameworks déjà instrumentés. ASP.NET et ASP.NET Core en sont deux exemples. 

Pour d’autres applications telles que les rôles de travail Azure Cloud Service et les API sans état Service Fabric, vous devez écrire du code pour indiquer à Application Insights où vos requêtes commencent et se terminent. Une fois ce code écrit, les données de télémétrie relatives aux requêtes sont envoyées à Application Insights. Vous pouvez voir les données de télémétrie dans la page Performances, et les profils sont collectés pour ces requêtes. 

Pour suivre manuellement les requêtes, procédez comme suit :

  1. Ajoutez le code suivant au début de la durée de vie de l’application :  

        ```csharp
        using Microsoft.ApplicationInsights.Extensibility;
        ...
        // Replace with your own Application Insights instrumentation key.
        TelemetryConfiguration.Active.InstrumentationKey = "00000000-0000-0000-0000-000000000000";
        ```
      Pour plus d’informations sur cette configuration de clé d’instrumentation générale, consultez [Use Service Fabric with Application Insights](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/blob/dev/appinsights/ApplicationInsights.md) (Utiliser Service Fabric avec Application Insights).  

  1. Pour tout bloc de code que vous souhaitez instrumenter, encadrez-le d'une instruction `StartOperation<RequestTelemetry>` **using**, comme indiqué dans l'exemple suivant :

        ```csharp
        using Microsoft.ApplicationInsights;
        using Microsoft.ApplicationInsights.DataContracts;
        ...
        var client = new TelemetryClient();
        ...
        using (var operation = client.StartOperation<RequestTelemetry>("Insert_Your_Custom_Event_Unique_Name"))
        {
          // ... Code I want to profile.
        }
        ```

        L’appel de `StartOperation<RequestTelemetry>` dans une autre étendue `StartOperation<RequestTelemetry>` n’est pas pris en charge. Vous pouvez utiliser `StartOperation<DependencyTelemetry>` dans l’étendue imbriquée à la place. Par exemple :  
        
        ```csharp
        using (var getDetailsOperation = client.StartOperation<RequestTelemetry>("GetProductDetails"))
        {
        try
        {
          ProductDetail details = new ProductDetail() { Id = productId };
          getDetailsOperation.Telemetry.Properties["ProductId"] = productId.ToString();
        
          // By using DependencyTelemetry, 'GetProductPrice' is correctly linked as part of the 'GetProductDetails' request.
          using (var getPriceOperation = client.StartOperation<DependencyTelemetry>("GetProductPrice"))
          {
              double price = await _priceDataBase.GetAsync(productId);
              if (IsTooCheap(price))
              {
                  throw new PriceTooLowException(productId);
              }
              details.Price = price;
          }
        
          // Similarly, note how 'GetProductReviews' doesn't establish another RequestTelemetry.
          using (var getReviewsOperation = client.StartOperation<DependencyTelemetry>("GetProductReviews"))
          {
              details.Reviews = await _reviewDataBase.GetAsync(productId);
          }
        
          getDetailsOperation.Telemetry.Success = true;
          return details;
        }
        catch(Exception ex)
        {
          getDetailsOperation.Telemetry.Success = false;
        
          // This exception gets linked to the 'GetProductDetails' request telemetry.
          client.TrackException(ex);
          throw;
        }
        }
        ```
