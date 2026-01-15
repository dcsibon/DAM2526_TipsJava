
# Comparación en Java: `==` vs `equals()`

En Java existen dos formas habituales de comparar:

* `==` (operador)
* `equals()` (método)

La diferencia principal es que **no comparan lo mismo**, y depende de si estás trabajando con **tipos primitivos** o con **objetos**.

---

## 1) Tipos primitivos vs objetos

### Tipos primitivos

Son valores “puros”, no objetos:

`int`, `double`, `float`, `long`, `short`, `byte`, `char`, `boolean`

✅ Con primitivos, `==` compara **el valor**:

```java
int a = 5;
int b = 5;
System.out.println(a == b); // true
```

---

### Objetos

Cualquier instancia creada con `new` (o que sea una clase), por ejemplo:

`String`, `Integer`, `Double`, `Persona`, etc.

✅ Con objetos, `==` compara **la referencia** (si son el mismo objeto en memoria):

```java
Persona p1 = new Persona("Diego", 40);
Persona p2 = new Persona("Diego", 40);

System.out.println(p1 == p2); // false (objetos distintos)
```

👉 “Referencia” significa: si ambas variables apuntan al **mismo** objeto.

---

## 2) ¿Qué hace `equals()`?

`equals()` es un método. La idea es que pueda comparar **contenido**, pero depende de si está sobrescrito.

### `equals()` por defecto (si NO lo sobrescribes)

Si una clase NO sobrescribe `equals()`, se usa la implementación de `Object`, que (simplificado) es:

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

✅ Por tanto, **si no sobrescribes `equals()` en tu clase, `equals()` y `==` hacen lo mismo**:

* comparan referencias
* no contenido

---

## 3) Por qué en `String` se usa `equals()` y no `==`

`String` es un objeto. Por tanto:

* `==` compara referencias
* `equals()` compara contenido (porque `String` sí sobrescribe `equals()`)

Ejemplo que engaña:

```java
String a = "hola";
String b = "hola";

System.out.println(a == b);      // true (a veces)
System.out.println(a.equals(b)); // true
```

### String pool (la razón del “a veces”)

Java optimiza los literales y puede reutilizar la **misma referencia** para el mismo texto (string pool). Por eso `==` puede dar `true` aquí.

Pero este caso falla:

```java
String a = new String("hola");
String b = new String("hola");

System.out.println(a == b);      // false (objetos distintos)
System.out.println(a.equals(b)); // true (contenido igual)
```

✅ Regla práctica: **para Strings, usa siempre `equals()`**.

---

## 4) Wrappers numéricos: `Integer`, `Double`, etc.

Los números primitivos funcionan con `==` (comparan valor), pero los wrappers son **objetos**.

### Caso “normal” (con `new`)

```java
Integer a = new Integer(5);
Integer b = new Integer(5);

System.out.println(a == b);      // false (referencias distintas)
System.out.println(a.equals(b)); // true  (contenido igual)
```

### Autoboxing y caché (caso “engañoso”)

Con autoboxing:

```java
Integer a = 5;
Integer b = 5;

System.out.println(a == b); // true (a veces)
```

Esto ocurre porque Java usa una caché interna para `Integer` (y otros wrappers) en el rango **-128 a 127** (aprox. comportamiento estándar).

Pero con valores fuera del rango:

```java
Integer a = 200;
Integer b = 200;

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

✅ Regla práctica: **para wrappers (`Integer`, `Long`, …) usa `equals()`** si quieres comparar valor.

---

## 5) Cuándo usar `==` y cuándo `equals()`

### Usa `==` cuando:

* compares **primitivos** (`int`, `double`, …)
* quieras comprobar si dos variables apuntan al **mismo objeto** (misma referencia)
* compares con `null` (porque `null` no tiene métodos)

Ejemplos:

```java
int a = 10, b = 10;
System.out.println(a == b); // true
```

```java
if (obj == null) { ... } // correcto
```

```java
System.out.println(obj1 == obj2); // “¿es el mismo objeto?”
```

---

### Usa `equals()` cuando:

* quieras comparar **contenido** de objetos (String, wrappers, clases propias)
* estés haciendo lógica “por valor”

Ejemplos:

```java
System.out.println("hola".equals(otroString));
System.out.println(Integer.valueOf(5).equals(otroInteger));
```

⚠️ Importante: evita NPE (NullPointerException)

```java
// MAL si otroString puede ser null:
otroString.equals("hola"); // puede lanzar NPE

// BIEN:
"hola".equals(otroString);
```

---

## 6) Sobrescribir `equals()` y `hashCode()` en clases propias

Si quieres comparar objetos por su contenido (como hace un `record`), debes sobrescribir `equals()`.

Y si sobrescribes `equals()`, **también debes sobrescribir `hashCode()`**, especialmente si vas a usar `HashSet`, `HashMap`, etc.

### Regla fundamental

> Si `a.equals(b)` es `true`, entonces `a.hashCode()` debe ser igual a `b.hashCode()`.

---

### Ejemplo completo: clase tradicional con `equals()` y `hashCode()`

```java
import java.util.Objects;

public class Persona {
    private String nombre;
    private int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String getNombre() {
        return nombre;
    }

    public int getEdad() {
        return edad;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                 // misma referencia
        if (o == null) return false;
        if (getClass() != o.getClass()) return false; // misma clase exacta

        Persona persona = (Persona) o;
        return edad == persona.edad
                && Objects.equals(nombre, persona.nombre);
    }

    @Override
    public int hashCode() {
        return Objects.hash(nombre, edad);
    }
}
```

---

## 7) Por qué `hashCode()` es importante en colecciones Hash

Colecciones como:

* `HashSet`
* `HashMap`

usan `hashCode()` para decidir “en qué zona” guardar/buscar el objeto.

Si `equals()` dice que dos objetos son iguales pero `hashCode()` no coincide, estas colecciones pueden comportarse mal:

* permitir duplicados en `HashSet`
* no encontrar claves en `HashMap`

Ejemplo conceptual:

```java
HashSet<Persona> set = new HashSet<>();
set.add(new Persona("Diego", 40));
System.out.println(set.contains(new Persona("Diego", 40))); // debería ser true
```

Para que eso funcione bien, deben estar correctamente implementados `equals()` y `hashCode()`.

---

## 8) Resumen rápido

* **Primitivos**: `==` compara **valor**
* **Objetos**: `==` compara **referencia**
* `equals()`:

  * si NO lo sobrescribes → equivale a `==`
  * si lo sobrescribes → compara contenido (según tu definición)
* `String`:

  * usa `equals()` (por pool, `==` engaña)
* Wrappers (`Integer`…):

  * usa `equals()` (por caché, `==` engaña)
* Si sobrescribes `equals()`, sobrescribe también `hashCode()`

