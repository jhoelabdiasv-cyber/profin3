# SIMTELE-WEB: Panel interactivo de prevención y monitoreo de Mi Teleférico

Proyecto académico de **Programación Web I**. Página única desarrollada con **HTML5, CSS3 y JavaScript puro**, sin frameworks. El acceso real con Google utiliza únicamente la biblioteca oficial **Google Identity Services**, que se carga solo si se configura un Client ID OAuth.

> **Aviso didáctico:** todos los datos, rangos, mediciones y umbrales son simulaciones educativas. **No son límites operativos reales de Mi Teleférico** ni deben utilizarse para tomar decisiones de seguridad.

## Integrantes del equipo

- Integrante 1: __________________________________
- Integrante 2: __________________________________
- Integrante 3: __________________________________
- Integrante 4: __________________________________

## Estructura de carpetas

```text
proyecto-simtele-web/
├── index.html
├── css/
│   └── styles.css
├── js/
│   └── app.js
└── README.md
```

## Inicio y cierre de sesión (mejora adicional)

Al abrir `index.html` se muestra **la pantalla de acceso de SIMTELE-WEB**, no una página que imita el acceso de Google. Tiene dos modalidades:

- **Modo demostración:** botón **Entrar en modo demostración**, sin contraseña y sin internet. Habilita los cuatro módulos. La sesión de demostración se mantiene al refrescar la pestaña; **Cerrar sesión** vuelve a la pantalla de acceso sin borrar los registros. Sirve para demostrar el proyecto sin una cuenta real.
- **Google real (opcional, con configuración):** botón **Continuar con Google**. Abre la ventana oficial de Google, nunca solicita contraseñas en SIMTELE-WEB y lee el perfil desde el endpoint de Google usando un token de acceso temporal. El token se conserva solo en memoria y no se escribe en localStorage. **Cerrar sesión** elimina el acceso local y el token de la memoria, **pero no cierra la cuenta Google en otros sitios**.

Los datos de los cuatro módulos se guardan en **localStorage del navegador actual**. El modo demostración utiliza la clave de la versión anterior `simtele_web_datos_v1`. Con Google, cada cuenta posee una clave separada por su identificador `sub` (`simtele_web_google_...`). **Esto NO sincroniza datos en Google Drive ni crea un servidor compartido**: si se abre otro navegador u otro dispositivo, los registros no aparecerán. El botón **Limpiar todos los datos guardados** borra solo los datos del usuario actual.

### Cómo habilitar el inicio real con Google

