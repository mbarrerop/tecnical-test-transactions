# **Prueba Técnica - Detección de Fraudes en Transacciones**
---

## **📌 Stack**

1. **Flask** como framework de desarrollo.
2. **GCP** como cloud services (no es necesario implementarlo, solo identificar que se necesita para el despligue).
3. **Python superior a 3.10** para el scripting necesario.

## **📌 Contexto del Problema**
En una plataforma de comercio electrónico, se registran miles de transacciones diariamente. Sin embargo, la empresa ha identificado que algunas de estas transacciones pueden ser fraudulentas, lo que genera pérdidas económicas y afecta la confianza de los clientes. Actualmente, las transacciones se almacenan en archivos CSV, y se necesita un sistema automatizado para identificarlas y gestionarlas de manera eficiente.

Para solucionar este problema, se requiere un sistema que:

1. **Permita la carga de transacciones desde un archivo CSV**.
2. **Detecte transacciones sospechosas** aplicando reglas de negocio predefinidas.
3. **Encole las transacciones sospechosas en Cloud Tasks** para ser procesadas de forma asíncrona.
4. **Guarde las transacciones sospechosas en una tabla separada** junto con la razón por la que fueron marcadas.
5. **Ejecute la detección de fraude automáticamente cada minuto** mediante Google Cloud Scheduler.

El objetivo final es optimizar la seguridad de la plataforma mediante la detección y gestión eficiente de fraudes.

## **📌 Descripción del Problema**
El sistema debe identificar **transacciones sospechosas** basándose en las siguientes reglas:

- **Más de 3 compras en menos de 1 minuto por el mismo usuario.**
- **Compras mayores a $5,000 en una sola transacción.**
- **Transacciones desde diferentes países en menos de 5 minutos por el mismo usuario.**

Si una transacción es sospechosa, debe **ser encolada en Google Cloud Tasks** y almacenada en una **tabla separada** para su posterior análisis, incluyendo el motivo de sospecha.

---

## **📌 Requisitos del Sistema**
El sistema tiene **cuatro módulos principales**:

### **1️⃣ Módulo de Carga de Transacciones desde CSV (`csv_loader`)**
📌 **Objetivo**: Permitir la carga manual de transacciones desde un archivo CSV y almacenarlas en la base de datos.

🔹 **Requerimientos:**
- Crear un endpoint `/upload-transactions` que permita subir un archivo CSV.
- Leer los datos del archivo y validarlos antes de almacenarlos.
- Insertar las transacciones en la base de datos.
- Generar logs de errores en caso de datos inválidos.

### **2️⃣ Módulo de Detección de Fraude (`fraud_detector`)**
📌 **Objetivo**: Obtener transacciones recientes, evaluarlas y encolarlas si son sospechosas.

🔹 **Flujo del proceso:**
1. **Cada minuto**, un **Cloud Scheduler** ejecuta una llamada al endpoint `/detect-fraud`.
2. Se obtienen las **transacciones del último minuto** desde la base de datos.
3. Se aplican **reglas de fraude**.
4. **Si la transacción es sospechosa:**
   - Se almacena en una **tabla separada** con la razón de sospecha.
   - Se encola en **Google Cloud Tasks** para su posterior procesamiento.

📌 **Nota**: Lo único que se debe implementar de esta parte es el **endpoint `/detect-fraud`**, el cual será llamado por el scheduler. La configuración del Cloud Scheduler no es parte de esta prueba.

### **3️⃣ Encolado de Transacciones en Cloud Tasks (`cloud_tasks_manager`)**
📌 **Objetivo**: Si una transacción es sospechosa, se envía a Cloud Tasks para su procesamiento posterior.

🔹 **Requerimientos:**
- Crear una tarea en **Google Cloud Tasks** con la transacción sospechosa.
- La tarea debe ser enviada a un **endpoint en la API (`/process-fraud`)**.
- Asegurar que la tarea incluya los datos necesarios para su análisis posterior.

### **4️⃣ Procesamiento de Tareas en Cloud Tasks (`fraud_processor`)**
📌 **Objetivo**: Cloud Tasks llama al endpoint `/process-fraud` para analizar transacciones sospechosas.

🔹 **Requerimientos:**
- Crear un endpoint `/process-fraud` que reciba las transacciones encoladas.
- Procesar la información y actualizar la **tabla de transacciones sospechosas** con el resultado del análisis.
- Opcionalmente, generar una alerta o registrar un log.

---

## **📌 Aspectos a Evaluar**

🔹 **Completitud de la Prueba**: Implementación de los módulos requeridos y correcto funcionamiento de cada uno.  
🔹 **Módulo de Test**: Se espera que el código tenga pruebas unitarias para validar su funcionalidad.  
🔹 **Clean Code**: Uso de buenas prácticas de programación, código modular, reutilizable y bien documentado.  
🔹 **Modelo de Datos**: Es fundamental identificar un buen diseño de la base de datos para almacenar transacciones y fraudes.  
🔹 **Seguridad** *(Bonus)*: Se valorará si los endpoints incluyen mecanismos de seguridad como autenticación JWT o API Keys.  

---

## **📌 Entregables Esperados**
El código debe incluir:

✅ `csv_loader/` → Módulo para cargar transacciones desde un archivo CSV.  
✅ `fraud_detector/` → Detección de fraude ejecutada por Cloud Scheduler (solo el endpoint `/detect-fraud`).  
✅ `cloud_tasks_manager/` → Módulo para encolar transacciones sospechosas.  
✅ `fraud_processor/` → Endpoint que recibe y procesa tareas desde Cloud Tasks.  
✅ **Base de datos con una tabla separada** para almacenar transacciones sospechosas junto con el motivo de sospecha.  
✅ `README.md` → Documentación con instrucciones de uso y despliegue.  
✅ **Pruebas unitarias** en cada módulo.  
✅ Configuración para **desplegar en Google Cloud Run**.  

---

## **📌 Resumen del Flujo**
✅ **1. Un usuario carga un archivo CSV con transacciones a través del endpoint `/upload-transactions`.**  
✅ **2. Las transacciones se almacenan en la base de datos.**  
✅ **3. Cloud Scheduler ejecuta `GET /detect-fraud` cada minuto.**  
✅ **4. Se obtienen transacciones del último minuto desde la base de datos.**  
✅ **5. Se aplican reglas de fraude a cada transacción.**  
✅ **6. Si una transacción es sospechosa, se guarda en una tabla separada con la razón de sospecha.**  
✅ **7. La transacción sospechosa se encola en Cloud Tasks.**  
✅ **8. Cloud Tasks llama al endpoint `/process-fraud` para su análisis final.**  

---

## **🚀 Beneficios de esta Arquitectura**
🔹 **Automatización total**: Cloud Scheduler ejecuta la detección cada minuto sin intervención manual.  
🔹 **Escalabilidad**: Cloud Tasks gestiona la carga sin afectar la API principal.  
🔹 **Reintentos automáticos**: Si `/process-fraud` falla, Cloud Tasks lo reintentará.  
🔹 **Independencia de servicios**: La API y el procesamiento de fraudes funcionan por separado.  
🔹 **Historial de fraudes**: Se mantiene un registro detallado de todas las transacciones sospechosas con su motivo.  

---
