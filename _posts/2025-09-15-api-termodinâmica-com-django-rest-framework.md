---
layout: post
title:  "API de Propriedades Termodinâmicas com Django REST Framework (Arquitetura Hexagonal)"
date:   2025-09-15 00:00:00 -0300
categories: posts python engenharia química
---
Este tutorial mostra passo a passo como criar uma API para cálculo de propriedades termodinâmicas da água usando **CoolProp** e **Django REST Framework** organizada em **arquitetura hexagonal**.


### **Instalação das dependências**

```bash
pip install django djangorestframework CoolProp
```


## Estrutura do projeto

```text
coolprop_api/
 ├── manage.py
 ├── coolprop_api/                # projeto django
 │    ├── settings.py
 │    ├── urls.py
 │    └── ...
 └── api/                         # app principal
      ├── __init__.py
      ├── apps.py
      ├── urls.py                 # infraestrutura (rotas)
      ├── views.py                # infraestrutura (views)
      ├── application/
      │     └── services.py       # camada de aplicação
      └── domain/
            └── properties.py     # camada de domínio
```


## Criar projeto e app

```bash
django-admin startproject coolprop_api
cd coolprop_api
python manage.py startapp api
```

No `coolprop_api/settings.py` adicione ao `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'api',
]
```

Crie as pastas `application` e `domain` dentro de `api/`.


## Camada de Domínio — `api/domain/properties.py`

```python
from CoolProp.CoolProp import PropsSI

class WaterProperties:
    """Regras de negócio para propriedades da água."""

    @staticmethod
    def pressao_saturacao(temperatura: float, qualidade: float) -> float:
        return PropsSI('P', 'T', temperatura, 'Q', qualidade, 'Water')

    @staticmethod
    def entalpia(temperatura: float, pressao: float) -> float:
        return PropsSI('H', 'T', temperatura, 'P', pressao, 'Water')

    @staticmethod
    def entropia(temperatura: float, pressao: float) -> float:
        return PropsSI('S', 'T', temperatura, 'P', pressao, 'Water')
```


## Camada de Aplicação — `api/application/services.py`

```python
from api.domain.properties import WaterProperties

class PropertyService:
    """Camada de aplicação: usa as regras do domínio."""

    @staticmethod
    def get_pressao_saturacao(temperatura: float, qualidade: float):
        pressao = WaterProperties.pressao_saturacao(temperatura, qualidade)
        return {
            "temperatura_K": temperatura,
            "qualidade": qualidade,
            "pressao_Pa": pressao
        }

    @staticmethod
    def get_entalpia(temperatura: float, pressao: float):
        entalpia = WaterProperties.entalpia(temperatura, pressao)
        return {
            "temperatura_K": temperatura,
            "pressao_Pa": pressao,
            "entalpia_Jkg": entalpia
        }

    @staticmethod
    def get_entropia(temperatura: float, pressao: float):
        entropia = WaterProperties.entropia(temperatura, pressao)
        return {
            "temperatura_K": temperatura,
            "pressao_Pa": pressao,
            "entropia_JkgK": entropia
        }
```


## Camada de Infraestrutura — `api/views.py`

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from api.application.services import PropertyService

class PressaoSaturacaoView(APIView):
    def get(self, request):
        temperatura = float(request.query_params.get('temperatura', 373.15))
        qualidade = float(request.query_params.get('qualidade', 0))
        data = PropertyService.get_pressao_saturacao(temperatura, qualidade)
        return Response(data)

class EntalpiaView(APIView):
    def get(self, request):
        temperatura = float(request.query_params.get('temperatura', 373.15))
        pressao = float(request.query_params.get('pressao', 101325))
        data = PropertyService.get_entalpia(temperatura, pressao)
        return Response(data)

class EntropiaView(APIView):
    def get(self, request):
        temperatura = float(request.query_params.get('temperatura', 373.15))
        pressao = float(request.query_params.get('pressao', 101325))
        data = PropertyService.get_entropia(temperatura, pressao)
        return Response(data)
```


## Rotas — `api/urls.py`

```python
from django.urls import path
from api.views import PressaoSaturacaoView, EntalpiaView, EntropiaView

urlpatterns = [
    path('pressao_saturacao/', PressaoSaturacaoView.as_view(), name='pressao_saturacao'),
    path('entalpia/', EntalpiaView.as_view(), name='entalpia'),
    path('entropia/', EntropiaView.as_view(), name='entropia'),
]
```

E no `coolprop_api/urls.py` inclua:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),
]
```


## Rodando

```bash
python manage.py migrate  # se ainda não fez migrations
python manage.py runserver
```


## Testando Endpoints

* **Pressão de saturação**
  `http://127.0.0.1:8000/api/pressao_saturacao/?temperatura=373.15&qualidade=0`

* **Entalpia**
  `http://127.0.0.1:8000/api/entalpia/?temperatura=373.15&pressao=101325`

* **Entropia**
  `http://127.0.0.1:8000/api/entropia/?temperatura=373.15&pressao=101325`


## Benefícios da Arquitetura Hexagonal

* **Domínio** isolado (regras de negócio podem ser reaproveitadas).
* **Aplicação** apenas orquestra os casos de uso.
* **Infraestrutura** (Django REST Framework) só expõe endpoints HTTP.
