# Comandos Básicos de OpenShift

## 1. Conexión y Autenticación
- **Iniciar sesión en el clúster:**
  ```bash
  oc login <URL_DEL_CLUSTER> --token=<TOKEN>
  ```
- **Cerrar sesión:**
  ```bash
  oc logout
  ```

## 2. Gestión de Proyectos/Namespaces
- **Listar proyectos disponibles:**
  ```bash
  oc get projects
  ```
- **Cambiar al proyecto actual:**
  ```bash
  oc project <nombre-del-proyecto>
  ```
- **Crear un nuevo proyecto:**
  ```bash
  oc new-project <nombre-del-proyecto>
  ```
- **Eliminar un proyecto:**
  ```bash
  oc delete project <nombre-del-proyecto>
  ```

## 3. Gestión de Recursos

### Listar Recursos
- **Listar todos los recursos en el proyecto actual:**
  ```bash
  oc get all
  ```
- **Listar pods:**
  ```bash
  oc get pods
  ```
- **Listar servicios (services):**
  ```bash
  oc get svc
  ```
- **Listar rutas (routes):**
  ```bash
  oc get routes
  ```
- **Listar despliegues (deployments):**
  ```bash
  oc get deployments
  ```

### Obtener Detalles
- **Mostrar detalles de un recurso específico:**
  ```bash
  oc describe <tipo-recurso> <nombre>
  ```
  Ejemplo:
  ```bash
  oc describe pod <nombre-del-pod>
  ```

## 4. Creación de Aplicaciones
- **Crear una aplicación desde un contenedor:**
  ```bash
  oc new-app <imagen>
  ```
  Ejemplo:
  ```bash
  oc new-app openshift/hello-openshift
  ```

- **Crear una aplicación desde un repositorio de código:**
  ```bash
  oc new-app https://github.com/<repo>.git
  ```

- **Exponer un servicio para acceso externo:**
  ```bash
  oc expose svc/<nombre-del-servicio>
  ```
  Ver la ruta generada:
  ```bash
  oc get route
  ```

## 5. Interactuar con Pods
- **Ver logs de un pod:**
  ```bash
  oc logs <nombre-del-pod>
  ```
- **Seguir logs en tiempo real:**
  ```bash
  oc logs -f <nombre-del-pod>
  ```
- **Ejecutar un comando dentro de un pod:**
  ```bash
  oc exec <nombre-del-pod> -- <comando>
  ```
  Ejemplo:
  ```bash
  oc exec <nombre-del-pod> -- ls /app
  ```
- **Abrir una sesión interactiva en un pod:**
  ```bash
  oc rsh <nombre-del-pod>
  ```

## 6. Gestión de Imágenes
- **Listar imágenes disponibles:**
  ```bash
  oc get is
  ```
- **Subir una imagen al registro interno:**
  ```bash
  oc import-image <nombre-imagen>:<tag>
  ```

## 7. Gestión de Configuraciones
- **Crear un ConfigMap:**
  ```bash
  oc create configmap <nombre> --from-literal=<clave>=<valor>
  ```
  Ejemplo:
  ```bash
  oc create configmap mi-config --from-literal=env=produccion
  ```
- **Crear un Secret:**
  ```bash
  oc create secret generic <nombre> --from-literal=<clave>=<valor>
  ```
  Ejemplo:
  ```bash
  oc create secret generic mi-secret --from-literal=password=1234
  ```
- **Montar ConfigMaps o Secrets en un Deployment:**
  Edita el deployment y agrega las configuraciones.

## 8. Gestión de Roles y Permisos
- **Ver roles asignados en un proyecto:**
  ```bash
  oc get rolebindings
  ```
- **Asignar un rol a un usuario:**
  ```bash
  oc adm policy add-role-to-user <rol> <usuario> -n <namespace>
  ```
  Ejemplo:
  ```bash
  oc adm policy add-role-to-user edit usuario -n mi-proyecto
  ```

## 9. Depuración y Resolución de Problemas
- **Ver eventos recientes en el proyecto:**
  ```bash
  oc get events
  ```
- **Comprobar acceso de un usuario:**
  ```bash
  oc auth can-i <acción> <recurso> -n <namespace>
  ```
  Ejemplo:
  ```bash
  oc auth can-i create pods -n mi-proyecto
  ```

## 10. Eliminar Recursos
- **Eliminar un recurso específico:**
  ```bash
  oc delete <tipo-recurso> <nombre>
  ```
  Ejemplo:
  ```bash
  oc delete pod mi-pod
  ```
- **Eliminar todos los recursos de un tipo:**
  ```bash
  oc delete <tipo-recurso> --all
  