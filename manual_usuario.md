# Manual de Usuario -- Módulo de Gestión de Propietarios (Vet KYMS)

## 1. Introducción

El módulo **Gestión de Propietarios** permite visualizar, registrar,
editar y eliminar la información de los dueños de mascotas dentro del
sistema Vet KYMS.

Ruta de acceso: `http://localhost:4200/owners`


## 2. Vista General

Al ingresar al módulo, se muestra una tabla con la lista de propietarios
registrados. Cada fila contiene:

-   **ID**
-   **Nombres**
-   **Apellidos**
-   **Teléfono**
-   **Email**
-   **Dirección**
-   **Acciones** (Editar / Eliminar)

También encontrarás el botón:\
**➕ Nuevo Propietario**

2. Tablas del Sistema
2.1 appointments

Descripción:
Registra las citas veterinarias programadas para las mascotas.
Incluye: fecha, hora, mascota, veterinario, motivo, estado.

2.2 laboratories

Descripción:
Almacena los resultados de los exámenes de laboratorio realizados a las mascotas.
Incluye: tipo de examen, resultados, fecha, mascota asociada.

2.3 medical_histories

Descripción:
Contiene los historiales médicos de cada mascota.
Incluye: diagnósticos, tratamientos, observaciones y fechas.

2.4 medications

Descripción:
Lista los medicamentos disponibles o utilizados en tratamientos.
Incluye: nombre, dosis, tipo, instrucciones.

2.5 owners

Descripción:
Registra la información de los dueños de las mascotas.
Incluye: nombre, dirección, teléfono, correo.

2.6 payments

Descripción:
Controla los pagos realizados por los servicios veterinarios.
Incluye: monto, fecha, método de pago, servicio asociado.

2.7 pets

Descripción:
Guarda la información de las mascotas atendidas por la veterinaria.
Incluye: nombre, especie, raza, edad, dueño.

2.8 prescription_medications

Descripción:
Asocia los medicamentos que forman parte de una receta.
Incluye: ID de la receta, medicamento, dosis, frecuencia.

2.9 prescriptions

Descripción:
Registra las recetas emitidas por los veterinarios.
Incluye: fecha, veterinario, mascota, indicaciones.

2.10 procedures

Descripción:
Almacena los procedimientos médicos realizados.
Incluye: tipo de procedimiento, descripción, mascota, veterinario.

2.11 vaccine_applications

Descripción:
Registra la aplicación de vacunas a las mascotas.
Incluye: fecha, vacuna aplicada, mascota, veterinario.

2.12 vaccines

Descripción:
Lista todas las vacunas disponibles.
Incluye: nombre, lote, fecha de expiración, descripción.

2.13 veterinarians

Descripción:
Guarda la información del personal veterinario.
Incluye: nombre, especialidad, contacto, horario.
