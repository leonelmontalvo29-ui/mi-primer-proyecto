# AGENTS.md — Factor GFC

## 1. Propósito del proyecto

Este repositorio contiene el desarrollo de un sistema de gestión documental de expedientes de crédito para Factor GFC / SOFOM.

El objetivo general es transformar un proceso principalmente manual —recepción de documentos por correo, organización en carpetas, impresión, integración física y marcado de documentos entregados— en una plataforma digital que permita:

- capturar información del cliente y del crédito;
- registrar y administrar múltiples créditos por cliente;
- registrar las personalidades relacionadas con cada crédito;
- mostrar únicamente los documentos que correspondan a cada personalidad;
- cargar documentos al expediente;
- validar documentos;
- controlar vigencias;
- generar y reutilizar formatos;
- almacenar documentos en Google Drive;
- persistir información en Google Sheets o backend equivalente;
- permitir operación mediante Portal Cliente, Portal Analista y Portal Administrador;
- construir posteriormente un expediente final ordenado y exportable.

La prioridad principal del proyecto es que el sistema sea estable, entendible, trazable y seguro antes de hacer refactorizaciones grandes o cambios estéticos.

---

## 2. Regla principal para trabajar en este repositorio

Antes de modificar código:

1. Leer este archivo completo.
2. Inspeccionar el código existente.
3. Identificar todos los archivos y funciones afectados.
4. Explicar brevemente el plan.
5. Evitar cambios innecesarios.
6. No eliminar funcionalidades existentes sin autorización.
7. No reescribir archivos completos cuando un cambio localizado sea suficiente.
8. Ejecutar pruebas o verificaciones razonables después del cambio.
9. Informar qué archivos fueron modificados.
10. Informar qué quedó pendiente o qué no pudo probarse.

Si una tarea requiere cambiar arquitectura, nombres de archivos, claves, estructura de datos, IDs, rutas de Drive o contratos entre frontend y backend, explicar primero el impacto antes de hacerlo.

---

## 3. Archivos principales

Los nombres siguientes son importantes y no deben modificarse sin autorización explícita.

Frontend principal:

```text
portal-cliente - desarrollo.html
```

Formatos:

```text
Formatos_Factor_GFC_Captura - desarrollo.html
```

El portal contiene una referencia explícita al archivo de formatos mediante:

```js
var ARCHIVO_FORMATOS = 'Formatos_Factor_GFC_Captura - desarrollo.html';
```

No renombrar este archivo ni cambiar esta constante sin revisar todas las referencias relacionadas.

También pueden existir archivos `.gs` de Google Apps Script, archivos de referencia `.xlsx`, archivos de documentación y respaldos.

No asumir que una copia con fecha o sufijo es la versión oficial. Confirmar cuál es el archivo activo antes de editar.

---

## 4. Arquitectura actual

La arquitectura actual está en transición.

Frontend:
- HTML
- CSS
- JavaScript
- `localStorage`
- IndexedDB para algunos archivos o pruebas locales

Backend:
- Google Apps Script

Persistencia:
- Google Sheets para datos estructurados
- Google Drive para documentos

El objetivo final es reducir la dependencia de `localStorage` y hacer que la información relevante pueda abrirse desde diferentes computadoras y sesiones.

`localStorage` puede seguir usándose temporalmente durante el desarrollo, pero no debe considerarse la fuente definitiva de verdad del sistema.

---

## 5. Modelo conceptual

La relación principal debe respetarse así:

```text
Cliente
└── Crédito
    ├── Datos del crédito
    ├── Personalidades
    │   ├── Acreditado
    │   ├── Representante legal
    │   ├── Aval
    │   ├── Garante
    │   └── Accionistas
    │
    ├── Documentos por personalidad
    ├── Documentos de operación
    ├── Formatos
    └── Solicitud / expediente
```

Un cliente puede tener múltiples créditos.

Cada crédito tiene:
- sus propios datos;
- su propio `idCredito`;
- su propia solicitud;
- sus propias personalidades;
- sus propios documentos;
- sus propios estados;
- sus propios formatos y datos relacionados.

Nunca mezclar datos entre créditos.

---

## 6. Personalidades soportadas

Las personalidades actualmente contempladas son:

