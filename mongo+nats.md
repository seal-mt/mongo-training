# Architecture

![Architecure](./architecture3.png)

## Mongo Config

### Cache einschränken

Auf Rechnern mit wenig RAM kann der MongoDB Cache eingeschränkt werden. Erfahrungsgemäß sollten es **mindestens* 2GB sein.

```yaml
storage:
  wiredTiger:
    engineConfig:
        cacheSizeGB: 2
```

### TLS CA Zertifikate

Wenn ein CA Zertifikat konfiguriert werden muss.

```yaml
net:
  tls:
    CAFile: xxx.pem
```

Dann muss das System Zertifikate entfernt werden.

```yaml
setParameter:
#  tlsUseSystemCA: true
```

## Mongo Probleme

### Platte voll

Lösungsschritte:

- SEAL Software inkl. MongoDB auf allen Rechnern im Cluster runterfahren

- Platte vergrößern lassen

- Zuerst die MongoDB Services wieder hochfahren und warten bis "rs.status()" anzeigt, dass alle Server synchronisiert sind

- Danach erst SEAL Software hochfahren

### Eine MongoDB im Cluster funktioniert nicht mehr

Nach längerem Ausfall eines MongoDB Service kann es passieren, dass er sich nach dem Hochfahren nicht mehr synchronisieren kann. Hier hilft nur eine vollständige Neusynchronisierung.

Lösungsschritte:

- SEAL Software auf allen Rechnern im Cluster runterfahren

- Betroffenen MongoDB Service herunterfahren

- Alle Dateien im MongoDB Datenverzeichnis (Linux: /opt/seal/data/seal-mongodb, Windows: c:\ProgrammData\SEAL Systems\data\seal-mongo) löschen. **Achtung:** nicht das Verzeichnis selbst löschen.

### TLS Zertifikat funktioniert nicht

Generell bei Fehlern mit Kundenzertifikaten prüfen:

- Liegen Zertifikat und Key im PEM Format vor. Andere Formate werden nicht akzeptiert.

- Wurden Zertifikat und Key korrekt in eine gemeinsame Datei kopiert. Die Struktur muss wie folgt aussehen, wobei die Reihenfolge von Zertifikat & Key egal ist:
  ```
  -----BEGIN CERTIFICATE-----
  <Base64 kodiertes Zertifikat>
  -----END CERTIFICATE-----
  -----BEGIN PRIVATE KEY-----
  <Base64 kodierter Key>
  -----END PRIVATE KEY-----
  ```

- Gibt es ein CA Zertifikat und wurde es korrekt konfiguriert, siehe [TLS CA Zertifikate](#tls-ca-zertifikate)

Die Fehlermeldungen im MongoDB Logfile sind in der Regel aussagekräftig und weisen sofort in die richtige Richtung, siehe Beispiele unten.

Beispiele:

"The use of TLS without specifying a chain of trust is no longer supported": es fehlt ein CA Zertifikat, siehe [TLS CA Zertifikate](#tls-ca-zertifikate).

"The use of both a CA File and the System Certificate store is not supported.":  "CAFile" und "tlsUseSystemCA" sind vorhanden, siehe [TLS CA Zertifikate](#tls-ca-zertifikate).

"Can not set up PEM key file.": die Datei auf die "certificateKeyFile" verweist ist nicht korrekt. Hier gib tes mehrere Möglichkeiten. Entweder die Datei enthält nur das Zertifikat oder nur den Key, oder Zertifikat/Key ist generell kaputt.


### ReplicaSet nicht initialisiert

Wenn ein "rs.status()" die Meldung "MongoServerError: no replset config has been received" liefert, wurde "rs.initiate()" vergessen.


## Mongo Kommandos

### MongoDB Shell Aufruf

Windows:
```powershell
& "C:\Program Files\mongosh\mongosh.exe" --tls --tlsAllowInvalidCertificates
```
Linux:
```bash
mongosh --tls --tlsAllowInvalidCertificates
```

### Batch Kommandos generell

```
mongosh --eval „<Kommando>“
```

### ReplicaSet prüfen

```
rs.status()
```

Vollständige Zeile: 

```bash
mongosh --tls --tlsAllowInvalidCertificates --eval "rs.status()"
```


## NATS Probleme

KEINE!

Aber: ein seal-co-notifier Problem den NATS KV Store korrekt zu benutzen. Ist mit seal-co-notifier 5.12.
