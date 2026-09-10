# Sabores Catering — Ecosistema de Automatización IA

## 📌 Descripción del proyecto

**Sabores Catering** es un sistema de automatización diseñado para gestionar de forma integral las consultas y solicitudes de presupuesto de un negocio de catering.

El sistema recibe consultas de clientes por correo electrónico, registra y consulta información en Airtable, utiliza Inteligencia Artificial para analizar los mensajes y generar una respuesta sugerida, y aplica un mecanismo de **Human-in-the-Loop (HITL)** antes de realizar el envío final al cliente.

El objetivo es automatizar el proceso de atención manteniendo control humano sobre las acciones críticas.

---

## 🎯 Objetivo

Automatizar el proceso de:

**Recepción → Registro → Análisis con IA → Validación → Aprobación humana → Envío → Registro del resultado**

El sistema permite reducir tareas manuales, mantener trazabilidad de las consultas y evitar el envío de presupuestos sin una validación previa.

---

## 🛠️ Tecnologías utilizadas

| Tecnología                   | Función                                          |
| ---------------------------- | ------------------------------------------------ |
| **n8n**                      | Orquestación y automatización del workflow       |
| **Airtable**                 | Base de datos y almacenamiento de información    |
| **Google Gemini**            | Análisis de consultas y generación de respuestas |
| **Gmail**                    | Recepción y envío de correos                     |
| **Webhooks**                 | Gestión de las acciones de aprobación y rechazo  |
| **Structured Output Parser** | Estructuración de la respuesta generada por IA   |

---

## ⚙️ Funcionamiento del sistema

El flujo principal funciona de la siguiente manera:

1. **Recepción de consulta**

   * Gmail Trigger detecta nuevos correos.

2. **Normalización de datos**

   * Se obtienen datos como email del cliente, identificador del mensaje e ID de consulta.

3. **Gestión de clientes**

   * n8n consulta Airtable para determinar si el cliente ya existe.
   * Si no existe, se crea automáticamente.

4. **Consulta de servicios**

   * Se obtiene información de los servicios disponibles desde Airtable.

5. **Análisis mediante IA**

   * Google Gemini analiza la consulta utilizando el mensaje del cliente y los servicios disponibles.
   * Identifica información relevante como:

     * tipo de evento
     * cantidad de invitados
     * fecha
     * servicio recomendado
     * restricciones alimentarias
     * prioridad
     * nivel de interés
     * datos faltantes

6. **Validación**

   * El sistema determina si están disponibles los datos mínimos necesarios para elaborar una propuesta.

7. **Registro**

   * La información procesada se almacena en Airtable.

8. **Human-in-the-Loop**

   * Si la información es suficiente, se genera una solicitud de aprobación.
   * El responsable puede aprobar o rechazar la propuesta.

9. **Envío**

   * Si se aprueba, n8n obtiene dinámicamente los datos del cliente y envía la respuesta mediante Gmail.

10. **Actualización**

    * La consulta queda registrada con su estado final:

      * Enviado
      * Rechazado
      * Error

---

## 🤖 Inteligencia Artificial

El sistema utiliza **Google Gemini** integrado mediante n8n.

La IA recibe como contexto:

* mensaje original del cliente
* información de los servicios disponibles en Airtable

A partir de ese contexto genera información estructurada para continuar el flujo.

Se incorporaron reglas para evitar la invención de información comercial y para diferenciar entre datos obligatorios y datos opcionales.

Además, el uso de **Structured Output Parser** permite que la salida de la IA mantenga una estructura compatible con las siguientes etapas de automatización.

---

## 🗄️ Base de datos — Airtable

El proyecto utiliza Airtable como base de datos y memoria operativa del sistema.

### Tablas principales

#### Clientes

Almacena información básica de los clientes:

* Nombre
* Email
* Empresa
* Teléfono
* Fecha de Alta

#### Consultas

Registra cada consulta recibida y su procesamiento:

* ID Consulta
* Cliente
* Mensaje
* Estado
* Tipo de Evento
* Cantidad de Invitados
* Fecha del evento
* Restricciones alimentarias
* Prioridad IA
* Nivel de Interés
* Resumen IA
* Datos faltantes
* Respuesta sugerida
* Servicio solicitado
* Error
* Fechas de procesamiento y envío

#### Servicios

Contiene la información comercial utilizada por la IA:

* Servicio
* Descripción
* Precio por persona
* Mínimo de personas
* Opciones vegetarianas
* Opciones sin TACC
* Estado activo

#### Errores

Permite registrar errores técnicos producidos durante la ejecución del sistema.

### 🔗 Vista de control

