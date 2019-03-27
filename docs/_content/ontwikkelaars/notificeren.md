---
title: "Notificeren"
date: '27-03-2019'
weight: 110
---

Als component moet je events publiceren naar een notificatiecomponent (NC), of
je wil kunnen acteren op events. Deze guide helpt je op weg.

Als uitgangspunt wordt de [gehoste notificatiecomponent](https://ref.tst.vng.cloud/nc/)
gebruikt.

Er zijn twee perspectieven:

* [Notificaties publiceren](#ik-wil-als-bron-notificaties-publiceren)
* [Notificaties ontvangen](#ik-wil-als-consumer-notificaties-ontvangen)

## Ik wil als bron notificaties publiceren

Je dient de scope `notificaties.scopes.publiceren` in het JWT te hebben
voor deze acties. Je kan de [tokentool][token-generator] gebruiken om een
JWT te genereren.

1. Bepaal de naam van het kanaal. Voor het ZRC bijvoorbeeld is dit `zaken`.

2. Zorg dat het kanaal bekend is bij het NC. Je kan dit controleren door een
   query te doen:

   ```http
   GET https://ref.tst.vng.cloud/nc/api/v1/kanaal?naam=zaken
   ```

   Indien een lege lijst terugkomt, dan bestaat het kanaal nog niet. Indien het
   kanaal wel al bestaat, ga dan naar stap 4.

3. Registreer het kanaal (indien het nog niet bestaat)

    ```http
    POST https://ref.tst.vng.cloud/nc/api/v1/kanaal

    Authorization: Bearer abcd1234
    Content-Type: application/json

    {
      "naam": "zaken",
      "documentatieLink": "https://ref.tst.vng.cloud/zrc/ref/kanalen/#zaken",
      "filters": [
        "bronorganisatie",
        "zaaktype",
        "vertrouwelijkheidaanduiding"
      ]
    }
    ```

    De documentatielink hoort te documenteren welke kenmerken relevant zijn
    voor een kanaal, en welke events gepubliceerd worden. Dit helpt consumers
    om te bepalen waarop ze willen abonneren.

4. Verstuur een bericht

    ```http
    POST https://ref.tst.vng.cloud/nc/api/v1/notificaties

    Authorization: Bearer abcd1234
    Content-Type: application/json

    {
      "kanaal": "zaken",
      "hoofdObject": "https://ref.tst.vng.cloud/zrc/api/v1/zaken/ddc6d192",
      "resource": "status",
      "resourceUrl": "https://ref.tst.vng.cloud/zrc/api/v1/statussen/44fdcebf",
      "actie": "create",
      "aanmaakdatum": "2019-03-27T10:59:13Z",
      "kenmerken": [
        {"bronorganisatie": "224557609"},
        {"zaaktype": "https://ref.tst.vng.cloud/ztc/api/v1/catalogussen/39732928/zaaktypen/53c5c164"},
        {"vertrouwelijkheidaanduiding": "openbaar"}
      ]
    }
    ```

## Ik wil als consumer notificaties ontvangen

Je dient de scope `notificaties.scopes.consumeren` in het JWT te hebben
voor deze acties. Je kan de [tokentool][token-generator] gebruiken om een
JWT te genereren.

1. Bouw een endpoint om notificaties op te ontvangen. Het formaat van berichten
   staat in de API spec en in stap 4 van
   [Notificaties publiceren](#ik-wil-als-bron-notificaties-publiceren). Voor
   het vervolg nemen we aan dat dit beschikbaar is op `https://my-consumer.nl/api/callbacks`

2. Vraag op welke kanalen beschikbaar zijn:

    ```http
    GET https://ref.tst.vng.cloud/nc/api/v1/kanaal

    Authorization: Bearer abcd1234
    ````

3. Registreer je abonnement bij het NC:

    ```http
    POST https://ref.tst.vng.cloud/nc/api/v1/abonnement

    Authorization: Bearer abcd1234
    Content-Type: application/json

    {
      "callbackUrl": "https://my-consumer.nl/api/callbacks",
      "auth": "Token abcde12345",
      "kanalen": [
        {
          "naam": "zaken",
          "filters": [
            {"bronorganisatie": "224557609"}
          ]
        }
      ]
    }
    ```

    * `callbackUrl` is de volledige URL naar je _eigen_ endpoint waar je
      notificaties wenst op te ontvangen

    * `auth` is de waarde van de `Authorization` header om je _eigen_ endpoint
      te kunnen benaderen. Deze waarde wordt gebruikt door het NC om berichten
      af te leveren.

    * `kanalen` is een lijst van kanalen waarop je wenst te abonneren, met de
      relevante filters. De beschikbare kenmerken waarop gefilterd kan worden
      horen gedocumenteerd te zijn op de kanalen die opgevraagd zijn in stap 2.

    * `filters` zijn optioneel. Indien je een filter weglaat, dan geldt dit als
      wildcard.

4. Berichten worden nu naar je eigen endpoint gestuurd met een POST request

    ```http
    POST https://my-consumer.nl/api/callbacks

    Content-Type: application/json
    Authorization: Token abcde12345

    {
      "kanaal": "zaken",
      "hoofdObject": "https://ref.tst.vng.cloud/zrc/api/v1/zaken/ddc6d192",
      "resource": "status",
      "resourceUrl": "https://ref.tst.vng.cloud/zrc/api/v1/statussen/44fdcebf",
      "actie": "create",
      "aanmaakdatum": "2019-03-27T10:59:13Z",
      "kenmerken": [
        {"bronorganisatie": "224557609"},
        {"zaaktype": "https://ref.tst.vng.cloud/ztc/api/v1/catalogussen/39732928/zaaktypen/53c5c164"},
        {"vertrouwelijkheidaanduiding": "openbaar"}
      ]
    }
    ```

[token-generator]: https://ref.tst.vng.cloud/tokens/