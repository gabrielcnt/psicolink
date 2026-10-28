# PsicoLink

Nome: PsicoLink
Descrição: API REST para gerenciamento de perfis profissionais de psicólogos.
Objetivo: demonstrar habilidades em desenvolvimento backend com Python, autenticação, regras de negócio e persistência de dados.

## Problema que o projeto busca resolver

O PsicoLink permite reunir em um perfil público informações profissionais de um psicólogo e links que podem ser úteis a quem o visita. A API oferece operações autenticadas para administrar o perfil e seus conteúdos, além de uma consulta pública para apresentar esses dados.

## Funcionalidades implementadas

- Cadastro de usuários e autenticação por e-mail e senha.
- Emissão de token JWT do tipo Bearer e consulta dos dados do usuário autenticado.
- Criação e atualização de um perfil por usuário, com slug gerado a partir do nome profissional e do CRP.
- Consulta pública do perfil pelo slug, incluindo links e especialidades associados.
- Criação, edição, listagem e remoção de links, com validação de URL, ordenação e restrição de acesso ao proprietário do perfil.
- Consulta do catálogo de especialidades e associação, ordenação e remoção de especialidades em um perfil.
- Solicitação e redefinição de senha com tokens temporários armazenados como hash.
- Migrações de banco de dados para as entidades da aplicação.
- Testes automatizados com `pytest` para autenticação, perfis, links e especialidades.

## Tecnologias

- **Python 3.12 ou superior**
- **FastAPI** para a API HTTP e documentação OpenAPI.
- **PostgreSQL** como banco de dados relacional.
- **SQLAlchemy 2** como ORM e acesso ao banco.
- **Alembic** para migrações de esquema.
- **Pydantic** e **pydantic-settings** para schemas, validação e configurações.
- **PyJWT** para tokens de acesso.
- **pwdlib** com suporte a **Argon2** para hash e verificação de senhas.
- **psycopg 3** como driver PostgreSQL.
- **uv** para gerenciamento do ambiente e dependências.
- **pytest** e **HTTPX** para testes.
- **Uvicorn** para servir a aplicação localmente.

## Arquitetura e organização

A aplicação está organizada por domínio dentro de `app/`:

| Diretório/arquivo | Responsabilidade |
| --- | --- |
| `app/auth/` | Rotas, schemas, serviços, repositório, modelos e dependências de autenticação. |
| `app/profile/` | Operações de perfil, geração de slug e consulta pública. |
| `app/link/` | Regras e operações dos links de um perfil. |
| `app/specialty/` | Catálogo de especialidades e associações com perfis. |
| `app/shared/config/` | Carregamento das configurações da aplicação. |
| `app/shared/database/` | Base declarativa, engine, sessões e dependência de banco. |
| `app/shared/security/` | Criação e validação de JWTs e hash de senhas. |
| `app/shared/exceptions/` | Mapeamento de exceções de domínio para respostas HTTP. |
| `migrations/` | Ambiente e revisões do Alembic. |
| `scripts/` | Scripts auxiliares relacionados aos dados da aplicação. |
| `tests/` | Testes organizados por domínio e fixtures compartilhadas. |

Em geral, as rotas recebem schemas validados pelo Pydantic e delegam regras de negócio aos serviços, que usam repositórios para consultar e persistir entidades via SQLAlchemy. A aplicação é iniciada por `app.main:app`.

## Modelo de dados

As migrações definem as seguintes entidades e relações:

- **`users`**: usuários com identificador UUID, nome, e-mail único, hash da senha, indicadores de estado e datas de criação/atualização.
- **`password_reset_tokens`**: tokens de redefinição armazenados como hash, com validade, data de uso e referência ao usuário.
- **`profiles`**: perfil associado a um usuário (um perfil por usuário), com slug único, nome profissional, CRP, biografia, cidade, URL de foto e template.
- **`links`**: links pertencentes a um perfil, com título, URL e posição. Título e URL são únicos dentro de cada perfil.
- **`specialties`**: catálogo de especialidades com nome único.
- **`profile_specialties`**: associação muitos-para-muitos entre perfis e especialidades, com posição e chave primária composta.

## Pré-requisitos

