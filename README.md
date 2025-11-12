# Project Demo: Integración GitHub + VSCode + Supabase (PostgreSQL) + Looker Studio

Este proyecto es una DEMO educativa end-to-end para que los alumnos aprendan a:
1. Clonar un repositorio de GitHub y trabajar en VSCode.
2. Crear y usar un entorno virtual de Python con dependencias en `requirements.txt`.
3. Cargar y procesar datos en un notebook (`demo_notebook.ipynb`).
4. Realizar clustering (K-Means) sobre datos de clientes.
5. Crear un proyecto en Supabase (PostgreSQL) y definir tablas para almacenar observaciones y resultados de clustering.
6. Conectar Looker Studio a la base de datos (vía API o exportando vistas) para crear visualizaciones.

---
## 1. Prerrequisitos
- Computadora con conexión a internet.
- Python 3.11+ instalado (recomendado).
- Cuenta de GitHub.
- Cuenta de Google (para Looker Studio, opcional en esta primera fase).

---
## 2. Instalar VSCode
1. Ir a https://code.visualstudio.com/
2. Descargar el instalador según tu sistema operativo (macOS o Windows).
3. Instalar y abrir VSCode.
4. Instalar extensiones recomendadas (opcional pero útil):
   - Python (Microsoft)
   - Jupyter (Microsoft)

---
## 3. Crear cuenta de GitHub (si no tienes una)
1. Ir a https://github.com/
2. Registrarte con correo institucional o personal.
3. Verificar tu correo.

---
## 4. Obtener el código del repositorio
Repositorio remoto: `https://github.com/jorgermzg15/project-demo`

Opciones:
### A) Clonar el repositorio completo (recomendado)
```bash
# Ubícate en el directorio donde quieres el proyecto
cd ~/Documents

# Clona el repositorio (esto puede ser en terminal de VSCode)
git clone https://github.com/jorgermzg15/project-demo.git

cd project-demo
```

### B) Descargar solo archivos específicos
Puedes descargar manualmente `demo_notebook.ipynb` y `requirements.txt` desde GitHub (botón Download Raw). Sin embargo, para trabajar en equipo y hacer commits se recomienda clonar.

---
## 5. Crear entorno virtual y instalar dependencias
El archivo `requirements.txt` contiene todas las librerías necesarias.

### macOS / Linux
```bash
# (Opcional) Verifica versión de Python
python3 --version

# Crear entorno virtual
python3 -m venv .venv

# Activar entorno
source .venv/bin/activate

# Instalar dependencias
python3 -m pip install --upgrade pip
pip install -r requirements.txt
```
(Si tu sistema usa `python` en lugar de `python3`, ajusta los comandos.)

### Windows (PowerShell)
```powershell
# Verifica versión de Python
python --version

# Crear entorno virtual
python -m venv .venv

# Activar entorno
.\.venv\Scripts\Activate.ps1

# Instalar dependencias
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Conda (Anaconda/Miniconda)
Si prefieres usar Conda, crea un entorno y usa pip dentro del entorno para instalar desde `requirements.txt`:

```bash
# Crear entorno con Python 3.11 (recomendado)
conda create -n project-demo python=3.11 -y

# Activar entorno
conda activate project-demo

# Instalar dependencias con pip dentro del entorno conda
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Para salir del entorno conda:

```bash
conda deactivate
```

Para desactivar el entorno virtual:
```bash
deactivate
```

---
## 6. Archivo de credenciales (NO subir a GitHub)
Crea un archivo `credentials.json` en la raíz del proyecto con tus claves para servicios externos (por ejemplo Supabase). Este archivo NO debe subirse al repositorio.

Ejemplo de `credentials.json` (reemplaza con tus valores reales):
```json
{
  "SUPABASE_URL": "https://<tu-proyecto>.supabase.co",
  "SUPABASE_KEY": "<public-anon-key>"
}
```
Agrega `credentials.json` a `.gitignore` (ver sección Bonus).

---
## 7. Crear cuenta y proyecto en Supabase
1. Ir a https://supabase.com/
2. Hacer login usando tu cuenta de GitHub (OAuth).
3. Crear un nuevo proyecto:
   - Nombre: `project-demo` (o similar).
   - Contraseña de la base de datos: genera una segura y guárdala.
