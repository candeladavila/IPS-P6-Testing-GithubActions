# Practica 6

## Antes de empezar

### Tareas que se cumplen: 
1. Se activa al hacer push o pull request.
2. Descarga el código del repositorio.
3. Configura Java.
4. Da permisos al Maven Wrapper.
5. Compila el proyecto.
6. Ejecuta los tests con Maven.
7. Genera informes de tests.
8. Genera badges de cobertura con JaCoCo.
9. Muestra los badges en el README.



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

### Ampliar workflow para que pueda escribir los badges en el repo
```yaml
name: CI

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

permissions:
  contents: write
  checks: write
  pull-requests: write
```

### Crear informe de los test 
Añadir como otro step dentro del job. 

Nota: Al poner if: always() implica que el informe  se va a crear siempre (aunque no pasen algunos tests)
```yaml
      - name: Publicar reporte de tests
        uses: ctrf-io/github-test-reporter@v1
        if: always()
        with:
          report-path: './target/*-reports/*.xml'
          github-report: true
          integrations-config: |
            {
              "junit-to-ctrf": {
                "enabled": true,
                "action": "convert",
                "options": {
                  "output": "./ctrf-reports/ctrf-report.json",
                  "toolname": "junit-to-ctrf"
                }
              }
            }
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Generar Badge de cobertura con JaCoCo
Nota: antes de seguir es necesario que en el fichero pom.xml está el siguiente plugin: 
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Añadir otro step dentro del job: 
```yaml
      - name: Generar badges de cobertura
        uses: cicirello/jacoco-badge-generator@v2
        if: always()
        with:
          jacoco-csv-file: target/site/jacoco/jacoco.csv
          badges-directory: .github/badges
          generate-branches-badge: true
          generate-summary: true
```

### Guardar los Badges dentro del repositorio
Este paso lo que hace es generar los ficheros (badges que después añadiremos):
- .github/badges/jacoco.svg
- .github/badges/branches.svg

Añadir este step al job: 
```yaml
      - name: Guardar badges en el repositorio
        uses: EndBug/add-and-commit@v9
        if: github.event_name == 'push'
        with:
          add: '.github/badges'
          message: 'Update coverage badges'
```

## Añadir badges al README.md

### Badge de estado
```md
![CI](https://github.com/NOMBRE_USUARIO_GITHUB/NOMBRE_REPO_GITHUB/actions/workflows/ci.yml/badge.svg)
```

### Badge de cobertura
```md
![Coverage](.github/badges/jacoco.svg)
![Branches](.github/badges/branches.svg)
```

