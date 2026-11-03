# SOCIAL MEDIA CENTER (SMC)
> **Progressive Web App (PWA) de Gestión Operativa, Alcance y Finanzas para Smaily Social Media**

---

## 🚀 Descripción General
**SOCIAL MEDIA CENTER** es una solución integral desarrollada con arquitectura web moderna (Mobile-First, Dark Mode sofisticado, Vanilla JavaScript modular, Tailwind CSS y Supabase JS v2). Está diseñada para gestionar el flujo de trabajo de agencias de marketing digital, productoras audiovisuales y creadores de contenido.

Permite:
- Evitar fugas de dinero detectando en tiempo real si un cliente agotó su cupo del abono mensual y alertando para cobrarlo como **Extra**.
- Monitorear en un **Semáforo Operativo** la carga semanal de trabajo (horas estimadas de tareas pendientes).
- Generar con 1 clic **Liquidaciones Mensuales detalladas** que muestran el ahorro de cortesías otorgadas (fidelización) y el total a pagar, exportables a **WhatsApp** formateado y **PDF corporativo**.
- Proteger la privacidad del negocio en celulares y PCs mediante un **PIN de seguridad de 4 dígitos**.

---

## 📦 Estructura del Proyecto
```
SOCIAL-MEDIA-CENTER/
├── index.html       # Aplicación SPA completa, autónoma y modular
├── manifest.json    # Manifiesto PWA para instalación como app nativa
├── sw.js            # Service Worker para caché offline y carga ultrarrápida
├── icon-192.svg     # Icono vectorizado PWA 192x192
├── icon-512.svg     # Icono vectorizado PWA 512x512
└── README.md        # Documentación técnica y guía de uso
```

---

## 🗄️ Conexión con Supabase
Configurada de manera nativa con las credenciales activas del proyecto:
- **URL**: `https://zcvkylmjikdcpkhpxwqg.supabase.co`
- **Key**: `sb_publishable_CZJw-iheCX3OwzuJ7LBwXQ_daFyvKwf`

### Tablas Conectadas:
1. **`smc_servicios`**:
   - `id` (uuid, PK)
   - `nombre` (text)
   - `categoria` (text)
   - `precio_base` (numeric)
   - `horas_estimadas` (numeric)
   - `activo` (boolean)

2. **`smc_clientes`**:
   - `id` (uuid, PK)
   - `nombre` (text)
   - `unidad_negocio` (text: 'Smaily', 'Independiente', 'Eventual')
   - `valor_abono` (numeric)
   - `cupo_reels` (integer)
   - `cupo_flyers` (integer)
   - `cupo_historias` (integer)
   - `cupo_rodajes` (integer)
   - `activo` (boolean)

3. **`smc_trabajos`**:
   - `id` (uuid, PK)
   - `cliente_id` (uuid, FK -> `smc_clientes`)
   - `servicio_id` (uuid, FK -> `smc_servicios`)
   - `descripcion` (text)
   - `fecha` (date)
   - `modalidad` (text: 'Abono', 'Extra', 'Cortesia', 'Puntual')
   - `tipo_cupo` (text: 'reels', 'flyers', 'historias', 'rodajes', null)
   - `recargo_urgencia` (numeric: 0, 25, 50)
   - `precio_final` (numeric)
   - `valor_real_cortesia` (numeric)
   - `horas_tarea` (numeric)
   - `estado` (text: 'Pendiente', 'En Proceso', 'Entregado', 'Cobrado')

---

## 🔑 Acceso y Seguridad (PIN de 4 dígitos)
- **PIN por defecto**: `1234`
- Teclado numérico táctil en pantalla con feedback háptico (vibración en celulares) y soporte para teclado físico de computadora.
- Botón **"Bloquear Ahora"** en el encabezado y en la pestaña de Ajustes para salir de forma segura.
- Cambio de PIN personalizable desde la pestaña **Ajustes**, guardado en el almacenamiento local del dispositivo (`localStorage`).

---

## 🛠️ Módulos Desarrollados al 100%

### 1. Dashboard Principal ("Mi Situación")
- **4 Tarjetas Financieras del Mes**:
  1. *Facturación Fija*: Suma de los abonos mensuales de clientes activos.
  2. *Adicionales / Extras*: Suma de los trabajos cobrados fuera de abono del mes seleccionado.
  3. *Cortesías Bonificadas*: Valor real del trabajo regalado (resalta el valor de fidelización).
  4. *Total Proyectado*: Facturación fija pactada + adicionales acumulados.
- **Semáforo de Carga Operativa**:
  - Suma las horas de las tareas pendientes y en proceso.
  - 🟢 **Verde (< 20 hs)**: Carga óptima. Capacidad disponible para nuevos rodajes o pedidos.
  - 🟡 **Amarillo (20 a 35 hs)**: Carga moderada. Capacidad ajustada.
  - 🔴 **Rojo (> 35 hs)**: Sobrecarga crítica. Aplicar recargos por urgencia o reprogramar fechas.
- **Entregas Pendientes**:
  - Lista interactiva de entregas urgentes con botones de 1 clic para marcar como **"Entregar"** o **"Cobrado"**.

