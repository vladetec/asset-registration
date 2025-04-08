
<div align="center">

![LogoRecadPatrimonial](./images_docs/LogoRecadPatrimonial.png)

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

## 4. Estrutura de Pastas do Projeto
   
```tree
patrimonio/
├── .env                    # Variáveis de ambiente
├── docker-compose.yml      # Configuração dos containers
├── Dockerfile              # Configuração da imagem Docker
├── requirements.txt        # Dependências Python
├── manage.py
├── README.md
├── patrimonio/             # Configuração principal do Django
│   ├── settings/
│   │   ├── base.py        # Configurações base
│   │   ├── development.py # Configs de desenvolvimento
│   │   └── production.py  # Configs de produção
│   ├── urls.py            # Rotas principais
│   └── asgi.py/wsgi.py
├── tenants/               # App de gerenciamento de tenants
│   ├── models.py          # Model Tenant
│   ├── api.py             # Endpoints Django Ninja
│   └── ...
├── core/                  # App principal de patrimônio
│   ├── models.py          # Model ItemPatrimonio
│   ├── api.py             # Endpoints Django Ninja
│   ├── services.py        # Lógica de negócio
│   └── ...
├── reports/               # Geração de relatórios
│   ├── templates/         # Templates HTML para relatórios
│   └── ...
├── static/                # Arquivos estáticos
├── media/                 # Arquivos de mídia (desenvolvimento)
└── mobile_app/            # Código do aplicativo Flet
    ├── main.py            # Ponto de entrada
    ├── assets/            # Imagens, etc.
    └── ...
```
