# Documentacion Tecnica

## Resumen del Proyecto

Este proyecto implementa un sistema de automatizacion end-to-end para la generacion de propuestas comerciales, integrando Notion como base de datos central, un modelo de IA (Claude Sonnet) para generar contenido personalizado, n8n como orquestador del flujo de trabajo, y Gmail para el envio final al cliente. El sistema incluye un checkpoint de aprobacion humana (Human-in-the-loop) antes de enviar cualquier propuesta, y un manejo centralizado de errores.

## Arquitectura del Sistema

El stack tecnologico se compone de cuatro herramientas principales. Notion almacena la base de datos Propuestas Comerciales - DB con las propiedades del cliente, el estado del proceso y el contenido generado. n8n aloja el workflow Generador de Propuestas Comerciales y orquesta la comunicacion entre todos los servicios. Claude (Anthropic) es el modelo de lenguaje que redacta la propuesta comercial a partir de los datos del cliente. Gmail se utiliza para el envio del documento final una vez aprobado.

## Descripcion del Flujo (Workflow n8n)

### Nodo 1: Notion Trigger

Se activa cuando se agrega una nueva pagina a la base de datos, iniciando el proceso con los datos del cliente (nombre, industria, objetivo, presupuesto).

### Nodo 2: Message a model

Envia un prompt dinamico al modelo Claude, construido con los datos del cliente, solicitando una propuesta comercial estructurada en formato JSON.

### Nodo 3: Parsear Respuesta IA

Nodo de codigo (JavaScript) que interpreta la respuesta cruda del modelo, extrae el JSON con manejo de errores (try/catch) y aplica valores de respaldo si el parseo falla.

### Nodo 4: Actualizar Propuesta en Notion

Actualiza la pagina de Notion con la propuesta generada, precio estimado, duracion estimada y nivel de confianza, marcando el estado como En Revision.

### Nodo 5: Registrar Error en Notion

Nodo centralizado de manejo de errores. Captura fallos de cualquier nodo anterior y los registra en Notion con estado Error y el detalle del error.

### Nodo 6: Trigger Pagina Actualizada (HITL)

Se activa cuando una pagina es actualizada en Notion, permitiendo detectar cuando un humano aprueba la propuesta manualmente.

### Nodo 7: Verificar Aprobacion Humana

Nodo IF que valida que el campo Aprobado sea verdadero y que el estado sea En Revision, actuando como checkpoint de control humano antes de cualquier envio.

### Nodo 8: Marcar Propuesta como Aprobada

Actualiza el estado de la pagina en Notion a Aprobado una vez validada la condicion anterior.

### Nodo 9: Enviar Propuesta por Gmail

Envia el correo electronico al cliente con el contenido de la propuesta generada, usando los datos dinamicos del trigger HITL (destinatario, asunto y cuerpo del mensaje).

### Nodo 10: Marcar Propuesta como Enviada

Actualiza el estado final de la propuesta en Notion a Enviado, cerrando el ciclo del proceso.

## Decisiones Tecnicas

El prompt enviado al modelo de IA es completamente dinamico, construido a partir de las propiedades de cada cliente (nombre, industria, objetivo y presupuesto), evitando cualquier valor hardcodeado. El parseo de la respuesta de la IA se diseno con multiples niveles de manejo de errores para garantizar que el flujo nunca se detenga por una respuesta mal formada. Se incorporo un checkpoint de aprobacion humana (HITL) antes del envio del correo, de manera que ninguna propuesta llegue al cliente sin revision manual previa. Finalmente, se centralizo el manejo de errores en un unico nodo de Notion, evitando duplicar logica de registro de errores en cada rama del flujo.

## Control de Calidad (Fase 10)

Durante la revision final del workflow se auditaron sistematicamente todas las expresiones dinamicas utilizadas en los nodos de Notion y Gmail. Se detectaron y corrigieron dos errores de configuracion: en el nodo Marcar Propuesta como Enviada, el campo Database Page ID contenia una expresion con llaves de cierre duplicadas que impedia resolver correctamente el ID de la pagina; y en el nodo Actualizar Propuesta en Notion, la propiedad Nivel de Confianza presentaba el mismo patron de error. Ambos casos fueron corregidos y verificados. Se confirmo ademas que el flujo no contiene bucles infinitos, que los tipos de datos estan correctamente mapeados en cada propiedad de Notion, que el prompt de la IA es completamente dinamico, y que el checkpoint de aprobacion humana funciona correctamente.

## Consideraciones de Seguridad

Por politica del proyecto, ninguna credencial, clave de API o token de acceso fue incluida en esta documentacion ni fue subida al repositorio de GitHub. Todas las credenciales (Notion, Anthropic/Claude, Gmail) se gestionan exclusivamente dentro de n8n mediante su sistema interno de credenciales.
