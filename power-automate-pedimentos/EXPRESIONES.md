# Expresiones listas para pegar

## Dominio @intelideck.com

```powerfx
endsWith(
  toLower(triggerBody()?['from']?['emailAddress']?['address']),
  '@intelideck.com'
)
```

Fallback si `from` viene como texto:

```powerfx
contains(toLower(string(triggerBody()?['from'])), '@intelideck.com')
```

## Año (desde fecha del correo)

```powerfx
formatDateTime(triggerBody()?['receivedDateTime'], 'yyyy')
```

## Mes en español (`MesEsp`)

```powerfx
switch(
  int(formatDateTime(triggerBody()?['receivedDateTime'], 'MM')),
  1, 'ENERO',
  2, 'FEBRERO',
  3, 'MARZO',
  4, 'ABRIL',
  5, 'MAYO',
  6, 'JUNIO',
  7, 'JULIO',
  8, 'AGOSTO',
  9, 'SEPTIEMBRE',
  10, 'OCTUBRE',
  11, 'NOVIEMBRE',
  12, 'DICIEMBRE',
  'MES'
)
```

## CarpetaMes

```powerfx
concat(
  formatDateTime(triggerBody()?['receivedDateTime'], 'MM'),
  ' ',
  outputs('MesEsp'),
  ' ',
  formatDateTime(triggerBody()?['receivedDateTime'], 'yyyy')
)
```

## Rutas OneDrive

```powerfx
concat(
  '/Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES ',
  variables('Anio')
)
```

```powerfx
concat(
  variables('RutaAnio'),
  '/',
  variables('CarpetaMes')
)
```

## ¿Es PDF? (nombre indistinto)

```powerfx
endsWith(toLower(items('Apply_to_each')?['name']), '.pdf')
```

## Filtro OData Excel (corregido, una sola línea)

En *Listar filas presentes en una tabla* → Filter Query:

```text
MessageID eq '@{triggerBody()?['id']}' and NombrePDF eq '@{replace(items('Apply_to_each')?['name'], '''', '''''')}'
```

Errores comunes a evitar:

| Incorrecto | Correcto |
|------------|----------|
| `MessageID = '...'` | `MessageID eq '...'` |
| `AND` en mayúsculas raro / saltos de línea rotos | `and` en la misma línea |
| Nombre con `'` sin escapar | `replace(..., '''', '''''')` |

## Alternativa: Filtrar matriz (sin OData)

Desde:

```powerfx
body('List_rows_present_in_a_table')?['value']
```

Condición:

```powerfx
@and(
  equals(item()?['MessageID'], triggerBody()?['id']),
  equals(item()?['NombrePDF'], items('Apply_to_each')?['name'])
)
```

## ¿Ya existe en Excel?

```powerfx
greater(length(body('List_rows_present_in_a_table')?['value']), 0)
```

O con Filtrar matriz:

```powerfx
greater(length(body('Filter_array')), 0)
```

## Tamaño del adjunto (para Excel)

```powerfx
items('Apply_to_each')?['size']
```

## RutaDestino (texto para Excel)

```powerfx
concat(
  'EXTRACCIONES ',
  variables('Anio'),
  '/',
  variables('CarpetaMes')
)
```

## Opcional: duplicado en carpeta OneDrive (nombre + tamaño)

Tras *Enumerar archivos en carpeta*, Filtrar matriz:

```powerfx
@and(
  equals(item()?['Name'], items('Apply_to_each')?['name']),
  equals(item()?['Size'], items('Apply_to_each')?['size'])
)
```
