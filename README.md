# Punto de Venta Dolorense

Punto de venta móvil hecho con **Flutter + SQLite local**, desarrollado con **TDD** (primero la prueba en rojo, después el código en verde, al final el refactor).

- **Materia:** Desarrollo Móvil Integral · UTNG
- **Profesor:** Anastacio Rodríguez García
- **Autor:** Carlos Alberto Camargo Villegas
- **Actividad:** Manual paso a paso con TDD, Punto de Venta

## Qué hace la app

Guarda productos y ventas en una base SQLite dentro del dispositivo (funciona sin internet), calcula totales, IVA y cambio sin errores de redondeo, descuenta el inventario automáticamente al cobrar y permite hacer el corte del día.

Ideas clave del proyecto:

- **El dinero nunca se guarda en `double`**: todo se maneja en centavos enteros.
- **El dominio es Dart puro**: no importa Flutter ni sqflite, por eso sus pruebas corren en milisegundos.
- **Cobrar usa una transacción**: o se guarda todo (venta, renglones e inventario) o no se guarda nada.

## Estructura del proyecto

```
punto_venta_dolores/
  lib/
    dominio/            reglas del negocio (Dart puro)
      dinero.dart  producto.dart  carrito.dart  venta.dart  excepciones.dart
    datos/              SQLite
      base_datos.dart  repositorio_productos.dart  repositorio_ventas.dart
    presentacion/       lo que ve el usuario
      caja_viewmodel.dart  pantalla_caja.dart
    main.dart           conecta todo (composition root)
  test/
    dominio_test.dart   11 pruebas
    datos_test.dart     6 pruebas
  capturas/             evidencia de las pruebas (rojo y verde)
```

## Cómo ejecutar las pruebas

```
flutter pub get
flutter test
```

Las pruebas de datos usan `sqflite_common_ffi`, así que corren en la computadora sin emulador, con una base SQLite en memoria que nace limpia en cada prueba.

Resultado final:

```
flutter analyze  ->  No issues found!
flutter test     ->  +17: All tests passed!
```

![Todas las pruebas en verde](capturas/final-todas-verde.jpeg)

## Pruebas realizadas

En total son **17 pruebas**: 11 de dominio y 6 de datos.

| Archivo | Grupo | Prueba | Qué valida |
|---|---|---|---|
| `dominio_test.dart` | Dinero | formatea centavos | 0, 5 y 123456 centavos se muestran como `$0.00`, `$0.05` y `$1,234.56` |
| `dominio_test.dart` | Dinero | convierte texto a centavos | `"12.5"` da 1250, `"$1,200"` da 120000 y `"0.07"` da 7 |
| `dominio_test.dart` | Dinero | rechaza texto inválido | Texto vacío y `"1.2.3"` lanzan `FormatException` |
| `dominio_test.dart` | Producto | ida y vuelta a mapa conserva los datos | `aMapa` y `desdeMapa` no pierden información |
| `dominio_test.dart` | Producto | copiarCon cambia solo lo indicado | Una copia modifica un campo y deja los demás igual |
| `dominio_test.dart` | Carrito | suma cantidades del mismo producto | Dos nieves dan un renglón con cantidad 2 y total de 5000 |
| `dominio_test.dart` | Carrito | no excede existencias | Pedir más de lo disponible lanza `ExistenciasInsuficientesException` |
| `dominio_test.dart` | Carrito | desglosa IVA 16% del total | Con $100.00 el IVA es 1379 centavos y subtotal + IVA = total |
| `dominio_test.dart` | Carrito | calcula cambio | Total de $25.00 pagando $50.00 da $25.00 de cambio |
| `dominio_test.dart` | Carrito | pago insuficiente lanza | Pago corto lanza `PagoInsuficienteException` |
| `dominio_test.dart` | Carrito | carrito vacío no cobra | Cobrar sin productos lanza `CarritoVacioException` |
| `datos_test.dart` | Productos | guarda y busca producto | Un producto guardado se recupera por código y tiene `id` |
| `datos_test.dart` | Productos | código duplicado lanza excepción de dominio | El error UNIQUE de SQL se traduce a `CodigoDuplicadoException` |
| `datos_test.dart` | Ventas | cobrar descuenta inventario y registra venta | Vender 2 de 5 deja 3 en existencias y calcula el cambio |
| `datos_test.dart` | Ventas | si falla el cobro NO se descuenta inventario | Con pago corto el inventario sigue en 5 |
| `datos_test.dart` | Ventas | corte del día suma ventas de hoy | Dos ventas suman $75.00 |
| `datos_test.dart` | Productos | bajo inventario alerta | Detecta productos con 5 o menos existencias |

## Evidencia por ronda (capturas)

### Ronda 1: Dinero

**1.1 Formatear pesos.** Rojo: no existe `dinero.dart`. Verde: pasa 1 prueba.

![Ronda 1 rojo](capturas/ronda1-rojo.jpeg)
![Ronda 1 verde](capturas/ronda1-verde.jpeg)

