# Concesionario CampusCar - Documentación de Base de Datos

Bienvenido a la documentación oficial del diseño de base de datos relacional para **CampusCar**. Este documento describe la arquitectura, justificaciones, restricciones de integridad, proceso metodológico de diseño y el modelo de relaciones para el sistema centralizado de gestión del concesionario.

---

## 📌 Tabla de Contenidos
1. [Contexto](#1-contexto)
2. [Objetivo del Sistema](#2-objetivo-del-sistema)
3. [Justificación del Sistema](#3-justificación-del-sistema)
4. [Proceso Metodológico de Diseño (E-R y Modelo de Tablas/UML)](#4-proceso-metodológico-de-diseño)
   - [Paso 1: Diseño del Modelo Entidad-Relación (Diagrama Conceptual)](#41-paso-1-diseño-del-modelo-entidad-relación-diagrama-conceptual)
   - [Paso 2: Transformación al Modelo Lógico / Tablas UML](#42-paso-2-transformación-al-modelo-lógico--tablas-uml)
5. [Justificación del Diseño](#5-justificación-del-diseño)
   - [Entidades Principales](#51-entidades-principales)
   - [Uso e Importancia de `DETALLE_VENTA`](#52-uso-e-importancia-de-detalle_venta)
   - [Uso e Importancia de `MANTENIMIENTO`](#53-uso-e-importancia-de-mantenimiento)
6. [Restricciones y Validaciones](#6-restricciones-y-validaciones)
   - [Reglas de Negocio de la Aplicación](#61-regla-de-negocio-adicional)
7. [Relaciones UML y Cardinalidad](#7-relaciones-uml)
   - [Detalle de Relaciones](#71-explicación-de-cada-relación)
   - [Mapeo Conceptual a Modelo Lógico](#72-correspondencia-entre-el-diagrama-conceptual-y-el-modelo-de-tablas)

---

## 1. Contexto

En el entorno operativo actual del concesionario **CampusCar**, la información relativa a las operaciones diarias se administra de manera fragmentada o descentralizada mediante formatos físicos y hojas de cálculo independientes[cite: 1]. Esta dinámica de trabajo genera inconvenientes recurrentes en la gestión de la empresa:
* **Incertidumbre en el inventario:** Dificultad para validar de forma confiable e inmediata si un vehículo específico continúa disponible para la venta o si ya fue comercializado[cite: 1].
* **Pérdida de trazabilidad operativa:** Fragmentación o pérdida del historial de servicios y mantenimientos preventivos/correctivos realizados a los vehículos[cite: 1].
* **Inconsistencias en los datos:** Riesgo elevado de duplicidad de datos críticos, como la captura errónea o duplicada de números de serie (VIN)[cite: 1].
* **Falta de consolidación:** Ausencia de una vista unificada que relacione los clientes, la fuerza de ventas y el historial vehicular en una sola plataforma[cite: 1].

---

## 2. Objetivo del Sistema

Desarrollar e implementar una **base de datos relacional centralizada** que responda de manera directa a las necesidades del negocio, optimizando la administración del concesionario **CampusCar** a través de:

* **Control Eficiente de Inventario:** Mantener el estado de disponibilidad y la ficha técnica de cada vehículo en tiempo real[cite: 1].
* **Trazabilidad de Transacciones:** Registrar de forma integral el flujo de ventas, asociando a cada cliente con el vendedor responsable y las unidades comercializadas[cite: 1].
* **Historial Integrado de Mantenimiento:** Garantizar el seguimiento completo de los servicios aplicados a cada auto, sin importar si este pertenece al stock actual del concesionario o si ya fue vendido[cite: 1].
* **Garantía de Integridad de Datos:** Eliminar la redundancia y prevenir duplicidades mediante la aplicación estricta de claves primarias, claves foráneas y reglas de unicidad[cite: 1].

---

## 3. Justificación del Sistema

El concesionario **CampusCar** requiere gestionar simultáneamente cuatro pilares operativos fundamentales[cite: 1]:
* **Inventario de Vehículos:** Disponibilidad y especificaciones técnicas[cite: 1].
* **Gestión de Clientes:** Información de contacto e historial de compra[cite: 1].
* **Equipo de Ventas:** Control de empleados e historial de transacciones gestionadas[cite: 1].
* **Historial de Mantenimientos:** Ficha de servicios prestados a cada unidad (vendida o no)[cite: 1].

El diseño de esta base de datos no solo busca cumplir con un requerimiento técnico o académico, sino solucionar una problemática concreta del negocio, garantizando **integridad referencial** y facilitando la ejecución de consultas complejas en tiempo real[cite: 1].

---

## 4. Proceso Metodológico de Diseño

Para construir la arquitectura final de la base de datos se siguió un proceso estructurado en dos fases jerárquicas: desde la abstracción del negocio hasta la implementación lógica de tablas.

### 4.1 Paso 1: Diseño del Modelo Entidad-Relación (Diagrama Conceptual)
Se utilizó la **Notación Chen** para representar de forma abstracta las reglas de negocio planteadas en los requerimientos[cite: 1]:
1. **Identificación de Entidades:** Se identificaron los objetos/sujetos principales del negocio representándolos con **rectángulos**: `CLIENTE`, `VENDEDOR`, `VEHICULO`, `VENTA` y `MANTENIMIENTO`[cite: 1].
2. **Definición de Atributos:** Se asoció cada atributo como un **óvalo** conectado a su entidad respectiva (por ejemplo, `nombre`, `correo` para `CLIENTE`; `vin`, `precio`, `disponible` para `VEHICULO`)[cite: 1].
3. **Establecimiento de Relaciones y Cardinalidad:** Se representaron las interacciones mediante **rombos** con sus respectivas proporciones cuantitativas ($1:N$, $N:M$):
   * `REALIZA` entre `CLIENTE` y `VENTA` ($1:N$)[cite: 1].
   * `GESTIONA` entre `VENDEDOR` y `VENTA` ($1:N$)[cite: 1].
   * `INCLUYE` / `CORRESPONDE` entre `VENTA` y `VEHICULO` ($N:M$)[cite: 1].
   * `REQUIERE` entre `VEHICULO` y `MANTENIMIENTO` ($1:N$)[cite: 1].
   * `SOLICITA` entre `CLIENTE` y `MANTENIMIENTO` ($0..1 : 0..N$)[cite: 1].
4. **Ausencia de Claves Foráneas:** En esta fase conceptual **no se incluyeron FKs**, manteniendo el modelo enfocado exclusivamente en la semántica del negocio[cite: 1].

### 4.2 Paso 2: Transformación al Modelo Lógico / Tablas UML
A partir del diagrama conceptual se realizó la conversión técnica al modelo relacional de tablas[cite: 1]:
1. **Conversión de Entidades a Tablas:** Cada entidad se transformó en una tabla con sus tipos de datos explícitos (`INT`, `VARCHAR`, `DECIMAL`, `DATE`, `BOOLEAN`)[cite: 1].
2. **Definición de Claves Primarias (PK):** Se asignó una clave única autoincremental a cada entidad para garantizar el acceso unívoco a cada fila[cite: 1].
3. **Migración de Claves Foráneas (FK):** Para cada relación $1:N$, la clave primaria de la entidad del lado `1` se migró como clave foránea dentro de la tabla del lado `N` (ejemplo: `id_cliente` de `CLIENTE` se colocó como FK en `VENTA`)[cite: 1].
4. **Resolución de la Relación $N:M$:** Al detectar que la relación entre `VENTA` y `VEHICULO` era de Muchos a Muchos, se creó la tabla puente **`DETALLE_VENTA`** con una clave primaria compuesta por ambas FK (`id_venta` + `id_vehiculo`)[cite: 1].
5. **Ajuste de Opcionalidades:** Se configuró el campo `id_cliente` en `MANTENIMIENTO` para permitir valores nulos (`NULL`), formalizando que un mantenimiento puede existir sin estar vinculado a un cliente[cite: 1].

---

## 5. Justificación del Diseño

El modelo fue estructurado identificando cada proceso de negocio (registro de autos, clientes, vendedores, transacciones y servicios) y traduciéndolo en entidades independientes[cite: 1]. Se establecieron **5 tablas principales** y **1 tabla asociativa (puente)** para resolver relaciones N:M[cite: 1].

### 5.1 Entidades Principales
* **`CLIENTE`**: Almacena los datos de contacto del comprador (`nombre`, `teléfono`, `correo`, `dirección`)[cite: 1].
* **`VENDEDOR`**: Almacena los datos del personal de ventas (`nombre`, `número de empleado`, `fecha de contratación`)[cite: 1].
* **`VEHICULO`**: Concentra las características técnicas y comerciales (`marca`, `modelo`, `año`, `VIN`, `precio`, `color`, `combustible`, `transmisión`, `estado`, `disponibilidad`)[cite: 1].
* **`VENTA`**: Encabezado de la transacción comercial (`fecha`, `total`, `método de pago`). Actúa como puente entre `CLIENTE` y `VENDEDOR`[cite: 1].
* **`MANTENIMIENTO`**: Registra los servicios ejecutados sobre un vehículo (`tipo de servicio`, `costo`, `fecha`)[cite: 1].

---

### 5.2 Uso e Importancia de `DETALLE_VENTA`

La tabla **`DETALLE_VENTA`** es indispensable porque resuelve técnicamente la relación de tipo **Muchos a Muchos ($N:M$)** que existe entre `VENTA` y `VEHICULO`[cite: 1].

* **Soporte de Ventas Múltiples:** Una misma venta/factura puede incluir más de un vehículo (por ejemplo, compras corporativas o de flotillas), y un modelo de vehículo a su vez está involucrado en transacciones comerciales[cite: 1].
* **Cumplimiento de la 1FN (Primera Forma Normal):** En el modelo relacional no se pueden guardar listas de valores en una sola celda (como colocar múltiples `id_vehiculo` en `VENTA`)[cite: 1].
* **Llave Primaria Compuesta:** Se define uniendo `(id_venta + id_vehiculo)`, garantizando que una combinación específica venta-vehículo no se duplique[cite: 1].
* **Almacenamiento de `precio_venta`:** Este campo guarda el valor real acordado en la transacción. Debe residir en esta tabla intermedia porque el precio de venta final puede variar con respecto al `precio` de lista grabado en la ficha del `VEHICULO` (debido a descuentos o negociaciones puntuales)[cite: 1].

---

### 5.3 Uso e Importancia de `MANTENIMIENTO`

La tabla **`MANTENIMIENTO`** desacopla y administra el historial de revisiones técnicas del vehículo sin saturar la entidad `VEHICULO` ni obligar a registrar un cliente de forma prematura[cite: 1].

* **Soporte de Historial ($1:N$):** Un vehículo acumula múltiples servicios mecánicos (cambios de aceite, frenos, repuestos) a lo largo del tiempo[cite: 1]. Guardar estos datos en `VEHICULO` obligaría a duplicar filas del auto por cada visita al taller.
* **Relación Opcional con `CLIENTE` (`id_cliente` es `NULL`able):**
  * **Acondicionamiento Pre-Venta:** Si el vehículo está en inventario y pertenece al concesionario, recibe mantenimiento sin requerir un cliente asociado (`id_cliente = NULL`)[cite: 1].
  * **Servicio Pos-Venta:** Cuando la unidad ya fue vendida y el comprador la ingresa al taller, se completa la clave foránea `id_cliente` asociándolo a la orden[cite: 1].
* **Trazabilidad de Costos:** Facilita auditar los gastos de mantenimiento de cada unidad para calcular el margen de ganancia real antes y después de comercializarla.

---

## 6. Restricciones y Validaciones

A continuación se detallan las restricciones de integridad asignadas a cada tabla del modelo[cite: 1]:

| Tabla | Campo | Tipo de Clave | Restricción / Justificación |
| :--- | :--- | :--- | :--- |
| **`CLIENTE`** | `id_cliente` | **PK** | Autoincremental. Identificador único de cliente[cite: 1]. |
| **`VENDEDOR`** | `id_vendedor` | **PK** | Autoincremental[cite: 1]. |
| **`VENDEDOR`** | `numero_empleado` | **UNIQUE** | Código de empleado único e irrepetible[cite: 1]. |
| **`VEHICULO`** | `id_vehiculo` | **PK** | Autoincremental. Identificador interno[cite: 1]. |
| **`VEHICULO`** | `vin` | **UNIQUE** | Número de serie real único por vehículo (requisito legal/técnico)[cite: 1]. |
| **`VENTA`** | `id_venta` | **PK** | Autoincremental[cite: 1]. |
| **`VENTA`** | `id_cliente` | **FK → CLIENTE** | Obligatoria (`NOT NULL`). Toda venta requiere un cliente[cite: 1]. |
| **`VENTA`** | `id_vendedor` | **FK → VENDEDOR** | Obligatoria (`NOT NULL`). Toda venta requiere un vendedor[cite: 1]. |
| **`DETALLE_VENTA`** | `id_venta` + `id_vehiculo` | **PK Compuesta / FK** | Identifica cada ítem dentro de la venta. Ambas actúan como FK a sus tablas origen[cite: 1]. |
| **`MANTENIMIENTO`** | `id_mantenimiento` | **PK** | Autoincremental[cite: 1]. |
| **`MANTENIMIENTO`** | `id_vehiculo` | **FK → VEHICULO** | Obligatoria (`NOT NULL`). Todo mantenimiento está asociado a un auto[cite: 1]. |
| **`MANTENIMIENTO`** | `id_cliente` | **FK → CLIENTE** | Opcional (`NULL`). Permite registrar mantenimientos a vehículos sin vender[cite: 1]. |

### 6.1 Regla de Negocio Adicional
* **Actualización de Disponibilidad:** Al insertar un nuevo registro en `DETALLE_VENTA`, el estado del campo `disponible` en la tabla `VEHICULO` debe cambiar automáticamente a `FALSE` (`0`)[cite: 1]. 
* *Implementación:* Se debe ejecutar mediante un **Trigger (Disparador)** a nivel de motor de base de datos o mediante lógica de negocio en la capa backend de la aplicación[cite: 1].

---

## 7. Relaciones UML

El modelado del sistema fue proyectado bajo dos enfoques metodológicos[cite: 1]:
1. **Nivel Conceptual (Modelo E-R, Notación Chen):** Entidades (rectángulos), Atributos (óvalos) y Relaciones (rombos con cardinalidad), abstraído de llaves foráneas[cite: 1].
2. **Nivel Lógico (Modelo Relacional):** Definición formal de tablas, tipos de datos, claves primarias (**PK**) y claves foráneas (**FK**)[cite: 1].

### Matriz de Cardinalidad y Obligatoriedad

| Relación | Cardinalidad | Obligatoriedad |
| :--- | :---: | :--- |
| **`CLIENTE` — `VENTA`** | `1 : N` | Obligatoria en ambos lados[cite: 1] |
| **`VENDEDOR` — `VENTA`** | `1 : N` | Obligatoria en ambos lados[cite: 1] |
| **`VENTA` — `DETALLE_VENTA`** | `1 : N` | Obligatoria (Relación identificante)[cite: 1] |
| **`VEHICULO` — `DETALLE_VENTA`** | `1 : N` | Obligatoria (Relación identificante)[cite: 1] |
| **`VEHICULO` — `MANTENIMIENTO`** | `1 : N` | Obligatoria en ambos lados[cite: 1] |
| **`CLIENTE` — `MANTENIMIENTO`** | `0..1 : 0..N` | Opcional (`id_cliente` acepta `NULL`)[cite: 1] |

### 7.1 Explicación de las Relaciones

* **`CLIENTE` — `VENTA` (REALIZA):** Un cliente puede registrar múltiples compras a lo largo del tiempo; cada venta es realizada por un único cliente[cite: 1].
* **`VENDEDOR` — `VENTA` (GESTIONA):** Un vendedor gestiona múltiples transacciones; cada venta queda asignada a un único responsable comercial[cite: 1].
* **`VENTA` — `DETALLE_VENTA` (INCLUYE) / `VEHICULO` — `DETALLE_VENTA` (CORRESPONDE):** Par de relaciones 1:N que descomponen la relación N:M original entre ventas y vehículos[cite: 1].
* **`VEHICULO` — `MANTENIMIENTO` (REQUIERE):** Un vehículo acumula historial de mantenimientos; cada orden de mantenimiento aplica a un único vehículo[cite: 1].
* **`CLIENTE` — `MANTENIMIENTO` (SOLICITA):** Relación opcional. Permite vincular al cliente solicitante cuando la unidad ya fue vendida[cite: 1].

### 7.2 Correspondencia entre Diagrama Conceptual y Modelo Relacional
Cada relación conceptual (rombo) se traduce formalmente en una clave foránea (**FK**) en el modelo de tablas[cite: 1]. En relaciones de tipo `1 : N`, la clave primaria del lado de la entidad `1` migra como clave foránea hacia la entidad del lado `N`[cite: 1].|

### 6.1 Explicación de las Relaciones

* **`CLIENTE` — `VENTA` (REALIZA):** Un cliente puede registrar múltiples compras a lo largo del tiempo; cada venta es realizada por un único cliente[cite: 1].
* **`VENDEDOR` — `VENTA` (GESTIONA):** Un vendedor gestiona múltiples transacciones; cada venta queda asignada a un único responsable comercial[cite: 1].
* **`VENTA` — `DETALLE_VENTA` (INCLUYE) / `VEHICULO` — `DETALLE_VENTA` (CORRESPONDE):** Par de relaciones 1:N que descomponen la relación N:M original entre ventas y vehículos[cite: 1].
* **`VEHICULO` — `MANTENIMIENTO` (REQUIERE):** Un vehículo acumula historial de mantenimientos; cada orden de mantenimiento aplica a un único vehículo[cite: 1].
* **`CLIENTE` — `MANTENIMIENTO` (SOLICITA):** Relación opcional. Permite vincular al cliente solicitante cuando la unidad ya fue vendida[cite: 1].

### 6.2 Correspondencia entre Diagrama Conceptual y Modelo Relacional
Cada relación conceptual (rombo) se traduce formalmente en una clave foránea (**FK**) en el modelo de tablas[cite: 1]. En relaciones de tipo `1 : N`, la clave primaria del lado de la entidad `1` migra como clave foránea hacia la entidad del lado `N`[cite: 1].