- Python 3.12 ou superior.
- PostgreSQL acessível localmente ou em outro host.
- [uv](https://docs.astral.sh/uv/) instalado.
- Um banco de dados para a aplicação e outro banco **descartável** para os testes.

## Configuração do ambiente e instalação

Execute os comandos a partir da raiz do repositório:

```bash
uv sync --dev
```

Crie o arquivo `.env` na raiz do projeto e preencha as configurações descritas abaixo antes de executar a aplicação, as migrações ou os testes.

## Variáveis de ambiente

As configurações são carregadas por `pydantic-settings` do ambiente ou de um arquivo `.env` no diretório de execução. Os cinco campos abaixo são obrigatórios para carregar as configurações:

| Variável | Uso |
| --- | --- |
| `DATABASE_URL` | URL de conexão PostgreSQL usada pela aplicação e pelo Alembic. |
| `TEST_DATABASE_URL` | URL de conexão usada pela suíte de testes. |
| `SECRET_KEY` | Chave usada para assinar e validar tokens JWT. |
| `ALGORITHM` | Algoritmo JWT configurado para emissão e validação. |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Duração do token de acesso, em minutos. |

Exemplo com valores fictícios para desenvolvimento local:

```dotenv
DATABASE_URL=postgresql+psycopg://postgres:senha-local@localhost:5432/psicolink
TEST_DATABASE_URL=postgresql+psycopg://postgres:senha-local@localhost:5432/psicolink_test
SECRET_KEY=chave-ficticia-de-desenvolvimento
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
```

Substitua as URLs, credenciais e demais valores conforme seu ambiente. Não reutilize a chave fictícia em ambientes compartilhados ou de produção.

## PostgreSQL

Crie os bancos `psicolink` e `psicolink_test` (ou outros nomes) e conceda ao usuário informado nas URLs as permissões necessárias. Com as ferramentas de linha de comando do PostgreSQL instaladas, por exemplo:

```bash
createdb psicolink
createdb psicolink_test
```

Garanta que cada URL em `.env` aponte para o banco correspondente. **O banco de testes deve ser isolado e descartável**: a fixture automática da suíte apaga e recria as tabelas nesse banco antes de cada teste.

## Migrações do banco

Com `DATABASE_URL` configurada e o PostgreSQL acessível, aplique as migrações:

```bash
uv run alembic upgrade head
```

As revisões do Alembic criam as tabelas descritas em [Modelo de dados](#modelo-de-dados). Elas não populam o catálogo de especialidades.

## Executar a aplicação

Com o ambiente configurado e as migrações aplicadas:

```bash
uv run uvicorn app.main:app --reload
```

Por padrão, o Uvicorn disponibiliza a aplicação em `http://127.0.0.1:8000`. A rota de verificação de saúde é `GET /health`.

## Endpoints

Todas as rotas da API, exceto `/health`, usam o prefixo `/api/v1`. As rotas indicadas como autenticadas exigem `Authorization: Bearer <token>`.

### Autenticação — `/api/v1/auth`

| Método | Caminho | Acesso | Descrição |
| --- | --- | --- | --- |
| `POST` | `/api/v1/auth/register` | Público | Cadastra usuário; senha com no mínimo 8 caracteres. |
| `POST` | `/api/v1/auth/login` | Público | Autentica com formulário OAuth2 (`username` recebe o e-mail e `password` a senha); retorna token Bearer. |
| `GET` | `/api/v1/auth/me` | Autenticado | Retorna os dados do usuário autenticado. |
| `POST` | `/api/v1/auth/forgot-password` | Público | Solicita a redefinição de senha. |
| `POST` | `/api/v1/auth/reset-password` | Público | Redefine a senha usando token e nova senha. |

### Perfis — `/api/v1/profile`

| Método | Caminho | Acesso | Descrição |
| --- | --- | --- | --- |
| `POST` | `/api/v1/profile/` | Autenticado | Cria o perfil do usuário autenticado. |
| `PATCH` | `/api/v1/profile/{profile_id}` | Autenticado | Atualiza os campos permitidos do perfil pertencente ao usuário. |
| `GET` | `/api/v1/profile/{slug}` | Público | Consulta o perfil público pelo slug, incluindo links e especialidades. |

### Links de perfil

| Método | Caminho | Acesso | Descrição |
| --- | --- | --- | --- |
| `POST` | `/api/v1/profile/{profile_id}/links/` | Autenticado | Cria um link e atribui sua posição inicial. |
| `GET` | `/api/v1/profile/{profile_id}/links/` | Autenticado | Lista os links do perfil em ordem de posição. |
| `PATCH` | `/api/v1/profile/{profile_id}/links/{link_id}/` | Autenticado | Atualiza título, URL e/ou posição de um link. |
| `DELETE` | `/api/v1/profile/{profile_id}/links/{link_id}/` | Autenticado | Remove um link. |

### Especialidades

| Método | Caminho | Acesso | Descrição |
| --- | --- | --- | --- |
| `GET` | `/api/v1/profile/specialties` | Público | Lista o catálogo de especialidades. |
| `POST` | `/api/v1/profile/{profile_id}/specialties/` | Autenticado | Associa uma especialidade existente ao perfil. |
| `GET` | `/api/v1/profile/{profile_id}/specialties/` | Autenticado | Lista as especialidades associadas ao perfil em ordem de posição. |
| `PATCH` | `/api/v1/profile/{profile_id}/specialties/{specialty_id}` | Autenticado | Atualiza a posição da associação. |
| `DELETE` | `/api/v1/profile/{profile_id}/specialties/{specialty_id}` | Autenticado | Remove a associação da especialidade ao perfil. |

## Documentação interativa

Com a aplicação em execução, acesse:

- Swagger UI: `http://127.0.0.1:8000/docs`
- ReDoc: `http://127.0.0.1:8000/redoc`
- OpenAPI JSON: `http://127.0.0.1:8000/openapi.json`

## Testes

Configure `TEST_DATABASE_URL` para apontar a um banco PostgreSQL exclusivo para testes e execute:

```bash
uv run pytest
```

**Atenção:** antes de cada teste, a fixture `clean_database` executa `drop_all` e `create_all` no banco de `TEST_DATABASE_URL`. Não use um banco com dados que precisam ser preservados. Os testes cobrem autenticação, validação, perfis públicos e privados, autorização, links e associações de especialidades.

## Limitações e pontos a revisar

- A solicitação de redefinição de senha gera e armazena o hash de um token, mas a rota não envia e-mail nem retorna o token; por isso, o fluxo não entrega ao usuário o token necessário para chamar a redefinição.
- As migrações criam o catálogo `specialties` vazio. A API permite listar especialidades existentes e associá-las a perfis, mas não oferece rota para cadastrar itens no catálogo. Há um script de seed, porém sua execução direta não inicializa a sessão de banco necessária.
- O slug é gerado na criação do perfil. Embora nome profissional e CRP possam ser atualizados, o serviço de atualização não recalcula o slug.
- O repositório implementa a API e os dados do perfil; não há uma interface web para renderizar as páginas profissionais.