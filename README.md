# TongoMart (Uganda) — Solución de Integración E-Commerce (Magento) + POS + Logistics Platform

Repositorio oficial de entrega para el caso práctico de **Solutions Engineer / ADSO** aplicado al cliente **TongoMart** (cadena de supermercados en Uganda con tienda virtual en **Magento** y facturación automatizada con su **Sistema POS**).

> [!IMPORTANT]
> **Cumplimiento de Confidencialidad (NDA):** Toda la documentación ejecutiva, diagramas de arquitectura y descripciones de la colección se refieren al motor tecnológico exclusivamente como **"La Plataforma Logística"** o **"Last-Mile Delivery Engine (LMD)"**, manteniendo las URLs base parametrizadas en el entorno mediante `{{base_url}}`.

---

## 1. Entregables Oficiales del Repositorio

```text
tongomart-solutions-challenge/
├── README.md                                           # Guía general de la entrega e instrucciones de uso
├── Solucion_Caso_Practico_TongoMart.pdf                # Documento Ejecutivo Final (Puntos 1, 2, 4 y 5 completos)
└── postman/
    ├── TongoMart_Logistics.postman_collection.json     # Punto 3: Colección Postman v2.1 lista para importar
    └── TongoMart_Environment.postman_environment.json  # Punto 3: Environment de Postman con variables requeridas
```

---

## 2. Contenido del Documento Ejecutivo (`Solucion_Caso_Practico_TongoMart.pdf`)

El documento [Solucion_Caso_Practico_TongoMart.pdf](./Solucion_Caso_Practico_TongoMart.pdf) compila el desarrollo técnico integral del caso práctico:

* **Punto 1 — Mapeo Completo de Endpoints e Integración:** Matriz y justificación técnica de los 8 módulos que conectan Magento, el Sistema POS de TongoMart y La Plataforma Logística (Catálogo Batch, `POST /jobs/availability/v2`, validación/extensión de slot, `POST /jobs` con `blocking_policy: "CHECKOUT"`, `GET /jobs/{jobId}`, Webhook `PICKING_FINISHED` con validación `SHA-256`, `PUT /jobs/{jobId}/payment_info` con *Automatic Shopper App Checkout*, y `Live Status Widget V3.x`).
* **Punto 2 — Arquitectura de Integración y Flujos de Secuencia:** Diseño desacoplado en 5 capas con diagramas de **Arquitectura de Componentes** y **Secuencia End-to-End**, detallando el flujo de facturación automática con el POS a partir de los ítems realmente recolectados (`found_quantity`) tras el evento `PICKING_FINISHED`.
* **Punto 3 — Automatización en Postman:** Especificación de las variables de entorno y la función JavaScript `setDynamicAvailabilityDates()` para el cálculo dinámico de ventanas horarias.
* **Punto 4 — Auditoría Técnica del Payload JSON (`Availability V2`):**
  * **4.1:** Identificación de los **6 errores bloqueantes de esquema** (`slot_size`, `origin`, `start > end`, `PICK_COLLECT`, `quantity: 0` y `quantity_found_limits: {}`).
  * **4.2:** Corrección de las **3 violaciones de tipos de datos** (`store_reference` a `string`, `is_big_order` a `boolean` y `picking_index` a `string`).
  * **4.3 & 4.4:** **5 recomendaciones consultivas de negocio para Uganda (`UGX`)** y el **Payload JSON 100% corregido**.
* **Punto 5 — Propuestas Técnicas de Mejora para la API:** 5 propuestas de ingeniería (unificación de `payment_info`, tipado polimórfico en `picking_index`, soporte de `Idempotency-Key` en `POST /jobs`, errores estructurados bajo el estándar RFC 7807 y cabeceras `X-RateLimit-*`).

---

## 3. Instrucciones para Importar y Ejecutar la Colección en Postman (Punto 3)

1. Abrir **Postman** (versión 10.x o 11.x).
2. Hacer clic en **Import** (esquina superior izquierda) y seleccionar los dos archivos ubicados en la carpeta [`postman/`](./postman/):
   * [`postman/TongoMart_Environment.postman_environment.json`](./postman/TongoMart_Environment.postman_environment.json)
   * [`postman/TongoMart_Logistics.postman_collection.json`](./postman/TongoMart_Logistics.postman_collection.json)
3. En el selector de entornos (esquina superior derecha de Postman), activar el environment **`TongoMart - Logistics Platform Environment (Uganda)`**.
4. Revisar las variables configuradas en el Environment:
   * `base_url`: URL base del servidor.
   * `api_key`: `tm_live_987654321_uganda_pos_key` (inyectada automáticamente en el header `x-api-key`).
   * `start` y `end`: Se calculan y asignan automáticamente en tiempo de ejecución mediante la función JavaScript `setDynamicAvailabilityDates()` incluida en la pestaña **Pre-request Script** de `POST Availability (Time Slots) V2`.
   * `slot_id` y `job_id`: Incluyen valores semilla para pruebas inmediatas y se actualizan automáticamente mediante los scripts de **Tests / Post-response** al recibir respuesta de la API.
5. Ejecutar en orden las peticiones de la carpeta **`1. Core Fulfilment Flow (Punto 3 Obligatorio)`**:
   1. **`POST Availability (Time Slots) V2`** (`{{base_url}}/jobs/availability/v2`)
   2. **`POST Create a Job`** (`{{base_url}}/jobs`)
   3. **`GET Get a Job by ID`** (`{{base_url}}/jobs/{{job_id}}`)
6. Opcionalmente, explorar la carpeta **`2. Automated POS Billing & Slot Validation (Complementario Punto 1)`**, que incluye la petición **`PUT Update Payment Info & Automated POS Invoice`** (`{{base_url}}/jobs/{{job_id}}/payment_info`) para simular el envío de la factura emitida por el POS (`POS-UG-INV-8841.pdf`) y el desbloqueo automático del checkout (`blocking_policy: "UNBLOCKED"`).
