# CentroCultural - Plataforma de Actividades Culturales
¡Bienvenidos al Centro Cultural UNGS!
La aplicación permite a los usuarios de la comunidad explorar, buscar y publicar actividades culturales y educativas en la zona. Proporciona una interfaz intuitiva para conectar colaboradores con la comunidad.

---

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Características](#características)
- [Cómo Empezar](#cómo-empezar)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)
- [Validaciones](#validaciones)
- [Consideraciones Generales](#consideraciones-generales)

---

## 📝 Descripción

CentroCultural es una plataforma web que facilita el registro y descubrimiento de talleres y actividades culturales en la comunidad. La aplicación implementa un sistema de autenticación seguro y validación de datos.
- **Usuarios no autenticados** (visitantes)
- **Usuarios autenticados** (colaboradores)

---

## ✨ Características

### Para Usuarios No Autenticados (Visitantes)

- ✅ **Explorar centros y talleres** - Acceder al listado general de actividades disponibles
- ✅ **Buscar actividades** - Por nombre del curso o dirección/ubicación
- ✅ **Interactuar con el mapa** - Ver zonas geográficas y seleccionar actividades
- ✅ **Ver información general** - Conocer sobre el Centro Cultural UNGS

### Para Usuarios Autenticados (Colaboradores)

- ✅ **Registrar talleres** - Crear nuevos centros/talleres con validación de datos
- ✅ **Gestionar talleres** - Ver, editar y eliminar talleres propios
- ✅ **Sección "Mis Talleres"** - Panel personal con:
  - Listado de talleres registrados
  - Búsqueda y filtrado por nombre/dirección
  - Mapa interactivo con ubicaciones
  - Opciones de gestión

- ✅ **Explorar otros talleres** - Ver actividades de otros colaboradores

---

## 🚀 Cómo Empezar

### Requisitos Previos

- **Visual Studio Code** - Descarga desde [code.visualstudio.com](https://code.visualstudio.com)
- Navegador web moderno (Chrome, Firefox, Safari, Edge)
- Conexión a internet (para geocodificación y mapas)

### Instalación y Ejecución

1. **Abre la carpeta del proyecto en VS Code**
   - Archivo → Abrir Carpeta → Selecciona la carpeta `CentroCultural`

2. **Abre `index.html` en el navegador**
   - Opción A: Haz clic derecho en `index.html` → "Abrir con Live Server" (si tienes la extensión)
   - Opción B: Haz doble clic en `index.html` para abrir en tu navegador predeterminado

3. **Comienza a explorar**
   - La plataforma se cargará completamente en tu navegador

### Primeros Pasos

1. **Visitar la plataforma** - La página principal se abrirá automáticamente
2. **Explorar sin registrarse** - Busca talleres en el mapa
3. **Registrarse como colaborador** - Haz clic en "Registrarse como Colaborador"
4. **Crear tu primer taller** - Accede a "Panel del Colaborador" y registra un taller
5. **Buscar tus talleres** - Ve a "Mis Talleres" para ver tus creaciones

---

## 📁 Estructura del Proyecto

```
CentroCultural/
├── index.html                          # Página principal (inicio/login)
├── registro.html                       # Formulario de registro de colaboradores
├── pantallaUsuario.html               # Panel del colaborador (registrar talleres)
├── misTalleres.html                   # Vista de talleres del usuario
├── app.js                             # Orquestador principal (inicializador)
├── style.css                          # Estilos globales
├── style-registro.css                 # Estilos formulario de registro
├── style-pantallaUsuario.css          # Estilos panel del colaborador
├── style-misTalleres.css              # Estilos vista "Mis Talleres"
├── img/                               # Imágenes y recursos
│   └── ubicacion.png                  # Icono de marcador del mapa
│   └── qr.jpg                         # Código QR de la app
│
└── modulos/                           # Módulos funcionales (separación de responsabilidades)
    ├── autenticacion.js               # Login, registro, sesiones
    ├── talleres.js                    # CRUD de talleres/centros
    ├── mapa.js                        # Mapa, marcadores, geocodificación
    ├── busqueda.js                    # Filtrado y búsqueda de talleres
    └── validaciones.js                # Validaciones de datos
```

### Descripción de Módulos

| Módulo | Responsabilidad |
|--------|-----------------|
| **autenticacion.js** | Registro, login, cierre de sesión, gestión de usuario activo, navegación dinámica |
| **talleres.js** | Crear, leer, actualizar y eliminar talleres; mostrar en lista y mapa |
| **mapa.js** | Inicializar mapa (Leaflet), gestionar marcadores, geocodificar direcciones |
| **busqueda.js** | Filtrar talleres por nombre o dirección en tiempo real |
| **validaciones.js** | Validar tipos de datos, formatos y campos obligatorios |

---

## 🛠️ Tecnologías Utilizadas

- **Frontend**: HTML5, CSS3, JavaScript (Vanilla)
- **Mapa**: [Leaflet.js](https://leafletjs.com/) - Mapas interactivos open-source
- **Geocodificación**: API USIG Buenos Aires
- **Almacenamiento**: localStorage (datos locales en navegador)
- **Estilos**: CSS puro (sin frameworks)

---

## ✅ Validaciones Implementadas

### Registro de Colaboradores

| Campo | Validación | Obligatorio |
|-------|-----------|------------|
| Nombre | Solo letras, 2+ caracteres | ✓ |
| Apellido | Solo letras, 2+ caracteres | ✓ |
| Teléfono | 10+ dígitos, formatos varios | ✓ |
| Email | Formato válido (usuario@dominio.com) | ✓ |
| Contraseña | 6+ caracteres, 1 mayúscula, 1 número | ✓ |

### Login

| Campo | Validación | Obligatorio |
|-------|-----------|------------|
| Email | Requerido | ✓ |
| Contraseña | Requerido | ✓ |

### Registro de Talleres

| Campo | Validación | Obligatorio |
|-------|-----------|------------|
| Nombre | 3+ caracteres | ✓ |
| Descripción | 10+ caracteres | ✓ |
| Rubro | Selección de opciones predefinidas | ✓ |
| Actividades | Texto libre | ✗ |
| Dirección | Formato: "Calle NRO, Localidad" + geocodificación | ✓ |
| Horarios | Franja horaria predefinida | ✓ |
| Teléfono | 10+ dígitos | ✓ |
| Redes Sociales | Texto libre | ✗ |
| Foto/Imagen | URL válida si se completa | ✗ |

### Características de Validación

- ✅ **Mensajes de error contextuales** - Cada campo muestra su error específico
- ✅ **Validación en tiempo de envío** - Se valida antes de guardar datos
- ✅ **Estilos visuales** - Campos con error se resaltan en rojo
- ✅ **Campos obligatorios marcados** - Asterisco rojo (*) en labels
- ✅ **Límites de caracteres** - maxlength en inputs
- ✅ **Geocodificación de direcciones** - Valida direcciones en Buenos Aires

---

## 📦 Almacenamiento de Datos

Los datos se almacenan en **localStorage** del navegador:

```javascript
// Colaboradores registrados
localStorage.getItem("colaboradoresPortal")

// Talleres creados
localStorage.getItem("talleres")

// Usuario activo (sesión actual)
localStorage.getItem("usuarioActivo")
```

**Nota**: Los datos se pierden si el navegador se limpia. Para producción, integrar con base de datos backend.

---

## 🔧 Desarrollo y Mantenimiento

### Agregar un Nuevo Campo a Talleres

1. Agregar input en `pantallaUsuario.html`
2. Agregar validador en `modulos/validaciones.js`
3. Actualizar función `registrarTaller()` en `modulos/talleres.js`
4. Actualizar función `crearTarjeta()` para mostrar el campo

### Cambiar Rubros o Franjas Horarias

- Edita los `<option>` en los `<select>` de `pantallaUsuario.html`

### Personalizar Estilos

- Cada página HTML tiene su propio CSS:
  - `style.css` - Estilos globales
  - `style-registro.css` - Formulario de registro
  - `style-pantallaUsuario.css` - Panel del colaborador
  - `style-misTalleres.css` - Vista de "Mis Talleres"

---

## 🚨 Consideraciones Generales

- **No es necesario estar logueado** para explorar o buscar actividades
- **Es obligatorio iniciar sesión** para registrar talleres o centros
- **Todos los usuarios** pueden acceder a la información general del Centro Cultural UNGS
- **Los datos se guardan localmente** en el navegador (localStorage)
- **La geocodificación funciona en Buenos Aires** mediante la API USIG

---

## 📋 Mejoras Futuras

- [ ] Backend con base de datos (Node.js, Python, etc.)
- [ ] Autenticación con tokens JWT
- [ ] Validación de email con confirmación
- [ ] Búsqueda avanzada y filtros
- [ ] Ratings y comentarios de usuarios
- [ ] Notificaciones en tiempo real
- [ ] Aplicación móvil (React Native, Flutter)
- [ ] Panel de administración

---

## 📧 Contacto y Soporte

Para reportar bugs o sugerir mejoras, contacta al equipo de desarrollo del Centro Cultural UNGS.

---

**Última actualización**: Mayo 2026

