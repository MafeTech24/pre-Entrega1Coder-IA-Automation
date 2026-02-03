# Pre-entrega 1 — n8n (IA Automation)

## Flujo implementado
**Schedule Trigger → HTTP Request (WorldTimeAPI) → Edit Fields → Google Sheets (Get Rows) → IF → Gmail (Send) + Google Sheets (Append Row)**

---

## Nombre del caso
**Alerta horaria:** consultar hora en Argentina (Córdoba) y notificar cuando se cumpla una condición (ej.: PM).  
Además, registrar el evento en Google Sheets y evitar duplicados.

---

## Trigger
**Schedule Trigger (cada X minutos)**  
Dispara el workflow automáticamente para simular monitoreo periódico.

---

## Nodos (2–3 líneas por nodo)

### 1) Schedule Trigger
Ejecuta el flujo cada X minutos para simular un control automático continuo.

### 2) HTTP Request (WorldTimeAPI)
**GET** `https://worldtimeapi.org/api/timezone/America/Argentina/Cordoba`  
Obtiene la fecha/hora actual en formato JSON (incluye `datetime`, `utc_datetime`, etc.).

### 3) Edit Fields (Set)
Normaliza y calcula campos clave:
- `datetime`: fecha/hora local
- `hour`: hora extraída (0–23)
- `isPM`: booleano (true si `hour >= 12`)
- `dateKey`: identificador por hora (ej.: `YYYY-MM-DDTHH`) para controlar duplicados

### 4) Google Sheets — Get row(s) in sheet
Busca en la planilla si ya existe la `dateKey`.  
Esto permite evitar duplicados (idempotencia: no repetir acciones por la misma hora).

> Recomendación: mantener la columna `dateKey` como **Plain text** en Google Sheets para que no auto-convierta el valor.

### 5) IF
Evalúa dos condiciones:
1) **Condición horaria** (ej.: `isPM = true`)  
2) **No duplicado**: si `Get Rows` **NO devuelve coincidencias** (ej.: `row_number` vacío)

- **True** → continúa a Gmail + Append Row  
- **False** → corta (no envía y no escribe)

> Nota: si usás `row_number` en el IF, puede ser necesario activar **Convert types where required**.

### 6) Gmail — Send a message
Envía un email automático con los datos del evento (datetime, hour, dateKey, sentAt).

### 7) Google Sheets — Append row in sheet
Inserta una fila con `dateKey`, `datetime`, `hour` y `sentAt` para dejar registro del envío.

---

## Condicional (resumen)
Se notifica **solo si**:
- `isPM = true` (o la condición definida por la consigna), **y**
- `dateKey` **no existe** previamente en el log de Google Sheets.

Esto evita múltiples notificaciones dentro de la misma hora.

---

## Notificación
Nodo **Gmail “Send a message”** configurado con credenciales en n8n.  
Las credenciales **no se exportan** en el JSON.

---

## Buenas prácticas aplicadas
- **Credenciales seguras:** OAuth/credenciales guardadas en *n8n Credentials* (no incluidas en el export).
- **Logs y evidencia:** validación mediante Execution Log y capturas.
- **Idempotencia:** `Get Rows` + `IF` para no enviar múltiples notificaciones por la misma `dateKey`.

---

## Evidencias incluidas
- `flujoAndando.jpg` (workflow funcionando)
- `mailRecibido.jpg` (email recibido)
- `evidencia_sheet_ok.png` (registro en Google Sheets)
- `pre-Entrega1Coder.json` (export del workflow)
