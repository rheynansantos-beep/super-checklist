# Guia de Desenvolvimento - Super Checklist

## 📋 Índice

1. [Setup do Projeto](#setup-do-projeto)
2. [Estrutura do Projeto](#estrutura-do-projeto)
3. [Criar um Modelo](#criar-um-modelo)
4. [Criar um Blueprint/Rota](#criar-um-blueprint-rota)
5. [Padrões de Código](#padrões-de-código)
6. [Migrações de Banco de Dados](#migrações-de-banco-de-dados)
7. [Testes](#testes)
8. [Deployment](#deployment)

---

## Setup do Projeto

### 1. Clonar o Repositório

```bash
git clone https://github.com/rheynansantos-beep/super-checklist.git
cd super-checklist
```

### 2. Criar Ambiente Virtual

```bash
# Linux/Mac
python3 -m venv venv
source venv/bin/activate

# Windows
python -m venv venv
venv\Scripts\activate
```

### 3. Instalar Dependências

```bash
pip install -r requirements.txt
```

### 4. Configurar Variáveis de Ambiente

```bash
cp .env.example .env

# Editar .env com suas credenciais
# - DB_HOST, DB_USER, DB_PASSWORD
# - ASKSUITE_API_KEY, ASKSUITE_API_URL
# - SECRET_KEY, JWT_SECRET_KEY
```

### 5. Criar Banco de Dados

```bash
# No MySQL
mysql -u root -p
CREATE DATABASE super_checklist CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;
```

### 6. Executar Migrações

```bash
flask db upgrade
```

### 7. Iniciar Servidor

```bash
python run.py
```

Acessar em: `http://localhost:5000`

---

## Estrutura do Projeto

```
super-checklist/
├── app/
│   ├── __init__.py              # Factory da aplicação
│   ├── config.py                # Configurações
│   │
│   ├── models/                  # Modelos SQLAlchemy
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── checklist.py
│   │   ├── round.py
│   │   ├── cadex_cleaning.py
│   │   ├── apartment_designation.py
│   │   ├── asksuite_tracking.py
│   │   ├── handover.py
│   │   ├── occurrence.py
│   │   ├── cash_control.py
│   │   └── audit.py
│   │
│   ├── routes/                  # Blueprints/Rotas
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── checklist.py
│   │   ├── round.py
│   │   ├── cadex_cleaning.py
│   │   ├── apartment_designation.py
│   │   ├── asksuite_tracking.py
│   │   ├── handover.py
│   │   ├── occurrence.py
│   │   ├── cash_control.py
│   │   ├── dashboard.py
│   │   ├── export.py
│   │   └── audit.py
│   │
│   ├── services/                # Lógica de negócio
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── checklist_service.py
│   │   ├── round_service.py
│   │   ├── export_service.py
│   │   └── asksuite_service.py
│   │
│   ├── schemas/                 # Validação com Marshmallow
│   │   ├── __init__.py
│   │   ├── user_schema.py
│   │   ├── checklist_schema.py
│   │   └── ...
│   │
│   ├── utils/                   # Utilitários
│   │   ├── __init__.py
│   │   ├── decorators.py        # @auth_required, etc
│   │   ├── errors.py            # Classes de erro
│   │   ├── helpers.py           # Funções auxiliares
│   │   └── asksuite_api.py      # Cliente API AskSuite
│   │
│   └── middleware/              # Middlewares
│       ├── __init__.py
│       └── auth_middleware.py
│
├── migrations/                  # Alembic migrations
│   └── versions/
│
├── tests/                       # Testes
│   ├── __init__.py
│   ├── test_auth.py
│   ├── test_checklist.py
│   └── ...
│
├── docs/                        # Documentação
│   ├── ESCOPO.md
│   ├── MODELOS_DADOS.md
│   ├── API.md
│   └── INTEGRACAO_ASKSUITE.md
│
├── .env.example                 # Exemplo de variáveis
├── .gitignore
├── requirements.txt
├── run.py                       # Ponto de entrada
└── README.md
```

---

## Criar um Modelo

### Exemplo: Novo Modelo `Notificacao`

#### 1. Criar arquivo `app/models/notification.py`

```python
from app import db
from datetime import datetime

class Notification(db.Model):
    """Modelo de Notificação"""
    __tablename__ = 'notificacoes'
    
    id = db.Column(db.Integer, primary_key=True)
    usuario_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    titulo = db.Column(db.String(255), nullable=False)
    mensagem = db.Column(db.Text)
    tipo = db.Column(db.Enum('INFO', 'AVISO', 'ERRO'), default='INFO')
    lida = db.Column(db.Boolean, default=False)
    data_criacao = db.Column(db.DateTime, default=datetime.utcnow)
    data_leitura = db.Column(db.DateTime)
    
    # Relacionamento
    usuario = db.relationship('User', backref='notificacoes')
    
    def __repr__(self):
        return f'<Notification {self.id}: {self.titulo}>'
    
    def to_dict(self):
        return {
            'id': self.id,
            'titulo': self.titulo,
            'mensagem': self.mensagem,
            'tipo': self.tipo,
            'lida': self.lida,
            'data_criacao': self.data_criacao.isoformat()
        }
```

#### 2. Adicionar a `app/models/__init__.py`

```python
from .notification import Notification

__all__ = [
    # ... outros modelos
    'Notification'
]
```

#### 3. Criar Migração

```bash
flask db migrate -m "Add Notification model"
flask db upgrade
```

---

## Criar um Blueprint/Rota

### Exemplo: Routes para Notificações

#### 1. Criar arquivo `app/routes/notification.py`

```python
from flask import Blueprint, request, jsonify
from flask_jwt_extended import jwt_required, get_jwt_identity
from app import db
from app.models import Notification, User
from app.utils.decorators import auth_required
from app.utils.errors import NotFoundError
from datetime import datetime

notification_bp = Blueprint('notification', __name__, url_prefix='/api/notification')

@notification_bp.route('', methods=['GET'])
@jwt_required()
def listar_notificacoes():
    """Listar notificações do usuário"""
    usuario_id = get_jwt_identity()
    
    # Filtros
    apenas_nao_lidas = request.args.get('nao_lidas', False, type=bool)
    limite = request.args.get('limite', 20, type=int)
    
    query = Notification.query.filter_by(usuario_id=usuario_id)
    
    if apenas_nao_lidas:
        query = query.filter_by(lida=False)
    
    notificacoes = query.order_by(Notification.data_criacao.desc()).limit(limite).all()
    
    return jsonify({
        'notificacoes': [n.to_dict() for n in notificacoes],
        'total': len(notificacoes)
    }), 200

@notification_bp.route('/<int:id>/lido', methods=['PUT'])
@jwt_required()
def marcar_como_lida(id):
    """Marcar notificação como lida"""
    usuario_id = get_jwt_identity()
    notificacao = Notification.query.filter_by(
        id=id,
        usuario_id=usuario_id
    ).first()
    
    if not notificacao:
        raise NotFoundError('Notificação não encontrada')
    
    notificacao.lida = True
    notificacao.data_leitura = datetime.utcnow()
    db.session.commit()
    
    return jsonify({'status': 'atualizado'}), 200

@notification_bp.route('/limpar', methods=['DELETE'])
@jwt_required()
def limpar_notificacoes():
    """Limpar todas as notificações lidas"""
    usuario_id = get_jwt_identity()
    
    notificacoes_lidas = Notification.query.filter_by(
        usuario_id=usuario_id,
        lida=True
    ).delete()
    
    db.session.commit()
    
    return jsonify({
        'status': 'limpas',
        'total_deletadas': notificacoes_lidas
    }), 200
```

#### 2. Registrar Blueprint em `app/__init__.py`

```python
def create_app(config_name='development'):
    # ... código anterior
    
    with app.app_context():
        from app.routes import (
            auth_bp, checklist_bp, round_bp, # ... outros
            notification_bp  # ← Adicionar
        )
        
        app.register_blueprint(notification_bp)  # ← Registrar
```

---

## Padrões de Código

### 1. Usar Type Hints

```python
from typing import Dict, List, Optional

def processar_checklist(checklist_id: int, items: List[Dict]) -> bool:
    """
    Processar checklist
    
    Args:
        checklist_id: ID do checklist
        items: Lista de itens
    
    Returns:
        bool: True se processado com sucesso
    """
    pass
```

### 2. Documentar Funções

```python
def criar_ronda(turno: str, data: str) -> Dict:
    """
    Criar nova ronda
    
    Args:
        turno: MANHA, TARDE ou NOITE
        data: Data da ronda (YYYY-MM-DD)
    
    Returns:
        Dict com informações da ronda criada
    
    Raises:
        ValueError: Se turno inválido
    """
    pass
```

### 3. Tratar Exceções

```python
from app.utils.errors import ValidationError, NotFoundError

try:
    # código
    pass
except ValidationError as e:
    return {'erro': str(e)}, 400
except NotFoundError as e:
    return {'erro': str(e)}, 404
except Exception as e:
    logger.error(f'Erro inesperado: {str(e)}')
    return {'erro': 'Erro interno do servidor'}, 500
```

### 4. Usar Schemas para Validação

```python
from marshmallow import Schema, fields, ValidationError

class NotificacaoSchema(Schema):
    titulo = fields.Str(required=True, validate=lambda x: len(x) > 0)
    mensagem = fields.Str(allow_none=True)
    tipo = fields.Str(validate=lambda x: x in ['INFO', 'AVISO', 'ERRO'])

schema = NotificacaoSchema()
erros = schema.validate(data)
if erros:
    return {'erros': erros}, 400
```

---

## Migrações de Banco de Dados

### Criar Nova Migração

```bash
# Criar automática
flask db migrate -m "Descrição da mudança"

# Revisar arquivo em migrations/versions/
# Editar se necessário

# Aplicar migração
flask db upgrade

# Reverter última migração
flask db downgrade
```

---

## Testes

### Executar Testes

```bash
# Todos os testes
pytest

# Testes específicos
pytest tests/test_auth.py

# Com coverage
pytest --cov=app tests/
```

### Escrever um Teste

```python
# tests/test_notification.py
import pytest
from app import create_app, db
from app.models import Notification, User

@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        db.create_all()
        yield app
        db.session.remove()
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

def test_criar_notificacao(client, app):
    with app.app_context():
        # Setup
        user = User(email='test@test.com', nome='Test')
        db.session.add(user)
        db.session.commit()
        
        # Test
        notif = Notification(
            usuario_id=user.id,
            titulo='Teste',
            mensagem='Mensagem de teste'
        )
        db.session.add(notif)
        db.session.commit()
        
        # Assert
        assert notif.id is not None
        assert notif.lida is False
```

---

## Deployment

### Deploy no Heroku

```bash
# 1. Criar Procfile
echo "web: gunicorn run:app" > Procfile

# 2. Instalar gunicorn
pip install gunicorn
pip freeze > requirements.txt

# 3. Deploy
git add .
git commit -m "Deploy para produção"
git push heroku main
```

### Deploy em Servidor Linux

```bash
# 1. Clonar repositório
git clone https://github.com/rheynansantos-beep/super-checklist.git
cd super-checklist

# 2. Instalar dependências do sistema
sudo apt-get install python3 python3-venv python3-pip mysql-server

# 3. Setup venv
python3 -m venv venv
source venv/bin/activate

# 4. Instalar dependências Python
pip install -r requirements.txt
pip install gunicorn

# 5. Executar migrações
flask db upgrade

# 6. Criar systemd service
sudo nano /etc/systemd/system/super-checklist.service
```

---

**Versão**: 1.0
**Data**: 2026-06-07
**Autor**: @rheynansantos-beep
