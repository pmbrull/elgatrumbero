title: Azure App Function & Key Vault
date: 2021-01-20 08:15:00
categories: 
  - Cloud
  - Azure
----

En general ens interessa vincular una Azure App Function amb un Key Vault per a extreure informació **sensible** de manera **segura**. Per sort, Azure ofereix diferents pràctiques que ens faciliten la vida a l'hora de desenvolupar la solució.

## Managed Identity

Podem activar una [Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) a la nostra App Function de manera que es crea de forma automàtica una entrada al **Azure AD** que identifica al nostre servei. Si la creem com a `System Managed Identity`, llavors l'entrada al AD està vinculada al cicle de vida del servei sense que s'hagi d'efectuar cap altra acció al damunt.

Gràcies a això, podem crear una pòliça d'accés al secrets (`GET`) de la instància del Key Vault per l'`ObjectId` generat, otorgant permisos sobre el pla de dades a la App Function de manera transparent.

## Key Vault References

El següent pas és escollir quins secrets necessitem per a la aplicació. La manera més senzilla és a través de [Key Vault References](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references), que ens permeten crear el lligam directament a la **configuració** de la App Function amb una sintaxi concreta:

```
@Microsoft.KeyVault(SecretUri=<URL del secret>)
```

## Private Endpoints

Tot i això, si parlem en termes de seguretat, en general ens interessa **limitar** l'accés del públic a la App Function. Un [Private Endpoint](https://docs.microsoft.com/en-us/azure/app-service/networking/private-endpoint) serveix justament per això, de manera que des de llavors ja solament es podrà accedir a la App Function des de dins de la VNet.

## Conseqüències

Una de les principals conseqüències (o limitacions) dels PE és que aleshores les referències al Key Vault fallen. Per tant hem de canviar l'estratègia i en comptes de poder accedir al KV directament per configuració, cal aplicar canvis al codi fent servir una [llibreria de Python](https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-python?tabs=cmd).

Cal dir que és bastant senzill d'usar.
