# 📋 ANÁLISIS DETALLADO: Tabla `TBSRDETParamApi`

## 🎯 **Propósito de la Tabla**

`TBSRDETParamApi` es una **tabla de relación/detalle** que conecta los **parámetros de reportes SSRS** con **endpoints de APIs externas** para proporcionar funcionalidad de búsqueda dinámica. Permite que los usuarios busquen y seleccionen valores desde APIs en lugar de escribirlos manualmente.

**Nomenclatura UNED:**
- **TBSR**: Tabla de Servicios de Reportes
- **DET**: Detalle/Relación (indica tabla de vínculo entre entidades)
- **ParamApi**: Parámetro-API (vincula parámetros con endpoints)

---

## 📊 **Estructura de Campos**

| Campo | Tipo | Propósito | Ejemplo |
|-------|------|-----------|---------|
| **IdParamApi** | `int` (PK) | Identificador único de la vinculación | `1, 2, 3...` |
| **IdParam** | `int` (FK) | ID del parámetro del reporte (→ TBSRPARReportes) | `15` (parámetro "Cédula") |
| **IdApiEndpoint** | `int` (FK) | ID del endpoint de API (→ TBSRMAEApiEndpoint) | `2` (API de personas) |
| **CampoMapeoValor** | `nvarchar(100)` | Campo del JSON de respuesta que se usará como valor del parámetro | `"identificacion"` |
| **TxtBtnBusq** | `nvarchar(100)` | Texto que aparece en el botón de búsqueda | `"🔍 Buscar Persona"` |
| **TituloModal** | `nvarchar(50)` | Título del modal de búsqueda | `"Búsqueda de Personas"` |
| **EstadoParamApi** | `bit` | Indica si la vinculación está activa | `1` (activo), `0` (inactivo) |
| **FecCreacion** | `datetime` | Fecha de creación del registro | `2025-11-27 10:30:00` |

---

## 🔑 **¿Por qué es necesario `IdParamApi`?**

El campo `IdParamApi` es **absolutamente esencial** por las siguientes razones:

### 1️⃣ **Identidad Única de la Relación**
- Cada vinculación entre un parámetro y un endpoint necesita su propio identificador único
- Permite referenciar específicamente UNA vinculación en operaciones CRUD
- Sin él, no podrías diferenciar entre múltiples vinculaciones

### 2️⃣ **Relación 1:1 con ParametroReporte**
```csharp
entity.HasOne(d => d.ParametroReporte)
      .WithOne(p => p.ApiEndpoint)
      .HasForeignKey<ParametroReporteApiEndpoint>(d => d.IdParametroReporte)
```
- Un parámetro puede tener **máximo UN endpoint** vinculado
- El `IdParamApi` permite rastrear cada vinculación individualmente
- El índice único en `IdParametroReporte` garantiza la unicidad

### 3️⃣ **Historial y Auditoría**
- Permite saber **cuándo** se creó cada vinculación (`FecCreacion`)
- Facilita auditoría: "¿Quién vinculó este parámetro con esta API?"
- Permite activar/desactivar vinculaciones sin borrarlas

### 4️⃣ **Operaciones de Actualización**
```csharp
// Sin IdParamApi sería difícil actualizar:
UPDATE TBSRDETParamApi 
SET TxtBtnBusq = 'Nuevo texto'
WHERE IdParamApi = 5  // ✅ Identificación precisa

// vs intentar sin PK:
WHERE IdParam = 15 AND IdApiEndpoint = 2  // ⚠️ Más complejo
```

### 5️⃣ **Configuraciones Múltiples Futuras**
Aunque actualmente es 1:1, el `IdParamApi` permite:
- Cambiar la vinculación de un parámetro sin perder historial
- Reactivar vinculaciones antiguas
- Soportar múltiples configuraciones de búsqueda en el futuro

---

## 🔄 **Relaciones con Otras Tablas**

```
TBSRMAEReporte (Reportes)
    ↓ (1:N)
TBSRPARReportes (Parámetros)
    ↓ (1:1)
TBSRDETParamApi (Vinculación) ←── ¡Esta tabla!
    ↓ (N:1)
TBSRMAEApiEndpoint (APIs)
```

**Ejemplo de flujo:**
1. Reporte: "Consulta de Estudiantes" → IdReporte = 10
2. Parámetro: "Cédula del Estudiante" → IdParam = 45
3. Vinculación: Conecta parámetro 45 con API de personas → **IdParamApi = 12**
4. Endpoint: "API Registro Civil" → IdApiEndpoint = 3

---

## 💡 **Casos de Uso Reales**

### **Escenario 1: Búsqueda de Personas**
```
Parámetro: "Identificación"
Endpoint: API de Registro Civil
CampoMapeoValor: "cedula"
TxtBtnBusq: "🔍 Buscar en Registro"
TituloModal: "Búsqueda de Ciudadanos"
```

Cuando el usuario hace clic en el botón:
1. Se abre un modal con campos de búsqueda
2. El usuario busca por nombre o cédula
3. El API retorna: `[{"cedula":"104560789","nombre":"Juan Pérez"}]`
4. El usuario selecciona un resultado
5. El valor `"104560789"` (campo `cedula`) se inserta en el parámetro del reporte

### **Escenario 2: Búsqueda de Cursos**
```
Parámetro: "Código de Curso"
Endpoint: API de Oferta Académica
CampoMapeoValor: "codigoCurso"
TxtBtnBusq: "Seleccionar Curso"
TituloModal: "Catálogo de Cursos UNED"
```

---

## 📝 **Script SQL: INSERT de 6 Registros de Ejemplo**

