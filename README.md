
<div align="center">

![LogoRecadPatrimonial](./docs/images_docs/LogoRecadPatrimonial.png)

</div>
<center>

# <font color="rgb(0, 34, 98)" size="7" face="Arial">Recad Patrimonial</font>
</center>

# DOCUMENTAÇÃO
# Sobre
Documentação para Aplicação Recad Patrimonial de Gestão de Patrimônio Municipal

|---|---|

## 1. Requisitos Funcionais (RFs)
#### RF01 - Gerenciamento de Tenants
- RF01.1: O sistema deve permitir o cadastro de múltiplos tenants (organizações/clientes)
- RF01.2: Cada tenant deve ter seus dados completamente isolados dos demais
- RF01.3: O sistema deve validar o domínio/identificador do tenant no login
#### RF02 - Cadastro de Itens Patrimoniais
- RF02.1: Permitir cadastro de novos itens via aplicativo mobile
- RF02.2: Gerar/ler QR Code ou Código de Barras para identificação do item
- RF02.3: Campos obrigatórios: numero (único por tenant), nome, descricao, tipo, valor, local, responsavel, aquisicao, estado
- RF02.4: Campos automáticos: id, usuario, data_criacao, data_atualizacao, ativo
#### RF03 - Conferência de Patrimônio
- RF03.1: Ler QR Code/Código de Barras via câmera do dispositivo
- RF03.2: Exibir todos os dados do item após leitura
- RF03.3: Permitir edição rápida de campos como local, responsavel e estado
#### RF04 - Relatórios
- RF04.1: Gerar relatórios consolidados por filtros (local, tipo, estado, período)
- RF04.2: Exportar em formatos: PDF, CSV, HTML
- RF04.3: Visualização prévia no navegador antes de exportar
#### RF05 - Autenticação e Autorização
- RF05.1: Login com usuário e senha (JWT)
- RF05.2: Controle de acesso baseado em roles (admin, gestor, conferente)
- RF05.3: Cada usuário associado a um tenant específico

## 2. Requisitos Não Funcionais (RNFs)
#### RNF01 - Desempenho
- RNF01.1: Tempo de resposta da API < 500ms para 95% das requisições
- RNF01.2: Suportar até 1000 requisições simultâneas por tenant
#### RNF02 - Segurança
- RNF02.1: Todos os dados em trânsito devem usar TLS 1.2+
- RNF02.2: Autenticação via JWT com tempo de expiração configurável
- RNF02.3: Isolamento completo de dados entre tenants
- RNF02.4: Hash de senhas usando bcrypt
#### RNF03 - Disponibilidade
- RNF03.1: SLA de 99.5% de disponibilidade
- RNF03.2: Backup automático diário dos bancos de dados
#### RNF04 - Usabilidade
- RNF04.1: Interface mobile responsiva e intuitiva
- RNF04.2: Tempo de cadastro de item < 30 segundos após leitura do código
#### RNF05 - Compatibilidade
- RNF05.1: Suporte a Android 10+ e iOS 14+
- RNF05.2: Navegadores modernos (Chrome, Firefox, Safari, Edge)

<center>

## 3. Diagrama de Arquitetura

```
+-------------------+       +-------------------+       +-------------------+
|   Mobile Client   |       |   Web Client      |       |   Admin Client    |
|   (Flet - Python) |       |   (HTML/JS)       |       |   (Django Admin)  |
+-------------------+       +-------------------+       +-------------------+
        |                           |                           |
        | HTTPS (REST API)          | HTTPS (REST API)          | HTTPS
        v                           v                           v
+----------------------------------------------------------------------------+
|                             API Gateway (Nginx)                            |
+----------------------------------------------------------------------------+
                                        |
                                        v
+----------------------------------------------------------------------------+
|                              Django Application                           |
| +----------------+  +----------------+  +----------------+  +------------+|
| |   Tenant       |  |   Auth         |  |   Patrimonio   |  |   Reports  ||
| |   Management   |  |   (JWT)        |  |   (Core)       |  |   Module   ||
| +----------------+  +----------------+  +----------------+  +------------+|
|                                                                          |
| Django Ninja (REST API)                                                  |
+----------------------------------------------------------------------------+
        |                           |                           |
        |                           |                           |
        v                           v                           v
+-------------------+       +-------------------+       +-------------------+
|   PostgreSQL      |       |   MinIO (S3)      |       |   Redis Cache     |
|   (Multi-tenant   |       |   (QR Codes,      |       |   (Rate Limiting, |
|   schemas)        |       |   Fotos)          |       |   Sessions)       |
+-------------------+       +-------------------+       +-------------------+
```
</center>

# 4. Passo a passo Criar e Rodar Localmente o Projeto
```shell
# Clone esse repositorio
git clone https://github.com/vladetec/asset-registration.git
cd asset-registration

# Rode com Pyenv e cria o ambiente virtual
pyenv local 3.12.3
pyenv exec python -m venv venv --prompt asset-reg

# Rode Sem Pyenv e cria o ambiente virtual
python -m venv venv --prompt asset-reg 

# Execute o ambiente virtual
source venv/bin/activate  Ou . venv/bin/activate  # Linux
#. venv\Scripts\activate  # Windows

pip install -U pip setuptools wheel

pip install -r requirements-dev.txt


# Iniciar ambiente de desenvolvimento
docker compose up -d --build

# Executar migrações
docker compose exec web python manage.py migrate

# Criar superusuário
docker compose exec web python manage.py createsuperuser --username root --email root@exmaple.com

# Executar testes
docker compose exec web pytest

# Executar linter
docker compose exec web ruff check .

# Acessar o banco de dados
docker compose exec db psql -U app_user -d app_dev

# Acessar o console MinIO
http://localhost:9001 (usuário/senha do .env.dev)

# Construa serviços individualmente:
docker-compose build web
docker-compose build celery
docker-compose build celery-beat

# Parar e remover tudo
docker compose down -v
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q) && docker rmi $(docker images -q) 
docker volume prune
docker system prune -f
docker builder prune -f

# Remover conflito de Porta
sudo kill -9 $(sudo lsof -t -i:8000)

```
Author: @VladeSouza 

Github: https://github.com/vladetec

Linkedin: https://www.linkedin.com/in/vlademir-souza-ads
