# Generador de Propuestas Comerciales

Sistema de automatizacion end-to-end para la generacion de propuestas comerciales, integrando Notion, n8n, Claude (IA) y Gmail, con un checkpoint de aprobacion humana (HITL) antes del envio final al cliente.

## Arquitectura

El stack se compone de cuatro herramientas principales. Notion aloja la base de datos Propuestas Comerciales - DB, con las propiedades del cliente y el estado del proceso. n8n orquesta el workflow Generador de Propuestas Comerciales. Claude (Anthropic) es el modelo de lenguaje que redacta la propuesta comercial. Gmail se utiliza para el envio del documento final una vez aprobado.

## Flujo de Trabajo (n8n)

El workflow esta compuesto por 10 nodos. Notion Trigger se activa al agregar una nueva pagina a la base de datos. Message a model envia un prompt dinamico a Claude solicitando la propuesta en formato JSON. Parsear Respuesta IA interpreta la respuesta con manejo robusto de errores. Actualizar Propuesta en Notion guarda el resultado y marca el estado como En Revision. Registrar Error en Notion centraliza el manejo de errores de todo el flujo. Trigger Pagina Actualizada detecta cuando un humano interactua con la propuesta. Verificar Aprobacion Humana es el checkpoint HITL que valida la aprobacion manual. Marcar Propuesta como Aprobada actualiza el estado correspondiente. Enviar Propuesta por Gmail envia el correo final al cliente. Marcar Propuesta como Enviada cierra el ciclo del proceso.

## Decisiones Tecnicas

El prompt enviado a la IA es completamente dinamico, construido a partir de las propiedades de cada cliente. El parseo de la respuesta cuenta con multiples niveles de manejo de errores para evitar que el flujo se detenga. Se incorporo un checkpoint de aprobacion humana antes del envio del correo. El manejo de errores esta centralizado en un unico nodo de Notion.

## Control de Calidad

Durante la revision final se auditaron todas las expresiones dinamicas del workflow y se corrigieron dos errores de configuracion relacionados con llaves de cierre duplicadas en expresiones de Notion. Se confirmo que el flujo no contiene bucles infinitos, que los tipos de datos estan correctamente mapeados, que el prompt es completamente dinamico y que el checkpoint de aprobacion humana funciona correctamente.

## Seguridad

Ninguna credencial, clave de API o token de acceso esta incluida en este repositorio. Todas las credenciales (Notion, Anthropic, Gmail) se gestionan exclusivamente dentro de n8n mediante su sistema interno de credenciales.

## Estructura del Repositorio

La carpeta docs contiene la documentacion tecnica completa del proyecto. La carpeta workflow contiene el blueprint JSON exportado desde n8n, sin credenciales. La carpeta screenshots contiene capturas del workflow en funcionamiento.
