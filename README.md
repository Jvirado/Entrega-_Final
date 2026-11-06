
LIFEMAP: REPUTACIÓN, UBICACIÓN Y SUPERVIVENCIA DE NEGOCIOS - ENTREGA 1


Curso: Big Data · Institución Universitaria de Envigado
Área Metropolitana: Philadelphia, PA
Categoría: Restaurants


--------------------------------------------------------------------
1. ARQUITECTURA
--------------------------------------------------------------------
El sistema está completamente dockerizado y se orquesta mediante docker-compose. Consta de:
- MongoDB: Base de datos para negocios (con índice 2dsphere), reseñas, ventanas y resultados.
- Spark: Un Master y un Worker para procesamiento batch (modelo y MapReduce).
- Dask: Scheduler y Worker para la ingesta y limpieza particionada de datos.
- API Flask: Endpoints para consultas geoespaciales, predicción de sentimiento y healthcheck.
- Jenkins: Servidor de CI/CD para pruebas automatizadas y despliegue.

--------------------------------------------------------------------
2. CÓMO DESPLEGAR DESDE CERO (REPRODUCIBILIDAD)
--------------------------------------------------------------------

Requisitos previos:
- Docker y Docker Compose instalados.
- Credenciales de Kaggle (kaggle.json).

Paso 1: Clonar el repositorio
git clone https://github.com/tu-usuario/lifemap-entrega1.git
cd lifemap-entrega1

Paso 2: Configurar credenciales (Fuera del repositorio)
El proyecto necesita descargar el dataset de Kaggle. Crea una carpeta de secretos y guarda tu token:
mkdir -p ~/.kaggle
cp /ruta/a/tu/kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json

Paso 3: Levantar los servicios
docker compose up -d --build

Paso 4: Verificar el estado
Accede a http://localhost:5000/health. Deberías ver un JSON con el estado de MongoDB, Spark y el hash del commit actual.

--------------------------------------------------------------------
4. ENDPOINTS DE LA API
--------------------------------------------------------------------

- GET /health
  Descripción: Estado de los servicios y hash del commit.
  Parámetros: Ninguno.

- GET /businesses/near
  Descripción: Negocios cerca de una coordenada usando $near.
  Parámetros: lat, lon, radius, category.

- POST /businesses/within
  Descripción: Negocios dentro de un polígono usando $geoWithin.
  Parámetros: geojson (en el body).

- GET /zones/summary
  Descripción: Agregación con $geoNear (sentimiento medio y tasa de cierre).
  Parámetros: lat, lon, radius.

- GET /stats/closure
  Descripción: Tasas de cierre por categoría o celda de geohash.
  Parámetros: group_by, min_days.

- POST /predict
  Descripción: Predice el sentimiento de un texto nuevo.
  Parámetros: text (en el body).

--------------------------------------------------------------------
5. INTEGRACIÓN Y DESPLIEGUE CONTINUO (JENKINS)
--------------------------------------------------------------------
1. Accede a Jenkins en http://localhost:8081.
2. Configura la credencial "kaggle-token" (tipo Secret file) con tu kaggle.json.
3. Crea un nuevo Pipeline apuntando a tu repositorio de GitHub.
4. Cualquier merge a la rama main disparará el Jenkinsfile (o el pollSCM configurado), el cual construirá las imágenes, correrá las pruebas y desplegará. Si una prueba falla, el despliegue se cancela.

