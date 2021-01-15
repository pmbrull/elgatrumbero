date: 2021-01-15 15:42:00
title: Azure SAS EventHub
category: azure
----

Quan tenim un EHUB dins d'un `namespace` al Event Hub i hem d'enviar missatges - per exemple des del Postman - ens cal un header d'autorització tal que `SharedAccessSignature sr=...`.

## 1. Crear Shared Access Policy

Dins del propi EHUB ens hem de crear una pòliça amb els mínims permisos necessaris. Per exemple, `send` per fer les proves del postman (i si escoltem amb una App Function, necessitarem una de `listen`).

Ens guardem la `Primary key`.

## 2. Crear la SAS

El mètode més senzill és amb una funció de bash:

```bash
get_sas_token() {
    local EVENTHUB_URI=$1
    local SHARED_ACCESS_KEY_NAME=$2
    local SHARED_ACCESS_KEY=$3
    local EXPIRY=${EXPIRY:=$((60 * 60 * 24 * 365 * 2))} # Default token expiry is 1 day -> set 2 years

    local ENCODED_URI=$(echo -n $EVENTHUB_URI | jq -s -R -r @uri)
    local TTL=$(($(date +%s) + $EXPIRY))
    local UTF8_SIGNATURE=$(printf "%s\n%s" $ENCODED_URI $TTL | iconv -t utf8)

    local HASH=$(echo -n "$UTF8_SIGNATURE" | openssl sha256 -hmac $SHARED_ACCESS_KEY -binary | base64)
    local ENCODED_HASH=$(echo -n $HASH | jq -s -R -r @uri)

    echo -n "SharedAccessSignature sr=$ENCODED_URI&sig=$ENCODED_HASH&se=$TTL&skn=$SHARED_ACCESS_KEY_NAME"
}
```

i la cridem com:

```bash
get_sas_token "https://<EHUBS>.servicebus.windows.net" "<nom-pòliça>" "<Primary Key>"
```

## 3. POST header

Un cop tenim la SAS l'hem de ficar al header del POST quan enviem missatges a la cua: `{Authorization: <SAS>}`.
