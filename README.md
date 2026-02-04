# 🛡️ Simple Presidio Anonymizer con UiPath

Este proyecto implementa un flujo de **anonimización y desanonimización de texto** utilizando:

- Microsoft Presidio Analyzer (ejecutado en Docker)
- UiPath Studio como POC para futuros proyectos
- Diccionarios en memoria para mantener la trazabilidad de los datos sensibles

El objetivo es:
 **detectar datos personales (PII)**, 
 **anonimizarlos de forma determinista** (`PERSON_1`, `ES_NIF_1`, etc.) y 
**poder revertir el proceso** cuando sea necesario.

---

## 📌 Características principales

✅ Detección automática de datos sensibles (PII)  
✅ Anonimización determinista y reversible  
✅ Independencia del Anonymizer de Presidio, no del Analyzer  
✅ Control total de entidades, posiciones y valores  


---

## 🧠 Arquitectura general

```
Texto original
     │
     ▼
Presidio Analyzer (/analyze)
     │
     ▼
UiPath
 ├─ dictValues  (ENTITY_ID → valor original)
 ├─ dictMeta    (ENTITY_ID → start, end, score, valor)
     │
     ▼
Anonimización local (por posiciones)
     │
     ▼
Texto anonimizado
     │
     ▼
Desanonimización (opcional, reversible)
```

---

## 🔧 Requisitos

- Docker
- Presidio Analyzer levantado en:
  ```
  http://localhost:3000/analyze
  ```
- UiPath Studio
- Paquetes UiPath:
  - UiPath.WebAPI.Activities
- Librerías .NET:
  - Newtonsoft.Json

---

## ⚙️ Instalación / Setup

1. Levantar Presidio Analyzer en Docker:
   ```bash
   docker-compose up -d
   ```

2. Abrir el proyecto en **UiPath Studio**

3. Verificar que el endpoint `/analyze` responde correctamente

---

## 🚀 Flujo del proceso

### 1️⃣ Texto de entrada

```
Hola, Marina Bargalló y Jordi Bargalló con NIF 44017895N e IP 192.168.1.1.
```

---

### 2️⃣ Llamada a Presidio Analyzer

**POST** `http://localhost:3000/analyze`

```json
{
  "text": "Hola, Marina Bargalló y Jordi Bargalló con NIF 44017895N e IP 192.168.1.1.",
  "language": "es"
}
```

---

### 3️⃣ Construcción de diccionarios

#### dictValues

```
PERSON_1      → Marina Bargalló
PERSON_2      → Jordi Bargalló
ES_NIF_1      → 44017895N
IP_ADDRESS_1  → 192.168.1.1
```

#### dictMeta

```
PERSON_1 → Start=6, End=21, Score=0.85
PERSON_2 → Start=24, End=38, Score=0.85
ES_NIF_1 → Start=47, End=56, Score=1.0
IP_ADDRESS_1 → Start=62, End=73, Score=0.95
```

---

## 🔐 Anonimización

Sustituciones **de derecha a izquierda** (para que no se alteren las longitudes del texto) usando posiciones:

```
Substring(0, start) + ENTITY_ID + Substring(end)
```

Resultado:

```
Hola, PERSON_1 y PERSON_2 con NIF ES_NIF_1 e IP IP_ADDRESS_1.
```

---

## 🔄 Desanonimización

Reemplazo usando `dictValues`, ordenando claves por longitud descendente.

Resultado final:

```
Hola, Marina Bargalló y Jordi Bargalló con NIF 44017895N e IP 192.168.1.1.
```

---

## 💾 Persistencia

```vb
JsonConvert.SerializeObject(dictValues)
```

```vb
JsonConvert.DeserializeObject(Of Dictionary(Of String, String))(json)
```

---

## ✅ Ventajas

- Trazabilidad completa
- Reversibilidad controlada
- Cumplimiento GDPR
- Automatización completa con UiPath

---

## 📄 Proyecto de ejemplo

**SimpleAnonymizeDemo** – UiPath + Presidio Anonymization Demo
