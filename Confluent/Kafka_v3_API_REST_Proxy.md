# Manual de la API v3 del Kafka REST Proxy

> **Nota:**  
> Este manual está basado en la documentación oficial de Confluent y en prácticas comunes. Te recomiendo revisar la [documentación oficial de Confluent Platform](https://docs.confluent.io/platform/current/kafka-rest/api.html) para obtener detalles adicionales y actualizaciones.

---

## 1. Introducción a la API v3

La API v3 del Kafka REST Proxy permite administrar y consultar el estado del clúster Kafka a través de HTTP. Con estos endpoints puedes obtener información sobre:
- Clústeres y su configuración.
- Brokers (nodos) que forman parte del clúster.
- Topics, particiones y réplicas.
- Estadísticas y métricas de los componentes.
- Información de consumidores, productores y otras entidades de administración.

La estructura base de las rutas suele ser:

```
http://<HOST>:<PUERTO>/v3/<recurso>
```

Algunos endpoints en versiones anteriores se encontraban bajo rutas como `/kafka/v3/`, pero en la versión estándar se utiliza la ruta `/v3/`. Asegúrate de revisar cuál es la ruta base configurada en tu REST Proxy.

---

## 2. Endpoints Principales de la API v3

A continuación se describen los endpoints más relevantes para la administración del clúster.

### 2.1. Clústeres

- **Listar clústeres**  
  **Endpoint:**  
  ```
  GET /v3/clusters
  ```  
  **Ejemplo con curl:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters
  ```
  **Descripción:**  
  Devuelve un listado de todos los clústeres que administra el REST Proxy, junto con metadatos (como el cluster ID, nombre, etc.).

- **Obtener detalles de un clúster**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>
  ```
  **Descripción:**  
  Proporciona detalles específicos del clúster identificado por `cluster_id`.

### 2.2. Brokers

- **Listar brokers en un clúster**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/brokers
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/brokers
  ```
  **Descripción:**  
  Obtienes la lista de brokers (nodos) que forman parte del clúster, junto con información como la dirección, puerto, estado, etc.

- **Obtener detalles de un broker**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/brokers/{broker_id}
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/brokers/<broker_id>
  ```
  **Descripción:**  
  Detalla la configuración y el estado de un broker específico.

### 2.3. Topics

- **Listar topics**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/topics
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/topics
  ```
  **Descripción:**  
  Devuelve la lista de topics existentes en el clúster, junto con información básica (nombre, número de particiones, estado, etc.).

- **Obtener detalles de un topic**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/topics/{topic_name}
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/topics/<topic_name>
  ```
  **Descripción:**  
  Proporciona detalles específicos del topic, como su configuración, número de particiones y réplicas.

### 2.4. Particiones

- **Listar particiones de un topic**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/topics/{topic_name}/partitions
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/topics/<topic_name>/partitions
  ```
  **Descripción:**  
  Muestra todas las particiones de un topic y detalles como el líder, réplicas, ISR (in-sync replicas), etc.

- **Obtener detalles de una partición**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/topics/{topic_name}/partitions/{partition_id}
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/topics/<topic_name>/partitions/<partition_id>
  ```
  **Descripción:**  
  Brinda información específica sobre la partición indicada.

### 2.5. Réplicas

- **Listar réplicas de una partición**  
  **Endpoint:**  
  ```
  GET /v3/clusters/{cluster_id}/topics/{topic_name}/partitions/{partition_id}/replicas
  ```  
  **Ejemplo:**
  ```bash
  curl -s http://<HOST>:<PUERTO>/v3/clusters/<cluster_id>/topics/<topic_name>/partitions/<partition_id>/replicas
  ```
  **Descripción:**  
  Permite conocer las réplicas configuradas para una partición, indicando cuál es la réplica líder y cuáles están en sincronía.

---

## 3. Otros Endpoints y Operaciones Administrativas

Además de la consulta de recursos básicos, la API v3 puede ofrecer endpoints para otras operaciones administrativas. Algunos ejemplos comunes (dependiendo de la versión y la configuración) son:

- **Obtener métricas y estadísticas:**  
  Algunos endpoints permiten consultar métricas del clúster, topics y brokers. Revisa la documentación para conocer los endpoints disponibles en tu versión.

- **Gestión de configuraciones:**  
  Es posible que existan endpoints para obtener o modificar configuraciones a nivel de clúster o de topic (aunque muchas veces estas operaciones se realizan a través de la CLI o herramientas administrativas).

- **Administración de consumidores y productores:**  
  Si bien la API v3 está más orientada a la administración del clúster, el REST Proxy también ofrece endpoints para la creación y gestión de consumidores y para la producción de mensajes. Para estos casos, consulta la sección de [Consumidores y Productores en la documentación oficial](https://docs.confluent.io/platform/current/kafka-rest/api.html#consumers).

---

## 4. Ejemplos Prácticos

A continuación, se muestran algunos ejemplos utilizando `curl`. Recuerda reemplazar `<HOST>`, `<PUERTO>`, `<cluster_id>`, `<topic_name>`, etc., por los valores correspondientes a tu entorno.

Normalmente el puerto por defecto es el <PUERTO>, pero hay que revisar la propia instalación para saber el puerto.

### 4.1. Listar Clústeres

```bash
curl -s http://<HOST>:<PUERTO>/v3/clusters | jq .
```

> **Nota:** Se utiliza `jq` para formatear la salida JSON.

### 4.2. Obtener el Detalle de un Clúster

```bash
curl -s http://<HOST>:<PUERTO>/v3/clusters/MkU3OEVBNTcwNTJENDM2Qk | jq .
```

### 4.3. Listar Topics del Clúster

```bash
curl -s http://<HOST>:<PUERTO>/v3/clusters/MkU3OEVBNTcwNTJENDM2Qk/topics | jq .
```

### 4.4. Obtener Información de un Topic Específico

```bash
curl -s http://<HOST>:<PUERTO>/v3/clusters/MkU3OEVBNTcwNTJENDM2Qk/topics/mi_topic | jq .
```

### 4.5. Listar Particiones de un Topic

```bash
curl -s http://<HOST>:<PUERTO>/v3/clusters/MkU3OEVBNTcwNTJENDM2Qk/topics/mi_topic/partitions | jq .
```

---

## 5. Consideraciones y Buenas Prácticas

- **Revisar la versión del REST Proxy:**  
  Antes de utilizar los endpoints v3, asegúrate de que la versión del REST Proxy que tienes desplegada los soporta. Puedes verificar la versión consultando la documentación interna o los logs del servicio.

- **Seguridad y Autenticación:**  
  Si tu clúster o el REST Proxy requiere autenticación (por ejemplo, mediante TLS/SSL o tokens), deberás incluir las cabeceras o parámetros necesarios en tus solicitudes `curl` o en las herramientas que utilices.

- **Uso de herramientas auxiliares:**  
  Utilizar herramientas como `jq` para formatear y explorar las respuestas JSON puede facilitar la administración y depuración.

- **Documentación oficial:**  
  La API puede incluir endpoints adicionales o modificaciones en función de la versión de Confluent Platform. Mantente al día consultando la [documentación oficial de Kafka REST Proxy API](https://docs.confluent.io/platform/current/kafka-rest/api.html).

- **Entornos de Prueba:**  
  Antes de ejecutar cambios en producción, realiza pruebas en entornos de desarrollo o staging para asegurarte de que las llamadas a la API tienen el efecto deseado.

---

## 6. Recursos Adicionales

- **Documentación Oficial de Confluent:**  
  [Confluent Platform Documentation](https://docs.confluent.io/platform/current/index.html)

- **API REST Proxy v3 Reference:**  
  [Kafka REST Proxy API](https://docs.confluent.io/platform/current/kafka-rest/api.html)

- **Ejemplos y Tutoriales en GitHub:**  
  Busca repositorios y ejemplos de configuración de Docker Compose y uso de REST Proxy para obtener ideas y soluciones a problemas comunes.

---

# Resumen

Este manual te brinda una visión general de la API v3 del Kafka REST Proxy para la administración de tu clúster de Kafka. Con los endpoints y ejemplos presentados, podrás:

- Consultar el estado y configuración de clústeres, brokers, topics y particiones.
- Integrar operaciones de administración a través de scripts o herramientas basadas en HTTP.
- Extender la administración a operaciones adicionales según las necesidades de tu entorno.
