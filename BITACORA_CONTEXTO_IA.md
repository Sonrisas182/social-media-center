# BITÁCORA TÉCNICA Y DE CONTEXTO: SOCIAL MEDIA CENTER (SMC)
> **Documento de Traspaso, Arquitectura y Estado de Proyecto para Inteligencias Artificiales**  
> **Cliente / Agencia:** Smaily Social Media (Sonri) — Mendoza, Argentina (Zona Este: Rodeo de la Cruz, San Martín, Junín, Maipú).  
> **Repositorio Oficial:** `https://github.com/Sonrisas182/social-media-center`  
> **Rama Principal:** `main`  
> **Fecha de Actualización:** Noviembre 2026  

---

## 1. RESUMEN EJECUTIVO Y PROPÓSITO DEL SISTEMA
**SOCIAL MEDIA CENTER (SMC)** es una Progressive Web App (PWA) Mobile-First construida en una arquitectura Single Page Application (SPA) monolítica limpia y modular. Su objetivo es gestionar de punta a punta la operación, el alcance de contenidos y las finanzas de **Smaily Social Media**, agencia de marketing digital, producción audiovisual y servicios creativos liderada por "Sonri".

La aplicación resuelve tres dolores críticos del negocio:
1. **Evitar fugas de dinero por exceso de alcance:** Monitorear en tiempo real los cupos consumidos de los clientes con abono mensual (Reels, Flyers, Historias, Rodajes) y alertar para facturar como **Extra** cualquier pedido fuera de contrato.
2. **Controlar la sobrecarga operativa:** Medir la carga semanal en horas de rodaje y edición mediante un **Semáforo Operativo** inteligente (<20 hs, 20-35 hs, >35 hs).
3. **Agilizar la venta multi-producto y presupuestación:** Cotizar combos para eventos (15 años, bodas) o aperturas comerciales (hamburgueserías, gastronomía, PyMEs) en segundos, aplicando bonificaciones de cortesía comercial (100% OFF) como gancho de venta, con exportación directa a WhatsApp y PDF formal.

---

## 2. STACK TECNOLÓGICO Y ARQUITECTURA
- **Formato de Entrega:** SPA autónoma en un único archivo `index.html` asistido por assets PWA (`manifest.json`, `sw.js`, `icon-192.svg`, `icon-512.svg`).
- **UI & Estilos:** Tailwind CSS (CDN oficial) configurado en Dark Mode profundo (`slate-950` / `slate-900` con acentos `indigo-500` para marca y `emerald-400` para finanzas).
- **Iconos:** Lucide Icons vía CDN con hidratación dinámica (`lucide.createIcons()`).
- **Backend & Base de Datos:** Supabase JS Client v2 vía CDN (`@supabase/supabase-js@2`), conectado a PostgreSQL en la nube.
- **Exportación:** `html2pdf.js` para PDFs vectoriales de liquidaciones y presupuestos; API nativa `navigator.clipboard` con fallback para WhatsApp.
- **PWA & Offline:** Manifiesto web standalone, Service Worker con estrategia de caché estática y bypass para peticiones a Supabase API.

### Credenciales de Supabase Configuradas:
- `SUPABASE_URL`: `"https://zcvkylmjikdcpkhpxwqg.supabase.co"`
- `SUPABASE_ANON_KEY`: `"sb_publishable_CZJw-iheCX3OwzuJ7LBwXQ_daFyvKwf"`

---

## 3. ESQUEMA DE BASE DE DATOS (SUPABASE SCHEMA)

### Tabla 1: `smc_servicios`
Almacena el catálogo base de servicios y tarifas operativas:
- `id` (uuid, PK, autogenerado)
- `nombre` (text, not null)
- `categoria` (text, not null) — *Categorías: 'Gestión y Redes', 'Diseño Gráfico', 'Producción Audiovisual', 'Conducción y Voz', 'Desarrollo y Estrategia'*
- `precio_base` (numeric, not null)
- `horas_estimadas` (numeric, not null)
- `activo` (boolean, default true)
- `created_at` (timestamptz)

