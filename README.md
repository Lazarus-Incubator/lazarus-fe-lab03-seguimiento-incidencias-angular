# Laboratorio 3 Reporte y Seguimiento de Incidencias Urbanas Los Olivos

## Contexto
En el distrito de Los Olivos, los vecinos reportan a diario problemas urbanos como baches, luminarias apagadas, acumulacion de basura, ruidos molestos y puntos inseguros. En dias de mayor demanda, el registro manual y el seguimiento por telefono se vuelven lentos: reportes duplicados, falta de trazabilidad y dificultad para saber en que etapa esta cada atencion.

## Objetivo del proyecto
Construir una aplicacion Angular que permita registrar y gestionar reportes de incidencias urbanas y realizar seguimiento con un historial de actualizaciones, manteniendo trazabilidad de asignaciones, cambios de estado y cierres.

## Alcance
Se trabajara con dos entidades relacionadas:
- Reporte
- Actualizacion

## Entidad Reporte
### Campos
- id numerico lo maneja JSON Server
- codigo autogenerado formato municipal R-YYYY-NNNN
- tipoIncidencia lista fija Bache Alumbrado Basura Ruido Seguridad Otros
- descripcion obligatoria minimo 10 caracteres
- ubicacion obligatoria direccion o punto de referencia
- referencia opcional max 120 recomendado
- fechaRegistro YYYY-MM-DD
- prioridad lista fija Baja Media Alta
- estadoActual lista fija Registrado Asignado En atencion Resuelto Cerrado
- areaResponsable lista fija Mantenimiento Obras Serenazgo Fiscalizacion Limpieza Publica
- fechaLimite YYYY-MM-DD calculada segun prioridad
- contactoVecino opcional nombre y telefono o correo max 120 recomendado
- evidenciaUrl opcional enlace a foto max 200 recomendado

### Regla de fecha limite
La fechaLimite se calcula automaticamente segun prioridad:
- Alta 2 dias
- Media 5 dias
- Baja 10 dias

## Entidad Actualizacion
Una actualizacion representa un evento del historial del reporte.

### Campos
- id numerico lo maneja JSON Server
- reporteId clave foranea hacia Reporte
- fecha YYYY-MM-DD
- accion lista fija Registro Asignacion Visita tecnica Cambio de estado Cierre
- areaOrigen lista fija
- areaDestino lista fija opcional segun accion
- nuevoEstado lista fija opcional segun accion
- comentario opcional max 200
- evidenciaUrl opcional enlace a foto o acta max 200 recomendado

## Funcionalidades

## Listado de reportes
- Tabla con columnas sugeridas
  - codigo
  - tipoIncidencia
  - ubicacion
  - prioridad
  - areaResponsable
  - estadoActual
  - fechaRegistro
  - fechaLimite
  - indicador de vencido si hoy es mayor que fechaLimite y estadoActual no es Resuelto ni Cerrado
- Buscador por codigo o por texto en ubicacion
- Filtros combinados
  - por estadoActual
  - por prioridad
  - por tipoIncidencia
  - por areaResponsable
  - por rango de fechas fechaRegistro desde y hasta
- Boton Nuevo reporte
- Acciones por fila
  - Ver detalle
  - Editar
  - Eliminar con confirmacion

## Formulario de reporte
Mismo formulario para crear y editar.

### Al crear un reporte
1. Generar codigo R-YYYY-NNNN
2. Calcular fechaLimite segun prioridad
3. Guardar reporte
4. Crear actualizacion inicial con accion Registro areaOrigen Atencion al Vecino y nuevoEstado Registrado

### Al editar un reporte
- Permitir editar datos base del reporte
- No se edita el historial desde este formulario

## Detalle del reporte
Ruta sugerida
- /reportes/:id

Contenido
- Resumen del reporte en una tarjeta
- Listado o timeline de actualizaciones ordenado por id descendente o fecha descendente
- Boton Registrar actualizacion

## Registrar actualizacion
Se puede registrar:
- Asignacion cambia areaResponsable y estadoActual a Asignado
- Visita tecnica agrega comentario y evidencia opcional mantiene o cambia estado
- Cambio de estado cambia estadoActual
- Cierre cambia estadoActual a Cerrado y requiere comentario

