# ESPHome Door

Configuratiebron voor twee Sonoff SV-boards: een elektrische pengrendel (`bolt-lock`) en een tijdelijke deuropener (`front-door`). Gericht op ESPHome 2026.10.1. De werking moet op de echte installatie worden getest; deze repository bevat geen kant-en-klare firmware.

## Overnemen zonder API-sleutel overtypen

Deze configuraties zijn de **bron na adoptie**, niet de eerste firmware voor een oud board. Voor de eerste migratie is apart voorbereide overdrachtsfirmware nodig: keyless API-encryptie, een koppelvenster van tien minuten, eigen hotspot- en webwachtwoorden en de bestaande OTA-toegangscode. Upload deze alleen tijdens de overdracht, op een vertrouwd netwerk zonder de oude HA-koppeling.

1. Gebruik Home Assistant 2026.10 of nieuwer en een actuele ESPHome Device Builder-app onder dezelfde HA Supervisor. De automatische sleuteloverdracht naar een losse Device Builder-server is niet gegarandeerd.
2. Voeg de privé ontvangen web- en hotspotwachtwoorden samen met de bestaande `secrets.yaml` van Device Builder. De sleutelnamen staan in `secrets.example.yaml`. Wis geen bestaande secrets. Dit bestand bevat geen API-key; die wordt automatisch ingesteld.
3. Verbind de boards via hun beveiligde fallback-hotspot met de eigen 2,4 GHz-wifi.
4. Voeg ieder apparaat **eerst in Home Assistant** toe. HA stelt een nieuwe API-encryptiesleutel in en deelt deze via Supervisor met Device Builder.
5. Open Device Builder en kies **Adopt / Take control** bij het ontdekte apparaat. Behoud de naam `bolt-lock` respectievelijk `front-door`. De door HA ingestelde sleutel moet automatisch in de lokale configuratie verschijnen. Laat geen nieuwe, andere sleutel genereren.
6. Controleer vóór installeren dat er lokaal een `api.encryption.key` staat, dat de configuratie valideert en dat de OTA-instellingen `encryption:` gebruiken. Breek af als een sleutel ontbreekt of om jouw oude sleutel wordt gevraagd; controleer versies en sleuteloverdracht.
7. Compileer en installeer de eigen firmware. De OTA-verbinding gebruikt dezelfde sleutel; het oude OTA-wachtwoord vervalt in deze firmware. Test bediening en een volgende OTA-verbinding.

De imports verwijzen naar `@codex/v0.2.0`: een vastgehouden versiebranch. Wijzig die branch niet nadat devices ermee zijn overgedragen; maak voor wijzigingen een nieuwe versiebranch. Een lokale package-import downloadt geen nieuwe firmware uit zichzelf; een update vereist compileren en installeren.

## Bediening

- `bolt-lock`: relais uit = softwarematig ontgrendeld. Volgens eigenaar ontgrendelt de pengrendel ook zonder voeding. Het weekschema staat aanvankelijk uit, is instelbaar via de webinterface en gebruikt SNTP met Europe/Amsterdam. Het schema bedient alleen de pengrendel en vergrendelt niet opnieuw.
- `front-door`: ontgrendelen bekrachtigt het relais gedurende 1–25 seconden (aanvankelijk 25). Herhaalde opdrachten tijdens de puls verlengen hem niet. Vergrendelen breekt de puls direct af. Er is geen deursensorafhankelijkheid.
- Beide: GPIO12 stuurt het relais. Na herstart is het relais uit; dit moet met de echte bedrading gecontroleerd worden. De getoonde status is een commandostatus, geen gemeten fysieke slotstand.

Elke lokale webinterface gebruikt Digest-authenticatie en ingebouwde webassets. Handmatige bediening vereist geen HA of internet. De web- en hotspotwachtwoorden zijn apart van de API-key en moeten privé worden overgedragen. Ze horen nooit in deze repository. Stel de apparaten niet via portforwarding beschikbaar op internet.

## Beveiliging en herstel

De afzonderlijke overdrachtsfirmware heeft zonder ingestelde API-key een eerste-koppelperiode van tien minuten na opstarten. De eerste succesvolle koppeling bepaalt de eigenaar van de API-key. Gebruik dit uitsluitend op het vertrouwde netwerk van de bewoners; laat de vorige HA-installatie niet als eerste koppelen. Na succesvolle instelling blijft de API-key behouden na stroomuitval. Een reboot wist de sleutel niet. Bij verloren sleutels kan fysiek herstel nodig zijn; publiceer geen factory-resetknop op de bewonerspagina.

Als geen HA wordt gebruikt, blijft lokale webbediening met de overdrachtsfirmware werken. Na het ongebruikte koppelvenster sluiten de oningerichte API en fallback-AP; herstart heropent de inrichting. Zodra API en wifi ingericht zijn, wordt de normale fallback bij wifi-uitval behouden. Controleer dit op locatie.

De publieke YAML bevat verwijzingen naar lokale web- en hotspotsecrets, maar geen echte credentials. Er wordt bewust niet met publieke standaardwachtwoorden gewerkt. Zonder het privébestand met die wachtwoorden compileert de import niet. Wifi-secrets komen uit Device Builder; de API-key uit de HA-overdracht. `secrets.example.yaml` bevat uitsluitend placeholders.

## Bronnen

- https://esphome.io/guides/creators/
- https://esphome.io/components/provisioning/
- https://esphome.io/components/ota/esphome/
- https://github.com/esphome/device-builder/blob/main/docs/ARCHITECTURE.md#ha-encryption-key-handoff

## Status

Concept 0.2.0. Beide overdrachtsfirmwares en beide lokaal samengevoegde adoptieconfiguraties compileren met ESPHome 2026.10.1. De HA-handoff is daarbij gesimuleerd. Op 2 november 2026 zijn beide echte GitHub-package-imports via `codex/v0.2.0` met tijdelijke testcredentials opgehaald en succesvol gevalideerd. De echte sleuteloverdracht via HA en Supervisor is nog niet getest. Koppeling met echte HA, flashen en elektrische praktijktests zijn nog nodig. De publieke configuratie is bewust alleen compileerbaar nadat de overgenomen API-key lokaal is toegevoegd: versleutelde OTA kan zonder die key niet worden ingesteld. Het tijdelijke koppelvenster is uitsluitend aanwezig in de afzonderlijke overdrachtsfirmware.