### Tabla 2: `smc_clientes`
Almacena los clientes con abono fijo o clientes eventuales:
- `id` (uuid, PK, autogenerado)
- `nombre` (text, not null)
- `unidad_negocio` (text, not null) — *'Smaily', 'Independiente', 'Eventual'*
- `valor_abono` (numeric, default 0)
- `cupo_reels` (integer, default 0)
- `cupo_flyers` (integer, default 0)
- `cupo_historias` (integer, default 0)
- `cupo_rodajes` (integer, default 0)
- `activo` (boolean, default true)
- `created_at` (timestamptz)

### Tabla 3: `smc_trabajos`
Registro de todos los pedidos, entregas y tareas:
- `id` (uuid, PK, autogenerado)
- `cliente_id` (uuid, FK -> `smc_clientes.id`, not null)
- `servicio_id` (uuid, FK -> `smc_servicios.id`, **nullable** para servicios a medida)
- `descripcion` (text)
- `fecha` (date, 'YYYY-MM-DD')
- `modalidad` (text: `'Abono'`, `'Extra'`, `'Cortesia'`, `'Puntual'`)
- `tipo_cupo` (text, nullable: `'reels'`, `'flyers'`, `'historias'`, `'rodajes'`)
- `recargo_urgencia` (**numeric**: `0` para Normal 0%, `25` para Prioritario +25%, `50` para En el día +50%. *Nota crítica: es numérico, no string*)
- `precio_final` (numeric, monto facturable real)
- `valor_real_cortesia` (numeric, monto bonificado 100% OFF para reportes de fidelización)
- `horas_tarea` (numeric, alimenta el semáforo semanal)
- `estado` (text: `'Pendiente'`, `'En Proceso'`, `'Entregado'`, `'Cobrado'`)
- `created_at` (timestamptz)

---

## 4. MÓDULOS DESARROLLADOS Y FUNCIONALIDADES

### 🔒 Pantalla de Bloqueo por PIN (Seguridad de Acceso)
- PIN inicial: `1234` (personalizable en Ajustes y respaldado en `localStorage`).
- Teclado numérico táctil en pantalla con respuesta háptica (`navigator.vibrate`), animación de temblor ante error y compatibilidad con teclado físico.
- Botón de bloqueo rápido en header y settings.

### 📊 Dashboard Principal ("Mi Situación")
- **Selector de Mes Dinámico:** Navegación mes a mes con recálculo global instantáneo.
- **4 KPIs Financieros:**
  1. *Facturación Fija:* Suma de `valor_abono` de clientes activos.
  2. *Adicionales y Extras:* Suma de `precio_final` de trabajos en modalidad 'Extra' o 'Puntual' del mes.
  3. *Cortesías Bonificadas:* Suma de `valor_real_cortesia` del mes (resalta el ahorro generado al cliente para fidelización).
  4. *Total Proyectado:* Fijo + Extras acumulados.
- **Semáforo de Carga Operativa:**
  - Suma las `horas_tarea` de trabajos con estado `'Pendiente'` o `'En Proceso'` de la semana.
  - 🟢 **Verde (< 20 hs):** Carga óptima / disponible para nuevos rodajes.
  - 🟡 **Amarillo (20 a 35 hs):** Carga moderada / capacidad ajustada.
  - 🔴 **Rojo (> 35 hs):** Sobrecarga crítica / alerta para aplicar recargo de urgencia o reprogramar.
- **Cola de Entregas Pendientes:** Lista interactiva con botones directos para marcar como **"Entregado"** o **"Cobrado"** en 1 clic.

### 👥 Gestión de Clientes y Control de Alcance
- Filtros por unidad (`Todos`, `Smaily`, `Independiente`, `Eventual`) y buscador predictivo.
- **Barras de progreso en tiempo real:** Muestran el consumo mensual exacto de **Reels**, **Flyers**, **Historias** y **Rodajes** respecto a los cupos pactados. Al superar el límite, emite la alerta visual roja **"AGOTADO"**.
- Acceso directo a la **Liquidación Mensual** y creación de pedidos directos por cliente.