**1.2 Texto a centavos.** Rojo: no existe `pesosACentavos`. Verde: pasan 3 pruebas.

![Ronda 1 rojo 2](capturas/ronda1-rojo-2.jpeg)
![Ronda 1 verde 2](capturas/ronda1-verde-2.jpeg)

### Ronda 2: Producto

Rojo: no existe `producto.dart`. Verde: pasan 5 pruebas.

![Ronda 2 rojo](capturas/ronda2-rojo.jpeg)
![Ronda 2 verde](capturas/ronda2-verde.jpeg)

### Ronda 3: Carrito

**Ciclo 3.1, sumar cantidades.** Rojo: no existe `carrito.dart`. Verde: pasan 6 pruebas.

![Ronda 3 ciclo 1 rojo](capturas/ronda3-ciclo1-rojo.jpeg)
![Ronda 3 ciclo 1 verde](capturas/ronda3-ciclo1-verde.jpeg)

**Ciclo 3.2, no vender lo que no hay.** Rojo: `agregar` no lanza error. Verde: pasan 7 pruebas.

![Ronda 3 ciclo 2 rojo](capturas/ronda3-ciclo2-rojo.jpeg)
![Ronda 3 ciclo 2 verde](capturas/ronda3-ciclo2-verde.jpeg)

**Ciclo 3.3, IVA incluido con enteros.** Rojo: no existe `ivaCentavos`. Verde: pasan 8 pruebas.

![Ronda 3 ciclo 3 rojo](capturas/ronda3-ciclo3-rojo.jpeg)
![Ronda 3 ciclo 3 verde](capturas/ronda3-ciclo3-verde.jpeg)

**Ciclo 3.4, cambio, pago corto y carrito vacío.** Rojo: no existe `cambio`. Verde: pasan 11 pruebas.

![Ronda 3 ciclo 4 rojo](capturas/ronda3-ciclo4-rojo.jpeg)
![Ronda 3 ciclo 4 verde](capturas/ronda3-ciclo4-verde.jpeg)

### Ronda 4: Base de datos SQLite

Rojo: no existe `repositorio_productos.dart`. Verde: pasan 2 pruebas de datos.

![Ronda 4 rojo](capturas/ronda4-rojo.jpeg)
![Ronda 4 verde](capturas/ronda4-verde.jpeg)

### Ronda 5: Cobrar con transacción

**Ciclo 5.1, cobrar descuenta inventario.** Rojo: no existe `repositorio_ventas.dart`. Verde: pasan 3 pruebas de datos.

![Ronda 5 ciclo 1 rojo](capturas/ronda5-ciclo1-rojo.jpeg)
![Ronda 5 ciclo 1 verde](capturas/ronda5-ciclo1-verde.jpeg)

**Ciclo 5.2, si falla el cobro no se descuenta inventario.** Sin rojo: la validación inicial de `cobrar` ya cubría el caso, así que la prueba pasó de inmediato y queda como protección contra errores futuros.

![Ronda 5 ciclo 2 verde](capturas/ronda5-ciclo2-verde.jpeg)

**Ciclo 5.3, corte del día y bajo inventario.** Sin rojo: `corteDelDia` y `bajoInventario` ya existían en los repositorios. Pasan 6 pruebas de datos.

![Ronda 5 ciclo 3 verde](capturas/ronda5-ciclo3-verde.jpeg)

## Hallazgos durante el desarrollo

- **Pruebas que compartían la misma base en memoria.** La prueba de código duplicado falló con un error que venía del `setUp`: sqflite reutiliza la conexión cuando se abre la misma ruta, y la base en memoria siempre tiene la misma ruta, así que el producto de la prueba anterior seguía guardado. Se corrigió en `base_datos.dart` con `singleInstance: ruta == null`, de modo que cada prueba obtiene una base nueva y limpia, y la app real sigue usando una sola conexión.
- **Prueba asíncrona.** La prueba de código duplicado se escribió con `await expectLater(...)`, porque `agregar` devuelve un `Future`.
- **Ciclos sin rojo.** Los ciclos 5.2 y 5.3 no tienen captura roja porque probaban comportamiento que el código ya tenía; se dejaron como pruebas de protección.

## Estado de la Ronda 6 (pantalla)

El ViewModel (`caja_viewmodel.dart`), la pantalla (`pantalla_caja.dart`) y `main.dart` están escritos y `flutter analyze` no reporta problemas. Como `sqflite` solo funciona en Android e iOS y no se contó con un emulador ni con un teléfono Android para esta entrega, **la app no se ejecutó en un dispositivo y no hay capturas de la pantalla**. Esta ronda no incluye pruebas automáticas nuevas.

Para correrla en un dispositivo Android:

```
flutter devices
flutter run
```

## Retos pendientes del manual

- Pruebas extra de dinero: `pesosACentavos('.5')` debe dar 50 y `formatearPesos(-250)` debe dar `-$2.50`.
- Descuento por porcentaje en el carrito, con su prueba primero.
- Prueba de `bajoInventario` con productos de 2 y 50 existencias.
- Prueba de vender las 5 nieves y luego intentar una más.
