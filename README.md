# 📦 Taller Práctico N° 4 — Update Firestore con React, TypeScript y Cloud Firestore

> Actividad académica que extiende el sistema de inventario de productos con la operación **UPDATE** sobre **Cloud Firestore**, practicando `updateDoc` junto con las operaciones existentes de escritura (`addDoc`) y lectura (`getDocs`) en una aplicación React + TypeScript + Vite.

## 🎯 Objetivo

Implementar la operación de actualización de documentos en Firestore manteniendo el flujo unidireccional de datos, el tipado estricto en TypeScript y un formulario inteligente con modo dual (CREATE / UPDATE):

- **CREATE:** Capturar datos del formulario y persistirlos como documentos en una colección de Firestore.
- **READ:** Consultar la colección al montar el componente y renderizar la lista en la UI.
- **UPDATE:** Seleccionar un producto existente, precargar sus datos en el formulario y actualizar el documento en Firestore con `updateDoc`.

## 📐 Diagrama de Flujo

```
┌──────────────────────────────────────────────────────────────────────┐
│                        FLUJO CREATE                                 │
│                                                                     │
│  ProductoForm ──► crearProducto() ──► addDoc() ──► Firestore 🔥     │
│  (formulario)     (servicio)          (SDK)        (colección:       │
│                                                     "productos")    │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                         FLUJO READ                                  │
│                                                                     │
│  Firestore 🔥  ──► getDocs() ──► obtenerProductos() ──► ProductoLista│
│  (colección:       (SDK)         (servicio)              (tabla UI)  │
│   "productos")                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                        FLUJO UPDATE                                 │
│                                                                     │
│  ProductoLista ──► [Editar] ──► ProductoForm ──► actualizarProducto()│
│  (selección)       (callback)   (modo edición)   (servicio)          │
│                                                                     │
│  actualizarProducto() ──► doc() + updateDoc() ──► Firestore 🔥      │
│  (servicio)                (SDK)                   (documento        │
│                                                     actualizado)    │
└──────────────────────────────────────────────────────────────────────┘
```

## 📁 Estructura del Proyecto

```
taller4-firebase-update/
├── Entregable/
│   ├── Antes de Editar.PNG        # Captura antes de editar un producto
│   ├── Editando.PNG               # Captura del formulario en modo edición
│   └── Editado.PNG                # Captura del producto ya actualizado
├── public/
│   └── image.png                  # Favicon personalizado
├── src/
│   ├── components/
│   │   ├── ProductoForm.tsx       # Formulario dual CREATE / UPDATE
│   │   ├── ProductoForm.css
│   │   ├── ProductoLista.tsx      # Lista READ con botón [Editar]
│   │   └── ProductoLista.css
│   ├── services/
│   │   └── productoService.ts     # Lógica Firestore (addDoc / getDocs / updateDoc)
│   ├── firebase.ts                # Inicialización de Firebase
│   ├── App.tsx                    # Componente raíz y orquestación
│   ├── App.css
│   ├── index.css                  # Tokens de diseño y reset global
│   └── main.tsx                   # Entry point React
├── DESIGN.md                      # Sistema de diseño visual (Craft Docs)
├── roadmap.md                     # Roadmap de implementación por fases
├── index.html                     # HTML con Google Fonts y favicon
├── package.json
├── tsconfig.json
└── vite.config.ts
```

## 🛠️ Tecnologías

| Tecnología | Versión | Rol |
|------------|---------|-----|
| React | 19.x | Librería de UI |
| TypeScript | 6.x | Tipado estático |
| Vite | 8.x | Bundler y dev server |
| Firebase | 13.x | Backend (Firestore) |

## 🚀 Instalación y Ejecución

### 1. Clonar el repositorio

```bash
git clone <https://github.com/luisAristeguieta/firebase_4>
cd taller4-firebase-update
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Configurar variables de entorno

Crea un archivo `.env` en la raíz del proyecto con las credenciales de tu proyecto Firebase:

```env
VITE_FIREBASE_API_KEY=tu-api-key
VITE_FIREBASE_AUTH_DOMAIN=tu-proyecto.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=tu-proyecto-id
VITE_FIREBASE_STORAGE_BUCKET=tu-proyecto.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=tu-sender-id
VITE_FIREBASE_APP_ID=tu-app-id
```

> **Nota:** El archivo `src/firebase.ts` actualmente usa las credenciales directamente. Para usar variables de entorno, actualiza la configuración usando `import.meta.env.VITE_FIREBASE_*`.

### 4. Ejecutar en modo desarrollo

```bash
npm run dev
```

La aplicación estará disponible en `http://localhost:5173`.

### 5. Build de producción

```bash
npm run build
```

## 📸 Entregables

| Captura | Descripción |
|---------|-------------|
| ![Antes de Editar](Entregable/Antes%20de%20Editar.PNG) | Vista de la lista de productos antes de iniciar la edición, con el botón [Editar] visible en cada fila. |
| ![Editando](Entregable/Editando.PNG) | Formulario en modo edición con los datos del producto precargados, botón "Actualizar" y opción de "Cancelar". |
| ![Editado](Entregable/Editado.PNG) | Producto actualizado exitosamente, reflejando los cambios en la lista y en Cloud Firestore. |

## 📋 Componentes Principales

### `ProductoForm`
Formulario inteligente con **modo dual**:
- **Modo CREATE** (`productoSeleccionado === null`): campos vacíos, botón "Guardar", llama a `crearProducto()`.
- **Modo UPDATE** (`productoSeleccionado !== null`): `useEffect` precarga los datos, botón "Actualizar" + "Cancelar", llama a `actualizarProducto()`.

### `ProductoLista`
Al montarse (`useEffect`), llama a `obtenerProductos()`. Muestra la información en una tabla con columnas **Producto** (con ID visible), **Precio**, **Stock** y **Acciones** (botón [Editar]). Se refresca automáticamente al crear o editar un producto mediante `recargarKey`.

### `productoService.ts`
Capa de servicio que encapsula las operaciones de Firestore:
- `crearProducto(nombre, precio, stock)` → `addDoc()`
- `obtenerProductos()` → `getDocs()`
- `actualizarProducto(id, datos)` → `doc()` + `updateDoc()`
