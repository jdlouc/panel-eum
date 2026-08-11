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

## Qué NO subir nunca a este repo
- Los CSV de roster o de reportes semanales (ya están en `.gitignore`).
- `CONTEXTO_PANEL_EUM.md` u otras notas internas que mencionen alumnas reales.

## Estructura
- `index.html` — la app completa (HTML + CSS + JS en un solo archivo).
- Todo el estado (roster oficial, cortes semanales, exclusiones) vive en Firestore, colección `eum_data`, un documento por llave.
