# **ZADANIE 1: Zdockeruj aplikację FastAPI (API z bazą i predykcją)**

## **Cel zadania**

Twoim zadaniem jest:

1. uruchomić poniższe API FastAPI (z predict + endpointami DB),
2. stworzyć **Dockerfile**,
3. zbudować obraz Dockera,
4. uruchomić go lokalnie,
5. oraz wypchnąć obraz do rejestru (DockerHub lub GitHub Container Registry — GHCR).

---

# **Kod aplikacji (FASTAPI + DB + ML Prediction)**

Plik `main.py`:

```python
from fastapi import FastAPI
from db.models import PricePrediction
import asyncio
from utils import predict_price
from sqlalchemy.orm import Session
from db import models
from db.database import SessionLocal
from functools import lru_cache
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

app = FastAPI(title="Housing API")

# Root
@app.get("/")
def root():
    return {"message": "API do mieszkań we Łodzi"}

@app.post("/predict")
async def predict(data: PricePrediction):
    predicted_price = await asyncio.get_event_loop().run_in_executor(
        None, predict_price, data.area_m2, data.rooms, data.floor, data.year_built, data.longitude, data.latitude, data.locality
    )
    return {"predicted_price": predicted_price}

@app.get("/offers/")
@lru_cache(maxsize=32)
def read_offers(db: Session = Depends(get_db)):
    return db.query(models.OfferDB).all()
```


---

# **Struktura projektu (wymagana)**

Upewnij się, że repo wygląda tak:

```
project/
  main.py
  db/
    __init__.py
    models.py
    database.py
  utils.py
  requirements.txt
```

> Jeśli uczestnik nie ma `requirements.txt`, musi go utworzyć.

---

# **Wymagany plik — `requirements.txt`**

Przykład minimalny:

```
fastapi
uvicorn[standard]
sqlalchemy
pydantic
```
I dla ML: 
```
joblib
scikit-learn
pandas
numpy
```

---

#  **Część 1: Stwórz Dockerfile**

Uczestnik musi utworzyć plik **Dockerfile**:

```Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

# 🛠 **Część 2: Zbuduj obraz Docker**

W katalogu projektu:

```bash
docker build -t housing-api:latest .
```

---

# ▶**Część 3: Uruchom obraz lokalnie**

```bash
docker run -p 8000:8000 housing-api:latest
```

Wejdź w przeglądarce:

```
http://localhost:8000
```

API powinno zwrócić:

```json
{ "message": "API do mieszkań we Łodzi" }
```

---

# **Część 4: Wypchnij obraz do rejestru**


## OPCJA 1: GitHub Container Registry (GHCR)

1. Zaloguj się:

```bash
echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin
```

2. Tagowanie:

```bash
docker tag housing-api:latest ghcr.io/USERNAME/housing-api:latest
```

3. Push:

```bash
docker push ghcr.io/USERNAME/housing-api:latest
```

---
---

---
