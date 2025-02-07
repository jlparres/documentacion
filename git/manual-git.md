# Manual Básico de Git

## 1. ¿Qué es Git?

Git es un sistema de control de versiones distribuido que te permite llevar un registro de los cambios en archivos y coordinar el trabajo en equipo en el desarrollo de proyectos. Fue creado por Linus Torvalds y es muy popular en el mundo del desarrollo de software.

## 2. Instalación de Git

Antes de comenzar, asegúrate de tener Git instalado en tu sistema. Puedes descargarlo desde [git-scm.com](https://git-scm.com/).

- **En Windows:** Descarga el instalador y sigue las instrucciones.
- **En macOS:** Puedes instalarlo usando Homebrew con el comando:
    ```bash
    brew install git
    ```
- **En Linux:** Usualmente se instala desde el gestor de paquetes. Por ejemplo, en Ubuntu:
    ```bash
    sudo apt update
    sudo apt install git
    ```

## 3. Configuración Inicial

Después de instalar Git, es recomendable configurar tu nombre de usuario y correo electrónico, ya que estos datos se incluirán en tus commits.

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@ejemplo.com"
```

Puedes verificar la configuración con:

```bash
git config --list
```

## 4. Creación y Clonación de Repositorios

### 4.1. Crear un Nuevo Repositorio

Para iniciar un nuevo proyecto con Git, crea una carpeta y luego ejecuta:

```bash
mkdir mi-proyecto
cd mi-proyecto
git init
```

Este comando crea un nuevo repositorio en la carpeta actual.

### 4.2. Clonar un Repositorio Existente

Si quieres copiar un repositorio remoto (por ejemplo, de GitHub) en tu máquina, usa:

```bash
git clone https://github.com/usuario/nombre-repositorio.git
```

Este comando descargará el repositorio y creá una carpeta con el nombre del proyecto.

## 5. Flujo de Trabajo Básico

### 5.1. Agregar Archivos al Área de Staging

Cuando haces cambios en tus archivos, primero debes agregarlos al área de staging para prepararlos para un commit:

```bash
git add archivo.txt
```

Para agregar todos los archivos modificados:

```bash
git add .
```

### 5.2. Guardar Cambios con un Commit

Después de agregar los archivos al área de staging, guarda los cambios en el historial del repositorio con un commit:

```bash
git commit -m "Mensaje descriptivo del cambio"
```

### 5.3. Visualizar el Historial de Commits

Puedes ver el historial de cambios con:

```bash
git log
```

Para una vista más resumida, puedes usar:

```bash
git log --oneline
```

## 6. Trabajando con Ramas (Branches)

Las ramas te permiten trabajar en funcionalidades o correcciones de manera aislada sin afectar la rama principal (usualmente `main` o `master`).

### 6.1. Crear una Nueva Rama

```bash
git branch nombre-de-la-rama
```

### 6.2. Cambiar de Rama

```bash
git checkout nombre-de-la-rama
```

O, combinando ambos pasos, crea y cambia de rama con:

```bash
git checkout -b nombre-de-la-rama
```

### 6.3. Fusionar (Merge) una Rama

Una vez que termines los cambios en una rama, puedes fusionarla con la rama principal:

1. Cambia a la rama principal:
    ```bash
    git checkout main
    ```
2. Fusiona la rama de desarrollo:
    ```bash
    git merge nombre-de-la-rama
    ```

## 7. Trabajando con Repositorios Remotos

### 7.1. Agregar un Repositorio Remoto

Si ya tienes un repositorio remoto (por ejemplo, en GitHub), puedes vincularlo a tu repositorio local:

```bash
git remote add origin https://github.com/usuario/nombre-repositorio.git
```

Verifica los repositorios remotos con:

```bash
git remote -v
```

### 7.2. Enviar Cambios al Repositorio Remoto (Push)

Después de realizar commits, envía los cambios al repositorio remoto:

```bash
git push origin main
```

Si estás trabajando en una rama diferente:

```bash
git push origin nombre-de-la-rama
```

### 7.3. Traer Cambios del Repositorio Remoto (Pull)

Para actualizar tu repositorio local con los cambios remotos:

```bash
git pull origin main
```

## 8. Otros Comandos Útiles

- **Ver el estado del repositorio:** 
    ```bash
git status
    ```
    Muestra qué archivos han cambiado y cuáles están en el área de staging.

- **Ver diferencias entre cambios (diff):** 
    ```bash
git diff
    ```
    Muestra las diferencias entre los cambios realizados y la última versión confirmada.

- **Deshacer cambios en área de staging:** 
    ```bash
git reset HEAD archivo.txt
    ```

- **Revertir un commit (si ya se ha enviado):** 
    ```bash
git revert <commit_id>
    ```

- **Eliminar una rama:** 
    ```bash
git branch -d nombre-de-la-rama
    ```

## 9. Buenas Prácticas

- **Commits Pequeños y Frecuentes:** 
  Realiza commits con cambios pequeños y bien definidos. Esto facilita el seguimiento y la resolución de problemas.

- **Mensajes de Commit Claros:** 
  Escribe mensajes descriptivos que expliquen claramente qué cambios se realizaron.

- **Mantén tu Rama Principal Actualizada:** 
  Integra cambios regularmente para evitar conflictos acumulados.

- **Utiliza Ramas para Funcionalidades o Correcciones:** 
  Trabaja en ramas separadas y fusiónalas una vez que estén probadas.

## Conclusión

Este manual básico te ofrece una introducción a Git y sus comandos fundamentales. A medida que te sientas más cómodo con estos conceptos, es recomendable explorar temas más avanzados como *rebasing*, *stash*, y la colaboración en equipos utilizando pull requests en plataformas como GitHub o GitLab.
