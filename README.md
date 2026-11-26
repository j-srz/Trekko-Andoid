# Trekko

App Android de turismo (Fase 1) hecha con **Kotlin + Jetpack Compose + Firebase**. Proyecto escolar desarrollado por un equipo de 5 personas en un sprint de 3 días.

**Recorrido principal:** registro/login → Inicio → directorio de lugares con filtros → ficha del lugar (llamada y WhatsApp) → generador de itinerario con costo y duración → guardar → consultar sin internet.

---

## Índice

1. [Requisitos](#1-requisitos)
2. [Cómo correr el proyecto](#2-cómo-correr-el-proyecto)
3. [Configurar Firebase](#3-configurar-firebase)
4. [Cargar los datos de prueba](#4-cargar-los-datos-de-prueba)
5. [Estructura del repositorio](#5-estructura-del-repositorio)
6. [Flujo de trabajo con Git](#6-flujo-de-trabajo-con-git)
7. [Equipo y responsabilidades](#7-equipo-y-responsabilidades)
8. [Guion de demostración](#8-guion-de-demostración)
9. [Generar el APK](#9-generar-el-apk)
10. [Problemas comunes](#10-problemas-comunes)
11. [Pendientes (fuera del sprint)](#11-pendientes-fuera-del-sprint)

---

## 1. Requisitos

- Android Studio (versión estable reciente) con el SDK de Android instalado.
- JDK incluido con Android Studio (no hace falta instalar otro).
- Un emulador o un teléfono con depuración USB activada.
- Git.
- Node.js (solo el Integrante 4 y quien necesite recargar los datos de prueba).

## 2. Cómo correr el proyecto

```bash
git clone <URL-DEL-REPOSITORIO>
cd Trekko
```

1. Abre la carpeta con Android Studio y espera a que termine la sincronización de Gradle.
2. Coloca el archivo `google-services.json` en `app/` (ver sección 3).
3. Selecciona un emulador o teléfono y presiona **Run ▶**.

## 3. Configurar Firebase

El equipo usa **un único proyecto Firebase** con Authentication (correo y contraseña), Firestore y Storage activados.

1. Pásale tu huella **SHA-1** al Integrante 4 para que la registre en Firebase:
   Android Studio → pestaña **Gradle** → `app` → Tasks → android → **signingReport**.
2. Descarga `google-services.json` (lo comparte el Integrante 4) y ponlo en `app/google-services.json`.
    - Si el repositorio es **privado**, el archivo puede estar subido.
    - Si el repositorio es **público**, no se sube: está en `.gitignore` y se comparte por mensaje privado.
3. Las reglas e índices viven en `firebase/` (`firestore.rules`, `storage.rules`, `firestore.indexes.json`). Solo el Integrante 4 los despliega.

> **Nunca** subas contraseñas, claves de servicio, `local.properties` ni archivos `.keystore`.

## 4. Cargar los datos de prueba

El script de datos crea categorías, 5 municipios, al menos 30 lugares y una ruta Trekko Originals. Usa IDs fijos, así que se puede ejecutar varias veces sin duplicar nada.

```bash
cd scripts/seed
npm install
node seed.js
```

> Ajustar este comando si el Integrante 4 usa otro nombre de archivo o herramienta. El detalle de credenciales y despliegue de reglas está en `firebase/README.md`.

Cuenta de demostración: `demo@trekko.mx` (la contraseña se comparte por el chat del equipo, no se escribe aquí).

## 5. Estructura del repositorio

```text
Trekko/
├── app/src/main/java/mx/trekko/app/
│   ├── MainActivity.kt            (Integrante 1)
│   ├── navigation/                (Integrante 1)
│   ├── auth/                      (Integrante 1)
│   ├── di/                        (Integrante 1)
│   ├── data/
│   │   ├── model/                 (contrato de datos: Integrantes 4 y 1)
│   │   ├── remote/                (Firestore)
│   │   └── local/                 (Room: Integrante 3)
│   ├── ui/                        (tema y componentes: Integrante 5)
│   └── feature/
│       ├── home/                  (Integrante 5)
│       ├── profile/               (Integrante 5)
│       ├── catalog/               (Integrante 2)
│       ├── map/                   (Integrante 2)
│       └── itineraries/           (Integrante 3)
├── firebase/                      (Integrante 4)
├── scripts/seed/                  (Integrante 4)
├── docs/                          (Integrante 5)
└── README.md
```

**Archivos compartidos** (`build.gradle.kts`, `libs.versions.toml`, `AndroidManifest.xml`, rutas de navegación): solo los modifica el Integrante 1. Si necesitas una dependencia o un permiso nuevo, pídelo explicando qué librería y para qué.

### Colecciones de Firestore

| Colección | Campos principales |
|---|---|
| `users/{uid}` | nombre, correo, modo, fechaCreacion |
| `categories/{id}` | nombre, icono |
| `municipalities/{id}` | nombre, latitud, longitud |
| `pois/{id}` | nombre, categoriaId, municipioId, descripcion, direccion, latitud, longitud, horario, costo, duracionEstimadaMin, telefono, whatsapp, fotos[], publicado |
| `routes/{id}` | nombre, municipioId, poiIds[], costoTotal, duracionTotal |
| `itineraries/{id}` | userId, municipioId, fecha, presupuesto, intereses[], paradas[], costoTotal, duracionTotalMin, creadoEn, actualizadoEn |

Solo se permiten cambios que no rompan lo existente: agregar campos sí, renombrar o borrar no.

## 6. Flujo de trabajo con Git

- **`main` siempre debe compilar.** Nadie hace push directo a `main`.
- Una rama por tarea: `feature/<integrante>-<tarea>`, por ejemplo `feature/i2-filtros-directorio`.

**Rutina diaria**

```bash
git checkout main
git pull
git checkout feature/i2-filtros-directorio
git merge main          # o: git rebase main
```

1. Commits pequeños y frecuentes con mensajes claros:
    - `feat(catalog): agrega filtro por municipio`
    - `fix(auth): corrige cierre de sesión`
2. Al terminar una tarea, sube la rama y abre un **Pull Request** hacia `main` con qué cambia y cómo probarlo.
3. El Integrante 1 revisa y mezcla los PRs de los demás; el Integrante 5 revisa los del Integrante 1. Quien revisa confirma que el proyecto compila.
4. Mezcla PRs varias veces al día, no al final.

**Para evitar conflictos**

- Trabaja solo dentro de tus carpetas.
- No reformatees archivos completos ni muevas archivos de otras personas.
- Si hay conflicto, resuélvelo con la persona dueña del archivo.

**Una tarea está terminada cuando:**

- [ ] Compila y se probó en emulador o teléfono.
- [ ] Su PR fue revisado por otra persona y mezclado a `main`.
- [ ] Maneja los casos de cargando, vacío, error y sin internet.
- [ ] No deja contraseñas, claves ni datos personales en código ni logs.

## 7. Equipo y responsabilidades

| Integrante | Nombre | Rol | Carpetas propias | Suplente |
|---|---|---|---|---|
| 1 | __________ | Líder Android: proyecto base, sesión, navegación e integración | `navigation/`, `auth/`, `di/`, `MainActivity` | 5 |
| 2 | __________ | Catálogo: directorio, filtros, ficha de lugar, contacto, mapa | `feature/catalog/`, `feature/map/` | 3 |
| 3 | __________ | Itinerarios y modo sin conexión (Room) | `feature/itineraries/`, `data/local/` | 2 |
| 4 | __________ | Firebase: configuración, datos de prueba y reglas | `firebase/`, `scripts/seed/` | 1 |
| 5 | __________ | Diseño, Inicio, Perfil, pruebas y entrega | `ui/`, `feature/home/`, `feature/profile/`, `docs/` | 1 |

El detalle día por día de cada integrante está en `docs/Trekko_Plan_de_Trabajo.docx`.

## 8. Guion de demostración

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Abrir la app sin sesión | Aparece el Login, no una pantalla en blanco |
| 2 | Registrarse o iniciar sesión | Llega a Inicio y el perfil queda creado |
| 3 | Abrir el directorio | Lista con lugares de 5 municipios |
| 4 | Filtrar (ej. Calvillo + gastronomía) | El conjunto cambia y los filtros se conservan |
| 5 | Abrir un lugar | Horario, costo, fotos y botones de contacto |
| 6 | Generar itinerario (Calvillo, $800, 1 día) | Paradas, costo total y duración |
| 7 | Guardar | Aparece en "Mis itinerarios" |
| 8 | Activar modo avión y reabrir la app | El itinerario sigue disponible |
| 9 | Volver a Inicio, abrir Perfil y cerrar sesión | Navegación estable, sin errores |
| 10 | Revisar Firebase | Los documentos existen y un acceso indebido es rechazado |

**Plan B:** si falla la red durante la demo, usar el APK ya instalado con datos precargados.

## 9. Generar el APK

Desde Android Studio: **Build → Build Bundle(s) / APK(s) → Build APK(s)**.

O desde terminal:

```bash
./gradlew assembleDebug
```

El archivo queda en `app/build/outputs/apk/debug/app-debug.apk`. Pruébalo instalándolo en un teléfono antes de la demo.

## 10. Problemas comunes

| Problema | Qué revisar |
|---|---|
| Login o Firestore fallan al abrir la app | Que `google-services.json` esté en `app/` y que tu SHA-1 esté registrada en Firebase |
| El directorio sale vacío | Que se hayan cargado los datos de prueba (sección 4) y que las reglas estén desplegadas |
| Error de permisos en Firestore | Estar autenticado; revisar `firestore.rules` con el Integrante 4 |
| El mapa no carga | Falta o está mal la clave de Maps; la lista de lugares sigue funcionando |
| Gradle no sincroniza | `File → Invalidate Caches…` y volver a sincronizar |
| Conflicto de Git | No elijas "mi versión" a ciegas: resuélvelo con la persona dueña del archivo |

## 11. Pendientes (fuera del sprint)

- Pagos y activación Premium.
- Grupos, votaciones y Viaje Activo.
- Reseñas, reputación, insignias y canjes.
- Portal completo de Anfitrión y analítica institucional.
- Mapas descargables para uso sin internet.
- Publicación en Google Play y endurecimiento de seguridad para producción.