1. Entra a [Google Cloud Console](https://console.cloud.google.com/) y crea o selecciona un proyecto.
2. Configura Google Auth Platform / pantalla de consentimiento OAuth. Para pruebas, añade los correos que vayan a utilizarlo si tu aplicación está en estado de prueba.
3. Crea un **ID de cliente OAuth 2.0**, tipo **Aplicación web**.
4. En **Orígenes autorizados de JavaScript**, agrega la dirección real desde la que abrirás la página: por ejemplo `http://localhost:8000` para desarrollo y `https://TU-USUARIO.github.io` para GitHub Pages (sin el nombre del repositorio en el origen).
5. Abre `js/app.js` y reemplaza:

   ```js
   const CLIENT_ID_GOOGLE = "REEMPLAZAR_POR_TU_CLIENT_ID.apps.googleusercontent.com";
   ```

   por el ID completo de cliente que Google te entregue. **Nunca pegues aquí una contraseña ni un secreto de cliente (`client_secret`)**.
6. Ejecuta un servidor local en la raíz del proyecto, por ejemplo con `python -m http.server 8000`, y visita `http://localhost:8000/`. No abras el archivo `index.html` mediante `file://` para probar Google.
7. Presiona **Continuar con Google**; se abrirá la ventana de cuentas de Google y, al autorizar, aparecerá el panel con el nombre y el correo de la cuenta.

El proyecto funciona sin configurar Google si se utiliza el **modo demostración**. Google requiere conexión a internet, HTTPS (o localhost autorizado) y el Client ID correcto. La autenticación en esta implementación es del **lado del navegador y apropiada para una demostración educativa**; no sirve como control de acceso seguro a datos sensibles sin un backend que valide credenciales y aplique permisos. La biblioteca Google Identity Services es la única dependencia externa y no se utiliza en el modo demostración.

Documentación oficial: [Obtener Client ID](https://developers.google.com/identity/oauth2/web/guides/get-google-api-clientid?hl=es-419) y [Modelo de token](https://developers.google.com/identity/oauth2/web/guides/use-token-model?hl=es-419).

## Módulos

1. **Detención de emergencia:** ingresa velocidad en km/h (0–120) y tiempo de reacción en segundos (0–60). Calcula `d = (v / 3.6) × t` y compara el resultado con el **umbral didáctico ficticio de 50 m**. Ejemplo: 72 km/h y 3 s → **60 m**.
2. **Mantenimiento:** agrega componentes con nombre, horas de uso (0–100 000) y estado (normal, revisión o urgente). Los registros se guardan en un arreglo de objetos usando `push()`, se muestran en una tabla dinámica y pueden eliminarse con `splice()`.
3. **Viento y oscilación:** introduce **exactamente cinco** mediciones ficticias (0–120 unidades simuladas), calcula suma, promedio y máximo usando arreglos, `for` y `forEach`. Clasifica el **promedio** como bajo (menos de 20), moderado (desde 20 y menos de 40) o alto (40 o más). Los rangos son didácticos, no reales.
4. **Sensores:** registra código, tipo y lectura (0–100 unidades didácticas). El código debe cumplir exactamente la expresión regular `/^SEN-\d{3}$/` usando `.test()`, por ejemplo `SEN-001`. Se validan campos vacíos, valores no numéricos y rangos. Los sensores válidos aparecen en una tabla y también pueden eliminarse.

## Instrucciones de uso

1. Abre **`index.html`** en un navegador moderno o ejecuta un servidor web local en esta carpeta; para Google real, usa `localhost` o HTTPS.
2. Presiona **Entrar en modo demostración** (o **Continuar con Google** después de configurarlo).
3. Utiliza el menú superior para pasar a uno de los cuatro módulos.
4. Completa los formularios y presiona el botón del módulo. Los resultados y errores aparecen **sin recargar** la página.
5. En mantenimiento y sensores, presiona **Eliminar** para quitar una fila.
6. Los registros se guardan automáticamente en **`localStorage`** cuando el navegador lo permite. Para borrar todos los registros y resultados, utiliza el botón **Limpiar todos los datos guardados** del pie de página. Para regresar a la pantalla de acceso, usa **Cerrar sesión** (no borra los datos).

## Pruebas mínimas de aceptación

| Prueba | Pasos | Resultado esperado |
| --- | --- | --- |
| Detención | Velocidad `72`, tiempo `3`; calcular | **60 m** y aviso de que supera 50 m ficticios |
| Mantenimiento | Agregar `Polea A` y `Motor B`; eliminar una fila | Primero **2 filas**, después **1 fila** |
| Viento | Introducir `10`, `20`, `30`, `40`, `50` | **Promedio 30**, **máximo 50**, clasificación moderada |
| Sensores | Registrar `SEN-001` con tipo y lectura; intentar `SEN-1` y código vacío | Primer registro válido; los otros muestran errores y no se agregan |
| Adaptabilidad | Abrir en escritorio y vista móvil | Navegación, formularios, tablas y botones utilizables |
| Inicio/cierre de sesión | Entrar en modo demo, agregar registro, cerrar sesión y volver a entrar | La aplicación se oculta al salir y los datos reaparecen al entrar |

Pruebas negativas recomendadas: horas no numéricas, una medición vacía, valores fuera de rango, código de sensor incorrecto, lectura no numérica; no deben agregarse registros inválidos.

## Publicación y entrega en GitHub

1. Crea un repositorio público en GitHub y sube el contenido de esta carpeta **sin modificar las rutas** de `css/styles.css` y `js/app.js`.
2. En GitHub, abre **Settings → Pages → Build and deployment** y selecciona la rama principal (`main`) con carpeta raíz (`/root`).
3. Cuando GitHub Pages publique la página, prueba los cuatro módulos en la URL pública.
4. Entrega a la docente **dos enlaces**: URL pública de GitHub Pages y URL del repositorio. Esos enlaces deben generarse en la cuenta del equipo; no están incluidos artificialmente en este proyecto.

## Detalles técnicos

- Etiquetas semánticas `header`, `nav`, `main`, cuatro `section` y `footer`.
- Campos con `label` asociado, texto de ayuda y mensajes de error accesibles.
- `addEventListener` para todos los eventos y `preventDefault()` para evitar recargas.
- Diseño adaptable con media queries para celulares y escritorios.
- Persistencia local: `simtele_web_datos_v1` en modo demostración y `simtele_web_google_<sub>` en modo Google, sin servidor de datos.
- Google real mediante OAuth 2.0 (token model) y llamada REST al endpoint `userinfo`; **requiere configuración propia**, no funciona sin un Client ID válido.
- Renderizado de datos de usuario con `textContent` para evitar insertar HTML arbitrario.

**Importante:** el archivo abierto directamente puede tener restricciones de `localStorage` en algunos navegadores o modos privados. El resto de la aplicación seguirá funcionando durante la sesión; para una prueba de persistencia consistente, usa un servidor web local o GitHub Pages.
