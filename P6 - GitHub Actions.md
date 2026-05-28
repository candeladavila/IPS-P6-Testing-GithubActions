# Practica 6

## GitHub Actions

### BÁSICOS:
- Los ficheros que definen workflows tienen extensión .yml
- Los ficheros de workflow están en el directorio .github/workflows/

La estructura del proyecto tiene que ser así:
```
Practica6Alumnos/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
├── pom.xml
├── mvnw
└── README.md
```

## Estructura básica workflow CI
```yaml
name: CI

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build-test:
    runs-on: ubuntu-latest

    steps:
      - name: Descargar codigo
        uses: actions/checkout@v4

      - name: Configurar Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Dar permisos a Maven Wrapper
        run: chmod +x ./mvnw

      - name: Compilar proyecto
        run: ./mvnw compile --no-transfer-progress

      - name: Ejecutar tests
        run: ./mvnw verify --no-transfer-progress
```