### ⚡ Detector de Extras y Carga Rápida (`+ Nuevo Pedido`)
- Formulario modal accesible desde cualquier pantalla (FAB flotante).
- Vinculado al catálogo de servicios y clientes.
- Auto-detección de cupos: Al elegir servicio sugiere el tipo de cupo y horas estimadas. Si el cupo está agotado, muestra alerta roja y botón directo: *"Facturar como Extra (+precio)"*.
- Modalidades: Abono ($0), Extra (precio base + recargo urgencia), Cortesía ($0 cobrado, valor guardado en cortesía), Puntual (clientes sin abono).

### 🧾 Liquidación Mensual y Exportación
- Desglose formal por cliente: Abono mensual pactado + Cupos consumidos + Lista cronológica de Extras + Lista de Cortesías bonificadas + Total general a pagar.
- **Botón "Copiar para WhatsApp":** Mensaje estructurado con negritas, emojis y totales listos para enviar al cliente.
- **Botón "Descargar PDF":** Documento corporativo generado con `html2pdf.js`.

---

## 5. EVOLUCIÓN BASADA EN EL FLUJO REAL DE LA AGENCIA

A partir de los presupuestos reales compartidos por Sonri (clientes corporativos como **CLU**, eventos como **15 Años en Salón María Victoria**, industrias como **MS Metalúrgica Sánchez**, marcas como **Vinos** y marcas personales como **Maxi Segura**), se incorporaron las siguientes mejoras avanzadas:

### 🚀 A. Alta de Clientes al Vuelo (Zero Fricción)
- En el modal de pedidos y en el cotizador, se agregó el selector: **`[ 👤 Habitual | ⚡ Ocasional / Nuevo ]`**.
- Permite escribir directamente el nombre del prospecto o evento (ej: *"Hamburguesería Rock & Burger"* o *"15 de Camila"*).
- El sistema lo da de alta automáticamente en Supabase como cliente **`Eventual`** (abono $0 y cupos 0), eliminando la necesidad de ir a la pestaña de clientes a configurar un perfil manual.

### 🛒 B. Cotizador Visual Multi-Producto con 1 Clic (Quick Catalog Picker)
- Se reemplazó el formulario manual de carga por un **Catálogo Visual Interactivo**:
  - Filtros por categoría de servicio.
  - Tarjetas de servicio con precio y steppers de adición rápida: **`[-]  [ Cantidad ]  [+]`**.
  - Si un cliente pide *1 Flyer, 2 Reels y 1 Aftermovie*, solo se tocan los botones `[+]` de cada servicio y se añaden automáticamente con sus nombres y precios de catálogo.
- **Botón de Cortesía Comercial `[ 🎁 Bonificar (100% OFF) ]`:** Permite marcar en la canasta cualquier servicio como regalo comercial para convencer al cliente (ej. el video resumen de la hamburguesería), mostrando:
  - Valor de lista: ej. `$163.000`
  - Ahorro en cortesías: `-$32.000`
  - Total cerrado del combo: `$131.000`

### ⚡ C. Plantillas Rápidas de Presupuestos Reales (1 Clic)
Botones preconfigurados en la cabecera del cotizador basados en las propuestas reales de Smaily:
1. 🎂 **15 Años Full ($620.000):** 6 hs evento + previa + 100 fotos editadas + video resumen + video de vida + video para redes.
2. 📸 **15 Años Solo Foto ($420.000):** 6 hs cobertura + previa + 100 fotos.
3. 🍔 **Apertura de Local / Combo:** Reel inauguración ($30k) + Pack 3 historias ($16k) + Hosteo ($85k) + Video Resumen ($32k bonificado al 100%).
4. 🚀 **Lanzamiento Digital Mes 1 ($220.000):** Setup integral, identidad visual, bio, 6 publicaciones y pauta inicial.

