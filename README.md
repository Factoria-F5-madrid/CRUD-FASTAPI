# Cómo Crear una Aplicación CRUD con FastAPI

## Índice

1. [Introducción](#1-introducción)
2. [Instalación y Configuración](#2-instalación-y-configuración)
3. [Estructura de Archivos y Carpetas](#3-estructura-de-archivos-y-carpetas)
4. [Conceptos Fundamentales](#4-conceptos-fundamentales)
5. [Flujo de Trabajo en FastAPI](#5-flujo-de-trabajo-en-fastapi)
6. [Comandos Esenciales](#6-comandos-esenciales)
7. [Ejemplo Completo: Aplicación CRUD](#7-ejemplo-completo-aplicación-crud)
8. [Despliegue en Producción](#8-despliegue-en-producción)
9. [Recursos Adicionales](#9-recursos-adicionales)

---

## 1. Introducción

FastAPI es un moderno y rápido (de alto rendimiento) framework web para construir APIs con Python 3.6+ basado en estándares Python. En este README, aprenderemos cómo crear una aplicación CRUD (Create, Read, Update, Delete) utilizando FastAPI.

🚀 FastAPI permite crear APIs RESTful de manera rápida y eficiente, con validación automática, serialización y documentación interactiva.

💻 Desarrollado por Sebastián Ramírez, FastAPI se basa en Starlette para el manejo web y Pydantic para la validación de datos, lo que lo hace extremadamente rápido y fácil de usar.

🔧 FastAPI es ideal para microservicios, aplicaciones serverless, y APIs que requieren alto rendimiento y facilidad de desarrollo.

## 2. Estructura de Archivos y Carpetas

```plaintext
book_crud/
│
├── main.py                   
├── config
│   ├── __init__.py
│   └── config_variables.py
|
├── database
│   ├── __init__.py
│   └── database.py
|                   
├── models/
│   ├── __init__.py
│   └── libro_model.py
|
├── schemas
│   ├── __init__.py
│   └── libro_schema.py
|
├── routes
│   ├── __init__.py
│   └── schema_model.py         
│
├── controllers/
│   ├── __init__.py
│   └── libro_controller.py   
│
├── .env #opcional
│
└── db.sqlite3 #Este archivo se creará solo ;)
```

- `venv/`: Directorio del entorno virtual de Python.
- `__init__.py`: Archivo que convierte el directorio en un paquete Python.
- `main.py`: Contiene la instancia principal de la aplicación FastAPI y laconfiguración global.
- `config_variables.py`: Guarda las variables de entorno utilizando pydantic_settings.
- `database.py`: Maneja la configuración y conexión a la base de datos.
- `models.py`: Define los modelos de SQLAlchemy que representan las tablas de la base de datos.
 - `schemas.py`: Contiene los esquemas Pydantic para la validación de datos yserialización.
- `controllers.py`: Implementa la lógica de negocio y las operaciones CRUD.
- `routes.py`: Define las rutas y endpoints de la API.
- `requirements.txt`: Lista todas las dependencias del proyecto para una fácil instalación.
- `README.md`: Proporciona documentación e instrucciones para el proyecto (este archivo).

## 3. Instalación y Configuración

1. Crear la estructura de directorios:
   ```
   mkdir book_crud
   cd book_crud
   ```
### Instalar FastAPI y dependencias

2. Crear un entorno virtual:
   ```
   python -m venv venv
   source venv/bin/activate
   ```

3. Instalar FastAPI y dependencias:
   ```
   pip install fastapi[all] sqlalchemy
   ```

   Crea las carpetas que necesites y dentro los archivos que vayas a utilizar, la arquitectura de tu proyecto puede cambiar, sin embargo recuerda que queremos **escalabilidad**, por lo que necesitamos dividir la lógica de los distintos servicios, y la conexiones con otras partes de la aplicación.

2. Configurar las variables de entorno y usar el diectorio y archivo `config/config_variables.py`:
**Install pydantic_settings**
    ```
    pip install pydantic-settings
    ```
   Pydantic es una librería de Python que se utiliza para validar, convertir y estructurar datos usando tipado de Python. Pydantic se asegura de que los datos que entran y salen de tu aplicación tengan la forma, el tipo y el contenido correcto.
    
    ```python
    from pydantic_settings import BaseSettings

    class Settings(BaseSettings):
        DB_USER: str = "nombre-del-ddbb-user"
        DB_PASSWORD: str = "contraseña-ddbb"
        DB_HOST: str = "ddbb-host"
        DB_NAME: str = "nombre-ddbb"

    settings = Settings()
    ```

    **Pero** si quisieramos hacer nuestro entorno más seguro podriamos usar el paquete `python-dotenv` y un archivo `.env`:

    ```python
    pip install python-dotenv
    ```
   Luego en tu archivo `.env`:
   
    ```python
    DB_USER=nombre-del-ddbb-user
    DB_PASSWORD=contraseña-ddbb
    DB_HOST=ddbb-host
    DB_NAME=nombre-ddbb
    ```
    Entonces el archivo `config/config_variables.py` se veria de esta manera:
   ```python
      # config/config_variables.py
   
      import os
      from dotenv import load_dotenv
      
      # Cargar variables del .env
      load_dotenv()
      
      class Settings:
          DB_USER: str = os.getenv("DB_USER", "default_user")
          DB_PASSWORD: str = os.getenv("DB_PASSWORD", "default_password")
          DB_HOST: str = os.getenv("DB_HOST", "localhost")
          DB_NAME: str = os.getenv("DB_NAME", "test_db")
      
      settings = Settings()
   ```
   

4. Configurar la base de datos en `database/database.py`:
   ```python
   # database/database.py

    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker
    from config.config_variables import settings

    # Variables de entorno para no exponer información sensible
    DB_USER = settings.DB_USER
    DB_PASSWORD = settings.DB_PASSWORD
    DB_HOST = settings.DB_HOST
    DB_NAME = settings.DB_NAME

    # Conexión con la base de datos
    DATABASE_URL = "mysql+pymysql://"+DB_USER+":"+DB_PASSWORD+"@"+DB_HOST+"/"+DB_NAME+""  # MySQL

    # Crea un engine
    engine = create_engine(DATABASE_URL)

    # Crea una clase para configurar la sesión
    Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)

    # Crea una clase base para los modelos
    Base = declarative_base()

    # función para obtener la sesión de la base de datos
    def get_db():
        db = Session()  # Crea una nueva sesión
        try:
            yield db  # Usa la sesión
        finally:
            db.close()  # Cierra la sesión al terminar

   ```

## 4. Conceptos Fundamentales

- **Modelos**: Representan las tablas en la base de datos.
- **Schemas**: Definen la estructura de los datos para la serialización/deserialización.
- **CRUD**: Operaciones básicas (Create, Read, Update, Delete) para manipular datos.
- **Dependencias**: Funciones que FastAPI ejecuta antes de las funciones de ruta.
- **Pydantic**: Biblioteca para validación de datos y configuraciones.

## 5. Flujo de Trabajo en FastAPI

1. Definir modelos de base de datos
2. Crear schemas Pydantic
3. Implementar operaciones CRUD
4. Definir rutas de la API
5. Configurar la aplicación principal

## 6. Comandos Esenciales

- `fastapi run app.py`: Inicia el servidor de desarrollo
- `uvicorn app.main:app --reload`: Inicia el servidor de desarrollo
- `pytest`: Ejecuta las pruebas (si están configuradas)
- `alembic revision --autogenerate`: Genera una migración de base de datos (si se usa Alembic)
- `alembic upgrade head`: Aplica las migraciones pendientes

## 7. Ejemplo Completo: Aplicación CRUD

### Modelos (`models/crud_model.py`)

```python
from sqlalchemy import Column, Integer, String
from .database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
```

### Schemas (`schemas/crud_schema.py`)

```python
from pydantic import BaseModel

class ItemBase(BaseModel):
    title: str
    description: str = None

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int

    class Config:
        orm_mode = True
```

### Operaciones CRUD (`controllers/crud_controller.py`)

```python
from sqlalchemy.orm import Session
from schemas import crud_schema
from models.curd_model import Item

class CrudControllers:

    @staticmethod
    async def get_item(db: Session, item_id: int):
        return db.query(Item).filter(Item.id == item_id).first()

    @staticmethod
    async def create_item(db: Session, item: crud_schema.ItemCreate):
        db_item = Item(**item.dict())
        db.add(db_item)
        db.commit()
        db.refresh(db_item)
        return db_item

# Implementar otras operaciones CRUD aquí
```

### Rutas de la API (`routes/crud_routes.py`)
```python
from fastapi import APIRouter, Depends
from fastapi import status
from sqlalchemy.orm import Session
from typing import List

from controllers.crud_controller import CrudController
from schemas.attendance_schema import AttendanceSchema
from database.database import get_db

router = APIRouter()

@router.post("/items/", response_model=schemas.Item)
def new_item(item: schemas.ItemCreate, db: Session = Depends(get_db)):
    item = await CrudController.create_item(item, db)
    return item

@app.get("/items/{item_id}", response_model=schemas.Item)
def read_item(item_id: int, db: Session = Depends(get_db)):
    db_item = CrudController.get_item(db, item_id=item_id)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item

# Implementar otras rutas aquí
```

### Archivo principal app.py (`app.py`)
```python
from fastapi import FastAPI
from routes.crud_routes import router

app = FastAPI()

#IMPORTANT: Aquí referenciamos el archivo "database" no la variable "db"
from database import database
def run():
    pass
if __name__ == '__main__':
    database.Base.metadata.create_all(database.engine)
    run()

@app.get("/")
async def root():
    return {"message": "Hello World"}
app.include_router(router, prefix="/v1")
```


## 8. Despliegue en Producción

1. Elegir un proveedor de hosting (por ejemplo, Heroku, DigitalOcean, AWS)
2. Configurar variables de entorno para la base de datos y otras configuraciones sensibles
3. Usar Gunicorn como servidor WSGI para producción
4. Configurar un servidor proxy inverso como Nginx (opcional, pero recomendado)
5. Implementar HTTPS para seguridad

## 9. Recursos Adicionales

- [Documentación oficial de FastAPI](https://fastapi.tiangolo.com/)
- [Tutorial de SQLAlchemy](https://docs.sqlalchemy.org/en/14/orm/tutorial.html)
- [Guía de despliegue de FastAPI](https://fastapi.tiangolo.com/deployment/)
- [Curso en video de FastAPI](https://www.youtube.com/watch?v=7t2alSnE2-I)
