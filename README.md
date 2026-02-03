# Pre-entrega n8n — Schedule → WorldTimeAPI → IF → Gmail + Google Sheets

## Nombre del caso
Alerta horaria: consultar hora en Argentina y notificar cuando se cumpla una condición (ej: PM). Además, registrar el evento en Google Sheets.

## Trigger
Schedule Trigger (cada X minutos). Dispara el workflow automáticamente.

## Nodos (2–3 líneas por nodo)
1) Schedule Trigger:
   Ejecuta el flujo cada X min para simular monitoreo automático.

2) HTTP Request (WorldTimeAPI):
   GET https://worldtimeapi.org/api/timezone/America/Argentina/Cordoba
   Obtiene fecha/hora actual en formato JSON.

3) Edit Fields (Set):
   Normaliza datos: datetime, hour, isPM y dateKey (identificador por hora).

4) Google Sheets - Get row(s) in sheet:
   Busca en la planilla si ya existe la dateKey para evitar duplicados.

5) IF:
   Evalúa si NO existe esa dateKey (idempotencia). Si no existe, continúa; si existe, corta.

6) Gmail - Send a message:
   Envía un email automático con los datos del evento.

7) Google Sheets - Append row in sheet:
   Inserta una fila con dateKey, datetime, hour y sentAt.

## Condicional
Se evalúa si debe enviarse notificación (por ejemplo isPM = true) y además si no existe la dateKey en el log (evita duplicados).

## Notificación
Gmail “Send a message” configurado con credenciales en n8n (no se exportan).

## Buenas prácticas aplicadas
- Credenciales: OAuth/credenciales guardadas en n8n Credentials (no incluidas en el JSON).
- Logs: validación mediante Execution Log y capturas.
- Idempotencia: lectura previa del log en Sheets + IF para no enviar múltiples notificaciones por la misma hora.