Regla principal
- Al guardar una actualizacion se debe actualizar el reporte con PATCH para reflejar areaResponsable y o estadoActual segun corresponda

## Autogeneracion del codigo de reporte
### Formato
R-YYYY-NNNN

Ejemplos
- R-2026-0001
- R-2026-0002
- R-2026-0123

### Regla
El correlativo incrementa dentro del mismo año. Al cambiar el año el correlativo vuelve a 0001.

### Implementacion sugerida
Al presionar Guardar en crear:
1. Obtener el ultimo reporte desde la API ordenado por id descendente
   - GET /reportes?_sort=id&_order=desc&_limit=1
2. Si el ultimo codigo es del mismo año sumar 1 al NNNN
3. Si no existe o es de otro año iniciar 0001
4. Guardar el reporte con el codigo generado

## Reglas de validacion

## Validaciones de reporte
- tipoIncidencia obligatorio
- descripcion obligatoria minimo 10 caracteres
- ubicacion obligatoria minimo 5 caracteres
- fechaRegistro obligatorio no vacio
- prioridad obligatorio de lista
- estadoActual obligatorio de lista
- areaResponsable obligatorio de lista
- fechaLimite calculada no editable
- referencia opcional max 120 recomendado
- contactoVecino opcional max 120 recomendado
- evidenciaUrl opcional max 200 recomendado

## Validaciones de actualizacion
- fecha obligatoria
- accion obligatoria
- si accion es Asignacion areaDestino obligatorio y debe ser distinto a areaOrigen
- si accion es Cierre comentario obligatorio minimo 10 caracteres
- si nuevoEstado es Resuelto o Cerrado comentario recomendado
- comentario max 200 recomendado
- evidenciaUrl opcional max 200 recomendado

## Regla anti duplicados
No permitir crear si ya existe un reporte con el mismo:
- tipoIncidencia
- ubicacion
- fechaRegistro

Implementacion sugerida
- GET /reportes?tipoIncidencia=...&ubicacion=...&fechaRegistro=...
- Si retorna uno o mas registros mostrar error y no guardar

## Instalacion y ejecucion
1. Instalar dependencias
   - npm install
2. Levantar la API JSON Server
   - npx json-server --watch db.json --port 3000
   - API en http://localhost:3000
3. Levantar Angular
   - ng serve -o
   - App en http://localhost:4200

## Endpoints esperados

## Reportes
- GET    /reportes
- GET    /reportes/:id
- POST   /reportes
- PUT    /reportes/:id
- PATCH  /reportes/:id
- DELETE /reportes/:id
- GET    /reportes?_sort=id&_order=desc&_limit=1

## Actualizaciones
- GET    /actualizaciones?reporteId=:id&_sort=id&_order=desc
- POST   /actualizaciones
- DELETE /actualizaciones/:id opcional

## Estructura sugerida del proyecto
```
src/app/
  core/
    services/
      reportes.service.ts
      actualizaciones.service.ts
    models/
      reporte.model.ts
      actualizacion.model.ts
    utils/
      codigo-reporte.util.ts
      validators.util.ts
      sla.util.ts
  features/
    reportes/
      pages/
        reportes-list/
        reporte-form/
        reporte-detail/
      components/
        actualizacion-form/
        actualizaciones-timeline/
      reportes.routes.ts
  app.routes.ts
```
## Criterios de aceptacion checklist
- Se puede crear un reporte y se guarda en JSON Server
- El sistema autogenera codigo con formato R-YYYY-NNNN
- Se calcula fechaLimite segun prioridad
- Se puede listar reportes en tabla con buscador y filtros
- Se puede editar un reporte existente
- Se puede eliminar un reporte con confirmacion
- Existe pantalla de detalle del reporte
- Se visualiza historial de actualizaciones del reporte
- Se puede registrar una actualizacion
- Al registrar actualizacion se actualiza estadoActual y o areaResponsable del reporte
- Las reglas condicionales de la actualizacion funcionan correctamente
- Regla anti duplicados implementada si se desarrolla la mejora plus
