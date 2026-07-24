# Asistente Marchi — Finanzas Personales Automatizadas

Sistema propio de gestión de finanzas personales que combina **Google Sheets**, **Google Apps Script**, **Telegram** e **iOS Shortcuts** para eliminar la carga manual de datos y automatizar recordatorios de pago.

> Nota: este repo muestra el código y la arquitectura reales del proyecto. Los datos (montos, cuentas, portfolio) son de ejemplo — la planilla real es privada.

## El problema

Llevar el control de gastos, tarjetas, servicios y cuotas a mano en una planilla implica cargar todo manualmente y acordarse de revisar vencimientos. Eso se pierde fácil. El objetivo fue automatizar todo lo posible: que los datos entren solos (desde el celular o desde el mail) y que el sistema avise cuando hay que pagar algo, sin tener que abrir la planilla.

## La solución

Google Sheets funciona como base de datos central (10 hojas: ingresos, gastos, cuotas, servicios, tarjetas, movimientos bancarios, presupuesto, dashboard general y portfolio de inversiones). Google Apps Script actúa como backend serverless que conecta todo:

- **Carga rápida desde el celular:** un atajo de iOS manda los datos de un gasto/ingreso/cuota por voz o con un toque, sin abrir la planilla.
- **Importación automática desde Gmail:** los resúmenes de tarjeta y facturas de servicios (incluso en PDF) se leen solos por regex y se cargan a la planilla, sin tipear nada.
- **Recordatorios por Telegram:** el día antes y el día del vencimiento, el bot avisa con un botón para marcar como pagado, que actualiza la planilla al instante.
- **Resumen mensual:** los días 5 y 15 de cada mes, un mensaje consolida todos los pagos pendientes.
- **Seguimiento de inversiones:** registro de compras de CEDEARs/acciones con cálculo de rentabilidad y evolución trimestral.

## Funcionalidades

| Funcionalidad | Cómo funciona |
|---|---|
| Registrar gasto/ingreso/cuota | Atajo de iOS → POST autenticado con secreto → fila nueva en Sheets |
| Importar resumen de tarjeta | Gmail (mail etiquetado) → regex extrae total y vencimiento → Sheets |
| Importar factura de servicio | Gmail (texto, HTML o PDF adjunto) → parseo → Sheets |
| Importar movimientos bancarios | Gmail (notificaciones del banco) → clasifica transferencia/compra crédito/débito → Sheets |
| Recordatorio de vencimiento | Trigger diario → Telegram con botón "Marcar como pagado" → callback actualiza Sheets |
| Resumen mensual de pagos | Trigger días 5 y 15 → Telegram |
| Lista de supermercado | Mensaje de texto libre en Telegram → parseo → lista |
| Dashboard del mes | Hoja "General": ingresos, gastos, saldo, presupuesto vs. real con semáforo |
| Portfolio de inversiones | Registro de compras, cálculo de rentabilidad y tiempo invertido |

## Arquitectura

Ver [`ARCHITECTURE.md`](./ARCHITECTURE.md) para el diagrama completo y el detalle de cada flujo (incluye decisiones de diseño: idempotencia del webhook con `CacheService`/`LockService`, autenticación con secreto compartido, mapeo dinámico de columnas).

## Stack técnico

Google Apps Script (JavaScript) · Google Sheets · Gmail API · Drive API / Google Docs (conversión de PDF) · Telegram Bot API · iOS Shortcuts · Time-driven triggers
## Cómo se replica (a grandes rasgos)

1. Crear la planilla con las hojas descritas en `data/finanzas_demo.xlsx`.
2. Crear un proyecto de Apps Script vinculado a esa planilla y pegar los archivos de `src/`.
3. Copiar `Config.example.gs` como `Config.gs` y completar token de Telegram, chat_id y nombres de hoja (este archivo no se sube al repo).
4. Publicar el proyecto como Web App (`doPost` como punto de entrada) y configurar el webhook de Telegram apuntando a esa URL.
5. Crear los triggers time-driven (`verificarVencimientos*`, `crearTriggersResumenMensual`, y los de importación de Gmail) desde el editor de Apps Script.
6. Configurar el Atajo de iOS para que mande un POST con el `secret` compartido.

