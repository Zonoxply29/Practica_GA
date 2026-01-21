# 📘 Práctica 2: Métricas de Tiempo y Retención de Usuarios (Nivel Avanzado)

**Objetivo:** Implementar un sistema que detecte el idioma del usuario, mida exactamente cuántos segundos pasa consumiendo el contenido y visualice el promedio de retención en un dashboard.

---

## 1. Conceptos Clave: La Diferencia Vital

Antes del código, es fundamental distinguir los dos tipos de datos que enviaremos en el mismo paquete:

* **🔤 Dimensiones (Cualidad / Texto):**
    * Describen **QUÉ** o **QUIÉN**. (Ej. `Español`, `Japonés`).
    * *Regla:* No se pueden sumar matemáticamente.
* **1️⃣ Métricas (Cantidad / Número):**
    * Describen **CUÁNTO**. (Ej. `45 segundos`, `3 clics`).
    * *Regla:* Se pueden sumar y promediar.

> **La Misión:** Enviaremos Dimensiones (`idioma_anterior`) junto con una Métrica Calculada (`tiempo_en_idioma`).

---

## 2. Implementación del Código (El Cronómetro)

Crearemos un archivo `idiomas.html` que usa `Date.now()` para calcular la diferencia de tiempo entre dos acciones.

### Código Fuente (`idiomas.html`)

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Práctica 2: Retención de Idiomas</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 50px; background-color: #f4f4f4; }
        .lang-btn { padding: 15px 25px; font-size: 18px; cursor: pointer; margin: 10px; border-radius: 5px; border: none; background: white; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .lang-btn:hover { background-color: #e0e0e0; }
        #content-area { margin-top: 40px; font-size: 28px; color: #333; font-weight: bold; background: white; padding: 20px; border-radius: 10px; display: inline-block;}
    </style>
    <!-- Google tag (gtag.js) -->
    <!---EN ESTA PARTE VA EL CODIGO DE GOOGLE ANALYTICS NOS DA-->
</head>
<body>

    <h1>Selector de Idioma</h1>
    <p>Lee el texto y cambia de idioma. Google medirá cuánto tardaste.</p>

    <button class="lang-btn" onclick="cambiarIdioma('es')">🇪🇸 Español</button>
    <button class="lang-btn" onclick="cambiarIdioma('en')">🇺🇸 English</button>
    <button class="lang-btn" onclick="cambiarIdioma('jp')">🇯🇵 日本語</button>

    <div id="content-area">Hola, bienvenido a la prueba.</div>

    <script>
        // --- LÓGICA DEL CRONÓMETRO ---
        let idiomaActual = 'es';
        let tiempoInicio = Date.now(); // Marca de tiempo inicial

        const textos = {
            'es': 'Hola, bienvenido a la prueba.',
            'en': 'Hello, welcome to the test.',
            'jp': 'こんにちは、テストへようこそ。' 
        };

        function cambiarIdioma(nuevoIdioma) {
            if (nuevoIdioma === idiomaActual) return; 

            // 1. MATEMÁTICAS: Calcular tiempo transcurrido
            // (Tiempo Actual - Tiempo Inicio) / 1000 = Segundos
            let tiempoFin = Date.now();
            let segundos = Math.round((tiempoFin - tiempoInicio) / 1000);

            // 2. ENVÍO A GA4: Paquete Completo
            if (typeof gtag !== 'undefined') {
                gtag('event', 'cambio_de_idioma', {
                    'idioma_anterior': idiomaActual,    // Dimensión
                    'idioma_nuevo': nuevoIdioma,        // Dimensión
                    'tiempo_en_idioma': segundos        // Métrica (Número)
                });
            }

            // 3. ACTUALIZAR INTERFAZ Y RELOJ
            document.getElementById('content-area').innerText = textos[nuevoIdioma];
            idiomaActual = nuevoIdioma;
            tiempoInicio = Date.now(); // Reiniciar el cronómetro
        }
    </script>
</body>
</html>
```
---

## 3. Configuración Avanzada en GA4

Ir a **Administrar (⚙️)** > **Visualización de datos** > **Definiciones personalizadas**.

### A. Configurar Dimensiones (Textos)

En la pestaña **Dimensiones personalizadas**, crea estas DOS:

1. **Nombre:** Idioma Anterior  
   **Parámetro:** `idioma_anterior`

2. **Nombre:** Idioma Nuevo  
   **Parámetro:** `idioma_nuevo`

### B. Configurar Métricas (Números)

En la pestaña **Métricas personalizadas**, crea:

* **Nombre:** Segundos por Idioma
* **Parámetro:** `tiempo_en_idioma`
* **Unidad:** Segundos

---

## 4. Visualización en Looker Studio

### Paso A: Actualizar Campos (Obligatorio)

1. Ir a **Recurso** > **Gestionar fuentes de datos** > **Editar**.
2. Clic en **"Actualizar campos"** (Abajo izquierda) para que aparezca `Idioma Nuevo`.
3. Clic en **Hecho**.

### Paso B: Gráfico 1 - Tabla de Retención (Tiempo)

**Objetivo:** ¿Cuánto tiempo leen?

1. Insertar una **Tabla**.
2. **Dimensión:** `Idioma Anterior`.
3. **Métrica:** Busca el campo automático `Average Segundos por Idioma`.

### Paso C: Gráfico 2 - Matriz de Flujo (Origen vs. Destino)

**Objetivo:** ¿A qué idioma se cambian?

1. Insertar una **Tabla Dinámica** (Pivot Table).
2. **Dimensión de Fila:** `Idioma Anterior` (De dónde vienen).
3. **Dimensión de Columna:** `Idioma Nuevo` (Hacia dónde van).
4. **Métrica:** `Recuento de eventos` (Cuántas veces pasó).


# 🛠️ Guía de Solución de Problemas (Troubleshooting)

En esta sección detallamos cómo resolver los obstáculos técnicos más comunes al conectar GA4 con Looker Studio.

---

## 🔴 Error 1: La Tabla dice "No hay datos"

**Síntoma:** Tu gráfico aparece vacío con un mensaje gris que dice "No hay datos", a pesar de que ya hiciste las pruebas en el sitio web.

**Causa A: El Filtro Fantasma**
Estás reutilizando un gráfico de la práctica anterior que tiene un filtro activo (ej. "Solo Ofertas"). Como los eventos de idioma no son ofertas, el filtro bloquea todo.

* **Solución:**
    1.  Selecciona la tabla con el error.
    2.  En la barra derecha (**Configuración**), baja hasta el final a la sección **Filtros**.
    3.  Pasa el mouse sobre el filtro activo y haz clic en la **X** para eliminarlo.
    4.  *(Opcional)* Crea un filtro nuevo: `Incluir` > `Nombre del evento` es igual a `cambio_de_idioma`.

**Causa B: La Fecha Predeterminada**
Looker Studio suele mostrar "Los últimos 28 días" pero **excluye el día de hoy** por defecto.

* **Solución:**
    1.  Busca el selector de fecha en tu informe (o en las propiedades del gráfico).
    2.  Cambia el rango a **Personalizado**.
    3.  En el calendario, selecciona **Hoy** (o asegúrate de que la fecha de fin incluya el día actual).
    4.  Haz clic en **Aplicar**.

---

## 🔴 Error 2: Métrica Bloqueada en "AUT" (No deja cambiar a Promedio)

**Síntoma:** Intentas cambiar la agregación de `Suma` a `Media` (Promedio), pero la opción aparece en gris, dice "AUT" (Automática) y no responde al clic.

**Diagnóstico:**
Looker Studio protege las métricas originales de GA4 para evitar errores de cálculo. No permite editarlas directamente.

**Solución (La Técnica del Campo Automático):**
Looker Studio genera automáticamente versiones calculadas de tus métricas numéricas.

1.  Selecciona tu tabla.
2.  En la sección **Métrica**, elimina el campo actual (`Segundos por Idioma`) haciendo clic en la X.
3.  En la lista de campos disponibles (derecha), busca el campo que empieza con la palabra **Average** o **Promedio**.
    * *Ejemplo:* `Average Segundos por Idioma`.
4.  Arrastra ese campo a la sección de Métrica.
    * *Resultado:* Ahora verás el promedio calculado automáticamente sin tener que configurar nada.

---

## 🔴 Error 3: El tiempo se ve raro (00:04:33) y quiero ver segundos (273)

**Síntoma:** La tabla muestra un formato de reloj (`HH:MM:SS`) que confunde la lectura rápida, tú prefieres ver el número entero de segundos.

**Diagnóstico:**
El "Tipo de dato" está configurado como **Duración**.

**Solución:**
1.  En la configuración del gráfico, ve a la sección **Métrica**.
2.  Pasa el mouse sobre tu métrica (`Average Segundos...`) y haz clic en el ícono de lápiz ✏️ (o el ícono "123" a la izquierda del nombre).
3.  Busca el menú desplegable **Tipo**.
4.  Cambia de `Duración` a **Número** > **Numérico**.
    * *Resultado:* El valor cambiará de `00:04:33` a `273.5`.

---

## 🔴 Error 4: Agregué una métrica en GA4 y no aparece en Looker Studio

**Síntoma:** Creaste "Segundos por Idioma" en GA4, pero cuando la buscas en la lista de campos de Looker Studio, no existe.

**Diagnóstico:**
Looker Studio no se sincroniza en tiempo real. Trabaja con una "foto" de tus datos del momento en que lo conectaste.

**Solución (Botón de Pánico):**
1.  En el menú superior de Looker Studio, haz clic en **Recurso**.
2.  Selecciona **Gestionar las fuentes de datos añadidas**.
3.  Identifica tu fuente ("Web GitHub Pages") y haz clic en **Editar** (ícono de lápiz).
4.  En la esquina inferior izquierda, haz clic en el botón **Actualizar campos**.
5.  Si hiciste todo bien, aparecerá un aviso: *"Se han encontrado campos nuevos"*.
6.  Haz clic en **Hecho**.

## 🔴 Error 5: Veo "(not set)" en mis dimensiones nuevas (Latencia)

**Síntoma:**
Tu filtro funciona (ya no ves 65 eventos, sino solo los 15 correctos), pero en las columnas de `Idioma Anterior` o `Idioma Nuevo` aparece el texto **(not set)** en lugar de "es", "en" o "jp".

**Diagnóstico (La Regla de las 24 Horas):**
Aunque el código esté perfecto, Google Analytics 4 tarda entre **24 y 48 horas** en procesar las **Dimensiones Personalizadas Nuevas** para que estén disponibles en la API de reportes (Looker Studio).

**La Prueba de la Verdad (Validación):**
Para confirmar que **NO** es tu culpa, haz esto:
1.  Ve a **GA4** > **Administrar** > **DebugView**.
2.  Haz un cambio de idioma en tu sitio web.
3.  Si en el DebugView ves el parámetro `idioma_nuevo` con el valor correcto (ej. `jp`), **TU PRÁCTICA ES UN ÉXITO**.
4.  **Solución:** Paciencia. Mañana por la mañana, Looker Studio mostrará los textos correctamente.