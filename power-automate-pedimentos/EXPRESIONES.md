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

## Condición: el correo es del dominio intelideck.com (recomendada)

Tras el desencadenador:

```powerfx
endsWith(
  toLower(
    trim(
      last(
        split(
          string(triggerOutputs()?['body/from']),
          '@'
        )
      )
    )
  ),
  'intelideck.com'
)
```

Versión simple (también sirve en la mayoría de casos):

```powerfx
contains(toLower(string(triggerOutputs()?['body/from'])), '@intelideck.com')
```

## Condición: PDF (cualquier nombre)

El nombre del archivo es indistinto (`1422CEX0126026494.pdf` u otro). Solo importa la extensión:

```powerfx
endsWith(toLower(items('Apply_to_each')?['name']), '.pdf')
```

## Valores fijos sugeridos

| Variable | Tipo | Valor |
|----------|------|-------|
| `RutaBase` | Cadena | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |
| `AnioExtracciones` | Cadena | `2026` |
