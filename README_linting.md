# API Linting - Einfache CLI Commands

Quick-Start für API Linting mit **Zally** und **Spectral**.

---

## Zally (Zalando Guidelines) - Docker

### ⚠️ Wichtig: Kein fertiges Docker Image!

Zalando stellt **keine fertigen Docker Images** bereit. Du hast 3 Optionen:

### Option 1: Mit Docker Compose (empfohlen für Zally)

```bash
# Zally Repository klonen
git clone https://github.com/zalando/zally.git
cd zally

# Mit Docker Compose starten
docker-compose up

# Server läuft dann auf http://localhost:8000
# Web-UI auf http://localhost:8080
```

**Dann testen:**
```bash
# Via curl
curl -X POST http://localhost:8000/api-violations \
  -H "Content-Type: application/yaml" \
  --data-binary @openapi.yaml

# Oder über Web-UI
# Browser öffnen: http://localhost:8080
```

### Option 2: Nur Spectral nutzen (einfachster Weg! ✅)

Da Zally kompliziert zu installieren ist, **empfehle ich dir Spectral mit dem Zalando Ruleset**:

```bash
# Installation
npm install -g @stoplight/spectral-cli

# Direkt mit Zalando Rules linten
spectral lint openapi.yaml \
  --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml
```

**Das ist viel einfacher und deckt die meisten Zalando Guidelines ab!**

### Option 3: Zally selbst bauen

```bash
# Repository klonen
git clone https://github.com/zalando/zally.git
cd zally/server

# Mit Gradle bauen
./gradlew bootRun

# Server läuft auf http://localhost:8080
```

---

## Spectral mit Zalando Ruleset

### Installation
```bash
# Via npm
npm install -g @stoplight/spectral-cli

# Oder via Docker
docker pull stoplight/spectral
```

### Option 1: Fertiges Zalando Ruleset (empfohlen!)

Es gibt ein **fertiges Zalando Spectral Ruleset**, das alle Zalando Guidelines implementiert!

```yaml
# .spectral.yaml - Einfach das Zalando Ruleset verwenden
extends:
  - https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml

# Optional: OpenAPI Standard-Rules hinzufügen
# extends:
#   - https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml
#   - spectral:oas
```

**Verwendung:**
```bash
# Direkt linten
spectral lint openapi.yaml

# Oder direkt mit URL (ohne .spectral.yaml Datei)
spectral lint openapi.yaml \
  --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml
```

### Option 2: Zalando Ruleset + Eigene Custom Rules

Wenn du das Zalando Ruleset erweitern willst:

```yaml
# .spectral.yaml - Zalando + eigene Rules
extends:
  - https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml

rules:
  # === EIGENE CUSTOM RULES ===
  
  # Custom Rule 1: Alle Responses brauchen Beispiele
  response-examples-required:
    description: "Alle Response Schemas brauchen Beispiele"
    message: "Response '{{property}}' benötigt ein 'example' Feld"
    severity: warn
    given: "$.paths[*][*].responses[*].content[*].schema"
    then:
      field: example
      function: truthy

  # Custom Rule 2: Beschreibung muss mindestens 50 Zeichen haben
  description-min-length:
    description: "API-Beschreibung muss mindestens 50 Zeichen haben"
    message: "Beschreibung zu kurz (min. 50 Zeichen): '{{value}}'"
    severity: error
    given: "$.info.description"
    then:
      function: length
      functionOptions:
        min: 50
```

### Option 3: Eigene Zalando-basierte Rules (wenn du nur einzelne brauchst)

### Option 3: Eigene Zalando-basierte Rules (wenn du nur einzelne brauchst)

Falls du nur einzelne Zalando Rules manuell definieren willst:

```yaml
extends: spectral:oas

rules:
  # Zalando Guidelines
  
  # 1. Kebab-case für Pfade
  path-kebab-case:
    description: "Pfade müssen kebab-case verwenden"
    message: "Pfad '{{property}}' muss kebab-case sein (z.B. /user-profile statt /userProfile)"
    severity: error
    given: "$.paths[*]~"
    then:
      function: pattern
      functionOptions:
        match: "^(\\/|[a-z0-9-.]+|\\{[a-z0-9-]+\\})+$"

  # 2. Plural für Ressourcen
  path-plural-resources:
    description: "Ressourcen-Pfade sollten Plural sein"
    message: "Ressource '{{value}}' sollte im Plural sein"
    severity: warn
    given: "$.paths.*~"
    then:
      function: pattern
      functionOptions:
        match: "^\\/([a-z0-9-]+s|\\{[a-z0-9-]+\\})"

  # 3. HTTP-Methoden müssen korrekt sein
  operation-success-response:
    description: "Jede Operation braucht einen 2xx Response"
    message: "Operation muss mindestens einen 2xx Response haben"
    severity: error
    given: "$.paths[*][*].responses"
    then:
      function: schema
      functionOptions:
        schema:
          type: object
          anyOf:
            - required: ["200"]
            - required: ["201"]
            - required: ["202"]
            - required: ["204"]

  # 4. API muss Version haben
  info-version:
    description: "API muss eine Version haben"
    message: "Version fehlt in info.version"
    severity: error
    given: "$.info"
    then:
      field: version
      function: truthy

  # 5. Alle Operationen brauchen operationId
  operation-operationId:
    description: "Alle Operationen brauchen eine operationId"
    message: "operationId fehlt"
    severity: error
    given: "$.paths[*][*]"
    then:
      field: operationId
      function: truthy
```

