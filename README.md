# Resumen General del Proyecto

## Tipo de Proyecto

- Aplicación web **ASP.NET Web Forms**.
- Arquitectura basada en páginas `.aspx`, controles de usuario `.ascx` y code-behind en C#.
- Proyecto sobre **.NET Framework 4.0** (según configuración actual).

## Tecnologías Usadas

- **C#** (backend / lógica de páginas y WebMethods)
- **ASP.NET Web Forms**
- **SQL Server** con procedimientos almacenados
- **AJAX** mediante WebMethods (`[WebMethod]`)
- **jQuery**
- **jQuery UI**
- **jqGrid** (grillas paginadas, filtros y exportación)
- **AjaxControlToolkit**
- **Bootstrap** (estilos y componentes visuales)

## Módulos Generales

- **Maestros de Clientes**
  - Clientes (`ClieCliente`)
  - Proyectos (`ClieProyecto`)
  - Sub Centros de Costo (`ClieSubCentroCosto`)
  - Holding (`ClieHolding`)
  - Contactos (`ClieContacto`)
  - Tipo de cliente, centro de costo, etc.

- **Maestros de Proveedores**
  - Proveedores (`ProvProveedores`)
  - Holding y tipos de proveedor
  - Parámetros comerciales/operacionales de proveedor

- **Servicios Transversales**
  - Servicio central `servicios_sigav.aspx` para operaciones genéricas (`Grabar`, `Eliminar`, `CargaDDL`, etc.)
  - Validaciones de negocio (por ejemplo, validación de RUT repetido)
  - Registro de accesos/eventos

- **Infraestructura de Interfaz**
  - Master page (`Site.master`)
  - Formularios modales reutilizables
  - Grillas con filtros, paginación y exportación (`Excel/CSV`)

## Documentación de Componentes

### Análisis de Componentes - Módulo Clientes
- [grid_ClieCliente.md](Maestros/Clientes/grid_ClieCliente.md) - Mantenimiento de Clientes
- [grid_ClieCentroCosto.md](Maestros/Clientes/grid_ClieCentroCosto.md) - Mantenimiento de Centros de Costo
- [grid_ClieProyecto.md](Maestros/Clientes/grid_ClieProyecto.md) - Mantenimiento de Proyectos
- [grid_ClieSubCentroCosto.md](Maestros/Clientes/grid_ClieSubCentroCosto.md) - Mantenimiento de Sub Centros de Costo
- [grid_ClieContacto.md](Maestros/Clientes/grid_ClieContacto.md) - Mantenimiento de Contactos de Clientes
- [grid_ClieHolding.md](Maestros/Clientes/grid_ClieHolding.md) - Mantenimiento de Holdings de Clientes

### Análisis de Componentes - Módulo Proveedores
- [grid_ProvProveedores.md](Maestros/Proveedores/grid_ProvProveedores.md) - Mantenimiento de Proveedores

### Documentación Transversal
- [Relaciones_Entidades.md](Relaciones_Entidades.md) - Catálogo centralizado de relaciones entre entidades

---

## Estructura de Documentación

Cada documento de análisis de componente sigue una estructura estándar:

1. **Descripción y función**: propósito del componente
2. **Artefactos involucrados**: archivos relacionados (ASPX, C#, JS, controles)
3. **Dependencias JS**: objetos y funciones JavaScript
4. **Dependencias C#**: WebMethods, clases DTO, servicios
5. **Procedimientos almacenados**: SPs directos e indirectos
6. **Flujo CRUD e interacciones**: detalle de cada operación (Create, Read, Update, Delete, Clone)
7. **Diagrama de objetos** (Mermaid): arquitectura y dependencias
8. **Diagrama de proceso CRUD** (Mermaid): secuencias de interacción
9. **Relaciones de datos**: link al documento centralizado
10. **Características especiales**: funcionalidades únicas del componente
11. **Estructura de datos**: tablas, campos e índices inferidos
12. **Resumen**: síntesis y casos de uso principales