### 2. Gestión de Clientes y Control de Alcance
- Filtros rápidos: `Todos`, `Smaily`, `Independiente`, `Eventual` y buscador predictivo.
- CRUD completo (Crear, Editar, Activar/Pausar, Eliminar clientes).
- **Barras de progreso en tiempo real**: Monitorean cuántos Reels, Flyers, Historias y Rodajes ha consumido el cliente en el mes actual sobre su cupo contratado. Al llegar al 100%, emite una insignia roja de **"AGOTADO"**.
- Acceso directo a la **Liquidación** y a la creación de pedidos específicos por cliente.

### 3. Detector de Extras, Carga Rápida y Cliente al Vuelo
- Accesible en cualquier momento mediante el botón flotante (FAB) o la barra de acciones.
- **Creación de Cliente al Vuelo (Sin Fricción):** Permite elegir entre un cliente habitual o crear un prospecto/cliente ocasional directamente escribiendo su nombre (ej: *"Hamburguesería Dylan"* o *"15 de Camila"*). El sistema lo da de alta automáticamente como cliente **Eventual** sin pedirte abonos ni cupos.
- **Detección inteligente de cupos:** Si un cliente con abono ya agotó su cupo del mes, la app muestra una alerta roja destacada y un botón de un clic: *"Facturar como Extra (+precio)"*.
- Selector de modalidad:
  - **Abono**: Descuenta del cupo mensual correspondiente ($0 extra).
  - **Extra**: Toma el valor base del servicio del catálogo y suma el recargo si corresponde.
  - **Cortesía**: Asigna precio $0 pero registra el valor real comercial para el reporte.
  - **Puntual**: Para trabajos individuales sin abono.
- **Recargos por Urgencia**: Normal (0%), Prioritario (+25%), En el día (+50%) con recálculo dinámico en pantalla.

### 4. Cotizador & Armador de Presupuestos Multi-Producto (Estrategia Smaily)
- **Selección Visual desde el Catálogo con 1 Clic:** No tenés que escribir nombres ni precios. Simplemente tocás `[+]` en los servicios del catálogo (ej: *Flyer*, *Reel Extra*, *Aftermovie*) y se cargan automáticamente a la canasta con sus precios de lista.
- **Packs Preconfigurados Frecuentes (1 Clic):**
  - 🎂 **15 Años Full ($620.000):** Foto + Video completo (6h de evento + sesión previa + 100 fotos + video resumen + video de vida + reels).
  - 📸 **15 Años Solo Foto ($420.000):** 6h cobertura + previa + 100 fotos editadas.
  - 🍔 **Apertura Local / Combo:** Reel de inauguración + Pack 3 Historias en vivo + Conducción/Hosteo + Video Resumen Aftermovie (🎁 100% Bonificado de cortesía).
  - 🚀 **Lanzamiento Digital Mes 1 ($220.000):** Setup integral, identidad visual, bio, 6 posts y campaña inicial.
- **Cortesía Comercial como Gancho de Venta:** Podés marcar cualquier servicio como 100% Bonificado con 1 botón para que el cliente vea que le regalás valor comercial real sin cobrárselo.
- **Condiciones Comerciales Reales de Smaily:** Selector automático de cláusulas:
  - *Eventos Sociales:* Seña del 50% para congelar fecha, saldo previo al evento, entrega 30 a 45 días corridos.
  - *Gestión Mensual:* Cobertura en Zona Este de Mendoza, pauta Meta Ads independiente, revisión de honorarios cada 4 meses.
- **Exportación Inmediata:**
  - 💬 **WhatsApp Listo:** Con el tono, las cláusulas y la firma de *Sonri – Smaily Social Media*.
  - 📄 **Presupuesto PDF Oficial:** Con membrete formal y desglose listo para imprimir o enviar.
  - 💾 **Aprobar en SMC:** Da de alta al cliente eventual e inserta todos los trabajos en la cola de entregas de la agencia.

### 4. Catálogo Dinámico de Servicios (ABM Completo)
- Permite crear, modificar precios base y estimaciones de horas operativas.
- Filtro por categorías:
  - *Gestión y Redes*
  - *Diseño Gráfico*
  - *Producción Audiovisual*
  - *Conducción y Voz*
  - *Desarrollo y Estrategia*

### 5. Liquidación Mensual y Exportación
- **Desglose completo y transparente**:
  - Abono base pactado.
  - Balance de cupos utilizados en el mes.
  - Listado cronológico de servicios extras realizados con fechas y recargos.
  - Listado de atenciones comerciales y cortesías (ahorro del cliente).
  - Total exacto a pagar.
- **Botón "Copiar para WhatsApp"**: Genera un mensaje formateado con emojis, negritas y datos listos para enviar al cliente por chat.
- **Botón "Descargar PDF"**: Genera un PDF formal y ejecutivo utilizando `html2pdf.js`.

---

## 📲 Cómo Instalar como PWA en tu Celular o PC
1. **En Google Chrome / Microsoft Edge (PC)**:
   - Hacé clic en el ícono de instalación en la barra de direcciones o abrí el menú (⋮) y seleccioná **"Instalar Social Media Center"**.
2. **En Android**:
   - Abrí el enlace en Chrome, tocá los 3 puntos (⋮) y seleccioná **"Agregar a la pantalla principal"**.
3. **En iPhone / iPad (Safari)**:
   - Tocá el botón de compartir (flecha hacia arriba) y elegí **"Agregar a pantalla de inicio"**.

---
Desarrollado con dedicación para **Smaily Social Media** 🚀

