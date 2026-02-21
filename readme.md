# README – Modelo Entidad Relación (ERD)  
## Pecker Nutrition S.A.S.

---

## 1. Contexto del Proyecto

Pecker Nutrition S.A.S. es una empresa dedicada a la comercialización de concentrado para animales (avicultura, porcicultura y ganadería).

El modelo de datos fue diseñado para soportar:

- Recepción de pedidos por WhatsApp (canal principal)
- Gestión de pedidos recurrentes
- Manejo de fórmulas personalizadas por cliente
- Facturación
- Registro de pagos
- Integración con logística y ERP

El sistema centraliza la información que interactúa con:

- Cliente  
- Bot de WhatsApp  
- Operador comercial  
- Sistema de facturación  
- Banco  
- ERP de producción  
- Sistema de logística  

---

## 2. Objetivos del ERD

El modelo entidad–relación fue diseñado bajo los siguientes principios:

1. Normalización adecuada para evitar redundancia.
2. Soporte a pedidos recurrentes.
3. Permitir fórmulas personalizadas por cliente.
4. Separación clara entre pedido, factura y pago.
5. Escalabilidad para integración futura con ERP y logística.
6. Trazabilidad completa del ciclo comercial.

---

## 3. Entidades Principales y Decisiones de Diseño

### 3.1 Cliente

Propósito: Representa el actor comercial principal.

Atributos clave:

- `id_cliente (PK)`
- `nit`
- `telefono_whatsapp (unique)`
- `tipo_cliente`
- `estado`

Decisiones:

- Se definió `telefono_whatsapp` como único porque el bot identifica al cliente por número.
- Se incluyó `tipo_cliente` para segmentación productiva (avicultura, porcicultura, ganadería).
- Se agregó `estado` para manejar clientes inactivos sin eliminar registros, manteniendo trazabilidad histórica.

---

### 3.2 Dirección

Propósito: Permitir múltiples direcciones por cliente.

Decisiones:

- Relación 1:N: un cliente puede tener varias direcciones.
- Se incluyó `es_principal (bool)` para identificar la dirección predeterminada.
- Se separó en tabla independiente para evitar repetir información en cada pedido.

---

### 3.3 Fórmula Personalizada

Propósito: Gestionar mezclas específicas solicitadas por clientes.

Decisiones:

- Relación 1:N: un cliente puede tener varias fórmulas.
- Se incluyen `especie_formula` y `etapa` para controlar producción.
- Se agregó `estado (vigente/no vigente)` para mantener histórico.
- No se embebe dentro del pedido porque una fórmula puede reutilizarse en múltiples pedidos.

---

### 3.4 Pedido

Propósito: Representa la intención comercial del cliente.

Atributos clave:

- `id_pedido`
- `id_cliente`
- `fecha_creacion`
- `canal`
- `estado`

Decisiones:

- Se incluyó `canal` porque puede originarse por WhatsApp o teléfono.
- Se incluyó `estado` (borrador, confirmado, despachado, etc.) para soportar el flujo operativo.
- El pedido se modela como entidad independiente de la factura.
- Permite soportar pedidos recurrentes consultados por el bot.

---

### 3.5 Detalle Pedido

Propósito: Desglosar productos o fórmulas dentro del pedido.

Decisiones:

- Relación 1:N: un pedido tiene múltiples detalles.
- Puede referenciar `id_producto` o `id_formula (nullable)` para soportar:
  - Productos estándar
  - Fórmulas personalizadas
- `precio_unitario` y `subtotal` se dejaron opcionales para permitir cálculo externo (ERP o facturación) y mantener histórico de precios.

---

### 3.6 Factura

Propósito: Documento contable asociado al pedido.

Decisiones:

- Relación 1:1 con Pedido.
- Incluye `fecha_emision`.
- `estado` permite controlar anulación o pago.
- `valor_total` se guarda para trazabilidad histórica.
- Se separa de Pedido porque no todo pedido genera factura inmediata y puede haber validaciones o crédito.

---

### 3.7 Pago

Propósito: Registrar transacciones financieras.

Decisiones:

- Relación 1:N: una factura puede tener múltiples pagos (abonos).
- Incluye `medio_pago`.
- Se agrega `respuesta_bot` para conciliación automática.
- Se mantiene independiente para facilitar integración con banco.

---

## 4. Flujo Comercial Soportado

El ERD permite modelar completamente el flujo mostrado en el diagrama de contexto:

1. Cliente inicia conversación por WhatsApp.
2. Sistema valida número.
3. Consulta pedidos recurrentes.
4. Genera Pedido.
5. Crea Factura.
6. Registra Pago.
7. Notifica a logística.
8. Integra con ERP.

El modelo permite trazabilidad completa, automatización y escalamiento a atención humana cuando sea necesario.

---

## 5. Principales Decisiones Arquitectónicas

### Separación Pedido–Factura–Pago

Se decidió desacoplar estas entidades porque representan:

- Pedido: intención comercial.
- Factura: obligación contable.
- Pago: evento financiero.

Esto mejora integración bancaria, manejo de crédito y control fiscal.

---

### Soporte a Recurrencia

No se creó una tabla específica de “pedido recurrente”.

En su lugar:

- Se reutilizan pedidos anteriores.
- El bot consulta historial.
- Se replica estructura cuando el cliente confirma.

Esto reduce complejidad innecesaria en el modelo.

---

### Normalización

Se evitó:

- Repetición de direcciones.
- Duplicación de fórmulas.
- Mezclar información financiera con operativa.

El modelo mantiene separación lógica y consistencia estructural, alineado con principios de tercera forma normal.

---

### Escalabilidad

El diseño permite:

- Integración con ERP de producción.
- Conexión con sistema de logística.
- Automatización por bot.
- Expansión a múltiples canales.

---

## 6. Conclusión

El modelo ERD de Pecker Nutrition S.A.S. fue diseñado alineado con el proceso real del negocio:

- Venta recurrente.
- Atención automatizada.
- Producción bajo demanda.
- Facturación formal.
- Registro financiero.

Se trata de una estructura modular, trazable y escalable que soporta tanto la operación actual como el crecimiento futuro de la empresa.
