# Diplomado en Aplicaciones Asistidas por IA

This repository contains the documentation site for the *Diplomado en Aplicaciones Asistidas por IA*, built using MkDocs with the Material theme.

## Prerequisites

To set up and run the documentation site, you need the following:

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or [Anaconda](https://www.anaconda.com/products/distribution) installed on your system.
- A terminal or command-line interface.
- The `environment.yml` file (included in this repository) to create the Conda environment.

## Installation Instructions

Follow these steps to set up the Conda environment and serve the MkDocs site locally:

1. **Clone the Repository** (if applicable):
   ```bash
   git clone git@github.com:introSoft-iud/Diplomado_IA_IUDigital.git
   cd Diplomado_IA_IUDigital


### Crear el Entorno Conda

Usa el archivo `environment.yml` proporcionado para crear un entorno Conda llamado `mkdocs-diplomado`:

```bash
    conda env create -f environment.yml
```

### Activar el Entorno Conda

Activa el entorno para utilizar las dependencias instaladas:

```bash
conda activate mkdocs-diplomado
```
### Servir el Sitio MkDocs Localmente

Ejecuta el servidor de desarrollo de MkDocs para visualizar el sitio localmente:

```bash
mkdocs serve
```
Abre tu navegador y navega a `http://127.0.0.1:8000` para ver el sitio.

El sitio se recargará automáticamente cuando realices cambios en los archivos fuente.

### Generar el Sitio (Opcional)

Para generar los archivos estáticos del sitio (por ejemplo, para su despliegue):

```bash
mkdocs build
```

### Sitio Desplegado

El sitio ha sido desplegado y está disponible en la siguiente URL:

[https://introsoft-iud.github.io/Diplomado_IA_IUDigital/Unidad%201/](https://introsoft-iud.github.io/Diplomado_IA_IUDigital/Unidad%201/)

<!-- Para desplegar uso 

mkdocs gh-deploy --remote-branch master   

Ensure GitHub Pages is set to use the master branch and / (root) in Settings > Pages.
   -->