- Acreditado Persona Moral
- Acreditado Persona Física
- Representante Legal
- Aval Persona Moral
- Aval Persona Física
- Garante Persona Moral
- Garante Persona Física
- Accionista Persona Moral
- Accionista Persona Física

Pueden existir cónyuges o relaciones derivadas cuando la lógica documental lo requiera, por ejemplo sociedad conyugal.

No inventar nuevos roles sin autorización.

Los accionistas son dinámicos:
- se pueden agregar;
- se pueden eliminar;
- pueden existir varios;
- pueden tener relaciones padre/hijo en la estructura societaria.

---

## 7. Regla de navegación por documentos

Los documentos no deben mostrarse como una sola lista global.

La experiencia esperada es:

```text
Crédito
↓
Personalidades
↓
Seleccionar personalidad
↓
Ver únicamente los documentos que correspondan a esa personalidad
```

Los documentos de operación que no pertenezcan a una personalidad concreta pueden mostrarse en un bloque separado.

Nunca volver a una lista global de documentos si eso rompe la separación por personalidad.

---

## 8. Catálogo documental y checklist

Existe un checklist oficial de Factor GFC que debe ser tratado como referencia funcional del sistema.

El portal debe respetar:
- categorías;
- documentos obligatorios;
- documentos opcionales;
- personalidades aplicables;
- vigencias;
- nombres internos;
- orden lógico del expediente.

Antes de cambiar reglas documentales, revisar el checklist de referencia disponible en el proyecto.

Si hay discrepancias entre versiones del checklist o entre el checklist y el código, no elegir arbitrariamente una versión. Reportar la discrepancia.

---

## 9. Estados documentales

Los estados de negocio esperados incluyen:

```text
Entregado
Pendiente
Pendiente de Hacer
No Aplica
```

Además, dentro del flujo digital pueden existir estados técnicos o de revisión como:

```text
En revisión
Aceptado
Rechazado
```

No mezclar sin explicación los estados de negocio con los estados técnicos.

Si se implementa un cambio de estados, debe mantenerse una conversión clara entre lo que ve el cliente, lo que ve el analista y lo que se guarda en backend.

---

## 10. Validación de documentos

El portal ya contempla reglas de validación como:
- extensión permitida;
- tamaño máximo;
- fecha del documento;
- fecha futura;
- antigüedad / vigencia.

Actualmente se manejan formatos como:
- PDF
- JPG / JPEG
- PNG
- XML
- ZIP

No ampliar formatos aceptados sin revisar el impacto en almacenamiento, seguridad y flujo de análisis.

La vigencia debe depender del documento correspondiente y no de una regla global.

---

## 11. Solicitudes

`DB.idSolicitud` representa la solicitud asociada al crédito activo.

Regla crítica:

```text
Un crédito existente no debe recibir una nueva solicitud solamente porque falte su carpeta de Drive.
```

Si `DB.idSolicitud` existe:
- conservarlo;
- comprobar si la carpeta relacionada sigue existiendo;
- si la carpeta existe, reutilizarla;
- si la carpeta fue eliminada o quedó inválida, recrearla;
- actualizar el ID de la carpeta en la fuente de datos;
- continuar usando el mismo `idSolicitud`.

No crear una nueva solicitud salvo que el flujo de negocio realmente requiera una nueva solicitud.

---

## 12. Google Drive

Existe una carpeta raíz de expedientes configurada desde Apps Script.

La función actual o equivalente puede crear una carpeta de solicitud y subcarpetas como:

```text
Solicitud
├── KYC
└── Documentos
```

Esta estructura puede evolucionar, pero no debe cambiarse sin revisar cómo afecta:
- cargas existentes;
- referencias guardadas;
- links;
- Apps Script;
- futuras exportaciones;
- expediente final.

Regla de recuperación:

Si una carpeta de solicitud fue eliminada, el backend debe poder recuperarse de forma segura manteniendo el mismo `idSolicitud`.

Antes de crear una carpeta duplicada:
- buscar por relación persistida;
- verificar si el ID existe;
- verificar si está en papelera;
- evitar duplicados;
- actualizar la referencia persistida después de recrear.

No usar manejo genérico de errores para recrear carpetas cuando el error podría ser de permisos o autorización.

