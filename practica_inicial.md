# 📘 Práctica 1: Monitoreo de Leads con Google Analytics 4

**Objetivo:** Implementar un sistema de rastreo para un botón de "Venta Simulada", configurar Google Analytics 4 (GA4) para recibir datos personalizados y visualizar los resultados en un dashboard de Looker Studio.

---

## 1. Conceptos Fundamentales

Antes de tocar el código, entendamos la jerarquía de Google Analytics usando la **"Analogía de Netflix"**:

* **📺 La Cuenta (Account) = La Suscripción Familiar**
    * Es el "Dueño" legal. Aquí se gestionan los usuarios y la facturación.
* **👤 La Propiedad (Property) = Tu Perfil Personal**
    * Es el sitio web específico. Lo que sucede en este perfil no se mezcla con los otros.
    * *Regla:* Debemos crear una "Propiedad" para nuestro sitio web en GitHub Pages.

---

## 2. Implementación del Sitio Web

Crearemos un archivo `index.html` que simula una landing page con un botón de conversión.

### Código Fuente (`index.html`)

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Demo Analytics en Vivo</title>
    <!-- Google tag (gtag.js) -->
    <!---EN ESTA PARTE VA EL CODIGO DE GOOGLE ANALYTICS NOS DA-->
</head>

<body>
    <h1>Práctica de Monitoreo en Tiempo Real</h1>
    <p>Este sitio es nuestro laboratorio de pruebas.</p>

    <button id="btn-promo"
        style="padding: 20px; font-size: 20px; background-color: #4CAF50; color: white; cursor: pointer;">
        ¡SOLICITAR OFERTA!
    </button>

    <br><br>
    <a href="practica_avanzada.html" 
       style="display: inline-block; padding: 15px 30px; font-size: 18px; background-color: #2196F3; color: white; text-decoration: none; border-radius: 5px; cursor: pointer;">
        Ir a Práctica Avanzada
    </a>

    <script>
        // Simularemos la lógica del botón más adelante en la integración
        document.getElementById('btn-promo').addEventListener('click', function () {
            alert('¡Click enviado a Google Analytics!');

            // Función para enviar evento personalizado manualmente
            // gtag('event', 'nombre_evento', { parametros });
            if (typeof gtag !== 'undefined') {
                gtag('event', 'click_oferta_especial', {
                    'tipo_boton': 'promocion_verano', // Esto será nuestra dimensión personalizada
                    'valor_lead': 50
                });
            }
        });
    </script>
</body>

</html>
```
---

## 3. Configuración en Google Analytics 4

Para que GA4 entienda los datos que enviamos, debemos configurar los "casilleros" receptores.

### A. Crear Propiedad

1. Ir a **Administrar (⚙️)** > **Crear Propiedad**.
2. **Plataforma:** Web.
3. **URL:** Tu dominio de GitHub Pages (`usuario.github.io`).
4. Copiar el **ID de Medición** (`G-XXXXXXXX`) e instalarlo en el HTML.

### B. Definiciones Personalizadas (Crucial)

Si no hacemos esto, Google ignorará el detalle del botón.

1. Ir a **Administrar** > **Visualización de datos** > **Definiciones personalizadas**.
2. **Crear Dimensión personalizada:**
   - **Nombre:** Tipo de Boton
   - **Ámbito:** Evento
   - **Parámetro del evento:** `tipo_boton` (Debe ser idéntico al código JS).

---

## 4. Validación (Testing)

Para verificar que el código funciona en tiempo real:

Se necesita el uso de la extension de **Google Analytics Debugger**

[Google Analytics Debugger](https://chromewebstore.google.com/detail/jnkmfdileelhofjcijamephohjechhna?utm_source=item-share-cb)

1. Abrir el sitio web y Google Analytics al mismo tiempo.
2. En GA4, ir a **Administrar** > **DebugView**.
3. Hacer clic en el botón del sitio web.

**Resultado esperado:** Ver aparecer el evento `click_oferta_especial` en la línea de tiempo del DebugView.

---

## 5. Visualización en Looker Studio

Crearemos un reporte para ver los datos de forma gráfica.

### A. Conexión

1. Crear informe vacío en Looker Studio.
2. Seleccionar conector **Google Analytics**.
3. Elegir la Propiedad creada.

### B. El Problema del "(not set)"

Al crear un gráfico de pastel por "Tipo de Botón", veremos que la mayoría de los datos dicen `(not set)`.

**Razón:** GA4 captura todos los eventos (vistas de página, scrolls, etc.), y esos eventos automáticos no tienen nuestra etiqueta personalizada.

### C. La Solución (Filtro)

Debemos limpiar el gráfico para ver solo lo que nos importa.

1. Seleccionar el gráfico.
2. En la barra derecha, ir a **Filtros** > **Añadir un filtro**.
3. Configurar:
   - **Acción:** Incluir
   - **Campo:** Nombre del evento
   - **Condición:** Es igual a (=)
   - **Valor:** `click_oferta_especial`
4. Guardar.

**Resultado Final:** Un gráfico limpio que muestra al 100% los datos de nuestra promoción.