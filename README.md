
# Deploy de Modelo de Machine Learning com Flask e Docker

## Visão geral

Neste roteiro prático, vamos transformar um modelo de Machine Learning treinado em Python em um serviço acessível por HTTP.

A ideia é sair de um modelo treinado em um notebook (por exemplo, no Google Colab) e criar uma pequena API capaz de receber dados e devolver previsões. Em seguida, essa aplicação será empacotada em um container Docker para execução de forma isolada e reproduzível.

Esse processo representa um passo importante na transição entre experimentação em notebooks e aplicações reais baseadas em serviços.

---

# Objetivos da atividade

Ao final desta prática, você será capaz de:

- salvar um modelo treinado em arquivo `.pkl`
- criar uma API simples em Python utilizando Flask
- carregar um modelo salvo dentro da API
- enviar dados para o modelo via requisição HTTP
- receber previsões em formato JSON
- empacotar a aplicação em um container Docker
- testar a API utilizando Postman

---

# Pré-requisitos

Antes de iniciar, verifique se você possui:

- Python instalado
- Docker Desktop instalado e em execução
- um editor de código (VS Code, por exemplo)
- Postman instalado
- um modelo de Machine Learning já treinado

---

# Estrutura do projeto

Crie uma pasta para o projeto com a seguinte estrutura:

```
deploy_ml/
├── modelo.pkl
├── inference.py
├── requirements.txt
└── Dockerfile
```

---

# Etapa 1 — Salvar o modelo treinado

Após treinar um modelo de Machine Learning em Python, é necessário salvá-lo em um arquivo para que ele possa ser reutilizado posteriormente.

Um formato comum para isso é o **pickle** (`.pkl`).

Exemplo:

```python
import pickle

with open("modelo.pkl", "wb") as f:
    pickle.dump(modelo, f)
```

Isso permite reutilizar o modelo posteriormente sem precisar treiná-lo novamente.

---

# Etapa 2 — Criar o arquivo `inference.py`

1. Abra o VS Code ou outro editor de código.
2. Dentro da pasta do projeto, crie um novo arquivo chamado:

```
inference.py
```

3. Cole o código abaixo:

```python
from flask import Flask, request, jsonify
import pickle
import pandas as pd

with open("modelo.pkl", "rb") as f:
    modelo = pickle.load(f)

app = Flask(__name__)

@app.route("/predict", methods=["POST"])
def predict():
    dados = request.json
    df = pd.DataFrame(dados)
    pred = modelo.predict(df)

    return jsonify({
        "predicao": pred.tolist()
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

---

# Etapa 3 — Criar o arquivo `requirements.txt`

Crie um arquivo chamado:

```
requirements.txt
```

Conteúdo:

```
flask
pandas
numpy
scikit-learn
```

---

# Etapa 4 — Executar a aplicação localmente

No terminal, dentro da pasta do projeto:

```
pip install -r requirements.txt
```

Depois execute:

```
python inference.py
```

Se tudo estiver correto, aparecerá algo semelhante a:

```
Running on http://0.0.0.0:8000
```

---

# Etapa 5 — Testar a API no Postman

## Criar a requisição

1. Abra o **Postman**
2. Clique em **New**
3. Clique em **HTTP Request**

## Configurar a requisição

Método:

```
POST
```

URL:

```
http://localhost:8000/predict
```

## Configurar o Body

1. Clique na aba **Body**
2. Selecione **raw**
3. No seletor à direita escolha **JSON**

Cole:

```json
[
  {
    "x": 1.2,
    "y": 3.4,
    "z": 5.6
  }
]
```

Clique em **Send**.

Resposta esperada:

```json
{
  "predicao": [0]
}
```

---

# Etapa 6 — Criar o Dockerfile

Crie um arquivo chamado:

```
Dockerfile
```

Conteúdo:

```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "inference.py"]
```

---

# Etapa 7 — Construir a imagem Docker

No terminal:

```
docker build -t modelo .
```

---

# Etapa 8 — Executar o container

```
docker run -p 8000:8000 modelo
```

---

# Etapa 9 — Testar novamente no Postman

POST:

```
http://localhost:8000/predict
```

Body:

```json
[
  {
    "x": 1.2,
    "y": 3.4,
    "z": 5.6
  }
]
```

---

# Fluxo completo

```
Treinar modelo
      ↓
Salvar modelo (.pkl)
      ↓
Criar API Flask
      ↓
Testar localmente
      ↓
Criar imagem Docker
      ↓
Executar container
      ↓
Consumir API via HTTP
```

---

# Referências

Flask Documentation  
https://flask.palletsprojects.com/

Scikit-learn Model Persistence  
https://scikit-learn.org/stable/model_persistence.html

Docker Documentation  
https://docs.docker.com/

Postman Learning Center  
https://learning.postman.com/

Python Pickle Documentation  
https://docs.python.org/3/library/pickle.html
