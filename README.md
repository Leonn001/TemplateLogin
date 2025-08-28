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


# Explicação do código
### 🏛️ Arquitetura e Fluxo de Interação das Classes

Este projeto segue uma arquitetura em camadas com uma clara separação de responsabilidades. Abaixo, descrevemos o papel de cada componente principal e como eles interagem durante os fluxos de registro e login.

#### Responsabilidades das Classes

-   **`config/SecurityConfig.java`**: É a classe central de configuração do Spring Security. Define a `SecurityFilterChain` (regras de acesso), e cria os `Beans` essenciais que a aplicação usa para segurança, como o `PasswordEncoder`, `AuthenticationManager`, `JwtEncoder` e `JwtDecoder`.
-   **`config/RsaKeyProperties.java`**: Classe de configuração que mapeia as propriedades do `application.properties` para objetos `RSAPublicKey` e `RSAPrivateKey`, fornecendo as chaves prontas para a `SecurityConfig` usar.
-   **`user/User.java`**: Entidade JPA que representa um usuário no banco de dados, mapeada para a tabela `users`.
-   **`user/UserRepository.java`**: Interface do Spring Data JPA que abstrai o acesso ao banco de dados para a entidade `User`.
-   **`security/UserDetailsServiceImpl.java`**: Implementa a interface `UserDetailsService`. É a "ponte" entre o `UserRepository` e o `AuthenticationManager`, ensinando ao Spring como encontrar um usuário.
-   **`security/jwt/JwtService.java`**: Serviço especialista em JWT. Sua única responsabilidade é gerar tokens seguros a partir de uma `Authentication` validada, usando o `JwtEncoder`.
-   **`auth/AuthenticationController.java`**: Controlador REST que expõe os endpoints públicos (`/login`, `/register`). Lida com as requisições e respostas HTTP.
-   **`auth/AuthenticationService.java`**: Orquestra a lógica de negócio de registro e login. Valida os dados, usa o `AuthenticationManager` e chama o `JwtService`.
-   **`auth/dto/`**: Contém os Data Transfer Objects (`LoginRequestDTO`, `RegisterRequestDTO`) que transportam os dados das requisições.

---
### Fluxo de Interação das Classes

#### ➡️ Fluxo de Login (`POST /api/auth/login`)

1.  A requisição chega no **`AuthenticationController`**, que recebe as credenciais dentro de um `LoginRequestDTO`.
2.  O `Controller` não possui lógica, ele apenas chama o método `authenticateAndGenerateToken` do **`AuthenticationService`**.
3.  O `Service` cria um `UsernamePasswordAuthenticationToken` e o passa para o **`AuthenticationManager`** (fornecido pela `SecurityConfig`) para validação.
4.  O `AuthenticationManager` chama internamente o **`UserDetailsServiceImpl`** para buscar o usuário no banco de dados.
5.  O `UserDetailsServiceImpl` usa o **`UserRepository`** para executar a query `findByUsername`.
6.  Se o usuário é encontrado e a senha (comparada usando o `PasswordEncoder`) está correta, o `AuthenticationManager` retorna um objeto `Authentication` completo.
7.  O `AuthenticationService` recebe este objeto e chama o **`JwtService`** para criar o token.
8.  O `JwtService` usa o `Bean` do **`JwtEncoder`** (configurado na `SecurityConfig` com a chave privada RSA) para gerar e assinar a string do token.
9.  O token é retornado pela cadeia (`Service` -> `Controller`) até o cliente como resposta da API.

#### ➡️ Fluxo de Registro (`POST /api/auth/register`)

1.  A requisição chega no **`AuthenticationController`** com os dados do novo usuário em um `RegisterRequestDTO`.
2.  O `Controller` chama o método `register` do **`AuthenticationService`**.
3.  O `Service` usa o **`UserRepository`** para verificar se o `username` ou `email` já existem, lançando uma exceção em caso afirmativo.
4.  O `Service` pega a senha em texto puro do DTO e usa o `Bean` do **`PasswordEncoder`** (da `SecurityConfig`) para gerar um hash seguro.
5.  O `Service` cria uma nova instância da entidade **`User`** com os dados validados e a senha já criptografada.
6.  O **`UserRepository`** é usado novamente para salvar (`save`) a nova entidade `User` no banco de dados.
7.  Uma mensagem de sucesso é retornada pela cadeia até o cliente.
