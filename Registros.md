# Registros en Java

## ¿Qué es un `record`?

Un **`record`** es un tipo especial de clase introducido en **Java 16**, diseñado para representar **datos inmutables** de forma sencilla.

Se utiliza cuando una clase:

* solo contiene **datos**
* no necesita setters
* debe compararse **por el contenido**
* no tiene lógica compleja

Ejemplo mínimo:

```java
public record Persona(String nombre, int edad) {}
```

Con una sola línea, Java genera automáticamente:

* atributos `private final`
* constructor
* getters
* `equals()`
* `hashCode()`
* `toString()`

---

## ¿Para qué se usa un `record`?

Un `record` se usa como **contenedor de datos**, por ejemplo:

* una persona
* una dirección
* un resultado
* un par de valores
* un objeto de transferencia de datos (DTO)

Es ideal cuando quieres:

* evitar código repetitivo
* garantizar que los datos no cambian
* comparar objetos por valor

---

## Código equivalente generado (simplificado)

Este `record`:

```java
public record Persona(String nombre, int edad) {}
```

Es equivalente, a grandes rasgos, a esta clase tradicional:

```java
public final class Persona {

    private final String nombre;
    private final int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String nombre() {
        return nombre;
    }

    public int edad() {
        return edad;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Persona p)) return false;
        return edad == p.edad &&
               java.util.Objects.equals(nombre, p.nombre);
    }

    @Override
    public int hashCode() {
        return java.util.Objects.hash(nombre, edad);
    }

    @Override
    public String toString() {
        return "Persona[nombre=" + nombre + ", edad=" + edad + "]";
    }
}
```

Todo esto lo genera Java automáticamente al usar `record`.

---

## ¿Se puede modificar un `record`?

❌ **No**

Los atributos:

* son `private`
* son `final`
* no existen setters

Ejemplo incorrecto:

```java
Persona p = new Persona("Diego", 40);
p.nombre = "Luis"; // ❌ No compila
```

### ¿Entonces cómo se “modifica”?

Se crea un **nuevo objeto**:

```java
Persona p1 = new Persona("Diego", 40);
Persona p2 = new Persona(p1.nombre(), 41);
```

Esto es una característica clave: **inmutabilidad**.

---

## ¿Se puede añadir lógica a un `record`?

Sí, **lógica simple y coherente con los datos**.

Ejemplo correcto:

```java
public record Persona(String nombre, int edad) {

    public boolean esMayorDeEdad() {
        return edad >= 18;
    }
}
```

No es recomendable:

* lógica compleja
* estado mutable
* comportamiento que no dependa directamente de los datos

---

## ¿Cuándo **NO** usar un `record`?

No debes usar un `record` cuando:

### ❌ 1. El objeto necesita cambiar de estado

Ejemplo:

* una cuenta bancaria
* un pedido que cambia de estado
* un usuario que se actualiza

Si necesitas setters → **no es un record**.

### ❌ 2. La clase tiene mucha lógica

Ejemplo:

* reglas complejas
* cálculos internos
* comportamiento que domina sobre los datos

Ahí conviene una clase tradicional.

### ❌ 3. Necesitas herencia

Los `record`:

* son `final`
* no se pueden extender

Si necesitas jerarquías → clase normal.

### ❌ 4. Representa una entidad con identidad

Si dos objetos **no deben considerarse iguales solo por sus datos**, no uses `record`.

Ejemplo:

* entidades persistentes (BD)
* objetos con identificador único

---

## Comparación: `class` vs `record`

### Clase tradicional

```java
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

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public int getEdad() {
        return edad;
    }

    public void setEdad(int edad) {
        this.edad = edad;
    }
}
```

Características:

* mutable
* setters
* más código
* igualdad por referencia (si no sobrescribes `equals`)

### `record`

```java
public record Persona(String nombre, int edad) {}
```

Características:

* inmutable
* sin setters
* menos código
* igualdad por contenido

### Tabla comparativa

| Característica          | `class` | `record`   |
| ----------------------- | ------- | ---------- |
| Inmutabilidad           | ❌ No    | ✅ Sí       |
| Setters                 | ✅ Sí    | ❌ No       |
| `equals()` automático   | ❌ No    | ✅ Sí       |
| `hashCode()` automático | ❌ No    | ✅ Sí       |
| Herencia                | ✅ Sí    | ❌ No       |
| Código repetitivo       | ❌ Mucho | ✅ Muy poco |

---

## Resumen

> Un `record` es una clase especial para representar datos que no cambian.
> Java genera automáticamente los métodos más comunes.
> Si necesitas modificar el objeto, usa una clase normal.

> **Usa `record` para datos.
> Usa `class` para comportamiento.**
