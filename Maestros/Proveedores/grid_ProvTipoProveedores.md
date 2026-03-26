# Análisis de `grid_ProvTipoProveedores.aspx`

## 1) Descripción y función

`grid_ProvTipoProveedores.aspx` es el componente de mantenimiento de **Tipos de Proveedores** en la capa WebForms del módulo de maestros de proveedores.

Su función principal es implementar el flujo CRUD sobre la entidad `ProvTipoProveedores` mediante:

- una **grilla jqGrid** para búsqueda, paginación y acciones por fila,
- un **formulario modal** (`Ctrl_ProvTipoProveedores.ascx`) para alta/edición/clonación,
- servicios AJAX (`WebMethod`) en `grid_ProvTipoProveedores.aspx.cs` y `servicios/servicios_sigav.aspx.cs`.

Un **tipo de proveedor** es una categoría que clasifica proveedores según su naturaleza o tipo de servicio/producto que ofrecen (ej: proveedor de servicios, materias primas, transporte, consultoría, etc.), permitiendo segmentar y filtrar proveedores por su función empresarial.

---

## 2) Artefactos involucrados

### Página y control

- `grid_ProvTipoProveedores.aspx`
- `grid_ProvTipoProveedores.aspx.cs` (la directiva apunta a `grId_ProvTipoProveedores.aspx.cs`)
- `ControlUser/Ctrl_ProvTipoProveedores.ascx`
- `ControlUser/Ctrl_ProvTipoProveedores.ascx.cs`
- `form_ProvTipoProveedores.aspx` (vista detalle/solo lectura)
- `form_ProvTipoProveedores.aspx.cs`

### JavaScript principal

- `js/Grid_ProvTipoProveedores.js`

### Servicios comunes

- `servicios/servicios_sigav.aspx.cs`

---

## 3) Dependencias JS (objetos y funciones)

### Grilla (`js/Grid_ProvTipoProveedores.js`)

- **`Grilla_ProvTipoProveedores(filtro, NombreCol, OrdenCol, filas)`**:
  - inicializa jqGrid con configuración responsiva,
  - llama por AJAX a `grid_ProvTipoProveedores.aspx/Grilla_ProvTipoProveedores`,
  - aplica hasta **3 filtros simultáneos** con operador `LIKE`,
  - construye botones de exportación (`Excel`, `CSV`),
  - calcula filas automáticamente según altura de ventana: `parseInt(($(window).height() - 200) / 23)`.

- **`Accion_ProvTipoProveedores(id, accion, idpadre, usuario)`**:
  - `0`: Nuevo → `popform_ProvTipoProveedores`
  - `1`: Editar → `popform_ProvTipoProveedores`
  - `2`: Clonar → `popform_ProvTipoProveedores`
  - `3`: Eliminar → `eliminareg`
  - `4`: Ver detalle → `SubFormJquery` con `form_ProvTipoProveedores.aspx`

- **`Caption(ifilter1, iColumn1, ifilter2, iColumn2, ifilter3, iColumn3, iTabla, filas, NombreCol, OrdenCol)`**:
  - genera barra de herramientas con botones (Buscar, Limpiar, Nuevo, Cerrar),
  - incluye los 3 filtros dinámicos.

- **`Filtros(ifilter, iColumn, iTabla, NroFiltro)`**:
  - genera UI de cada filtro (combo + input),
  - usa `servicios_sigav.aspx/Caption_Option` para obtener columnas filtrables,
  - aplica restricción de entrada: `onkeypress="return SoloAlfanumerico(event)"`.

### Formulario modal (`Ctrl_ProvTipoProveedores.ascx`)

- **`popform_ProvTipoProveedores(...)`**: 
  - abre modal jQuery UI (750x550px),
  - orquesta flujo CRUD con tres botones: Grabar, Eliminar, Cerrar,
  - registra eventos de apertura/cierre mediante `Registrar_LogEvento`.

- **`BuscarDatos_ProvTipoProveedores(id, tabla, accion, hijo, idpadre, TablaOrigen, id_Origen)`**: 
  - carga datos para edición/clonado llamando `grid_ProvTipoProveedores.aspx/Buscar`,
  - popula campos del formulario (`IdMaeEmpresa`, `Nombre`).

- **`Grabar_ProvTipoProveedores(id, tabla, accion, hijo, usuario, idProceso, CallBack)`**: 
  - persiste cambios llamando `servicios_sigav.aspx/Grabar`,
  - construye 3 conjuntos de parámetros: `ParametrosGrabar`, `ParametrosValidacion`, `ParamValObligatorios`,
  - muestra indicadores de procesamiento (`ProcesandoGrabar`, `OcultarProcesandoGrabar`),
  - refresca grilla al éxito.

- **`DatosValidacion_ProvTipoProveedores()`**: 
  - valida formato de campos con expresiones regulares:
    - `IdProvTipoProveedores`: `/^[0-9]{0,10}$/`
    - `IdMaeEmpresa`: `/^[0-9]{0,10}$/`
    - `Nombre`: `/^[a-zA-Z0-9_.,:ñÑáéíóúÁÉÍÓÚ()=<>°$%@\/\*\+\s\-\\]+$/`
  - muestra mensajes específicos por campo con ejemplo de formato esperado.

