# Laboratorio 00- Setup Reproducible (Docker + Postgres + Jupyter) 🛠️

**Objetivo:**
Al final tendrás una “mini pila” de datos biomédicos reproducible:

1. Una base de datos PostgreSQL corriendo en Docker
2. JupyterLab corriendo en Docker
3. Un script SQL versionado que crea una tabla e inserta datos
4. Un Notebook que consulta la base de datos y muestra resultados
5. Un Pull Request (PR) como entrega

> ✅ Si logras ver un DataFrame en Jupyter con datos desde Postgres, **pasaste el lab**.

---

## 0) Reglas del curso (muy importante)

- **Nunca trabajes en `main`.**
- **No instales Python local para este lab.** Todo corre dentro de Docker.
- Si algo falla, **no te quedes atorado**: pasa al Troubleshooting y/o trabaja con un compañero.

---

## 1) Prerrequisitos

### 1.1 Crear cuenta en GitHub

- Crea/usa una cuenta en GitHub.
- Asegúrate de poder iniciar sesión.

### 1.2 Instalar VS Code

- Instala Visual Studio Code.

### 1.3 Instalar Git

- Instala Git.
- Verifica en terminal (PowerShell o VS Code Terminal):

```powershell
git --version
```

### 1.4 Instalar Docker Desktop

- Instala Docker Desktop.
- **Abre Docker Desktop** y espera a que diga **Engine running**.
- Verifica en terminal:

```powershell
docker --version
docker version
```

> ✅ Debes ver sección **Server: Docker Desktop** en `docker version`.

---

## 2) Descargar el repositorio del curso (clonar)

### 2.1 Abrir terminal en VS Code

- Abre VS Code
- Abre la carpeta donde quieres guardar el proyecto
- Abre Terminal: `Terminal → New Terminal`

### 2.2 Clonar el repositorio

Copia el link del repo y ejecuta:

```powershell
git clone <URL_DEL_REPO>
cd <NOMBRE_DEL_REPO>
```

> Tip: si ya lo clonaste antes, entra a la carpeta con `cd`.

---

## 3) Entrar al laboratorio

Ve a la carpeta del lab (ajusta según tu estructura):

```powershell
cd labs/lab-0-setup
```

(En tu repo puede llamarse distinto; sigue la estructura que te dieron.)

---

## 4) Levantar la pila (Docker Compose)

### 4.1 Confirmar que Docker Desktop está abierto

Docker Desktop debe estar **abierto**. Si lo cerraste, vuelve a abrirlo.

### 4.2 Levantar servicios

Ejecuta:

```powershell
docker compose up -d
```

Esto debe crear/levantar:

- `db` (PostgreSQL)
- `jupyter` (JupyterLab)

### 4.3 Verificar que está corriendo

```powershell
docker ps
```

✅ Debes ver algo como:

- `lab-0-setup-db-1` (postgres)
- `lab-0-setup-jupyter-1` (jupyter)

---

## 5) Abrir JupyterLab

Abre en tu navegador:

- [http://localhost:8888](http://localhost:8888)

Deberías ver JupyterLab con archivos del repo en la carpeta `/work`.

---

## 6) Prueba de conexión (Notebook)

Abre `connection_test.ipynb` y ejecuta las celdas.

Debe salir algo como:

- “Connection Successful…”
- una tabla con `connection_status = 1`

> Si esto funciona, tu infraestructura está lista.

---

## 7) Crear datos en la base de datos (SQL reproducible)

Ahora crearemos una tabla simple `patient` y cargaremos 5 filas.

### 7.1 Crear el archivo SQL

En el repositorio, crea la carpeta sql/ (si no existe) y el archivo:

sql/001_init.sql


Copia tal cual el siguiente contenido:

```
CREATE TABLE IF NOT EXISTS patient (
  patient_id SERIAL PRIMARY KEY,
  sex TEXT CHECK (sex IN ('M','F','O')),
  birth_date DATE,
  city TEXT,
  has_chronic_condition BOOLEAN,
  created_at TIMESTAMP DEFAULT now()
);
```

INSERT INTO patient (sex, birth_date, city, has_chronic_condition) VALUES
('F', '1990-05-12', 'Guatemala', true),
('M', '1984-11-03', 'Antigua', false),
('O', '2001-07-21', 'Quetzaltenango', true),
('F', '1975-02-18', 'Guatemala', false),
('M', '1998-09-30', 'Escuintla', false);

> Esta tabla representa un ejemplo simplificado de pacientes: sexo, fecha de nacimiento, ciudad y si tienen una condición crónica.

---

7.2 Ejecutar el SQL en PostgreSQL (Windows PowerShell)

⚠️ Importante (Windows): PowerShell no soporta < archivo.sql como bash.

Ejecuta este comando desde la carpeta del laboratorio:

Get-Content .\sql\001_init.sql | docker compose exec -T db psql -U uvg_user -d health_data

Si todo salió bien, deberías ver algo como:

CREATE TABLE

INSERT 0 5


Esto confirma que los datos fueron creados correctamente.


---

8) Primera consulta (copiar y ejecutar)

