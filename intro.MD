# Informe del Proyecto de Extracción de Trades de Koinly

## Descripción General del Proyecto

Este proyecto fue creado para extraer datos de operaciones (trades) de criptomonedas desde la interfaz web de Koinly sin necesidad de contar con una suscripción de pago. Aprovechando las herramientas de desarrollo del navegador (F12), fue posible acceder a los datos JSON subyacentes y crear un parser personalizado para extraer y formatear la información de las operaciones.

## Origen y Motivación

### El Problema
- **Objetivo**: Extraer el historial de operaciones de Koinly sin pagar por funciones premium
- **Descubrimiento**: Al utilizar la interfaz web de Koinly, los datos JSON que contienen toda la información de transacciones eran visibles en la consola de desarrollo del navegador (F12)
- **Oportunidad**: Estos datos JSON podían ser copiados y procesados para extraer la información de trades de forma independiente

### La Idea
En lugar de pagar por las funciones de exportación de Koinly, podíamos:
1. Abrir Koinly en un navegador web
2. Usar las herramientas de desarrollo (F12) para acceder a los datos de red/consola
3. Extraer los datos JSON de transacciones
4. Parsearlos para extraer únicamente los trades de exchanges
5. Formatear la salida en un formato utilizable (CSV)

## Cronograma del Proyecto

### Fase 1: Descubrimiento de Datos
- Se identificó que la interfaz web de Koinly carga los datos de transacciones como JSON
- Se localizó la estructura JSON en las herramientas de desarrollo del navegador (F12)
- Se examinó la estructura de las transacciones de exchange
- Se identificó un ejemplo específico de trade: 31,356.779802 USDC → 31,383.338735 USDT

### Fase 2: Análisis de la Estructura
Se analizó la estructura JSON para transacciones de exchange:
```json
{
  "type": "exchange",
  "from": {
    "amount": "31356.779802",
    "currency": {
      "symbol": "USDC"
    }
  },
  "to": {
    "amount": "31383.338735",
    "currency": {
      "symbol": "USDT"
    }
  }
}
```

### Fase 3: Desarrollo del Script
Se creó `parse_koinly_trades.py` con las siguientes características:
- **Entrada**: Archivo JSON extraído desde la consola del navegador (`dump.txt`)
- **Filtrado**: Solo procesa transacciones con `type: "exchange"` (ignora depósitos, retiros, accruals, etc.)
- **Extracción**: Extrae:
  - Moneda de origen
  - Monto de origen
  - Moneda de destino
  - Monto de destino
  - Fecha
  - Hash de la transacción
- **Salida**:
  - Tabla formateada en consola
  - Archivo CSV (`koinly_trades.csv`) con todos los datos de trades

### Fase 4: Pruebas y Verificación
- Probado con un archivo JSON grande (594,321 líneas, 2,115 transacciones)
- Se extrajeron correctamente 340 trades de exchange
- Se verificó que el trade específico de 30K USDC → USDT fue correctamente extraído
- Se confirmó que todos los trades siguen la misma estructura y están correctamente formateados

## Implementación Técnica

### Características Clave

1. **Parseo JSON Robusto**
   - Maneja archivos grandes de forma eficiente
   - Indicadores de progreso para tiempos de procesamiento largos
   - Manejo de errores para datos malformados

2. **Filtrado Inteligente**
   - Solo extrae transacciones con `type: "exchange"`
   - Ignora: depósitos, retiros, accruals, transferencias, etc.
   - Valida que existan datos tanto en `from` como en `to`

3. **Formateo de Datos**
   - Formatea montos con decimales y separadores de miles
   - Preserva montos crudos para la exportación CSV
   - Incluye metadatos (fecha, hash de transacción) para referencia

4. **Formatos de Salida**
   - **Consola**: Tabla legible para humanos
   - **CSV**: Formato compatible con hojas de cálculo

### Estructura del Script

```text
parse_koinly_trades.py
├── format_amount()
├── parse_koinly_trades()
├── print_trades_table()
├── save_to_csv()
└── main()
```

## Resultados

### Métricas de Éxito
- ✅ Parseo exitoso de 594,321 líneas de datos JSON
- ✅ Extracción de 340 trades de exchange de 2,115 transacciones totales
- ✅ Identificación correcta del trade objetivo (30K USDC → USDT)
- ✅ Generación de salida CSV limpia y usable
- ✅ Cero pérdida de datos: todos los trades válidos fueron extraídos

### Formato de Salida

**Salida en Consola:**
```
From coin       From amount          To coin         To amount
================================================================================
USDC            31,356.78            USDT            31,383.34
USDC            7,008.00             USN             7,004.59
...
```

**Salida CSV:**
- Columnas: Moneda origen, Monto origen, Moneda destino, Monto destino, Fecha, Hash de transacción
- 341 filas (340 trades + encabezado)
- Listo para importar en Excel, Google Sheets u otras herramientas de análisis

## Uso

### Uso Básico
```bash
python3 parse_koinly_trades.py
```

### Con Archivos Personalizados
```bash
python3 parse_koinly_trades.py dump.txt output.csv
```

### Proceso
1. Abrir Koinly en el navegador
2. Abrir herramientas de desarrollo (F12)
3. Encontrar los datos JSON en Network/Console
4. Copiar y guardar en `dump.txt`
5. Ejecutar el script
6. Obtener los trades formateados en `koinly_trades.csv`

## Beneficios Obtenidos

1. **Ahorro de Costos**: No es necesario pagar por funciones premium de Koinly
2. **Acceso a Datos**: Acceso completo a todos los datos de trades de la cuenta
3. **Formateo Personalizado**: Salida adaptada a necesidades específicas
4. **Reutilización**: El script puede ejecutarse en cualquier momento con nuevos datos
5. **Transparencia**: Control total sobre la extracción y procesamiento de datos

## Desafíos Técnicos Superados

1. **Manejo de Archivos Grandes**: Archivo con más de 594K líneas
2. **Estructura JSON Compleja**: Requirió parseo cuidadoso
3. **Filtrado de Datos**: Diferenciar trades de otros tipos de transacciones
4. **Consistencia de Formato**: Asegurar que todos los trades sigan la misma estructura

## Mejoras Futuras (Opcional)

Posibles mejoras que podrían añadirse:
- Filtrar por rango de fechas
- Filtrar por monedas específicas
- Calcular ganancias/pérdidas por trade
- Incluir información de comisiones
- Soporte para otros tipos de transacciones
- Automatización de scraping web
- Monitoreo y actualizaciones en tiempo real

## Archivos Creados

1. **`parse_koinly_trades.py`**: Script principal de extracción
2. **`koinly_trades.csv`**: Archivo de salida con todos los trades extraídos
3. **`KOINLY_TRADE_EXTRACTION_REPORT_ES.md`**: Este informe

## Conclusión

Este proyecto demuestra exitosamente cómo las herramientas de desarrollo y el scripting personalizado pueden proporcionar acceso a datos que de otro modo requerirían servicios pagos. Al comprender la estructura subyacente de los datos y crear un parser específico, se logró:

- Extracción completa de trades desde la interfaz de Koinly
- Acceso gratuito a los datos de operaciones
- Salida formateada de forma personalizada para análisis
- Una solución reutilizable para futuras exportaciones

---

**Fecha del Proyecto**: Enero de 2026  
**Estado**: ✅ Completo y Verificado  
**Versión del Script**: 1.0