### 📌 D. Cláusulas y Condiciones Comerciales Reales
Selector automático en el cotizador que inyecta las cláusulas contractuales exactas de Smaily en WhatsApp y PDF:
- **Eventos Sociales:** Seña del 50% para congelar fecha en agenda, saldo restante previo a la noche del evento, entrega en 30 a 45 días corridos en formato digital.
- **Gestión Mensual:** Incluye coberturas en Zona Este de Mendoza, pauta Meta Ads independiente de honorarios, revisión cuatrimestral de precios por costos operativos.
- **Firma Oficial:** *"Sonri – Smaily Social Media | Producción Audiovisual, Marketing & Contenido Creativo • Mendoza"*.

### 🌐 E. Ficha de Ecosistema Digital en Clientes
En la pestaña de Clientes y modal de edición:
- **Redes trabajadas (badges):** Instagram, Google Maps, WhatsApp Business, TikTok, Facebook, YouTube, LinkedIn.
- **Etapa de madurez:** *⚡ Desde Cero*, *🛠️ Optimización de Perfil*, *🚀 Gestión y Crecimiento Activo*.
- **Servicios de ecosistema:** Guionado & Copywriting, Posicionamiento en Maps (SEO Local), Setup de WhatsApp Business, Rodaje presencial.

---

## 6. ESTADO DE SINCRONIZACIÓN Y DESPLIEGUE
- Repositorio Git local inicializado en rama `main`.
- Código sincronizado con el repositorio remoto: `https://github.com/Sonrisas182/social-media-center.git`.
- Token de acceso personal utilizado para el push inicial y removido de la configuración local por seguridad.
- PWA lista para publicación directa en **GitHub Pages** desde `Settings -> Pages -> Deploy from branch (main / root)`.

---

## 7. PAUTAS PARA CUALQUIER IA QUE CONTINÚE EL DESARROLLO
1. **Preservar la simplicidad monolítica:** El archivo `index.html` contiene toda la lógica reactiva en el objeto global `App` y el estado centralizado `State`. Evitar introducir frameworks pesados (React/Vue/Webpack/Vite) a menos que el usuario lo solicite expresamente; la ligereza actual en Vanilla JS + CDN permite abrir el archivo directamente en cualquier navegador y dispositivo.
2. **Respetar tipos de datos en Supabase:** El campo `recargo_urgencia` en `smc_trabajos` es estrictamente `numeric` (`0`, `25`, `50`), no string.
3. **Mantener el enfoque Mobile-First:** Todas las modificaciones de UI deben probarse con la barra de navegación inferior móvil (`pb-safe`) y el FAB flotante en cuenta.
4. **Respetar la identidad de marca de Sonri:** Smaily Social Media opera en Mendoza (Zona Este) y combina el rol de agencia de redes con productora audiovisual de eventos y hosteo. Toda plantilla y comunicación debe reflejar este tono cercano, profesional y enfocado en la rentabilidad de los combos.



---

## 8. ACTUALIZACIÓN A VERSIÓN 2.0 (FASE 2 - IMPLEMENTACIONES CRÍTICAS)

### 🔒 1. Seguridad, Limpieza y Persistencia de PIN
- **Bugfix de Persistencia en localStorage:** Corregida la sincronización de PIN entre sesiones. Se unificaron las claves `smc_user_pin_v2` y `smc_user_pin`, asegurando que el PIN personalizado nunca retorne a `"1234"` tras recargar o cerrar el navegador.
- **Enmascaramiento en Bloqueo:** Eliminada toda pista visual `(Default: 1234)`. Se incorporó un input enmascarado `type="password"` (`#lockPinInput`) sincronizado con los 4 indicadores luminosos y el teclado físico/virtual, garantizando confidencialidad absoluta en entornos compartidos.
- **Eliminación Total del Botón Seed:** Removida por completo la tarjeta y el botón de carga inicial del DOM en Ajustes para evitar reinicios o sobreescrituras accidentales de la base de datos de producción.
- **Lanzador Directo de WhatsApp en Liquidación:** Arreglado el botón de liquidación para que abra automáticamente `https://wa.me/?text=...` en una nueva pestaña/aplicación nativa de WhatsApp, además de respaldar el texto copiado al portapapeles.

