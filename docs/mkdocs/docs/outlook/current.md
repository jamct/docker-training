# Aktuelles: Rate-Limiting im Hub

Anfang November, parallel zum Workshop, hat Docker eine Änderung am Docker Hub in Kraft gesetzt. Seitdem werden lesende Zugriffe auf den Hub beschränkt (per API-Rate-Limiting). Das bedeutet: Ab 100 Pull-Versuchen in 6 Stunden von einer IP erreichen Sie schon das Limit. Das ist ein Problem, wenn Sie `watchtower` verwenden. Gezählt werden nämlich auch Requests, bei denen nichts heruntergeladen wird, weil die Checksumme schon bekannt ist! Watchtower übergeben Sie einfach eine ENV-Variable:

```
WATCHTOWER_POLL_INTERVAL = 1800
```

Standardwert ist 300 (Sekunden). Damit alle Container auf Ihrem Host versorgt werden, ist also bei der Wahl des Intervalls etwas Dreisatz nötig.

## Limit erhöhen

Das Limit kann man erhöhen. Dafür reicht schon ein kostenlose Account und ein passendes Token. Wie Sie das mit Watchtower mitsenden, steht in der [Doku](https://containrrr.dev/watchtower/private-registries/). Als angemeldeter Benutzer liegt das Limit bei 200 Anfragen in 6 Stunden.

Wenn Sie das Limit ganz loswerden wollen, zahlen Sie 5 US-Dollar für einen Pro-Account bei Docker.