Ahora verificaremos que los datos se pueden consultar desde el entorno analítico.

En connection_test.ipynb, agrega una celda y copia exactamente este código:

df = pd.read_sql("""
SELECT patient_id, sex, birth_date, city, has_chronic_condition, created_at
FROM patient
ORDER BY patient_id;
""", engine)

df

✅ Deberías ver una tabla con 5 pacientes.

Hasta aquí, todo es copia guiada.
A partir de ahora, escribirás tus propias consultas, con ayuda.


---

9) Consultas guiadas (escritas por ti)

En esta sección NO copies consultas completas.
Sigue las instrucciones y escribe el SQL dentro del notebook.

9.1 Contar pacientes por sexo

Pregunta:

> ¿Cuántos pacientes hay por sexo?



Pistas:

Usa COUNT(*)

Agrupa por sex


Estructura sugerida (completa lo que falta):

query = """
-- selecciona el sexo
-- cuenta cuántos registros hay
-- indica la tabla
-- agrupa por sexo
"""
pd.read_sql(query, engine)

Resultado esperado:

Una tabla con columnas sex y count

Una fila por cada sexo presente



---

9.2 Pacientes con condición crónica

Pregunta:

> ¿Cuántos pacientes tienen una condición crónica?



Pistas:

La columna es has_chronic_condition

Filtra solo los que tengan true


Escribe una consulta que:

1. Cuente pacientes


2. Donde has_chronic_condition = true



(No necesitas agrupar).


---

9.3 Pacientes por ciudad (opcional, reto corto)

Pregunta:

> ¿Cuántos pacientes hay por ciudad?



Pistas:

Agrupa por city

Ordena de mayor a menor


Resultado esperado:

Ciudades con el número de pacientes en cada una


---

## 10) Entrega (Pull Request) — se hace AL FINAL (cuando todos tengan setup)


### 10.1 Crear rama del laboratorio

(El nombre exacto lo dirá el docente; ejemplo:)

```powershell
git checkout main
git pull
git checkout -b lab00-setup/grupo-XX
```

### 10.2 Guardar cambios (commit)

```powershell
git status
git add sql/001_init.sql connection_test.ipynb
git commit -m "Lab00-setup: setup reproducible + patient table + query"
```

### 11.3 Subir tu rama (push)

```powershell
git push -u origin lab00-setup/grupo-XX
```

### 11.4 Crear Pull Request (PR) en GitHub

En GitHub:

- Abre el repo
- Te aparecerá “Compare & pull request”
- Base: `main`
- Compare: `lab00-setup/grupo-XX`

**Título:** `Lab00-setup – Grupo XX`
**Descripción (pega esto):**

- Docker compose levanta Postgres + Jupyter
- Ejecuté `sql/001_init.sql` (tabla `patient` + 3 inserts)
- El notebook consulta `patient` y genera un KPI simple

✅ Entrega completa = PR creado.

---

# Troubleshooting (soluciones rápidas)

## A) “Docker daemon is not running”

- Abre Docker Desktop
- Espera “Engine running”
- Reintenta:

```powershell
docker version
```

## B) “Internal Server Error … API version …”

1. Asegúrate que Docker Desktop esté abierto
2. Quita variables de entorno (PowerShell):

```powershell
Remove-Item Env:DOCKER_API_VERSION -ErrorAction SilentlyContinue
Remove-Item Env:DOCKER_HOST -ErrorAction SilentlyContinue
Remove-Item Env:DOCKER_CONTEXT -ErrorAction SilentlyContinue
```

3. Reintenta:

```powershell
docker ps
```

## C) No abre [http://localhost:8888](http://localhost:8888)

Ver logs:

```powershell
docker compose logs -f jupyter
```

## D) Puerto 5432 ocupado

Si tu máquina ya usa Postgres en 5432:

- cambia el puerto host en `docker-compose.yml` a `5433:5432`
- vuelve a levantar:

```powershell
docker compose up -d
```

> O pide ayuda al docente.

## E) En PowerShell no funciona `< archivo.sql`

Correcto: PowerShell no usa `<` así.
Usa siempre:

```powershell
Get-Content .\sql\001_init.sql | docker compose exec -T db psql -U uvg_user -d health_data
```

---

## Checklist de éxito ✅

- [ ] Docker Desktop abierto y “Engine running”
- [ ] `docker compose up -d` funciona
- [ ] `docker ps` muestra `db` y `jupyter`
- [ ] Jupyter abre en `localhost:8888`
- [ ] Notebook muestra conexión exitosa
- [ ] Tabla `patient` creada y consultada
- [ ] PR abierto en GitHub