### 📈 2. Control de Inflación y Aumentos Trimestrales
- **Campo "Fecha de Último Aumento":** Integrado en el perfil extendido de cada cliente (`smc_client_profiles[id].fecha_ultimo_aumento`).
- **Alerta Visual Automática (90 Días):** Si transcurrieron 3 meses o más desde el último ajuste, la tarjeta del cliente destaca una insignia amarilla: `🟡 Aumento Pendiente (X meses sin ajuste)`.
- **Modal de Ajuste Rápido (`inflationModal`):**
  - Botones de cálculo inmediato: `+15%`, `+20%`, `+25%`, `+30%` y campo de nuevo fee mensual editable.
  - Generador dinámico de mensaje profesional y cordial para WhatsApp explicando los costos operativos.
  - Botón *"✅ Aplicar Nuevo Valor y Actualizar Fecha a Hoy"*: actualiza `valor_abono` directamente en Supabase y resetea la fecha de aumento al día actual en un solo clic.

### ⚡ 3. Liquidación Flexible (Mensual o Semanal)
- **Selector de Período en Liquidación:** Alternador `[ 📅 Mes Completo | ⚡ Solo esta Semana (7 días) ]`.
- **Filtro Semanal:** Filtra dinámicamente los extras y cortesías comprendidos en los últimos 7 días con prorrateo de abono (`abono / 4`).
- **Adaptación Integral:** Modifica los títulos, subtotales, texto de WhatsApp y encabezado del PDF corporativo según la periodicidad elegida.

### ⏳ 4. Presupuestos Emitidos con Vencimiento a 7 Días
- **Historial de Cotizaciones (`smc_presupuestos_emitidos`):** Guarda la propuesta con fecha de emisión y vencimiento automático a 7 días.
- **Semáforo de Vigencia:** Badges reactivos `🟢 Vigente (Vence en X días)` vs `🔴 Vencido (Venció hace X días)`.
- **Conversión Directa:** Botón *"Aprobar (Pasa a Trabajos)"* que inserta automáticamente cada ítem del presupuesto en Supabase `smc_trabajos` bajo el cliente correspondiente (o lo crea si era prospecto) y marca la cotización como aprobada.
- **Reenvío con Urgencia Comercial:** Mensaje de WhatsApp que recuerda al cliente la fecha límite para congelar tarifas y agenda.

### 🎥 5. Agenda Simple de Rodajes & Eventos (Prevención de Superposición)
- **Módulo de Agenda Cronológica (`tab-agenda` & Widget en Dashboard):** Lista ordenada por fecha y hora con insignias relativas (`🔴 ¡HOY!`, `🟡 Mañana`, `🔵 En X días`, `⚪ Pasado`).
- **Campos del Rodaje:** Cliente, Fecha, Hora, Lugar (con enlace directo a Google Maps), Costo de Viaje/Viáticos ($) y Plan de Grabación.
- **Detector de Colisiones:** Al seleccionar una fecha ya ocupada por otro evento, despliega una alerta roja destacada: `⚠️ Ya tienes un evento registrado este día: [Cliente] en [Lugar] a las [Hora] hs.`, exigiendo confirmación explícita para guardar.

### 💰 6. Rentabilidad Neta y Gastos Fijos
- **Parámetros Financieros en Ajustes:**
  - *"Mi Valor Hora Sugerido ($)"*: Tarifa de referencia profesional.
  - *"Gastos Fijos del Mes ($)"*: Monto único para herramientas, internet, impuestos y suscripciones (`smc_finanzas_config`).
- **Hero Card en Dashboard:**
  - Métrica de **"Ganancia Limpia Estimada"** = `Total Proyectado − Gastos Fijos del Mes` en tipografía esmeralda de alto contraste.
  - Comparativa con el valor hora sugerido para evaluar la rentabilidad real de la agencia en Mendoza.
