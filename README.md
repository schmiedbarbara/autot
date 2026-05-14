# CentroCultural - Plataforma Web de Gestión de Actividades Culturales

## Resumen Ejecutivo

CentroCultural es una aplicación web desarrollada para facilitar la exploración, búsqueda y publicación de actividades culturales y educativas en la comunidad universitaria. La plataforma implementa un sistema de autenticación basado en roles (visitante/colaborador) con validación robusta de datos y geolocalización interactiva mediante mapas.

---

## 1. Ejecución del Programa

### 1.1 Requisitos Previos

- **Visual Studio Code** - Editor recomendado ([descargar](https://code.visualstudio.com))
- Navegador web moderno (Chrome, Firefox, Safari, Edge)
- Conexión a internet (requerida para geocodificación y mapas)

### 1.2 Instalación

1. Descargar o clonar el repositorio
2. Abrir la carpeta en Visual Studio Code: `Archivo → Abrir Carpeta`
3. Ejecutar `index.html` en el navegador (doble clic o Live Server)

### 1.3 Verificación de Funcionamiento

- La aplicación carga en `http://localhost` (o directamente desde archivo)
- Se debe visualizar el mapa interactivo y el formulario de login
- Los datos se almacenan automáticamente en localStorage del navegador

---

## 2. Arquitectura y Estructura del Proyecto

### 2.1 Diagrama Estructural

```
CentroCultural/
├── Archivos HTML (páginas principales)
│   ├── index.html                      # Portal principal
│   ├── registro.html                   # Registro de colaboradores
│   ├── pantallaUsuario.html           # Panel de administración
│   └── misTalleres.html               # Mis talleres
│
├── Archivos CSS (presentación)
│   ├── style.css                      # Estilos globales
│   ├── style-registro.css             # Estilos de registro
│   ├── style-pantallaUsuario.css      # Estilos del panel
│   └── style-misTalleres.css          # Estilos de talleres
│
├── app.js                             # Orquestador principal
├── img/                               # Recursos multimedia
│
└── modulos/                           # Módulos funcionales
    ├── autenticacion.js               # Gestión de usuarios y sesiones
    ├── talleres.js                    # CRUD de actividades
    ├── mapa.js                        # Integración de mapas
    ├── busqueda.js                    # Filtrado en tiempo real
    └── validaciones.js                # Validación de datos
```

### 2.2 Descripción de Módulos

| Módulo | Funcionalidad | Responsabilidades |
|--------|--------------|-------------------|
| **autenticacion.js** | Gestión de usuarios | Registro, login, sesiones, navegación dinámica |
| **talleres.js** | CRUD de actividades | Crear, leer, eliminar talleres; renderizar en UI |
| **mapa.js** | Geolocalización | Inicializar mapa, gestionar marcadores, geocodificar |
| **busqueda.js** | Filtrado | Búsqueda por nombre/dirección en tiempo real |
| **validaciones.js** | Control de datos | Validar tipos, formatos y campos obligatorios |

### 2.3 Patrón de Arquitectura

La aplicación sigue un patrón **modular con separación de responsabilidades**:
- **app.js** actúa como orquestador central (inicializador)
- Cada módulo gestiona un dominio específico
- Las validaciones son centralizadas en un módulo dedicado
- El almacenamiento de datos utiliza localStorage

---

## 3. Características Principales

### 3.1 Para Usuarios No Autenticados

- Exploración de centros y talleres en tiempo real
- Búsqueda por nombre de actividad o dirección
- Visualización interactiva en mapa geográfico
- Acceso a información general del Centro Cultural

### 3.2 Para Usuarios Autenticados (Colaboradores)

- Registro seguro de nuevos talleres con validación
- Panel de administración personal ("Mis Talleres")
- Gestión completa: crear, visualizar y eliminar actividades
- Búsqueda y filtrado dentro de talleres propios
- Mapa interactivo con ubicaciones de actividades

---

## 4. Navegación de la Plataforma

### 4.1 Flujo de Usuario No Autenticado

```
Inicio (index.html)
├─→ Ver información general
├─→ Buscar talleres en mapa
└─→ Opción: Ir a Registro
```

### 4.2 Flujo de Usuario Autenticado

```
Login (index.html)
├─→ Panel del Colaborador (pantallaUsuario.html)
│   ├─→ Registrar nuevo taller
│   └─→ Ir a "Mis Talleres"
│
├─→ Mis Talleres (misTalleres.html)
│   ├─→ Ver talleres propios
│   ├─→ Buscar/filtrar
│   └─→ Eliminar taller
│
└─→ Explorar todos los talleres (index.html → Centros)
```

### 4.3 Secciones Principales

| Sección | URL | Acceso | Descripción |
|---------|-----|--------|------------|
| Inicio | index.html | Público | Página principal, login y exploración |
| Registro | registro.html | Público | Formulario de registro de colaboradores |
| Panel | pantallaUsuario.html | Autenticado | Crear y gestionar talleres |
| Mis Talleres | misTalleres.html | Autenticado | Visualizar actividades propias |

---

## 5. Sistema de Validación de Datos

### 5.1 Validaciones en Registro de Colaboradores

| Campo | Tipo | Restricciones | Obligatorio |
|-------|------|---------------|------------|
| Nombre | Texto | Solo letras, 2+ caracteres | ✓ |
| Apellido | Texto | Solo letras, 2+ caracteres | ✓ |
| Teléfono | Tel | 10+ dígitos, múltiples formatos | ✓ |
| Email | Email | Formato RFC válido | ✓ |
| Contraseña | Password | 6+ caracteres, 1 mayúscula, 1 número | ✓ |

### 5.2 Validaciones en Registro de Talleres

| Campo | Tipo | Restricciones | Obligatorio |
|-------|------|---------------|------------|
| Nombre | Texto | 3+ caracteres | ✓ |
| Descripción | Textarea | 10+ caracteres | ✓ |
| Rubro | Select | Opciones predefinidas | ✓ |
| Dirección | Texto | Formato: "Calle NRO, Localidad" + geocodificación | ✓ |
| Horarios | Select | Franja predefinida | ✓ |
| Teléfono | Tel | 10+ dígitos | ✓ |
| Actividades | Texto | Libre | ✗ |
| Redes Sociales | Texto | Libre | ✗ |
| Imagen | URL | URL válida si se completa | ✗ |

### 5.3 Mecanismo de Validación

```javascript
// Flujo de validación
1. Captura de datos del formulario
2. Validación de tipo de dato
3. Validación de formato específico
4. Mensaje de error contextualizado (si aplica)
5. Persistencia en localStorage
```

**Características:**
- Mensajes de error específicos por campo
- Resaltado visual de campos inválidos (rojo)
- Validación antes de guardar datos
- Límites de caracteres implementados

---

## 6. Tecnologías Utilizadas

### 6.1 Stack Tecnológico

| Capa | Tecnología | Propósito |
|------|-----------|----------|
| Frontend | HTML5 | Estructura y semántica |
| Estilos | CSS3 | Presentación responsiva |
| Lógica | JavaScript (Vanilla) | Comportamiento dinámico |
| Mapas | Leaflet.js | Visualización geográfica |
| Geolocalización | API USIG BA | Geocodificación de direcciones |
| Persistencia | localStorage | Almacenamiento local |

### 6.2 Justificación Técnica

- **JavaScript Vanilla**: Sin dependencias externas; máxima portabilidad
- **Leaflet.js**: Librería ligera y open-source para mapas
- **localStorage**: Adecuado para prototipo; facilita demostración sin backend
- **API USIG**: Geocodificación específica para Buenos Aires

---

## 7. Almacenamiento de Datos

Los datos se almacenan en localStorage bajo las siguientes claves:

```javascript
// Estructura de almacenamiento
{
  "colaboradoresPortal": [
    {
      "id": 1,
      "nombre": "...",
      "apellido": "...",
      "correo": "...",
      "telefono": "...",
      "contrasenia": "..."
    }
  ],
  "talleres": [
    {
      "id": 1,
      "idColaborador": 1,
      "nombre": "...",
      "descripcion": "...",
      "rubro": "...",
      "direccion": "...",
      "lat": -34.5431,
      "lng": -58.7126,
      // ... otros campos
    }
  ],
  "usuarioActivo": { /* objeto usuario activo */ }
}
```

**Limitaciones:**
- Los datos se pierden al limpiar caché del navegador
- Sin sincronización entre dispositivos
- Seguridad limitada (credenciales en texto plano)

---

## 8. Guía de Mantenimiento y Extensión

### 8.1 Agregar un Nuevo Campo a Talleres

1. Adicionar `<input>` o `<select>` en `pantallaUsuario.html`
2. Crear función validadora en `modulos/validaciones.js`
3. Actualizar `registrarTaller()` en `modulos/talleres.js`
4. Incluir campo en `crearTarjeta()` para renderizar

### 8.2 Modificar Opciones de Selectores

Editar los `<option>` correspondientes en `pantallaUsuario.html`:
- Rubros: línea ~70
- Franjas horarias: línea ~85

### 8.3 Personalizar Estilos

Cada página tiene CSS independiente:
- `style.css` - Estilos base globales
- `style-registro.css` - Formulario de registro
- `style-pantallaUsuario.css` - Panel del colaborador
- `style-misTalleres.css` - Visualización de talleres

---

## 9. Consideraciones Funcionales

### 9.1 Restricciones de Acceso

- ✓ No requiere autenticación: exploración y búsqueda
- ✗ Requiere autenticación: crear/editar/eliminar talleres

### 9.2 Alcance Geográfico

- Geocodificación limitada a Buenos Aires (API USIG)
- Mapas centrados en coordenadas: -34.5431, -58.7126

### 9.3 Ciclo de Vida de Sesión

```
Usuario accede → Autentica → sesión guardada en localStorage
    ↓
Navega por aplicación → Sesión persiste
    ↓
Cierra sesión → localStorage["usuarioActivo"] se elimina
```

---

## 10. Perspectivas Futuras

### 10.1 Mejoras Técnicas Recomendadas

- [ ] Backend con base de datos relacional
- [ ] Autenticación con JWT
- [ ] HTTPS y validación de email
- [ ] API REST para operaciones CRUD
- [ ] Caching de imágenes
- [ ] Progressive Web App (PWA)

### 10.2 Expansiones Funcionales

- [ ] Búsqueda avanzada con filtros múltiples
- [ ] Sistema de ratings y comentarios
- [ ] Notificaciones por email
- [ ] Aplicación móvil nativa
- [ ] Panel administrativo
- [ ] Reportes de actividad

---

## 11. Referencias y Documentación

- [Documentación Leaflet.js](https://leafletjs.com/reference.html)
- [API USIG Buenos Aires](https://servicios.usig.buenosaires.gob.ar/)
- [MDN - localStorage](https://developer.mozilla.org/es/docs/Web/API/Storage/localStorage)
- [HTML5 Validation](https://developer.mozilla.org/es/docs/Learn/Forms/Form_validation)

---

## 12. Información del Proyecto

**Institución:** Centro Cultural UNGS  
**Objetivo:** Plataforma comunitaria de difusión de actividades culturales  
**Versión:** 1.0  
**Última actualización:** Mayo 2026  
**Licencia:** Abierta (uso comunitario)

Para reportar problemas o sugerir mejoras, contactar al equipo de desarrollo del Centro Cultural UNGS.
