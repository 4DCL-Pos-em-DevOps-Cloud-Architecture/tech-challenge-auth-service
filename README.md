# auth-service (Go)

Este é o serviço de autenticação do projeto ToggleMaster. Ele é responsável por criar e validar chaves de API.

## 📦 Pré-requisitos (Local)

* [Go](https://go.dev/doc/install) (versão 1.21 ou superior)
* [PostgreSQL](https://www.postgresql.org/download/) (rodando localmente ou em um contêiner Docker)

## 🚀 Rodando Localmente

1.  **Clone o repositório** e entre na pasta `auth-service`.

2.  **Prepare o Banco de Dados:**
    * Crie um banco de dados no seu PostgreSQL (ex: `auth_db`).
    * Execute o script `db/init.sql` para criar a tabela `api_keys`:
        ```bash
        psql -U seu_usuario -d auth_db -f db/init.sql
        ```

3.  **Configure as Variáveis de Ambiente:**
    Crie um arquivo chamado `.env` na raiz desta pasta (`auth-service/`) com o seguinte conteúdo:
    ```.env
    # String de conexão do seu banco de dados PostgreSQL
    DATABASE_URL="postgres://SEU_USUARIO:SUA_SENHA@localhost:5432/auth_db"
    
    # Porta que o serviço irá rodar
    PORT="8001"
    
    # Chave mestra para criar novas chaves de API
    MASTER_KEY="admin-secreto-123"
    ```

4.  **Instale as Dependências:**
    ```bash
    go mod tidy
    ```

5.  **Inicie o Serviço:**
    ```bash
    go run .
    ```
    O servidor estará rodando em `http://localhost:8001`.

## 🧪 Testando os Endpoints

Você pode usar `curl` ou Postman.

**1. Verifique a Saúde (Health Check):**
```bash
curl http://localhost:8001/health
```

Saída esperada: `{"status":"ok"}`

**2. Crie uma nova Chave de API (requer a MASTER_KEY):**

```bash
curl -X POST http://localhost:8001/admin/keys \
-H "Content-Type: application/json" \
-H "Authorization: Bearer admin-secreto-123" \
-d '{"name": "meu-primeiro-servico"}'
``` 

Saída esperada (A SUA CHAVE SERÁ DIFERENTE):

```json
{
  "name": "meu-primeiro-servico",
  "key": "tm_key_a1b2c3d4...",
  "message": "Guarde esta chave com segurança! Você não poderá vê-la novamente."
}
```

**3. Valide a Chave que você acabou de criar: (Substitua tm_key_... pela chave exata que você recebeu no passo anterior)**

```bash
curl http://localhost:8001/validate \
-H "Authorization: Bearer tm_key_a1b2c3d4..."
```

Saída esperada: `{"message":"Chave válida"}`

**4. Teste uma Chave Inválida:**

```bash
curl http://localhost:8001/validate \
-H "Authorization: Bearer chave-errada-123"
```

Saída esperada: `Chave de API inválida ou inativa`

## Manifestos Kubernetes

Os manifestos deste serviço estão em `k8s/`. O serviço é publicado pelo Ingress na rota `/auth`.

Antes da aplicação, substitua os seguintes valores:

| Placeholder | Descrição |
|---|---|
| `<REGISTRY_URL>` | URI do registro ECR, por exemplo `123456789012.dkr.ecr.us-east-1.amazonaws.com` |
| `<BASE64_ENCODED_DATABASE_URL>` | `DATABASE_URL` codificada em Base64 |
| `<BASE64_ENCODED_MASTER_KEY>` | `MASTER_KEY` codificada em Base64 |

Não versione os valores reais do Secret no repositório.

Aplicação dos recursos:

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/jobs/db-schema-init-job.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

O endpoint externo fica disponível em `/auth`. O health check interno é `/health` na porta `8001`.