---

## 13. Google Sheets / backend

La información estructurada debe persistirse en backend.

Entidades esperadas:

```text
Clientes
Créditos
Personalidades
Solicitudes
Documentos
Estados
Usuarios
```

Pueden existir hojas adicionales según la implementación.

La fuente definitiva debe permitir:
- abrir el mismo crédito desde otra computadora;
- recuperar personalidades;
- recuperar documentos;
- recuperar estados;
- recuperar relación con Drive;
- evitar depender del navegador.

No borrar datos persistidos al limpiar `localStorage`.

---

## 14. Múltiples créditos por cliente

Este requisito es obligatorio.

Cada cliente puede tener más de un crédito.

El sistema debe distinguir:
- `idCliente`;
- `idCredito`;
- `idSolicitud`.

No asumir que cliente = crédito = solicitud.

El portal ya maneja una clave de crédito actual y almacenamiento separado por `idCredito`.

Cualquier migración al backend debe conservar esa separación.

---

## 15. Paquetes de formatos

Los paquetes actuales son:

```js
pm-acreditado
pf-acreditado
pf-aval
pf-accionistas
pm-accionistas
```

La lógica existente de selección por personalidad debe conservarse salvo que el checklist oficial exija cambios.

Relación conceptual actual:

```text
pm-acreditado
- solicitud
- carta-control
- kyc-cliente
- aviso
- autorizacion-pm

pf-acreditado
- solicitud
- kyc-cliente
- aviso
- autorizacion-pm
- autorizacion-pf

pf-aval
- situacion-patrimonial
- kyc-propietario
- aviso
- autorizacion-pm
- autorizacion-pf

pf-accionistas
- kyc-propietario
- aviso
- autorizacion-pf
- autorizacion-pm

pm-accionistas
- kyc-propietario
- aviso
- autorizacion-pm
- carta-control
```

Antes de cambiar paquetes:
- revisar `PAQUETES_FMT`;
- revisar `paqueteDe(per)`;
- revisar el checklist;
- revisar las referencias dentro del archivo de formatos.

---

## 16. Puente Portal → Formatos

El portal construye un paquete de datos que alimenta `Formatos_Factor_GFC_Captura - desarrollo.html`.

El sistema usa claves canónicas vinculadas con atributos como:

```html
data-c="..."
```

Reglas:

- No renombrar claves `data-c` sin revisar todas sus referencias.
- No eliminar una clave sin comprobar si otra sección la usa.
- No mapear automáticamente un dato a varias personalidades si el dato pertenece a una persona específica.
- Datos globales del crédito pueden reutilizarse.
- Datos personales deben mantenerse aislados por personalidad.

Si existe un mapeo global como `MAPA`, revisar que no cause contaminación entre personalidades.

---

## 17. KYC

El sistema incluye procesos KYC para nuevos clientes y para personalidades relacionadas.

Objetivo:
- capturar una vez;
- reutilizar información;
- evitar recaptura manual;
- mantener la identidad de cada personalidad;
- generar los formatos correspondientes.

Debe distinguirse entre:
- KYC Cliente;
- KYC Propietario Real;
- otros formatos KYC relacionados.

Regla crítica:

```text
Nunca mezclar datos de una personalidad con otra.
```

Por ejemplo:
- RFC del aval no debe aparecer como RFC del acreditado;
- cargo público del representante no debe copiarse al acreditado;
- domicilio de una personalidad no debe sobreescribir otra.

Los datos de crédito sí pueden reutilizarse cuando corresponda.

---

## 18. Declaratorias KYC

El archivo de formatos contiene declaratorias como:

```text
D1 — cargo público propio
D2 — cargo público de cónyuge / familiar
D3 — propietario real / beneficiario
D4 — proveedor de recursos
```

Estas declaratorias deben:
- existir en el portal cuando corresponda;
- persistirse;
- enviarse al paquete de formatos;
- activar correctamente Sí / No;
- alimentar el detalle correspondiente;
- mantenerse por personalidad cuando el dato sea personal.

No dejar inputs de declaratoria sin clave canónica si deben recibir datos del portal.

---

## 19. Situación Patrimonial