```sql
-- ============================================
-- INSERTS DE EJEMPLO PARA TBSRDETParamApi
-- ============================================

-- Registro 1: Búsqueda de personas por cédula
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (1, 1, 'identificacion', '🔍 Buscar Persona', 'Búsqueda de Personas', 1, GETDATE());

-- Registro 2: Búsqueda de estudiantes
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (2, 2, 'numeroEstudiante', '🎓 Buscar Estudiante', 'Consulta de Estudiantes UNED', 1, GETDATE());

-- Registro 3: Búsqueda de cursos
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (3, 3, 'codigoCurso', '📚 Seleccionar Curso', 'Catálogo de Cursos', 1, GETDATE());

-- Registro 4: Búsqueda de empleados
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (4, 4, 'codigoEmpleado', '👤 Buscar Empleado', 'Directorio de Personal', 1, GETDATE());

-- Registro 5: Búsqueda de centros universitarios
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (5, 5, 'codigoCentro', '🏛️ Seleccionar Centro', 'Centros Universitarios UNED', 1, GETDATE());

-- Registro 6: Búsqueda de expediente digital (Parámetro con API compleja)
INSERT INTO TBSRDETParamApi 
    (IdParam, IdApiEndpoint, CampoMapeoValor, TxtBtnBusq, TituloModal, EstadoParamApi, FecCreacion)
VALUES 
    (15, 1, 'identificacion', '📂 Buscar Expediente', 'Sistema de Expediente Digital', 1, GETDATE());
```

---

## 🔍 **Consultas Útiles para Verificar Datos**

### Ver todas las vinculaciones con nombres legibles
```sql
SELECT 
    pa.IdParamApi,
    r.NombReporte AS 'Reporte',
    pr.NombreEtiqueta AS 'Parámetro',
    ae.NombreEndpoint AS 'API',
    pa.CampoMapeoValor,
    pa.TxtBtnBusq AS 'Texto Botón',
    pa.TituloModal,
    CASE WHEN pa.EstadoParamApi = 1 THEN '✅ Activo' ELSE '❌ Inactivo' END AS 'Estado'
FROM TBSRDETParamApi pa
INNER JOIN TBSRPARReportes pr ON pa.IdParam = pr.IdParam
INNER JOIN TBSRMAEReporte r ON pr.IdReporte = r.IdReporte
INNER JOIN TBSRMAEApiEndpoint ae ON pa.IdApiEndpoint = ae.IdApiEndpoint
ORDER BY r.NombReporte, pr.OrdenVisual;
```

### Contar vinculaciones por API
```sql
SELECT 
    ae.NombreEndpoint,
    COUNT(*) AS 'Parámetros Vinculados'
FROM TBSRDETParamApi pa
INNER JOIN TBSRMAEApiEndpoint ae ON pa.IdApiEndpoint = ae.IdApiEndpoint
WHERE pa.EstadoParamApi = 1
GROUP BY ae.NombreEndpoint;
```

### Parámetros SIN vinculación a API
```sql
SELECT 
    r.NombReporte,
    pr.NombreEtiqueta,
    pr.TipoDato
FROM TBSRPARReportes pr
INNER JOIN TBSRMAEReporte r ON pr.IdReporte = r.IdReporte
WHERE NOT EXISTS (
    SELECT 1 FROM TBSRDETParamApi pa 
    WHERE pa.IdParam = pr.IdParam
);
```

---

## ⚙️ **Funcionamiento en la Aplicación**

### **1. Renderizado del Parámetro**
```csharp
// En Index.cshtml.cs (Ejecución de Reporte)
var parametro = await _context.ParametrosReportes
    .Include(p => p.ApiEndpoint)
        .ThenInclude(a => a.ApiEndpoint)
    .FirstOrDefaultAsync(p => p.IdParametro == id);

if (parametro.ApiEndpoint != null)
{
    // Mostrar input con botón de búsqueda
    <input type="text" id="@parametro.IdParametro" />
    <button onclick="abrirModalBusqueda(@parametro.ApiEndpoint.IdApiEndpoint)">
        @parametro.ApiEndpoint.TextoBotonBusqueda
    </button>
}
```

### **2. Llamada a la API**
```javascript
// wwwroot/js/busqueda-api.js
function buscarEnApi(idParamApi, terminoBusqueda) {
    fetch(`/api/busqueda/${idParamApi}?q=${terminoBusqueda}`)
        .then(response => response.json())
        .then(data => {
            // data viene del campo "CampoMapeoValor" del API
            mostrarResultados(data);
        });
}
```

### **3. Selección del Resultado**
```javascript
function seleccionarResultado(valor, idParametro) {
    // El valor viene del campo especificado en "CampoMapeoValor"
    document.getElementById(idParametro).value = valor;
    cerrarModal();
}
```

---

## 📌 **Resumen Ejecutivo**

| Aspecto | Descripción |
|---------|-------------|
| **Función Principal** | Vincular parámetros de reportes con APIs de búsqueda |
| **Relación** | 1:1 con ParametroReporte, N:1 con ApiEndpoint |
| **IdParamApi** | PK necesaria para identidad única, auditoría y operaciones CRUD |
| **Ventaja** | Evita escribir manualmente valores complejos (cédulas, códigos) |
| **UX** | Botón de búsqueda → Modal → Selección → Autocompletado |
| **Ejemplo Real** | Parámetro "Cédula" conectado a API del Registro Civil |

---

## 🎓 **Conclusión**

El `IdParamApi` es el **corazón** de esta funcionalidad, permitiendo rastrear, modificar y gestionar cada vinculación de forma única e independiente. Sin él, sería imposible mantener la integridad referencial y realizar operaciones precisas sobre las vinculaciones.

Esta tabla representa una de las características más innovadoras del sistema, transformando la experiencia del usuario de tener que memorizar y escribir códigos complejos a simplemente buscar y seleccionar desde interfaces amigables conectadas a APIs institucionales.