- **`LimpiaDatos_ProvTipoProveedores(accion)`**: 
  - limpia campos del formulario,
  - deshabilita `IdProvTipoProveedores` (auto-generado),
  - carga combo `IdMaeEmpresa` vía `DDLIdMaeEmpresa`.

- **`CambiosProvTipoProveedores()`**: 
  - detecta cambios en formulario mediante evento `change`,
  - habilita botón Grabar solo si hay modificaciones (`$("#mod_ProvTipoProveedores").val()`),
  - controla cambios en combobox customizado con evento `select`.

- **`DDLIdMaeEmpresa(id, tabla, filtro, id_selected, accion)`**: 
  - carga combo de empresas mediante `servicios_sigav.aspx/CargaDDL`,
  - campo de búsqueda: `Nombre2` (razón social),
  - usa combobox customizado con autocompletado jQuery UI.

- **`PopIdMaeEmpresa_ProvTipoProveedores()`**: 
  - abre detalle de empresa seleccionada en modal,
  - usa `SubFormJquery` con `form_MaeEmpresa.aspx`.

- **Funciones auxiliares**:
  - `ParametrosGrabar_ProvTipoProveedores()`: construye string de parámetros para inserción/actualización
  - `ParametrosValidacion_ProvTipoProveedores()`: parámetros para validaciones de negocio
  - `ParamValObligatorios_ProvTipoProveedores()`: parámetros de campos obligatorios

---

## 4) Dependencias C# (métodos y clases)

### `grid_ProvTipoProveedores.aspx.cs`

- **`Page_Load(object sender, EventArgs e)`**:
  - controla autenticación (`HttpContext.Current.User.Identity.IsAuthenticated`),
  - valida perfil del usuario (`Autentificacion.ValidaPerfil(Session["IdUser"], "ProvTipoProveedores")`),
  - registra acceso mediante `ClassSigav.GrabaLogAccesos` (evento tipo "3" - acceso a grilla),
  - soporta apertura directa en modo edición por parámetro `IdRegistro` (encriptado vía `ClassGrid.EncriptaDesEncriptaDatos`),
  - redirige a `Dashboard.aspx` si usuario sin permisos,
  - abre modal de login si sesión no autenticada.

- **WebMethods**:
  - **`InicializaProvTipoProveedores(string idUsuario)`**: 
    - devuelve valores iniciales para nuevos registros,
    - ejecuta `sp_InicializaProvTipoProveedores`,
    - retorna `IdMaeEmpresa` y `Nombre` por defecto basados en perfil del usuario.
  
  - **`Buscar(string id_reg, string tabla)`**: 
    - busca registro por ID usando `sp_generico_sel`,
    - retorna objeto `ProvTipoProveedores` con campos `ws_IdMaeEmpresa` y `ws_Nombre`.
  
  - **`Grilla_ProvTipoProveedores(int pPageSize, int pCurrentPage, string pSortColumn, string pSortOrder, string tabla, string pSearchField, string pSearchString)`**: 
    - implementa paginación servidor,
    - ejecuta `sp_Paginacion_Grilla2`,
    - construye botones de acción HTML por fila (Editar, Clonar, Eliminar, Ver),
    - procesa filtros dinámicos reemplazando `#%` por `'%` y `%#` por `%'`,
    - aplica formato `String.Format("{0:N0}", row["Nombre"])` a columna Nombre,
    - retorna `JQGridJsonResponse_ProvTipoProveedores` con metadatos de paginación.

- **Clases**:
  - **`ProvTipoProveedores`**: 
    - DTO de la entidad con propiedades: 
      - `ws_IdProvTipoProveedores` (Int32)
      - `ws_IdMaeEmpresa` (string)
      - `ws_Nombre` (string)
      - `ws_Botones` (string, HTML de botones de acción)
    - método `Encontrar(id_reg, sp_buscar, tabla)`: ejecuta búsqueda por ID mediante `sp_generico_sel`
  
  - **`BtnProvTipoProveedores`**: 
    - mensajes de estado: `ws_IdMensaje`, `ws_Descripcion`
  
  - **`JQGridJsonResponse_ProvTipoProveedores`**: 
    - respuesta paginada para jqGrid con propiedades:
      - `PageCount`: número total de páginas
      - `CurrentPage`: página actual
      - `RecordCount`: total de registros
      - `Items`: lista de `ClassGrid.JQGridItem`

### `ControlUser/Ctrl_ProvTipoProveedores.ascx.cs`

- **`Page_Load(object sender, EventArgs e)`**: 
  - maneja apertura directa vía QueryString (`tabla=ProvTipoProveedores`, `id=...`),
  - llama a `Inicio()` para modo visualización si no es PostBack.

