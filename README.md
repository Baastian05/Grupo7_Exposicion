# 📘 Triggers y Cursores Explícitos en PL/SQL

---

## 🔹 Triggers (BEFORE / AFTER)

### 📌 ¿Qué es un Trigger?

Un trigger es un bloque de código en **PL/SQL** que se ejecuta automáticamente cuando ocurre un evento sobre una tabla.

No necesita ser llamado manualmente, ya que la base de datos lo dispara automáticamente.

### ⚡ Eventos que activan un trigger:

* `INSERT`
* `UPDATE`
* `DELETE`

---

## 🔹 BEFORE vs AFTER

Cuando se ejecuta una operación (por ejemplo `UPDATE`), el orden es:

```
Sentencia → BEFORE → Cambio → AFTER
```

| Tipo   | Función                       |
| ------ | ----------------------------- |
| BEFORE | Se ejecuta antes del cambio   |
| AFTER  | Se ejecuta después del cambio |

👉 El **BEFORE** puede evitar que el cambio ocurra
👉 El **AFTER** actúa cuando el cambio ya fue realizado

---

## 🔹 Variables :OLD y :NEW

En triggers de tipo fila (`FOR EACH ROW`), se utilizan:

* `:OLD` → valor anterior del dato
* `:NEW` → valor nuevo del dato

Esto permite comparar cambios o tomar decisiones.

---

## 🔹 Cuándo usar BEFORE y AFTER

| BEFORE                        | AFTER                      |
| ----------------------------- | -------------------------- |
| Valida datos antes de guardar | Registra lo que ya ocurrió |
| Puede modificar `:NEW`        | No puede modificar datos   |
| Puede cancelar operación      | No puede cancelar          |
| Uso: validaciones             | Uso: auditoría, logs       |

---

## 🔹 Estructura de un Trigger

```sql
CREATE OR REPLACE TRIGGER nombre_trigger
  BEFORE | AFTER
  INSERT OR UPDATE OR DELETE
  ON nombre_tabla
  FOR EACH ROW
BEGIN
  -- lógica
END;
/
```

---

## 🔹 Ejemplo BEFORE (Validación)

```sql
CREATE OR REPLACE TRIGGER trg_valida_salario
  BEFORE INSERT OR UPDATE OF salario ON empleados
  FOR EACH ROW
BEGIN
  IF :NEW.salario < 500 THEN
    RAISE_APPLICATION_ERROR(-20001,
      'Error: El salario no puede ser menor a 500.');
  END IF;
END;
/
```

👉 Si el salario es menor a 500, la operación se cancela.

---

## 🔹 Ejemplo AFTER (Auditoría)

```sql
CREATE OR REPLACE TRIGGER trg_auditoria_salario
  AFTER UPDATE OF salario ON empleados
  FOR EACH ROW
BEGIN
  INSERT INTO auditoria_salarios
  VALUES (:OLD.id_empleado, :OLD.salario, :NEW.salario, SYSDATE, USER);
END;
/
```

👉 Se registra el cambio solo si fue exitoso.

---

## 🔹 Cursores Explícitos

### 📌 ¿Qué es un cursor?

Un cursor explícito permite recorrer múltiples filas de una consulta SQL de forma controlada, una por una.

---

## 🔹 Importancia

* Permite procesar datos fila por fila
* Da control total sobre los resultados
* Es útil en lógica compleja

---

## 🔹 Estructura del Cursor

1. `DECLARE`
2. `OPEN`
3. `FETCH`
4. `LOOP`
5. `CLOSE`

---

## 🔹 Ejemplo de Cursor

```sql
DECLARE
   CURSOR c_empleados IS
      SELECT nombre, salario FROM empleados;

   v_nombre empleados.nombre%TYPE;
   v_salario empleados.salario%TYPE;

BEGIN
   OPEN c_empleados;

   LOOP
      FETCH c_empleados INTO v_nombre, v_salario;
      EXIT WHEN c_empleados%NOTFOUND;

      DBMS_OUTPUT.PUT_LINE(v_nombre || ' - ' || v_salario);
   END LOOP;

   CLOSE c_empleados;
END;
```

---

## 🔹 Diferencia: Cursores vs Triggers

| Aspecto    | Cursores       | Triggers             |
| ---------- | -------------- | -------------------- |
| Ejecución  | Manual         | Automática           |
| Uso        | Procesar datos | Reaccionar a cambios |
| Control    | Total          | Limitado             |
| Aplicación | Reportes       | Auditoría            |

---

## 🔹 Idea clave

👉 Los cursores recorren datos
👉 Los triggers reaccionan a eventos