El paquete de Persona Física — Aval incluye el formato de Situación Patrimonial.

Debe revisarse y terminarse el mapeo de:
- identidad;
- rol;
- fecha;
- bienes;
- pasivos;
- patrimonio;
- otros campos aplicables.

No asumir que todos los datos provienen del acreditado.

---

## 20. Portal Cliente

Objetivo final:

El cliente debe poder:
- iniciar sesión;
- ver únicamente sus créditos;
- abrir un crédito;
- ver sus personalidades;
- capturar información;
- abrir formatos;
- subir documentos;
- sustituir documentos cuando corresponda;
- ver observaciones;
- ver pendientes;
- consultar el avance.

El acceso de cliente todavía debe tratarse como autenticación real, no como una simulación de rol.

Nunca confiar en un botón del frontend para proteger información.

---

## 21. Portal Analista

Objetivo final:

El analista debe tener un dashboard donde pueda:
- ver créditos;
- buscar;
- filtrar;
- abrir un crédito;
- revisar datos;
- revisar personalidades;
- revisar documentos;
- aceptar;
- rechazar;
- agregar observaciones;
- revisar vigencias;
- identificar faltantes;
- consultar el expediente completo.

El código actual puede contener vistas de analista simuladas dentro del mismo HTML.

No confundir esa simulación con un portal multiusuario terminado.

---

## 22. Portal Administrador

Objetivo final:

El administrador debe poder gestionar:
- usuarios;
- roles;
- permisos;
- documentos requeridos;
- reglas;
- vigencias;
- paquetes;
- configuraciones del sistema;
- cambios futuros de arquitectura documental.

No hardcodear en el frontend lo que eventualmente deba ser configurable por administrador, salvo que sea una etapa temporal de desarrollo.

---

## 23. Seguridad

Principios mínimos:

- No confiar en controles del frontend como seguridad.
- Validar acceso también en backend.
- No exponer contraseñas.
- No guardar secretos en `AGENTS.md`.
- No incluir tokens privados en el repositorio.
- No permitir que cambiar un `idCredito` en la URL permita ver información ajena.
- Validar que el usuario tenga permiso sobre el crédito solicitado.
- Separar permisos de Cliente, Analista y Administrador.
- Registrar acciones relevantes cuando se implemente auditoría.

Si una tarea requiere introducir credenciales reales, pedir al usuario que las configure fuera del código fuente.

---

## 24. Historial documental

Objetivo futuro:

Cuando un documento sea sustituido:
- conservar trazabilidad;
- saber qué archivo era el anterior;
- saber cuándo se reemplazó;
- saber quién lo subió;
- saber quién lo validó;
- conocer observaciones.

No eliminar versiones anteriores de forma irreversible sin una decisión explícita del negocio.

---

## 25. Expediente final

El objetivo final es poder generar un expediente completo en orden lógico.

Debe contemplar:
- documentos;
- personalidades;
- formatos;
- orden del checklist;
- estados;
- posibles documentos no aplicables;
- revisión final.

No implementar un expediente final juntando archivos en orden arbitrario.

---

## 26. Exportaciones

Requisitos contemplados:

- exportación para formatos;
- PDF de entregados / pendientes;
- Excel respetando la estructura del checklist;
- expediente final ordenado.

Actualmente puede existir una exportación JSON usada como puente entre portal y formatos.

No confundir ese JSON con la exportación final del sistema.

---

## 27. Flujo de crédito

El portal contiene una línea de tiempo de crédito basada en pasos y etapas.

Antes de cambiar el flujo:
- revisar la fuente de referencia;
- revisar `FLUJO`;
- revisar `PASOS`;
- revisar documentos relacionados;
- revisar efectos sobre progreso.

No cambiar tiempos o etapas solo por criterio visual.

---

## 28. Regla sobre código legado

Puede existir código viejo o de pruebas.

Antes de eliminarlo:
- comprobar si aún se ejecuta;
- buscar todas sus referencias;
- revisar si algún botón, evento o función lo llama;
- determinar si ya existe una sustitución funcional.

No borrar código solo porque parezca duplicado.

Si se identifica código legado, reportarlo primero.

---

## 29. Regla sobre errores de sintaxis