- **`DLLIdMaeEmpresa(string filtro)`**: 
  - llena combo de empresas mediante `sp_llenaDropDown`,
  - tabla origen: `MaeEmpresa`,
  - campo display: `Nombre2` (razón social),
  - campo value: `IdMaeEmpresa`,
  - usa `ClassSigav.LlenarDropDown(sql)`.

- **`Inicio()`**: 
  - modo visualización/detalle,
  - obtiene `id` desde QueryString,
  - carga registro vía `BuscaProvTipoProveedores`,
  - aplica `SoloLectura()`.

- **`BuscaProvTipoProveedores(string IdProvTipoProveedores)`**: 
  - ejecuta `sp_generico_sel 'ProvTipoProveedores', '{id}'`,
  - popula controles del formulario:
    - `IdMaeEmpresa`: carga combo filtrado con empresa del registro
    - `Nombre`: campo de texto con nombre del tipo.

- **`SoloLectura()`**: 
  - deshabilita todos los controles para modo visualización,
  - `IdMaeEmpresa.Enabled = false`,
  - `Nombre.Enabled = false`,
  - oculta botón de búsqueda de empresa (`PopIdMaeEmpresa.Visible = false`),
  - deshabilita combobox customizado vía ScriptManager.

### `servicios/servicios_sigav.aspx.cs`

- **`Grabar(...)`**: 
  - delega validación y persistencia en `ClassForm.Validacion(...)`,
  - registra evento de grabación mediante `LogAcceso` (evento tipo "6"),
  - parámetros: `id_reg`, `tabla`, `par`, `parval`, `parvalobligatorio`, `accion`, `hijo`, `usuario`, `idProceso`, `DirPrincipal`, `DirDespacho`,
  - resuelve dinámicamente SP de validación y persistencia por tabla.

- **`Eliminar(id_reg, tabla, usuario)`**: 
  - eliminación genérica vía `ClassForm.Eliminacion(..., "sp_generico_del", ...)`,
  - registra evento de eliminación.

- **`CargaDDL(id_padre, tabla, filtro, campo, accion)`**: 
  - genera query dinámica para llenar combos,
  - detecta columna a mostrar desde metadatos (`ZysCampo.MostrarEnCombo=1`),
  - ejecuta `sp_llenaDropDown`,
  - retorna `ArrayList` de `ListItem` con pares `Value`/`Text`.

- **`Caption_Option(filtro, columna, tabla)`**: 
  - genera opciones de columnas filtrables para dropdown de filtros,
  - ejecuta `sp_generico_option`,
  - retorna lista de objetos con propiedad `ws_Caption`.

---

## 5) Procedimientos almacenados de servidor (detectados)

### Directos desde el componente:

- **`sp_Paginacion_Grilla2`**: 
  - listado paginado con soporte de ordenamiento y filtros múltiples,
  - parámetros: 
    - `@PageSize` (int): registros por página
    - `@CurrentPage` (int): página actual
    - `@SortColumn` (varchar 40): columna de ordenamiento
    - `@SortOrder` (varchar 4): dirección (asc/desc)
    - `@tabla` (varchar 50): nombre de tabla
    - `@filtro` (varchar 1000): condiciones WHERE dinámicas
    - `@IdUsuario` (varchar 50): usuario para filtros personalizados
  - retorna dos tablas:
    - tabla 0: `PageCount`, `CurrentPage`, `RecordCount`
    - tabla 1: filas de datos con todas las columnas

- **`sp_generico_sel`**: 
  - búsqueda de registro por ID,
  - parámetros: `@tabla='ProvTipoProveedores'`, `@id_reg`,
  - retorna todos los campos del registro:
    - `IdProvTipoProveedores`
    - `IdMaeEmpresa`
    - `Nombre`
    - campos de auditoría (FechaCreacion, UsuarioCreacion, etc.)

- **`sp_InicializaProvTipoProveedores`**: 
  - valores iniciales para nuevos registros,
  - parámetro: `@idUsuario`,
  - retorna `IdMaeEmpresa` y `Nombre` por defecto basados en perfil del usuario,
  - puede incluir lógica de empresa predeterminada según rol.

### Indirectos/generales usados en el flujo:

- **`sp_generico_del`**: 
  - eliminación genérica (vía `servicios_sigav.aspx/Eliminar`),
  - parámetros: `@tabla`, `@id_reg`, `@usuario`,
  - realiza soft delete (`Eliminado=1`) o hard delete según configuración de tabla,
  - puede incluir validaciones de integridad referencial.

- **`sp_llenaDropDown`**: 
  - carga de combos dinámicos,
  - parámetros: 
    - `@campos`: columnas a seleccionar (ej: 'IdMaeEmpresa,Nombre')
    - `@tabla_filtro`: tabla y filtro WHERE (ej: 'MaeEmpresa where Estado=1')
    - `@campo_orden`: columna de ordenamiento
  - retorna pares `Id`/`Texto` para poblar DropDownList.

- **`sp_generico_option`**: 
  - genera opciones de filtros de columnas,
  - parámetros: `@filtro`, `@columna`, `@tabla`,
  - retorna lista de campos disponibles para filtrado con estructura HTML `<option>`,
  - campo retornado: `NombreCampo`.

