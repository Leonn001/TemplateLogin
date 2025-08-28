# Template de Segurança com Spring Boot 3 e JWT (RSA)

Este projeto é um boilerplate robusto para implementar autenticação e autorização usando Spring Security 6 com tokens JWT assinados de forma assimétrica (RSA 2048 bits).

## ✨ Funcionalidades

-   **Autenticação via Endpoint de Login:** Rota `POST /api/auth/login` para validar credenciais.
-   **Geração de Token JWT:** Utiliza chaves RSA (pública/privada) para máxima segurança.
-   **Arquitetura Stateless:** Sem armazenamento de sessão no servidor.
-   **Configuração de Segurança Moderna:** Baseado em `SecurityFilterChain` e `Lambdas`.
-   **Estrutura Preparada para RBAC (Role-Based Access Control):** A base para um sistema de permissões por papéis.

## 🚀 Tecnologias Utilizadas

-   Java 21
-   Spring Boot 3.3+
-   Spring Security 6+
-   Spring Data JPA
-   PostgreSQL
-   Lombok
-   Maven

## ⚙️ Configuração do Ambiente

Siga estes passos para rodar o projeto localmente.

### 1. Banco de Dados
-   Garanta que você tenha uma instância do PostgreSQL rodando.
-   Crie um banco de dados (ex: `auth_db`).
-   Ajuste as credenciais do banco no arquivo `src/main/resources/application.properties`.

### 2. Gerar Chaves RSA
Este projeto usa um par de chaves RSA para assinar e verificar os tokens JWT. Para gerar seus próprios arquivos de chave:

-   Navegue até a pasta `src/main/resources` no seu terminal.
-   Execute os seguintes comandos do OpenSSL:

    ```bash
    # Gerar a chave privada
    openssl genpkey -algorithm RSA -out app.key -pkeyopt rsa_keygen_bits:2048

    # Extrair a chave pública
    openssl rsa -in app.key -pubout -out app.pub
    ```
-   **Importante:** Adicione o arquivo da chave privada (`app.key`) ao seu `.gitignore` para nunca enviá-lo a um repositório público!

### 3. Configurar `application.properties`
Garanta que o arquivo aponte para a localização das suas chaves:
```properties
rsa.public-key-location=classpath:app.pub
rsa.private-key-location=classpath:app.key
```

## ▶️ Como Rodar

1.  Clone este repositório.
2.  Siga os passos de configuração acima.
3.  Execute a aplicação.
4.  Use um cliente de API (como o Postman) para testar.

## Endpoints da API

### Autenticação

-   **`POST /api/auth/login`**: Realiza o login.
    -   **Body (JSON):**
        ```json
        {
            "username": "seu_usuario",
            "password": "sua_senha"
        }
        ```
    -   **Resposta de Sucesso (200 OK):**
        ```
        eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJzcHJpbmctc2VjdXJpdHktand0Iiwic3ViIjoidXNlciIsImV4cCI6MTc1MjQzMjE4NywiaWF0IjoxNzUyNDI4NTg3LCJzY29wZSI6IlJPTEVfVVNFUiJ9.abc...
        ```
