# **Manual Básico de REGO**

## **Introducción**
REGO es un lenguaje declarativo para definir políticas. Se utiliza para escribir reglas que evalúan datos y devuelven decisiones (permitir/denegar, valores específicos, etc.).

Las políticas en REGO se definen en términos de **reglas**, **consultas** y **datos**. Las consultas verifican si los datos cumplen las reglas.

---

## **Estructura Básica**
1. **Módulo**: Agrupa reglas relacionadas.
2. **Reglas**: Son las unidades básicas de una política. 
3. **Consultas**: Evalúan reglas para verificar datos.

### Ejemplo Básico
```rego
package ejemplo

allow {
    input.usuario == "admin"
    input.accion == "leer"
}
```
Este código permite que solo los administradores lean datos.

---

## **Operaciones Básicas**

### **Operadores Relacionales**
| Operador  | Descripción                | Ejemplo                |
|-----------|----------------------------|------------------------|
| `==`      | Igualdad                   | `x == 10`             |
| `!=`      | No es igual                | `x != "admin"`        |
| `<`       | Menor que                  | `x < 100`             |
| `<=`      | Menor o igual que          | `x <= 50`             |
| `>`       | Mayor que                  | `x > 10`              |
| `>=`      | Mayor o igual que          | `x >= 20`             |

---

### **Operadores Lógicos**
| Operador  | Descripción                | Ejemplo                |
|-----------|----------------------------|------------------------|
| `and`     | Y lógico                   | `x == 10 and y == 20` |
| `or`      | O lógico                   | `x == 10 or y == 20`  |
| `not`     | Negación                   | `not x == 10`         |

---

### **Funciones Básicas**

1. **Condicionales**
   ```rego
   allow {
       some x
       x in input.roles
       x == "admin"
   }
   ```
   Evalúa si algún valor de `input.roles` es `"admin"`.

2. **Filtrado de datos**
   ```rego
   admins := {u | u := input.usuarios[_]; u.rol == "admin"}
   ```
   Obtiene una colección de usuarios con rol `admin`.

3. **Agregación**
   - `count`: Cuenta elementos.
     ```rego
     count(input.usuarios) > 5
     ```
   - `sum`: Suma elementos.
     ```rego
     total := sum(input.precios)
     ```
   - `max`/`min`: Encuentra el valor máximo/mínimo.
     ```rego
     max(input.edades) > 60
     ```

---

### **Estructuras de Control**

#### **Reglas con Condiciones**
```rego
allow {
    input.usuario == "admin"
}

deny {
    input.usuario != "admin"
}
```

#### **Iteraciones**
```rego
valid_users := [user | user := input.usuarios[_]; user.activo == true]
```

---

## **Consultas**
Para ejecutar políticas, utilizamos consultas en OPA.

1. **Ejemplo de Entrada**
   ```json
   {
       "usuario": "admin",
       "accion": "leer",
       "roles": ["admin", "editor"]
   }
   ```

2. **Consulta con OPA**
   ```bash
   opa eval -i input.json -d policy.rego "data.ejemplo.allow"
   ```

---

## **Ejemplo Completo**
```rego
package authz

default allow = false

allow {
    input.usuario == "admin"
    input.accion in ["leer", "escribir"]
}

allow {
    input.usuario == "editor"
    input.accion == "leer"
}
```

### **Entrada**
```json
{
    "usuario": "editor",
    "accion": "leer"
}
```

### **Salida**
```json
{
    "result": true
}
```