- **Procedimientos de inserción/actualización** (inferidos): 
  - invocados desde `ClassForm.Validacion(...)` en `servicios_sigav.aspx/Grabar`,
  - resolución dinámica por tabla,
  - probablemente `sp_ProvTipoProveedores_ins` y `sp_ProvTipoProveedores_upd`,
  - pueden incluir validaciones de negocio específicas (ej: nombre único por empresa).

---

## 6) Flujo CRUD e interacciones

### Create (Nuevo)

1. Usuario pulsa botón **Nuevo** en Caption de grilla.
2. `Accion_ProvTipoProveedores(0, 0, 0)` en `Grid_ProvTipoProveedores.js` invoca `popform_ProvTipoProveedores` con `accion=0`.
3. Modal se abre con título "Nuevo ProvTipoProveedores" (750x550px).
4. `LimpiaDatos_ProvTipoProveedores(0)`:
   - limpia campos del formulario,
   - deshabilita `IdProvTipoProveedores` (auto-generado),
   - ejecuta `DDLIdMaeEmpresa` para cargar combo de empresas.
5. `CambiosProvTipoProveedores()` activa detección de cambios (botón Grabar deshabilitado inicialmente).
6. `Registrar_LogEvento` registra apertura de formulario (evento tipo "4").
7. Usuario ingresa datos en campos:
   - **IdMaeEmpresa** (obligatorio, combobox con autocompletado)
   - **Nombre** (obligatorio, máx. 50 caracteres, alfanumérico con caracteres especiales)
8. Al modificar cualquier campo, `CambiosProvTipoProveedores` habilita botón **Grabar**.
9. Usuario pulsa **Grabar**.
10. `DatosValidacion_ProvTipoProveedores()` valida formato de campos con regex:
    - retorna `false` y muestra mensaje específico si hay error,
    - incluye ejemplo de formato esperado en el mensaje.
11. Si validación OK, muestra indicador `ProcesandoGrabar()`.
12. `Grabar_ProvTipoProveedores` construye 3 conjuntos de parámetros:
    - `ParametrosGrabar`: valores de campos para inserción
    - `ParametrosValidacion`: valores para validación de negocio
    - `ParamValObligatorios`: campos obligatorios
13. Llama `servicios/servicios_sigav.aspx/Grabar` con parámetros.
14. Backend (`ClassForm.Validacion`) ejecuta SP de validación y persistencia.
15. Al éxito, retorna `IdPadre` (nuevo ID), `Id` (resultado), `Name` (mensaje).
16. `OcultarProcesandoGrabar()` oculta indicador.
17. `Mensaje_Grabacion` muestra resultado al usuario.
18. `Grilla_ProvTipoProveedores(1)` refresca la grilla.
19. Modal se cierra.
20. `Registrar_LogEvento` registra cierre de formulario (evento tipo "5").

### Read (Listar / Ver)

#### Listar
1. `Grilla_ProvTipoProveedores()` se ejecuta al cargar página (jQuery ready).
2. Obtiene filtros iniciales mediante `GrillaFiltroInicial('0', 'ProvTipoProveedores')`.
3. Construye `StringFiltro` con operadores `LIKE` para cada filtro activo.
4. Calcula filas por página: `parseInt(($(window).height() - 200) / 23)`.
5. Ejecuta AJAX a `grid_ProvTipoProveedores.aspx/Grilla_ProvTipoProveedores`.
6. Backend:
   - ejecuta `sp_Paginacion_Grilla2` con parámetros de paginación, ordenamiento y filtros,
   - obtiene `PageCount`, `CurrentPage`, `RecordCount` de tabla 0,
   - obtiene filas de datos de tabla 1,
   - construye HTML de botones por fila,
   - aplica formato `{0:N0}` a columna Nombre.
7. Retorna JSON con estructura jqGrid.
8. Grilla renderiza filas con columnas:
   - **IdProvTipoProveedores** (10% ancho)
   - **Nombre** (75% ancho, formato numérico)
   - **Botones** (180px fijo): Editar, Clonar, Eliminar, Ver
9. Navega con paginador jqGrid.

#### Ver detalle
1. Usuario pulsa botón **Ver** (icono info) en fila de grilla.
2. `Accion_ProvTipoProveedores(id, 4, id, usuario)` invoca `SubFormJquery`.
3. Abre `form_ProvTipoProveedores.aspx?id={id}&accion=&tabla=ProvTipoProveedores` en modal jQuery UI.
4. `Ctrl_ProvTipoProveedores.ascx.cs/Page_Load` detecta QueryString.
5. `Inicio()`:
   - obtiene `id` desde QueryString,
   - llama `BuscaProvTipoProveedores(id)`.
6. `BuscaProvTipoProveedores`:
   - ejecuta `sp_generico_sel 'ProvTipoProveedores', '{id}'`,
   - carga combo `IdMaeEmpresa` filtrado con empresa del registro,
   - popula campo `Nombre`.
