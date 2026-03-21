# Resumen General del Proyecto Sigav

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
  - Holding (`ClieHolding`)
  - Contactos (`ClieContacto`)
  - Tipo de cliente, centro de costo, subcentro, etc.

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
