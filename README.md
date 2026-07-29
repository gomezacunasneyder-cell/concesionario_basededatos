# Concesionario CampusCar - Documentación de Base de Datos

Bienvenido a la documentación oficial del diseño de base de datos relacional para **CampusCar**, un sistema diseñado para la gestión integral de inventario de vehículos, trazabilidad de clientes y vendedores, control de ventas y seguimiento de mantenimientos preventivos y correctivos.

---

## 📌 Tabla de Contenidos
1. [Justificación del Sistema](#1-justificación-del-sistema)
2. [Justificación del Diseño](#2-justificación-del-diseño)
   - [Entidades Principales](#21-entidades-principales)
   - [Inclusión de DETALLE_VENTA](#22-por-qué-se-creó-detalle_venta)
   - [Relación Opcional en MANTENIMIENTO](#23-por-qué-mantenimiento-tiene-una-relación-opcional-con-cliente)
3. [Restricciones y Validaciones](#3-restricciones-y-validaciones)
   - [Reglas de Negocio de la Aplicación](#31-regla-de-negocio-adicional)
4. [Relaciones UML y Cardinalidad](#4-relaciones-uml)
   - [Detalle de Relaciones](#41-explicación-de-cada-relación)
   - [Mapeo Conceptual a Modelo Lógico](#42-correspondencia-entre-el-diagrama-conceptual-y-el-modelo-de-tablas)

---

## 1. Justificación del Sistema

El concesionario **CampusCar** requiere gestionar simultáneamente cuatro pilares operativos fundamentales:
* **Inventario de Vehículos:** Disponibilidad y especificaciones técnicas.
* **Gestión de Clientes:** Información de contacto e historial de compra.
* **Equipo de Ventas:** Control de empleados e historial de transacciones gestionadas.
* **Historial de Mantenimientos:** Ficha de servicios prestados a cada unidad (vendida o no).

Manejar esta información mediante formatos en papel u hojas de cálculo descentralizadas genera problemas críticos como falta de certeza sobre la disponibilidad de inventario, pérdida de trazabilidad de servicios y duplicidad accidental de registros.

El diseño de esta **base de datos relacional** centraliza la información operativa, garantizando **integridad referencial**, eliminación de redundancias mediante claves primarias/foráneas y facilitando la ejecución de consultas complejas de negocio en tiempo real.

---

## 2. Justificación del Diseño

El modelo fue estructurado identificando cada proceso de negocio (registro de autos, clientes, vendedores, transacciones y servicios) y traduciéndolo en entidades independientes. Se establecieron **5 tablas principales** y **1 tabla asociativa (puente)** para resolver relaciones N:M.

### 2.1 Entidades Principales
* **`CLIENTE`**: Almacena los datos de contacto del comprador (`nombre`, `teléfono`, `correo`, `dirección`).
* **`VENDEDOR`**: Almacena los datos del personal de ventas (`nombre`, `número de empleado`, `fecha de contratación`).
* **`VEHICULO`**: Concentra las características técnicas y comerciales (`marca`, `modelo`, `año`, `VIN`, `precio`, `color`, `combustible`, `transmisión`, `estado`, `disponibilidad`).
* **`VENTA`**: Encabezado de la transacción comercial (`fecha`, `total`, `método de pago`). Actúa como puente entre `CLIENTE` y `VENDEDOR`.
* **`MANTENIMIENTO`**: Registra los servicios ejecutados sobre un vehículo (`tipo de servicio`, `costo`, `fecha`).

### 2.2 Por qué se creó `DETALLE_VENTA`
Dado que una venta puede incluir múltiples vehículos y un vehículo puede estar en una venta, existe una relación **Muchos a Muchos (N:M)** entre `VENTA` y `VEHICULO`. Como las columnas de una tabla relacional no deben almacenar valores multivaluados (1FN), se diseñó la tabla asociativa `DETALLE_VENTA`:
* **Llave Primaria Compuesta:** (`id_venta` + `id_vehiculo`).
* **Atributos Propios:** Almacena `precio_venta`, ya que el valor convenido depende de la combinación específica entre la transacción y la unidad vendida.

### 2.3 Por qué `MANTENIMIENTO` tiene una relación opcional con `CLIENTE`
Los vehículos en stock pueden recibir acondicionamiento o mantenimiento preventivo antes de ser comercializados. Por lo tanto, la clave foránea `id_cliente` dentro de la tabla `MANTENIMIENTO` se definió como **opcional (`NULL`)**, permitiendo registrar mantenimientos a unidades de inventario que aún pertenecen al concesionario.

---

## 3. Restricciones y Validaciones

A continuación se detallan las restricciones de integridad asignadas a cada tabla del modelo:

| Tabla | Campo | Tipo de Clave | Restricción / Justificación |
| :--- | :--- | :--- | :--- |
| **`CLIENTE`** | `id_cliente` | **PK** | Autoincremental. Identificador único de cliente. |
| **`VENDEDOR`** | `id_vendedor` | **PK** | Autoincremental. |
| **`VENDEDOR`** | `numero_empleado` | **UNIQUE** | Código de empleado único e irrepetible. |
| **`VEHICULO`** | `id_vehiculo` | **PK** | Autoincremental. Identificador interno. |
| **`VEHICULO`** | `vin` | **UNIQUE** | Número de serie real único por vehículo (requisito legal/técnico). |
| **`VENTA`** | `id_venta` | **PK** | Autoincremental. |
| **`VENTA`** | `id_cliente` | **FK → CLIENTE** | Obligatoria (`NOT NULL`). Toda venta requiere un cliente. |
| **`VENTA`** | `id_vendedor` | **FK → VENDEDOR** | Obligatoria (`NOT NULL`). Toda venta requiere un vendedor. |
| **`DETALLE_VENTA`** | `id_venta` + `id_vehiculo` | **PK Compuesta / FK** | Identifica cada ítem dentro de la venta. Ambas actúan como FK a sus tablas origen. |
| **`MANTENIMIENTO`** | `id_mantenimiento` | **PK** | Autoincremental. |
| **`MANTENIMIENTO`** | `id_vehiculo` | **FK → VEHICULO** | Obligatoria (`NOT NULL`). Todo mantenimiento está asociado a un auto. |
| **`MANTENIMIENTO`** | `id_cliente` | **FK → CLIENTE** | Opcional (`NULL`). Permite registrar mantenimientos a vehículos sin vender. |

### 3.1 Regla de Negocio Adicional
* **Actualización de Disponibilidad:** Al insertar un nuevo registro en `DETALLE_VENTA`, el estado del campo `disponible` en la tabla `VEHICULO` debe cambiar automáticamente a `FALSE` (`0`). 
* *Implementación:* Se debe ejecutar mediante un **Trigger (Disparador)** a nivel de motor de base de datos o mediante lógica de negocio en la capa backend de la aplicación.

---

## 4. Relaciones UML

El modelado del sistema fue proyectado bajo dos enfoques metodológicos:
1. **Nivel Conceptual (Modelo E-R, Notación Chen):** Entidades (rectángulos), Atributos (óvalos) y Relaciones (rombos con cardinalidad), abstraído de llaves foráneas.
2. **Nivel Lógico (Modelo Relacional):** Definición formal de tablas, tipos de datos, claves primarias (**PK**) y claves foráneas (**FK**).

### Matriz de Cardinalidad y Obligatoriedad

| Relación | Cardinalidad | Obligatoriedad |
| :--- | :---: | :--- |
| **`CLIENTE` — `VENTA`** | `1 : N` | Obligatoria en ambos lados |
| **`VENDEDOR` — `VENTA`** | `1 : N` | Obligatoria en ambos lados |
| **`VENTA` — `DETALLE_VENTA`** | `1 : N` | Obligatoria (Relación identificante) |
| **`VEHICULO` — `DETALLE_VENTA`** | `1 : N` | Obligatoria (Relación identificante) |
| **`VEHICULO` — `MANTENIMIENTO`** | `1 : N` | Obligatoria en ambos lados |
| **`CLIENTE` — `MANTENIMIENTO`** | `0..1 : 0..N` | Opcional (`id_cliente` acepta `NULL`) |

### 4.1 Explicación de las Relaciones

* **`CLIENTE` — `VENTA` (REALIZA):** Un cliente puede registrar múltiples compras a lo largo del tiempo; cada venta es realizada por un único cliente.
* **`VENDEDOR` — `VENTA` (GESTIONA):** Un vendedor gestiona múltiples transacciones; cada venta queda asignada a un único responsable comercial.
* **`VENTA` — `DETALLE_VENTA` (INCLUYE) / `VEHICULO` — `DETALLE_VENTA` (CORRESPONDE):** Par de relaciones 1:N que descomponen la relación N:M original entre ventas y vehículos.
* **`VEHICULO` — `MANTENIMIENTO` (REQUIERE):** Un vehículo acumula historial de mantenimientos; cada orden de mantenimiento aplica a un único vehículo.
* **`CLIENTE` — `MANTENIMIENTO` (SOLICITA):** Relación opcional. Permite vincular al cliente solicitante cuando la unidad ya fue vendida.

### 4.2 Correspondencia entre Diagrama Conceptual y Modelo Relacional
Cada relación conceptual (rombo) se traduce formalmente en una clave foránea (**FK**) en el modelo de tablas. En relaciones de tipo `1 : N`, la clave primaria del lado de la entidad `1` migra como clave foránea hacia la entidad del lado `N`.