7. `SoloLectura()` deshabilita todos los controles para visualización.
8. Usuario consulta datos sin poder editarlos.

### Update (Editar)

1. Usuario pulsa botón **Editar** en fila de grilla.
2. `Accion_ProvTipoProveedores(id, 1, id, usuario)` abre `popform_ProvTipoProveedores` con `accion=1`.
3. Modal se abre con título "Edición del ProvTipoProveedores".
4. `BuscarDatos_ProvTipoProveedores` llama `grid_ProvTipoProveedores.aspx/Buscar` con `id_reg`.
5. Backend ejecuta `sp_generico_sel 'ProvTipoProveedores', '{id}'`.
6. Respuesta JSON popula campos del formulario:
   - `$('#IdProvTipoProveedores').val(id)` (deshabilitado),
   - `$('#IdMaeEmpresa').val(ws_IdMaeEmpresa).combobox("refresh")`,
   - `$('#Nombre').val(ws_Nombre)`.
7. Botón **Eliminar** habilitado (solo en edición).
8. Usuario modifica datos (habilita botón Grabar por detección de cambios).
9. Validaciones de formato con `DatosValidacion_ProvTipoProveedores()`.
10. `Grabar_ProvTipoProveedores` con `accion=1` persiste cambios.
11. Backend actualiza registro existente en tabla `ProvTipoProveedores`.
12. Grilla se refresca mostrando cambios.
13. Modal se cierra.

### Delete (Eliminar)

#### Desde grilla
1. Usuario pulsa botón **Eliminar** en fila.
2. `Accion_ProvTipoProveedores(id, 3, id, usuario)` invoca `eliminareg(id, 'ProvTipoProveedores', '', '', usuario)`.
3. Modal de confirmación jQuery UI con texto: "Está a punto de eliminar un registro?".
4. Si confirma, ejecuta AJAX a `servicios/servicios_sigav.aspx/Eliminar`.
5. Backend:
   - llama `ClassForm.Eliminacion`,
   - ejecuta `sp_generico_del 'ProvTipoProveedores', '{id}', '{usuario}'`,
   - SP realiza soft delete (`UPDATE ... SET Eliminado=1`) o hard delete según configuración.
6. Retorna objeto con `Id` (resultado), `Name` (mensaje).
7. UI muestra mensaje de resultado en modal `#dialog-mensaje`.
8. `Grilla_ProvTipoProveedores(1)` refresca datos.

#### Desde formulario
1. Usuario abre tipo de proveedor en modo edición (`accion=1`).
2. Pulsa botón **Eliminar** en modal.
3. Valida que `accion == "1"` (solo permite eliminar en edición, no en nuevo).
4. Ejecuta `eliminareg(id, tabla, '', '', usuario)`.
5. Mismo flujo de confirmación y eliminación.
6. Adicional: modal se cierra automáticamente.

### Clone (Clonar)

1. Usuario pulsa botón **Clonar** en fila.
2. `Accion_ProvTipoProveedores(id, 2, id, usuario)` abre modal con `accion=2`.
3. Título del modal: "Clonación del ProvTipoProveedores".
4. `BuscarDatos_ProvTipoProveedores` carga datos del registro original.
5. `IdProvTipoProveedores` permanece deshabilitado y vacío (se generará nuevo ID).
6. Usuario puede modificar `IdMaeEmpresa` y `Nombre` según necesidad.
7. `Grabar_ProvTipoProveedores` con `accion=2` crea nuevo registro.
8. Backend lo trata como inserción.
9. Nuevo registro aparece en grilla con ID diferente.

---

## 7) Diagrama de objetos (Mermaid)

```mermaid
graph TD
    U[Usuario]
    G[grid_ProvTipoProveedores.aspx]
    JS[js/Grid_ProvTipoProveedores.js]
    UC[ControlUser/Ctrl_ProvTipoProveedores.ascx]
    FV[form_ProvTipoProveedores.aspx]
    GCS[grid_ProvTipoProveedores.aspx.cs]
    UCCS[Ctrl_ProvTipoProveedores.ascx.cs]
    SVC[servicios/servicios_sigav.aspx.cs]
    CF[ClassForm]
    CG[ClassGrid]
    CS[ClassSigav]
    AUTH[Autentificacion]
    DB[(SQL Server)]

    U --> G
    G --> JS
    G --> UC
    G --> FV

    JS -->|AJAX: Grilla_ProvTipoProveedores, Buscar, InicializaProvTipoProveedores| GCS
    JS -->|AJAX: Grabar, Eliminar, CargaDDL, Caption_Option| SVC
    UC --> UCCS
    FV --> UCCS

    GCS -->|Valida perfil| AUTH
    GCS -->|Log accesos evento 3| CS
    GCS -->|Conexión BD| CG
    GCS -->|SPs directos| DB

    SVC -->|Eliminar/Grabar genérico| CF
    CF -->|SP dinámicos por tabla| DB
    CG -->|Utilidades BD| DB
    CS -->|GrabaLogAccesos| DB

    UCCS -->|sp_generico_sel| DB
    UCCS -->|sp_llenaDropDown| DB
    UCCS -->|SoloLectura modo vista| UC

    JS -->|Registrar_LogEvento tipo 4,5| CS
```

