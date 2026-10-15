# Expresiones listas para pegar en Power Automate

## Nombre carpeta mes (estable)

### Compose `MesEsp`

```powerfx
switch(
  int(formatDateTime(utcNow(), 'MM')),
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

### Variable / Compose `NombreCarpetaMes`

```powerfx
concat(
  formatDateTime(utcNow(), 'MM'),
  ' ',
  outputs('MesEsp'),
  ' ',
  formatDateTime(utcNow(), 'yyyy')
)
```

Ejemplos:

- Septiembre 2026 → `09 SEPTIEMBRE 2026`
- Octubre 2026 → `10 OCTUBRE 2026`

## Ruta base dinámica por año

```powerfx
concat(
  'Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES ',
  formatDateTime(utcNow(), 'yyyy')
)
```

## Ruta completa de la carpeta del mes

```powerfx
concat(
  variables('RutaBase'),
  '/',
  variables('NombreCarpetaMes')
)
```

## Condición: el correo es de Intelideck

Tras el desencadenador (si no filtraste en el campo *De*):

```powerfx
or(
  contains(toLower(triggerOutputs()?['body/from']), 'intelideck'),
  contains(toLower(triggerOutputs()?['body/from']), 'intelideck.com')
)
```

Si en tu conector el remitente viene en otra propiedad, prueba también:

```powerfx
contains(toLower(string(triggerOutputs()?['body/from'])), 'intelideck')
```

## Condición: PDF pedimento Intelideck (recomendada)

Nombres reales tipo `1422CEX0126026494.pdf`:

```powerfx
and(
  endsWith(toLower(items('Apply_to_each')?['name']), '.pdf'),
  contains(toUpper(items('Apply_to_each')?['name']), 'CEX')
)
```

## Condición: todos los PDF del correo (si no hay otros adjuntos)

Solo si Intelideck **solo** manda PDF de pedimentos y ningún otro PDF:

```powerfx
endsWith(toLower(items('Apply_to_each')?['name']), '.pdf')
```

## Condición alternativa: PDF cuyo nombre es solo código alfanumérico

Útil si algún pedimento no trae `CEX` en el nombre. Ejemplo válido: `1422CEX0126026494.pdf`.

```powerfx
and(
  endsWith(toLower(items('Apply_to_each')?['name']), '.pdf'),
  greater(
    length(
      replace(
        replace(
          toUpper(items('Apply_to_each')?['name']),
          '.PDF',
          ''
        ),
        ' ',
        ''
      )
    ),
    10
  ),
  not(contains(toLower(items('Apply_to_each')?['name']), 'factura')),
  not(contains(toLower(items('Apply_to_each')?['name']), 'guia'))
)
```

## Valores fijos sugeridos

| Variable | Tipo | Valor |
|----------|------|-------|
| `RutaBase` | Cadena | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |
| `AnioExtracciones` | Cadena | `2026` |