4. Espera a que el proyecto se aprovisione.
5. En el panel ve a: Project Settings > API
   - Copia tu `Project URL` (SUPABASE_URL).
   - Copia tu `anon public` key (SUPABASE_KEY).
6. Llena tu `credentials.json` con esos valores.

---
## 8. Crear tablas en Supabase (PostgreSQL)
Puedes hacerlo desde la sección SQL del panel de Supabase. Ejecuta los siguientes scripts (ajusta si el esquema ya existe):

### Tabla: observations
Registra cada observación del dataset original.
```sql
create table if not exists public.observations (
   customer_id integer primary key not null,
   genre smallint not null,           -- 0 = Male, 1 = Female
   age integer not null,
   annual_income_k integer not null,
   spending_score integer not null
);
```

### Tabla: cluster_labels
Almacena la asignación de cluster para cada cliente.
```sql
create table if not exists public.cluster_labels (
   customer_id integer not null,
   cluster integer not null,
   constraint fk_cluster_customer foreign key (customer_id)
      references public.observations(customer_id)
);
```

### (Opcional) Tabla de centroides
Si quieres persistir los centroides calculados:
```sql
create table if not exists public.cluster_centroids (
  id bigint generated always as identity primary key,
  cluster integer not null,
  age_center numeric not null,
  annual_income_k_center numeric not null,
  spending_score_center numeric not null,
  computed_at timestamptz default now()
);
```

---
## 9. Ejecutar el notebook
1. Abre VSCode.
2. Abre la carpeta del proyecto (`File > Open Folder`).
3. Abre `demo_notebook.ipynb`.
4. Selecciona el kernel de Python del entorno virtual (`.venv`).
5. Ejecuta las celdas en orden:
   - Importaciones
   - Lectura de datos
   - Preprocesamiento
   - Clustering
   - Conexión a Supabase y carga de datos

Si todo está correcto, los registros se insertarán en las tablas creadas.

---
## 10. Visualización en Looker Studio (Resumen)
1. Ir a https://lookerstudio.google.com/
2. Crear un nuevo informe.
3. Elegir fuente de datos (puedes:
   - Exponer una API desde Supabase (via REST) y conectarla usando un conector JSON.
   - O exportar datos a CSV desde Supabase y subirlos manualmente.)
4. Crear gráficos: distribución por cluster, ingreso vs gasto, etc.

(Detalle extendido puede agregarse en futuras versiones.)

---
## 11. Buenas prácticas
- NO subir `credentials.json` ni claves a GitHub.
- Usar ramas para nuevas mejoras: `git checkout -b mejora-readme`.
- Hacer commits claros y pequeños.

---
## 12. Comandos Git básicos
```bash
# Ver estado
git status

# Agregar cambios
git add README.md demo_notebook.ipynb requirements.txt

# Commit
git commit -m "Inicial: entorno y notebook"

# Establecer rama principal (si aplica)
git branch -M main

# Subir cambios
git push origin main
```

---
## 13. Bonus: Crear `.gitignore`
Crea un archivo `.gitignore` con:
```
.venv/
__pycache__/
.credentials.json
credentials.json
.ipynb_checkpoints/
```

---
## 14. Flujo resumido (Checklist)
- [ ] Instalar VSCode
- [ ] Crear cuenta GitHub
- [ ] Clonar repo
- [ ] Crear `.venv`
- [ ] Instalar dependencias
- [ ] Crear proyecto Supabase
- [ ] Crear tablas vía SQL
- [ ] Añadir `credentials.json`
- [ ] Ejecutar notebook
- [ ] Validar inserciones en Supabase
- [ ] Conectar Looker Studio (opcional)

---
## 15. Siguientes pasos (Extensiones de la demo)
- Añadir autenticación y roles
- Automatizar carga con scripts (ETL)
- Crear vistas o materialized views para Looker Studio
- Añadir tests unitarios para el código de clustering

---
## Licencia
Uso académico interno. Ajustar según necesidades institucionales.

---
Si encuentras algún problema, documenta el paso donde ocurrió y compártelo para soporte.
