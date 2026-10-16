# Panel EUM

Panel de administración y seguimiento académico para las alumnas de **Educación Universitaria para las Mujeres** (Grupo KX / Ayuntamiento de Mérida). Un solo archivo (`index.html`), sin build ni dependencias propias — solo Firebase y PapaParse por CDN.

Este repo **no contiene datos de ninguna alumna**. Todo vive en Firestore, protegido por login. Si abres `index.html` sin configurar Firebase (ver abajo), la app corre en modo de prueba local (guarda en el `localStorage` de tu navegador, sin login) — útil para probar cambios antes de publicar.

## Configuración (una sola vez)

### 1. Crear el proyecto de Firebase
1. Entra a [console.firebase.google.com](https://console.firebase.google.com) con tu cuenta de Google y crea un proyecto nuevo (gratis, plan "Spark").
2. **Authentication** → pestaña *Sign-in method* → activa **Correo electrónico/contraseña**.
3. **Authentication** → pestaña *Users* → **Add user** → crea una cuenta por cada persona autorizada (tú, Maite, etc.) con su correo y una contraseña temporal. La app no tiene "registrarse", solo "iniciar sesión" — nadie más puede crearse una cuenta.
4. **Firestore Database** → **Crear base de datos** → modo producción, cualquier región.
5. En Firestore, pestaña **Reglas**, reemplaza todo por:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /eum_data/{doc} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   y publica.
6. **Configuración del proyecto** (ícono de engranaje) → **General** → sección "Tus apps" → **Agregar app → Web** (el ícono `</>`). No necesitas activar Firebase Hosting. Copia el objeto `firebaseConfig` que te muestra.

### 2. Conectar el código con tu proyecto
Abre `index.html`, busca `const firebaseConfig = {` cerca del final del archivo, y reemplaza los valores `"REEMPLAZA..."` con los que copiaste. No son datos secretos — la protección real la dan las Reglas de Firestore más el hecho de que solo existen las cuentas que creaste a mano.

### 3. Publicar en GitHub Pages
1. Crea un repositorio en GitHub (puede ser público — Pages gratis lo requiere) y sube este proyecto.
2. En el repo: **Settings → Pages → Build and deployment → Deploy from a branch**, elige la rama `main` y la carpeta `/ (root)`.
3. Espera un par de minutos y abre la URL que te da GitHub Pages.

### 4. Cargar el roster oficial la primera vez
Ya con el sitio publicado, entra con tu cuenta y ve a la pestaña **Administración → Importar roster inicial desde CSV**. Pega el contenido de tu roster base (columnas: `Matricula,NombreCompleto,Programa,Email,UltimoAcceso,Estatus,CalifFinal`) y da clic en Importar. Ese archivo nunca debe subirse a este repo — solo se pega directamente en la app, ya logueado.

Después, cada semana: pestaña **Seguimiento → Cargar nuevo reporte semanal de Talisis**, sube el CSV que envía la plataforma.

### 5. Calendario de materias
En la pestaña **Calendario** viven las materias programadas por carrera con su fecha de inicio y de fin (todo editable). La primera vez usa **Importar desde Excel** con el archivo CALENDARIZACIÓN EUM, revisa el borrador y aplícalo. Seguimiento compara el avance de cada alumna contra ese calendario (columna "Calendario" y sección en su ficha), evaluado a la fecha del último corte cargado. La tolerancia para "ligeramente atrasada" es `TOLERANCIA_CALENDARIO` al inicio del script. En la ficha de cada alumna, la sección **Ruta según calendario** muestra sus materias cerradas, las que lleva en curso (avance real contra el esperado de hoy, semana y módulo que le toca) y las próximas.

### 6. Seguimiento: prioridad, filtros y bitácora
Cada alumna activa recibe **una sola prioridad**:
- **Contactar** (rojo): rezago o materia atrasada contra el calendario (más de 50 puntos por debajo de lo esperado), 15 o más días sin entrar, o nunca ha entrado.
- **Vigilar** (ámbar): ligeramente atrasada (hasta 50 puntos por debajo de lo esperado), o entre 10 y 14 días sin entrar.
- **Al corriente** (verde): el resto. Si la carrera no tiene calendario cargado, se usa el semáforo por avance.

La tabla muestra el motivo, el **ritmo** frente al corte anterior (avanzó N puntos o sin avance), el último acceso y el último contacto. Los botones de filtro (Contactar, Vigilar, Rezago, Sin avance, Sin entrar 14+ d, Por contactar, Seguimiento vencido, Bajas…) se combinan con el buscador y las tarjetas por carrera, y la lista visible se puede **copiar (correos)** o **exportar a CSV**. Los umbrales están al inicio del script (`DIAS_INACTIVIDAD_ALERTA`, `DIAS_INACTIVIDAD_CRITICA`, `TOLERANCIA_CALENDARIO`).

En la ficha de cada alumna hay un resumen con **acción sugerida**, la **bitácora de contactos** (fecha, medio, nota y próximo seguimiento; se guarda en Firestore bajo `bitacora_idx` y `bitacora:<correo>` con las mismas reglas) y la **evolución por corte** de cada materia. Si cambia de correo, su bitácora se mantiene unida a su correo principal.

### 7. Asistencia a clases presenciales
La pestaña **Asistencia** registra la asistencia física de cada alumna, una sesión por semana (sábado; viernes en Contabilidad). En **Pasar lista** eliges carrera y sesión y marcas **asistencia, retardo, justificado, falta o baja**; se guarda al instante. **Historial** muestra la matriz alumna × sesión (clic en una celda para cambiarla). También se puede marcar una sesión como "sin clase" (no cuenta para nadie).

- **% de asistencia** = (asistencias + retardos) ÷ (asistencias + retardos + faltas); el justificado no suma ni resta. Solo cuentan las sesiones con al menos una marca en su carrera.
- **Faltas seguidas**: consecutivas desde la sesión más reciente; el justificado no corta la racha. `ASIST_FALTAS_AMARILLO` (2, Vigilar) y `ASIST_FALTAS_ROJO` (3, Contactar) están al inicio del bloque de asistencia en el script, junto con `ASIST_CLASES` (fecha de la primera sesión de cada carrera).
- Aparece en la tabla de Seguimiento (columna Asistencia y filtro Faltas seguidas), en el semáforo y en la ficha de cada alumna.
- **Importar historial**: CSV con columnas `Carrera, Alumna, Fecha, Estado` (opcional `Correo`); las alumnas se buscan por nombre en el roster y se muestra un borrador antes de guardar. **Exportar** genera el mismo formato.
- Se guarda en Firestore como `asistencia_idx` y `asistencia:<carrera>:<AAAA-MM>` (un documento por carrera y mes), con las mismas reglas de acceso.

## Qué NO subir nunca a este repo
- Los CSV de roster o de reportes semanales (ya están en `.gitignore`).
- `CONTEXTO_PANEL_EUM.md` u otras notas internas que mencionen alumnas reales.

## Estructura
- `index.html` — la app completa (HTML + CSS + JS en un solo archivo).
- Todo el estado (roster oficial, cortes semanales, exclusiones) vive en Firestore, colección `eum_data`, un documento por llave.
