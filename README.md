# Music Language / Music Recommendation Backend
Realizado por: Marco Duarte, Elkin Benitez, Alejandro Caro, Johan Garcia.

---

# Resumen del Proyecto
La Music Recommendation API es una aplicación web completa y lista para producción diseñada para mejorar el descubrimiento y la reproducción de música. Al integrar las APIs de YouTube, Spotify y Genius, proporciona a los usuarios la capacidad de buscar y reproducir música, recuperar metadatos y acceder a letras. La plataforma incluye recomendaciones inteligentes basadas en las preferencias del usuario, un panel de administración para la gestión de usuarios y seguimiento del uso de la API, convirtiéndola en una solución robusta para entusiastas de la música y desarrolladores.

# Descripción de Módulos del Proyecto
El proyecto consta de los siguientes módulos funcionales:
- **Endpoints de la API**: Gestionan autenticación, búsqueda musical, recomendaciones y obtención de letras.
- **Integración con Base de Datos**: Administra cuentas de usuario, metadatos de canciones, playlists y recomendaciones.
- **Servicios de APIs Externas**: Integra las APIs de YouTube, Spotify y Genius para datos musicales y letras.
- **Motor de Recomendación**: Proporciona sugerencias musicales personalizadas basadas en preferencias e historial de escucha.
- **Panel de Administración**: Administra roles de usuario, estadísticas y seguimiento del uso de la API.

# Árbol de Directorios
```
.
├── app/
│   ├── api/
│   │   ├── v1/
│   │   │   ├── auth.py            # Autenticación y OAuth de Spotify
│   │   │   ├── music.py           # Búsqueda musical, streaming, letras
│   │   │   ├── recommendations.py # Endpoints de recomendación
│   │   │   └── admin.py           # Panel de administración para gestión de usuarios
│   │   ├── __init__.py
│   ├── core/
│   │   └── config.py              # Configuración de la aplicación
│   ├── models/
│   │   └── models.py              # Modelos de base de datos
│   ├── schemas/
│   │   └── schemas.py             # Esquemas Pydantic
│   ├── services/
│   │   ├── youtube_service.py
│   │   ├── spotify_service.py
│   │   ├── genius_service.py
│   │   └── recommendation_service.py
│   ├── database.py                # Conexión a la base de datos
│   └── main.py                    # Aplicación FastAPI
├── Dockerfile                     # Configuración Docker
├── docker-compose.yml             # Entorno de desarrollo
├── docker-compose.prod.yml        # Despliegue en producción
├── .dockerignore                  # Archivos a ignorar en build Docker
├── scripts/
│   ├── start.sh                   # Script para iniciar servicios
│   ├── stop.sh                    # Script para detener servicios
│   ├── logs.sh                    # Script para ver logs
│   ├── setup_api_keys.py          # Script interactivo para configurar claves API
├── requirements.txt               # Dependencias Python
├── pytest.ini                     # Configuración de pytest
├── Makefile                       # Comandos de setup y testing
└── README.md                      # Documentación del proyecto
```

# Inventario y Descripción de Archivos
- **app/api/v1/auth.py**: Maneja registro de usuarios, login e integración OAuth con Spotify.
- **app/api/v1/music.py**: Gestiona búsqueda musical, URLs de streaming y obtención de letras.
- **app/api/v1/recommendations.py**: Proporciona recomendaciones musicales personalizadas.
- **app/api/v1/admin.py**: Panel administrativo para gestión de usuarios y monitoreo de uso de la API.
- **app/core/config.py**: Contiene configuración de la aplicación y claves de API con validación.
- **app/models/models.py**: Define modelos de base de datos para usuarios, canciones, playlists y recomendaciones.
- **app/schemas/schemas.py**: Esquemas Pydantic para peticiones y respuestas de la API.
- **app/services/youtube_service.py**: Integra con la API de YouTube para búsqueda y streaming musical.
- **app/services/spotify_service.py**: Integra con la API de Spotify para metadatos y recomendaciones.
- **app/services/genius_service.py**: Conecta con la API de Genius para obtener letras.
- **app/services/recommendation_service.py**: Implementa la lógica para generar recomendaciones musicales.
- **app/database.py**: Gestiona conexiones y sesiones de base de datos.
- **Dockerfile**: Define la imagen Docker de la aplicación.
- **docker-compose.yml**: Configuración para entorno local de desarrollo.
- **docker-compose.prod.yml**: Configuración para despliegue en producción.
- **.dockerignore**: Especifica archivos y carpetas a ignorar en builds Docker.
- **scripts/start.sh**: Inicia servicios Docker y verifica salud.
- **scripts/stop.sh**: Detiene servicios Docker.
- **scripts/logs.sh**: Muestra logs de los servicios Docker.
- **scripts/setup_api_keys.py**: Script interactivo para configurar claves de APIs externas.
- **requirements.txt**: Lista de dependencias del proyecto.
- **pytest.ini**: Configuración para pruebas con pytest.
- **Makefile**: Proporciona comandos para setup, pruebas y operaciones Docker.
- **README.md**: Proporciona documentación del proyecto.

# Stack Tecnológico
- **FastAPI**: Framework web para construir APIs con Python.
- **Docker**: Plataforma de contenedores para despliegue.
- **PostgreSQL**: Base de datos relacional para almacenamiento.
- **Redis**: Almacenamiento en memoria para caché (opcional).
- **YouTube Data API**: Para búsqueda de videos musicales y streaming.
- **Spotify Web API**: Para metadatos musicales y recomendaciones.
- **Genius API**: Para obtener letras de canciones.
- **Pydantic**: Validación de datos y gestión de settings.
- **SQLAlchemy**: ORM para interacción con la base de datos.
- **Uvicorn**: Servidor ASGI para ejecutar FastAPI.
- **pytest**: Framework de pruebas.
- **Makefile**: Automatización de tareas de build y despliegue.

# Uso
1. Clonar el repositorio y navegar al directorio del proyecto:
   ```bash
   git clone music_filtering_backend
   cd music-recommendation-api
   ```
2. Configurar variables de entorno de forma interactiva:
   ```bash
   python scripts/setup_api_keys.py
   ```
3. Instalar dependencias:
   ```bash
   make install
   ```
4. Iniciar la aplicación:
   ```bash
   make docker-up
   ```

(Dependiendo de la implementación real, los nombres de comandos en el `Makefile` pueden variar.)

---