Antes de implementar una tarea grande:
- validar JavaScript;
- buscar errores obvios;
- buscar comas dobles;
- buscar bloques fuera de `<html>`;
- buscar etiquetas mal cerradas;
- buscar funciones duplicadas;
- buscar referencias a IDs inexistentes.

Si hay un error de sintaxis que impide probar el proyecto, corregirlo primero con el cambio mínimo posible.

---

## 30. Pruebas mínimas después de cambios

Para cambios de frontend:

- abrir el portal;
- verificar que no existan errores de consola;
- cambiar entre vistas;
- guardar datos;
- recargar;
- comprobar persistencia;
- abrir una personalidad;
- abrir documentos;
- abrir formatos.

Para cambios de documentos:

- probar carga válida;
- probar extensión inválida;
- probar tamaño inválido;
- probar fecha futura;
- probar vigencia vencida.

Para cambios de Drive:

- solicitud nueva;
- solicitud existente;
- carpeta existente;
- carpeta eliminada;
- carpeta en papelera;
- fallo de permisos;
- subida de archivo;
- actualización de referencia.

Para cambios multiusuario:
- cliente autorizado;
- cliente no autorizado;
- analista;
- administrador;
- acceso directo por ID.

---

## 31. Forma de reportar cada tarea

Después de una modificación, responder con este esquema:

```text
Objetivo
Qué se cambió
Archivos modificados
Funciones principales afectadas
Pruebas realizadas
Resultado
Pendientes / limitaciones
Cómo probarlo manualmente
```

Evitar respuestas vagas como:
- "ya quedó";
- "todo funciona";
- "se corrigió";

si no se ejecutaron pruebas suficientes.

---

## 32. Git y control de versiones

Antes de cambios grandes:
- comprobar estado del repositorio;
- evitar sobrescribir trabajo no guardado;
- sugerir commit si existe una versión estable;
- no hacer `git reset --hard` sin autorización;
- no borrar ramas;
- no forzar push;
- no reescribir historial.

Si hay cambios no relacionados en el repositorio, no mezclarlos con la tarea actual.

---

## 33. Prioridad de implementación

Orden recomendado:

```text
1. Estabilizar archivos actuales
2. Corregir errores de sintaxis / estructura
3. Terminar KYC y formatos
4. Validar paquetes contra checklist
5. Cerrar recuperación y estructura de Drive
6. Consolidar backend
7. Migrar persistencia crítica fuera de localStorage
8. Implementar login real
9. Implementar Portal Cliente multiusuario
10. Implementar Portal Analista
11. Implementar Portal Administrador
12. Historial documental
13. Expediente final
14. Exportaciones
15. Seguridad, auditoría y pruebas finales
```

No comenzar una refactorización estética grande antes de estabilizar la lógica.

---

## 34. Tarea inicial recomendada para Codex

Si Codex abre este proyecto por primera vez, ejecutar primero una auditoría sin modificar archivos.

Prompt recomendado:

```text
Lee AGENTS.md completo.

Después revisa todo el proyecto sin modificar archivos.

Analiza:

- arquitectura;
- portal principal;
- formatos;
- Apps Script;
- persistencia;
- Google Drive;
- Google Sheets;
- creación de solicitudes;
- múltiples créditos;
- personalidades;
- documentos por personalidad;
- paquetes;
- KYC;
- Portal Cliente;
- Portal Analista;
- Portal Administrador;
- errores de sintaxis;
- funciones duplicadas;
- código legado;
- inconsistencias frontend/backend;
- riesgos de seguridad;
- dependencias entre archivos.

Compara el estado actual contra AGENTS.md.

Entrega:

1. Qué funciona.
2. Qué está parcial.
3. Qué falta.
4. Bugs detectados.
5. Riesgos.
6. Orden recomendado de implementación.

No modifiques archivos todavía.
```

---

## 35. Regla final

Este sistema maneja expedientes de crédito y documentos sensibles.

Por lo tanto:

```text
Primero integridad de datos.
Después estabilidad.
Después seguridad.
Después automatización.
Después refactorización.
Después mejoras visuales.
```

Nunca sacrificar trazabilidad o aislamiento entre clientes / créditos / personalidades por hacer el código más corto o "más limpio".
