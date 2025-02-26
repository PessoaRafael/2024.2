# Docker compose com Django e banco de dados

## Informações gerais

- **Aluno**:
  - Nome: Rafael Pessoa Bento do Nascimento
  - Matrícula: 20211014040003

## Relato

Nesse projeto, configurei um ambiente Docker para rodar uma aplicação Django com um banco de dados PostgreSQL. A ideia era deixar tudo pronto para rodar sem precisar instalar dependências manualmente na máquina, apenas com `docker-compose up`.

### O que foi feito

1. Criei um projeto Django básico para servir como aplicação web.
2. Escrevi um `Dockerfile` para gerar a imagem do Django, garantindo que todas as dependências fossem instaladas corretamente.
3. Testei o container isoladamente para ver se tudo rodava como esperado.
4. Montei o `docker-compose.yml` para subir dois serviços: a aplicação Django (`webapp`) e o banco de dados PostgreSQL (`db`).
5. Ajustei as configurações do Django para que ele se conectasse ao banco via o serviço `db` do Docker.
6. Testei a configuração com `docker-compose up`, garantindo que a aplicação estivesse acessível e conectada ao banco corretamente.
7. Registrei esse processo aqui para documentar tudo o que foi feito.

## Arquivos Docker e configuração do Django

### `Dockerfile`
```dockerfile
# Usando a imagem oficial do Python como base
FROM python:3.10

# Definir diretório de trabalho no container
WORKDIR /app

# Copiar os arquivos do projeto para o container
COPY . /app/

# Instalar dependências
RUN pip install --no-cache-dir -r requirements.txt

# Expor a porta da aplicação
EXPOSE 8000

# Comando para rodar a aplicação
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### `docker-compose.yml`
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  webapp:
    build: .
    restart: always
    depends_on:
      - db
    ports:
      - "8000:8000"
    environment:
      DATABASE_NAME: mydatabase
      DATABASE_USER: user
      DATABASE_PASSWORD: password
      DATABASE_HOST: db
      DATABASE_PORT: 5432
    volumes:
      - .:/app

volumes:
  postgres_data:
```

### Configuração do banco de dados no Django (`settings.py`)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DATABASE_NAME', 'mydatabase'),
        'USER': os.getenv('DATABASE_USER', 'user'),
        'PASSWORD': os.getenv('DATABASE_PASSWORD', 'password'),
        'HOST': os.getenv('DATABASE_HOST', 'db'),
        'PORT': os.getenv('DATABASE_PORT', '5432'),
    }
}
```

## Testes e Considerações

Depois de configurar tudo, testei com os seguintes passos:

- Rodei `docker-compose build` para garantir que as imagens fossem criadas corretamente.
- Subi os containers com `docker-compose up` e verifiquei se a aplicação estava rodando em `http://localhost:8000`.
- Testei a conexão entre o Django e o banco PostgreSQL, rodando migrações e criando alguns registros.

No final, tudo funcionou bem e agora o projeto pode ser rodado facilmente em qualquer máquina com Docker instalado.