---

## 8) Diagrama de proceso CRUD (Mermaid)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant JS as Grid_ProvTipoProveedores.js
    participant GCS as grid_ProvTipoProveedores.aspx.cs
    participant UCCS as Ctrl_ProvTipoProveedores.ascx.cs
    participant SVC as servicios_sigav.aspx.cs
    participant CF as ClassForm
    participant CS as ClassSigav
    participant DB as SQL Server

    Note over U,DB: FLUJO CREATE/UPDATE

    U->>JS: Clic Nuevo/Editar
    JS->>JS: popform_ProvTipoProveedores(id, accion)
    JS->>CS: Registrar_LogEvento(usuario, "Apertura Formulario", "4")
    
    alt accion == 1 (Editar)
        JS->>GCS: Buscar(id, "ProvTipoProveedores")
        GCS->>DB: sp_generico_sel 'ProvTipoProveedores', id
        DB-->>GCS: Registro ProvTipoProveedores
        GCS-->>JS: JSON {ws_IdMaeEmpresa, ws_Nombre}
        JS->>JS: Poblar formulario + Habilitar Eliminar
    else accion == 0 (Nuevo)
        JS->>GCS: InicializaProvTipoProveedores(idUsuario)
        GCS->>DB: sp_InicializaProvTipoProveedores idUsuario
        DB-->>GCS: Valores por defecto
        GCS-->>JS: {IdMaeEmpresa, Nombre default}
    end

    JS->>SVC: CargaDDL("MaeEmpresa", "Nombre2")
    SVC->>DB: sp_llenaDropDown
    DB-->>SVC: Lista empresas
    SVC-->>JS: Array<ListItem>

    U->>JS: Modifica datos
    JS->>JS: CambiosProvTipoProveedores() → Habilita Grabar
    
    U->>JS: Clic Grabar
    JS->>JS: DatosValidacion_ProvTipoProveedores() → Regex
    
    alt Validación OK
        JS->>JS: ProcesandoGrabar() → Muestra loading
        JS->>SVC: Grabar(id, tabla, par, parval, parvalobligatorio, ...)
        SVC->>CS: LogAcceso(usuario, "Grabar", "6")
        SVC->>CF: Validacion(...)
        CF->>DB: SP validación + SP inserción/actualización
        DB-->>CF: IdPadre + Resultado
        CF-->>SVC: {Id, IdPadre, Name}
        SVC-->>JS: JSON response
        JS->>JS: OcultarProcesandoGrabar()
        JS->>JS: Mensaje_Grabacion(resultado)
        JS->>JS: Grilla_ProvTipoProveedores(1) → Refresca
        JS->>JS: Cerrar modal
        JS->>CS: Registrar_LogEvento(usuario, "Cierre Formulario", "5")
    else Validación Error
        JS->>U: Mensaje_Eliminacion("Error en los datos")
    end

    Note over U,DB: FLUJO READ (Listar)

    U->>JS: Cargar página / Buscar / Filtrar
    JS->>JS: GrillaFiltroInicial("0", "ProvTipoProveedores")
    JS->>JS: Construye StringFiltro con 3 filtros LIKE
    JS->>GCS: Grilla_ProvTipoProveedores(pageSize, currentPage, sort, filtros)
    GCS->>CS: Log acceso evento tipo "3"
    GCS->>DB: sp_Paginacion_Grilla2
    DB-->>GCS: Tabla0(PageCount,RecordCount) + Tabla1(Rows)
    GCS->>GCS: Construir HTML botones por fila
    GCS->>GCS: Format("{0:N0}", Nombre)
    GCS-->>JS: JQGridJsonResponse_ProvTipoProveedores
    JS->>JS: Renderiza grilla + botones exportación

    Note over U,DB: FLUJO DELETE

    U->>JS: Clic Eliminar
    JS->>JS: eliminareg(id, 'ProvTipoProveedores', usuario)
    JS->>U: Diálogo confirmación (#dialog-confirm)
    U->>JS: Confirma
    
    JS->>SVC: Eliminar(id, "ProvTipoProveedores", usuario)
    SVC->>CF: Eliminacion(...)
    CF->>DB: sp_generico_del 'ProvTipoProveedores', id, usuario
    DB-->>CF: Resultado
    CF-->>SVC: {Id, Name}
    SVC-->>JS: JSON response
    
    JS->>U: Muestra mensaje (#dialog-mensaje)
    JS->>JS: Grilla_ProvTipoProveedores(1) → Refresca

    Note over U,DB: FLUJO VIEW (Ver detalle)

    U->>JS: Clic Ver
    JS->>JS: SubFormJquery → form_ProvTipoProveedores.aspx
    UCCS->>UCCS: Page_Load → Inicio()
    UCCS->>DB: sp_generico_sel 'ProvTipoProveedores', id
    DB-->>UCCS: Registro completo
    UCCS->>DB: sp_llenaDropDown (cargar combo empresa)
    DB-->>UCCS: Lista empresas
    UCCS->>UCCS: Poblar controles
    UCCS->>UCCS: SoloLectura() → Deshabilita todo
    UCCS-->>U: Formulario en modo vista
```

---

## 9) Relaciones de datos

`ProvTipoProveedores` es una entidad de clasificación del módulo de proveedores que categoriza proveedores según su tipo de servicio o producto.

### Relaciones detectadas

- **ProvTipoProveedores → MaeEmpresa** (Many-to-One): 
  - campo `IdMaeEmpresa` (obligatorio)
  - asocia el tipo de proveedor a una empresa del sistema
  - permite tener diferentes tipos de proveedores por empresa

- **ProvProveedores → ProvTipoProveedores** (Many-to-One): 
  - campo `IdProvTipoProveedores` en tabla `ProvProveedores`
  - permite clasificar múltiples proveedores bajo un mismo tipo
  - relación opcional (un proveedor puede no tener tipo asignado)

Para información detallada sobre las relaciones de esta entidad con otras del sistema, consultar:  
📘 **[Relaciones entre Entidades - Sistema SIGAV](../../Relaciones_Entidades.md#módulo-proveedores)**

---

## 10) Características especiales

### Diseño minimalista idéntico a ProvHolding
- Solo **2 campos editables**: `IdMaeEmpresa` y `Nombre`
- Enfocado en simplicidad y facilidad de uso
- Formulario compacto (750x550px)
- Arquitectura gemela de `grid_ProvHolding.aspx`

### Validaciones robustas con mensajes explicativos
- **Formato de nombre**: permite alfanuméricos, acentos, caracteres especiales (`.,:ñÑáéíóúÁÉÍÓÚ()=<>°$%@/*+-\`)
- **Campos numéricos**: validación estricta con regex `/^[0-9]{0,10}$/`
- **Mensajes específicos**: incluyen ejemplo de formato esperado (ej: "Debe utilizar este formato : 999999999")
- **Detección de cambios**: botón Grabar solo se habilita si hay modificaciones
- **Validación HTML5**: atributo `required` y `pattern` en inputs

### Filtros dinámicos avanzados
- Soporte para **3 filtros simultáneos** con operador `LIKE`
- Columnas filtrables generadas dinámicamente vía `Caption_Option`
- Filtros persistentes entre recargas mediante `GrillaFiltroInicial`
- Restricción de entrada: `onkeypress="return SoloAlfanumerico(event)"`
- Conversión automática a mayúsculas en filtros: `.toUpperCase().trim()`

### Responsividad completa
- **Ancho de columnas** calculado como porcentajes de ventana:
  - `IdProvTipoProveedores`: 10%
  - `Nombre`: 75% (con formato numérico `{0:N0}`)
  - `Botones`: 180px fijo
- **Alto de grilla** adaptativo: `$(window).height() - 200`
- **Filas por página** dinámicas: `parseInt(($(window).height() - 200) / 23)`
- Ancho total: `$(window).width() - 50`

### Seguridad multi-capa
- **Validación de perfil** por usuario: `Autentificacion.ValidaPerfil(Session["IdUser"], "ProvTipoProveedores")`
- **Log de accesos** detallado: usuario, hostname, IP, user agent, referrer, tipo de evento
- **Parámetros encriptados** en URLs: `ClassGrid.EncriptaDesEncriptaDatos` para `IdRegistro`
- **Redirección automática** a Dashboard si sin permisos
- **Modal de login** si sesión no autenticada
- **Control de botón Eliminar**: solo habilitado en modo edición (`accion==1`)

### Exportación de datos
- Botón **Excel** (`.xls`) con delimitador `\t` (tabulador)
- Botón **CSV** (`.csv`) con delimitador `;` (punto y coma)
- Función genérica `ExportGrilla` del framework
- Exporta datos visibles en grilla según filtros aplicados

### Auditoría completa
- **Log de apertura de formulario**: evento tipo "4" al abrir modal
- **Log de cierre de formulario**: evento tipo "5" al cerrar modal
- **Log de grabación**: evento tipo "6" en operación CREATE/UPDATE
- **Log de acceso a grilla**: evento tipo "3" en `Page_Load`
- Usuario y timestamp implícitos en todas las operaciones
- Trazabilidad completa mediante `ClassSigav.GrabaLogAccesos` y `Registrar_LogEvento`

### Integración con modal de empresa
- Botón de búsqueda (lupa) junto a combo `IdMaeEmpresa`
- `PopIdMaeEmpresa_ProvTipoProveedores()` abre `form_MaeEmpresa.aspx` en modal
- Consulta rápida de detalle de empresa seleccionada sin salir del formulario
- Usa `SubFormJquery` para modal contextual

### Combobox mejorado con jQuery UI
- Uso de **jQuery UI Combobox** con autocompletado
- Mejora UX permitiendo búsqueda rápida en lista de empresas
- Deshabilitado en modo visualización
- Evento `select` integrado con detección de cambios
- Campo de búsqueda por `Nombre2` (razón social)

### Indicadores de procesamiento
- `ProcesandoGrabar()`: muestra loading gif y mensaje "Procesando..."
- `OcultarProcesandoGrabar()`: oculta indicador al completar
- Mejora UX en operaciones asíncronas largas
- Bloquea botones Grabar/Eliminar durante procesamiento

### Formato de datos
- **Columna Nombre**: `String.Format("{0:N0}", row["Nombre"])`
- Formato numérico con separadores de miles (aunque es campo de texto)
- Puede ser configuración heredada de plantilla genérica

---

## 11) Estructura de datos

### Tabla ProvTipoProveedores (inferida)

| Campo | Tipo | Null | Descripción |
|-------|------|------|-------------|
| IdProvTipoProveedores | int | No | PK, Identity |
| IdMaeEmpresa | int | No | FK a MaeEmpresa (empresa del sistema) |
| Nombre | varchar(50) | No | Nombre del tipo de proveedor |
| Estado | bit | Sí | Flag activo/inactivo (inferido) |
| FechaCreacion | datetime | Sí | Timestamp de creación |
| UsuarioCreacion | varchar(50) | Sí | Usuario que creó el registro |
| FechaModificacion | datetime | Sí | Timestamp de última modificación |
| UsuarioModificacion | varchar(50) | Sí | Usuario que modificó el registro |
| Eliminado | bit | Sí | Flag de soft delete |

### Índices (sugeridos)
- **PK** en `IdProvTipoProveedores`
- **FK** en `IdMaeEmpresa`
- **Index compuesto único** en `(IdMaeEmpresa, Nombre)` para evitar duplicados por empresa
- **Index** en `Nombre` para búsquedas y ordenamiento
- **Index filtrado** en `Eliminado = 0` para consultas de registros activos

### Campos obligatorios en formulario
- `IdMaeEmpresa` (obligatorio, combobox)
- `Nombre` (obligatorio, max 50 caracteres, pattern HTML5)

### Validadores ASP.NET WebForms
- **RegularExpressionValidator** en `Nombre`: valida patrón alfanumérico con especiales
- **RequiredFieldValidator** en `Nombre`: valida campo obligatorio
- Color de error: `ForeColor="Red"`

---

## 12) Resumen

`grid_ProvTipoProveedores.aspx` implementa un CRUD WebForms **simplificado pero completo** para la gestión de tipos de proveedores, con:

- **jqGrid responsiva** con paginación automática, ordenamiento y 3 filtros simultáneos
- **Modal jQuery UI** compacto (750x550) con solo 2 campos editables
- **Servicios genéricos** de persistencia y eliminación reutilizados del framework
- **SPs parametrizados** para consulta y manipulación de datos
- **Validaciones robustas** con mensajes explicativos (formato, obligatoriedad)
- **Exportación** a Excel y CSV con delimitadores configurables
- **Seguridad multi-capa** basada en perfiles, encriptación y auditoría completa (5 tipos de eventos)
- **Diseño responsivo** adaptado a tamaño de ventana con cálculos dinámicos
- **Integración contextual** con maestro de empresas (búsqueda rápida)
- **Indicadores UX** de procesamiento asíncrono

Es un componente **maestro de clasificación** del módulo de proveedores, con **baja densidad de campos** (solo 2 editables) pero **alta cohesión funcional**. Sirve como entidad de categorización para segmentar proveedores por tipo de servicio/producto.

### Casos de uso principales

1. **Gestión de tipos**: crear, modificar, consultar y eliminar tipos de proveedores
2. **Clasificación de proveedores**: categorizar proveedores por tipo de servicio/producto
3. **Búsqueda y filtrado**: localizar tipos por empresa o nombre
4. **Exportación de catálogo**: generar listados de tipos en Excel/CSV
5. **Consulta rápida**: visualización de detalle en modo solo lectura
6. **Auditoría**: rastrear cambios y accesos al maestro de tipos

### Diferencias con componentes hermanos

**Similitudes con `grid_ProvHolding`**:
- Misma arquitectura de 2 campos editables
- Mismo tamaño de modal (750x550)
- Mismas validaciones de formato
- Mismo flujo CRUD
- Misma estructura de código (plantilla genérica)

**Diferencias con `grid_ProvProveedores`** (que tiene validaciones complejas de RUT, sincronización SICON, y múltiples campos):
- No tiene validaciones de duplicidad específicas
- No tiene sincronización externa
- No tiene pestañas o secciones
- Solo 2 campos editables vs 20+ en componentes complejos
- Ideal para maestros auxiliares de clasificación

### Patrón de diseño
Este componente sigue el patrón **Maestro Auxiliar de Clasificación**, caracterizado por:
- Entidad simple con mínimos campos
- Alta frecuencia de lectura, baja frecuencia de escritura
- Relación Many-to-One con entidad principal (ProvProveedores)
- Serve como tabla de lookup/catálogo
- Facilita filtrado y agrupación en reportes
