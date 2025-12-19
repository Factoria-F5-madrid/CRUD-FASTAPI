# Cómo Crear una Aplicación CRUD con FastAPI

## Índice

1. [Introducción](#1-introducción)
2. [Estructura de Archivos y Carpetas](#2-Estructura-de-Archivos-y-Carpetas)
3. [Instalación y Configuración](#3-Instalación-y-Configuración)
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
│   └── routes.py         
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
si no te permite utilizar `[all]` entonces instala:

```
pip install fastapi sqlalchemy uvicorn pymysql
```

Crea las carpetas que necesites y dentro los archivos que vayasa utilizar, la arquitectura de tu proyecto puede cambiar, sinembargo recuerda que queremos **escalabilidad**, por lo quenecesitamos dividir la lógica de los distintos servicios, y laconexiones con otras partes de la aplicación, es decir crea lascarpetas y archivos (los archivos son los que tienen extensionescomo ".py", las carpetas no tienen extensión):

```plaintext
book_crud/
│
├── main.py                   
├── config
│  ├── __init__.py
│  └── config_variables.py
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
│   └── routes.py         
│
├── controllers/
│   ├── __init__.py
│   └── libro_controller.py   
│ 
└──.env #opcional
```

2. Configurar las variables de entorno y usar el diectorio y archivo `config/config_variables.py`:

   Para eso primero debemos instalar la dependencia pertinente, en este caso **pydantic_settings**, librería que nos da una serie de herramientas para permitir que nuestro sistema pueda acceder a información pertinente durante el desarrollo.
   
   Pydantic es una librería de Python que se utiliza para validar, convertir y estructurar datos usando tipado de Python. Pydantic se asegura de que los datos que entran y salen de tu aplicación tengan la forma, el tipo y el contenido correcto.
   
   ```
   pip install pydantic-settings
   ```

   Nos dirigimos al directorio `config/` y dentro de esta carpeto nos dirigimos al archivo `config_variables.py` y escribimos:
    
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

   Ahora para guardar estas dependencias con sus versiones en un archivo `requirements.txt` hacemos:

   ```bash
   pip freeze >> requirements.txt
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
        DB_PASSWORD: str = os.getenv("DB_PASSWORD","default_password")
        DB_HOST: str = os.getenv("DB_HOST", "localhost")
        DB_NAME: str = os.getenv("DB_NAME", "test_db")
      
    settings = Settings()
   ```
   

4. Configurar la base de datos en `database/database.py` y en `MySQL Workbench`:
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
   
   # Esta función crea una sesión para trabajar con la base de datos, la devuelve mientras haces algo (yield db) y la cierra automáticamente al terminar.
   # Se usa mucho en frameworks como FastAPI para que cada petición tenga su propia sesión limpia.
   ```
   
   ### Creamos nuestra conexión en Workbech

   <img width="1028" height="451" alt="image" src="https://github.com/user-attachments/assets/3b3dabab-5b60-4021-bd21-df190734bd12" />
   <img width="1150" height="610" alt="image" src="https://github.com/user-attachments/assets/0e28d86d-f226-4ac7-9459-6461daf7a07a" />

   Ahora entramos a la conexión y creamos una base de datos
   
   <img width="808" height="338" alt="image" src="https://github.com/user-attachments/assets/46230438-0d49-4181-aeb8-f97e230d7a4f" />
   
   Nos aseguramos de estar en `schemas` y no en `Administration`
   
   <img width="347" height="600" alt="image" src="https://github.com/user-attachments/assets/d251d4fd-2b12-4f4a-ad36-86760de9da76" />
   
   Creamos la base de datos
   
   <img width="887" height="580" alt="image" src="https://github.com/user-attachments/assets/f648e24a-6747-4baf-9d7b-37ea987870a9" />

   Le ponemos como nombre `book_crud` y hacemos click en el botón `apply` que está al final de esa vista:
   
   <img width="735" height="461" alt="image" src="https://github.com/user-attachments/assets/aea823d8-a415-4388-9526-e8f29893a35e" />
   <img width="671" height="419" alt="image" src="https://github.com/user-attachments/assets/55e2d801-e486-48e0-ba29-db83decc94c9" />

   Si no ves tu base de datos, o que haya habido alg[un cambio, refresca haciendo click en estas flechas:
   
   <img width="360" height="495" alt="image" src="https://github.com/user-attachments/assets/80aa8a20-ca1f-4a91-b141-71ac7931aabf" />

   Hacemos click derecho sobre la opción `tables` (la cual está dentro de book_crud en el menú lateral), y hacemos click en `create table` para así crear una nueva tabla:
   
   <img width="668" height="612" alt="image" src="https://github.com/user-attachments/assets/a3a158a1-9114-40f7-9ace-d1b1eae6463e" />

   A la tabla le colocaremos como nombre `libros` y sus atributos serán `id`, `title`, `description` y volvemos a hacer click en `apply`:
   
   <img width="1596" height="273" alt="image" src="https://github.com/user-attachments/assets/6888c96c-1006-473b-a1c6-2cdd48a41af8" />

   ### Comprobamos que estamos correctamente conectados a la base de datos

   Para comprobar que estamos conectados a la base de datos necesitamos que nuestro archivo `main.py` tenga definido el arranque de la aplicación:

   ```python
   from fastapi import FastAPI

   app = FastAPI()
   
   #IMPORTANTE: Aquí referenciamos el archivo "database" no la variable "db"
   from database import database
   
   def run():
       pass
   if __name__ == '__main__':
       database.Base.metadata.create_all(database.engine)
       run()
   
   @app.get("/")
   async def root():
       return {"message": "Hello World"}
   ```

   Y luego en nuestra terminal ejecutamos:

   ```bash
   uvicorn main:app --reload
   ```

## 4. Conceptos Fundamentales

- **create_engine**: Piensa en esto como construir la tubería que conecta tu código con la base de datos.

- **declarative_base**: Es una plantilla para crear tus tablas como si fueran clases de Python.

- **sessionmaker**: Es como un generador de “mesas de trabajo” para interactuar con la base de datos sin tocarla directamente. Con la sesión puedes consultar, insertar, actualizar o borrar datos sin afectar inmediatamente la base de datos real hasta que confirmes los cambios.
  - **autocommit=False** → No confirma automáticamente los cambios, tú decides cuándo guardar.
  - **autoflush=False** → No envía automáticamente los cambios hasta que decidas.
  - **bind=engine** → Le decimos a la sesión que use esa tubería que creamos antes para conectarse.

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

- `fastapi run main.py`: Inicia el servidor de desarrollo
- `uvicorn main:app --reload`: Inicia el servidor de desarrollo
- `pytest`: Ejecuta las pruebas (si están configuradas)
- `alembic revision --autogenerate`: Genera una migración de base de datos (si se usa Alembic)
- `alembic upgrade head`: Aplica las migraciones pendientes

## 7. Ejemplo Completo: Aplicación CRUD

### Modelos (`models/libro_model.py`)

```python
from sqlalchemy import Column, Integer, String
from database.database import Base

class Libro(Base):
    __tablename__ = "libros"

    id = Column(Integer, primary_key=True)
    title = Column(String, index=True, nullable=False)
    description = Column(String, index=True)
```

### Schemas (`schemas/libro_schema.py`)

```python
from pydantic import BaseModel

# Hereda todo de ItemBase.
# Se usa cuando el usuario envía datos para crear un item.
# “pass” significa que no añadimos nada nuevo, solo usamos lo que está en ItemBase.

class LibroBase(BaseModel):
    title: str
    description: str = None

class LibroCreate(LibroBase):
    pass

class Libro(LibroBase): # Hereda de ItemBase (title y description) y añade el id que ya existe en la base de datos.
    id: int

    class Config:
        orm_mode = True # permite que Pydantic lea directamente objetos de SQLAlchemy como si fueran diccionarios, para enviarlos en respuestas JSON.
```

### Operaciones CRUD (`controllers/libro_controller.py`)

```python
from sqlalchemy.orm import Session
from schemas import libro_schema
from models.libro_model import Libro

class CrudControllers:

    '''
    Si no usaramos @static methos tendriamos que escribir:

    def __init__(self, db: Session): # El trabajador ya tiene su caja de herramientas (self.db) y no necesita traer nada externo cada vez.
            self.db = db

    y el controlador se vería así:

    def create_item(self, item: crud_schema.ItemCreate): # No hace falta pasar db:session cada vez
            db_item = Item(**item.dict())
            self.db.add(db_item)
            self.db.commit()
            self.db.refresh(db_item)
            return db_item
    '''

    @staticmethod
    def get_Libros(db: Session):
        return db.query(Libro).all()

    @staticmethod # Cada función es como un trabajador independiente que llega con su propio conjunto de herramientas (db) y hace su tarea.
    def get_Libro_by_id(db: Session, Libro_id: int):
        return db.query(Libro).filter(Libro.id == Libro_id).first()

    @staticmethod
    def create_Libro(db: Session, libro: libro_schema.LibroCreate):
        db_libro = Libro(**libro.dict())  # libro.dict() funciona porque es un Pydantic model
        db.add(db_libro)
        db.commit()
        db.refresh(db_libro)
        return db_libro

    @staticmethod
    def update_Libro(db: Session, Libro_id: int, libro: libro_schema.LibroCreate):
        db_Libro = db.query(Libro).filter(Libro.id == Libro_id).first()
        if db_Libro:
            db_Libro.title = libro.title 
            db_Libro.description = libro.description
            db.commit()
            db.refresh(db_Libro)
        return db_Libro
   
    @staticmethod
    def delete_Libro(db: Session, Libro_id: int):
        db_Libro = db.query(Libro).filter(Libro.id == Libro_id).first()
        if db_Libro:
            db.delete(db_Libro)
            db.commit()
        return {"message": "Libro eliminado exitosamente"}
```

**También** nuestros controladores pueden ser funciones independientes, no tienen por qué estar dentro de una clase:

```python
from sqlalchemy.orm import Session
from schemas import libro_schema
from models.libro_model import Libro

def get_Libros(db: Session):
    return db.query(Libro).all()

def get_Libro_by_id(db: Session,Libro_id: int):
    return db.query(Libro).filter(Libro.id == Libro_id).first()


def create_Libro(db: Session, libro:libro_schema.LibroCreate):
    db_libro = Libro(**libro.dict()) # libro.dict() funciona porque esun Pydantic model
    db.add(db_libro)
    db.commit()
    db.refresh(db_libro)
    return db_libro


def update_Libro(db: Session,Libro_id: int, libro: libro_schemaLibroCreate):
    db_Libro = db.query(Libro).filte(Libro.id == Libro_id).first()
    if db_Libro:
        db_Libro.title = libro.title 
        db_Libro.description = librodescription
        db.commit()
        db.refresh(db_Libro)
    return db_Libro
   

def delete_Libro(db: Session,Libro_id: int):
    db_Libro = db.query(Libro).filte(Libro.id == Libro_id).first()
    if db_Libro:
        db.delete(db_Libro)
        db.commit()
    return {"message": "Libroeliminado exitosamente"}
```

### Rutas de la API (`routes/routes.py`)

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

# Obtener lista de items
@router.get("/items/", response_model=List[crud_schema.Item])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    items = CrudControllers.get_items(db, skip=skip, limit=limit)
    return items

# Actualizar un item
@router.put("/items/{item_id}", response_model=crud_schema.Item)
def update_item(item_id: int, item: crud_schema.ItemCreate, db: Session = Depends(get_db)):
    db_item = CrudControllers.update_item(db, item_id, item)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item

# Borrar un item
@router.delete("/items/{item_id}", response_model=crud_schema.Item)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    db_item = CrudControllers.delete_item(db, item_id)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item
```

Si definimos los controladores como funciones en lugar de una clase, entonces nuestras rutas se van a ver de esta manera:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List

from controllers.libro_controller import CrudControllers
from schemas.libro_schema import LibroBase, LibroCreate, Libro
from database.database import get_db

router = APIRouter()

@router.post("/libros/", response_model=LibroBase, status_code=status.HTTP_201_CREATED)
async def new_libro(libro: LibroCreate, db: Session = Depends(get_db)):
    libro = CrudControllers.create_Libro(db,libro)
    return libro

@router.get("/libros/{libro_id}", response_model=LibroBase)
def get_libro(libro_id: int, db: Session = Depends(get_db)):
    libro = CrudControllers.get_Libro_by_id(db, libro_id)
    if not libro:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
    return libro

@router.put("/libros/{libro_id}", response_model=LibroBase)
def update_Libro(libro_id: int, libro: LibroCreate, db: Session = Depends(get_db)):
    updated = CrudControllers.update_Libro(db, libro_id, libro)
    if not updated:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
    return updated

@router.delete("/libros/{libro_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_libro(libro_id: int, db: Session = Depends(get_db)):
    deleted = CrudControllers.delete_Libro(db, libro_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
```

### Archivo principal main.py (`main.py`)
```python
from fastapi import FastAPI
from routes.routes import router

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