**[Ver Airtable — Vista de control](https://airtable.com/app91JknHff3ig2wJ/shrll87Tv2bRWS8Vq)**

---

## 👤 Human-in-the-Loop

Las propuestas no se envían directamente al cliente.

Antes del envío final, el responsable recibe un correo de aprobación con:

* información del evento
* cantidad de invitados
* servicio sugerido
* respuesta preparada por la IA
* opciones para aprobar o rechazar

### Aprobación

La aprobación activa un webhook que:

1. identifica la consulta
2. recupera los datos del cliente
3. envía la respuesta mediante Gmail
4. actualiza la consulta a **Enviado**

### Rechazo

La propuesta no se envía y la consulta queda registrada como **Rechazado**.

Este mecanismo permite mantener supervisión humana sobre una acción crítica del sistema.

---

## 🛡️ Seguridad y resiliencia

El workflow contempla diferentes mecanismos de protección:

* credenciales administradas mediante el sistema de credenciales de n8n
* variables dinámicas en lugar de datos comerciales hardcodeados
* validación de datos antes del envío
* separación entre información interna y respuesta enviada al cliente
* control mediante aprobación humana
* registro de estados y errores
* manejo de consultas con información incompleta
* protección de credenciales y datos sensibles fuera del repositorio público

El repositorio **no debe contener API Keys, contraseñas, credenciales ni URLs privadas de webhooks**.

---

## ❌ Manejo de errores

El sistema contempla dos tipos principales de errores.

### Datos incompletos

Si la consulta no contiene la información mínima necesaria, no se solicita aprobación.

La consulta se registra indicando los datos faltantes.

### Errores técnicos

Los errores producidos durante la ejecución del workflow se gestionan mediante un flujo de errores basado en **Error Trigger**, con registro en Airtable.

De esta manera se mantiene trazabilidad sobre los incidentes y se evita continuar con acciones críticas cuando existe un fallo técnico.

---

## 📊 Dashboard y KPIs

Airtable funciona también como panel de control operativo.

Entre los indicadores que pueden monitorearse se encuentran:

* Total de consultas
* Consultas pendientes
* Consultas procesadas por IA
* Consultas pendientes de aprobación
* Consultas aprobadas
* Consultas enviadas
* Consultas rechazadas
* Consultas con error
* Tasa de error
* Tasa de aprobación
* Tasa de envío
* Consultas con datos faltantes
* Distribución por servicio
* Distribución por prioridad

**Tasa de error = (Consultas con error / Total de consultas) × 100**

---

## 🧪 Pruebas realizadas

El sistema fue probado mediante diferentes escenarios para validar tanto el flujo principal como las rutas alternativas.

Se contemplaron casos como:

* consulta completa
* consulta con información incompleta
* diferentes tipos de eventos
* diferentes cantidades de invitados
* restricciones alimentarias
* recomendación de servicios
* aprobación de presupuesto
* rechazo de presupuesto
* envío dinámico al cliente
* actualización del estado de la consulta
* manejo de errores

Como ejemplo de prueba, se utilizó una consulta para **40 personas** con un servicio de **$28.000 por persona**, generando un presupuesto estimado de **$1.120.000**.

---

## 📁 Archivos de la entrega

### 📄 Documentación completa


Contiene la documentación completa del proyecto, incluyendo:

* arquitectura
* funcionamiento del workflow
* estructura de Airtable
* implementación de IA
* Human-in-the-Loop
* seguridad
* manejo de errores
* costos
* dashboard y KPIs
* pruebas
* manual de operación y mantenimiento

### ⚙️ Workflow de n8n


Contiene la exportación del workflow utilizado para implementar la automatización.

> Para reutilizar el workflow en otra instalación de n8n es necesario configurar nuevamente las credenciales y adaptar las conexiones externas correspondientes.

---

## 🎥 Video demostrativo

El video muestra el funcionamiento del sistema de principio a fin.

**[Ver video demostrativo](https://drive.google.com/file/d/1hmawq6lkZre8bXIygZXHwdio-TYK_cDd/view?usp=drive_link)**

La demostración incluye:

1. recepción de una consulta
2. procesamiento mediante n8n
3. análisis con IA
4. registro en Airtable
5. solicitud de aprobación
6. aprobación humana
7. envío al cliente
8. actualización del estado

---

## 📂 Estructura del repositorio

```text
Sabores-Catering-Automatizacion-IA/
│
├── README.md
├── Entrega_Final_Sabores_Catering.pdf
└── Sabores_Catering_Workflow.json
```

---

## 👨‍💻 Información del proyecto

**Proyecto:** Sabores Catering
**Curso:** Automatización con Inteligencia Artificial — Coderhouse
**Alumno:** Luciano Mellone
**Año:** 2026

---

## ✅ Resultado final

El proyecto implementa un ecosistema de automatización de extremo a extremo que integra:

**n8n + Airtable + Google Gemini + Gmail + Webhooks + Human-in-the-Loop**

El resultado es un sistema capaz de recibir consultas, procesarlas mediante IA, utilizar información almacenada en una base de datos, validar los datos disponibles, solicitar aprobación humana y realizar el envío final de manera automatizada y trazable.