### Eigene Custom Rules

Füge deine 2 eigenen Rules hinzu:

```yaml
# .spectral.yaml - mit eigenen Rules
extends: spectral:oas

rules:
  # === ZALANDO RULES ===
  path-kebab-case:
    description: "Pfade müssen kebab-case verwenden"
    message: "Pfad '{{property}}' muss kebab-case sein"
    severity: error
    given: "$.paths[*]~"
    then:
      function: pattern
      functionOptions:
        match: "^(\\/|[a-z0-9-.]+|\\{[a-z0-9-]+\\})+$"

  path-plural-resources:
    description: "Ressourcen-Pfade sollten Plural sein"
    message: "Ressource '{{value}}' sollte im Plural sein"
    severity: warn
    given: "$.paths.*~"
    then:
      function: pattern
      functionOptions:
        match: "^\\/([a-z0-9-]+s|\\{[a-z0-9-]+\\})"

  # === EIGENE RULES ===
  
  # Custom Rule 1: Alle Responses brauchen Beispiele
  response-examples-required:
    description: "Alle Response Schemas brauchen Beispiele"
    message: "Response '{{property}}' benötigt ein 'example' Feld"
    severity: warn
    given: "$.paths[*][*].responses[*].content[*].schema"
    then:
      field: example
      function: truthy

  # Custom Rule 2: Beschreibung muss mindestens 50 Zeichen haben
  description-min-length:
    description: "API-Beschreibung muss mindestens 50 Zeichen haben"
    message: "Beschreibung zu kurz (min. 50 Zeichen): '{{value}}'"
    severity: error
    given: "$.info.description"
    then:
      function: length
      functionOptions:
        min: 50
```

### Spectral Verwendung

```bash
# Option 1: Mit fertigem Zalando Ruleset (empfohlen!)
spectral lint openapi.yaml \
  --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml

# Option 2: Mit eigenem .spectral.yaml
spectral lint openapi.yaml

# Option 3: Docker
docker run --rm -v $(pwd):/work \
  stoplight/spectral lint /work/openapi.yaml

# Bash-Funktion für Zalando Rules
zalando-spec() {
  spectral lint "$1" \
    --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml
}

# Bash-Funktion für eigene Rules
spec() {
  spectral lint "$1" --ruleset .spectral.yaml
}

# Dann:
zalando-spec openapi.yaml  # Mit Zalando Ruleset
spec openapi.yaml          # Mit eigenem Ruleset
```

---

## Beide Tools kombinieren

```bash
#!/bin/bash
# lint.sh

echo "🔍 Zally (Original Zalando Tool)..."
docker run --rm -v $(pwd):/specs \
  ghcr.io/zalando/zally:latest \
  lint /specs/openapi.yaml

echo -e "\n🔍 Spectral (Zalando Ruleset)..."
spectral lint openapi.yaml \
  --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml
```

```bash
chmod +x lint.sh
./lint.sh
```

---

## Quick Commands

```bash
# Zally (Original)
docker run --rm -v $(pwd):/specs ghcr.io/zalando/zally:latest lint /specs/openapi.yaml

# Spectral mit Zalando Ruleset
spectral lint openapi.yaml \
  --ruleset https://raw.githubusercontent.com/baloise-incubator/spectral-ruleset/main/zalando.yml

# Beide
./lint.sh
```

---

## Was ist der Unterschied?

**Zally** = Offizielles Zalando Tool (Java-basiert, Docker)
- ✅ Direkt von Zalando
- ✅ Immer aktuell mit Guidelines
- ❌ Braucht Docker
- ❌ Langsamer

**Spectral mit Zalando Ruleset** = Community Implementation
- ✅ Schneller
- ✅ Einfacher zu integrieren
- ✅ Mehr Flexibilität für eigene Rules
- ❌ Community-maintained (nicht von Zalando selbst)
- ❌ Könnte leicht hinter den Guidelines zurückliegen

**Empfehlung:** Nutze **beide** für maximale Sicherheit!
