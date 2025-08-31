# 🌐 MACRO – Ecossistema ShipLink (Para ChatGPT)

## 1️⃣ Visão Geral

O **ShipLink** é um ecossistema de **microsserviços desacoplados**, cada um responsável por um domínio específico (CRUD, Organização, Usuário, TMS, CTe, etc.).  

**Objetivos principais:**

- Cada microserviço é **autônomo**, recebendo apenas requisições HTTP e **nunca acessa o banco diretamente**.
- **Core-CRUD** é exclusivo e central: único responsável por conexão, persistência, tradução de campos e gerenciamento de transações.
- Banco de dados sempre separado por microserviço (MySQL, PostgreSQL, Mongo, etc.), independente de ambiente (localhost ou cloud).
- Comunicação entre micros: **HTTP/JSON**, com possibilidade futura de **brokers** (Kafka, RabbitMQ).
- **Padrão Silicon Valley**: clean code, modular, escalável, Dockerizado e pronto para produção.
- Suporte SaaS / multi-tenant: múltiplos bancos, múltiplos drivers, múltiplos micros.

---

## 2️⃣ Core-CRUD: Backbone Exclusivo

- **Exclusivo, nunca exposto** aos produtos ou micros.
- Possui **DbConfig**: mapeia tokens de micros (`MICRO_KEY`) para suas conexões e drivers.
- Possui **CrudBase**: traduz campos camelCase do JSON para os nomes reais no banco (snake_case).
- Gerencia **transações multi-banco/multi-driver**, incluindo **rollback completo em falhas**.
- Montagem dinâmica de payloads com herança: **SIMPLE** (opera no nível atual) ou **CASCATE** (percorre hierarquia Pai → Filho → Neto).
- Autenticação obrigatória: **MASTER_KEY** no header HTTP (Bearer, hash SHA256).

### Fluxo de processamento

1. **EngineManager**: Valida MASTER_KEY.
2. **CrudManager**: Percorre request, abre conexões, executa operações, gerencia transações e devolve JSON padronizado.
3. Suporte a operações complexas: insert/update/delete/fetch, joins, group by, order by, subqueries e herança de campos.

---

## 3️⃣ Estrutura e Convenções de Microsserviço

### 3.1 Diretórios e Arquivos Principais

```
/<nome-micro>/
├─ public/ (endpoint)
│  └─ index.(php/js/java/cs)
├─ src/
│  └─ Controllers/     # Recebe JSON e responde em JSON padronizado
│  └─ Models/          # Regra de negócio
│  └─ Routes/          # Recebe a rota do index e encaminha ao controller
│  └─ Services/        # Lógica específica e integração com Core-CRUD
├─ Dockerfile
├─ docker-compose.yml (opcional)
└─ README.md
```

### 3.2 Docker

- Core-CRUD: precisa de drivers de bancos (MySQL, Postgres, Mongo, SQLServer, etc.)
- Outros micros: apenas tecnologias do micro, sem drivers de banco
- Micros enviam **JSON HTTP para core-crud** e recebem JSON padronizado

| Micro | Tecnologia | Docker base |
|-------|-----------|------------|
| core-organization | PHP | php:8.2-apache |
| Boleto | Node.js | node:20-alpine |
| Dashboard | Java | openjdk:20-jdk |
| XPTO | C# | mcr.microsoft.com/dotnet/sdk:8.0 |

### 3.3 Convenções de JSON

- Campos em **camelCase**
- Operações: `create`, `update`, `fetch`, `delete`
- Token de identificação do micro: `tokenIdentidade` (SHA256)
- Tipos de serviço:
  - **SIMPLE** → opera no nível atual, sem herança
  - **CASCATE** → percorre hierarquia Pai → Filho → Neto, com herança de campos
- Transações disponíveis mesmo em SIMPLE

---

## 4️⃣ Conexão e Autenticação

- Core-CRUD `.env` armazena **MASTER_KEY** (hash)
- Micros `.env` armazenam **MICRO_KEY** (SHA256) - esse token é o identificador do micro para o crud saber o driver/banco/credenciais.
- Comunicação via HTTP:
  - Header: `Authorization: Bearer <MASTER_KEY>`
  - Body JSON: inclui `"tokenIdentidade": "<MICRO_KEY>"`

Exemplo request PHP:

```php
$token = getenv('MASTER_API_KEY');
$ch = curl_init('http://core-crud:8080/crud');
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer ' . $token,
    'Content-Type: application/json'
]);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode([
    'service' => [
        [
            'serviceType' => 'SIMPLE',
            'tokenIdentidade' => getenv('MICRO_KEY'),
            'operation' => 'insert',
            'setCampos' => [['campo'=>'nome','valor'=>'Produto X']],
            'setFrom' => [['tabela'=>'produto','alias'=>'p']]
        ]
    ]
]));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
$data = json_decode($response, true);
```

---

## 5️⃣ Exemplos de Operações

- **Insert, Update, Delete**
- **Fetch simples**
- **Fetch avançado**: joins múltiplos, group by, order by, subqueries, filtros complexos
- **Payload cascata**: herança de campos entre múltiplos micros
- **Transações multi-banco/multi-driver**

---

## 6️⃣ Estrutura de Drivers Core-CRUD

```
core-crud/
└─ src/
   ├─ drivers/
   │  ├─ mysql/
   │  │  ├─ Crud.php   # CRUD simples
   │  │  └─ DB.php     # Conexões, dialetos e transações
   │  ├─ postgresql/
   │  ├─ oracle/
   │  ├─ mongo/
   │  ├─ sqlite/
   │  └─ sqlserver/
   ├─ CrudManager.php  # Gerencia transações e execução
   ├─ CrudBase.php     # Tradutor camelCase → snake_case
   ├─ DbConfig.php     # Mapeia MICRO_KEY → conexão
   └─ EngineManager.php # Valida MASTER_KEY
```

---

## 7️⃣ Produtos e Micros

```
ShipLink/
├── CORE/
│   ├── core-crud/
│   ├── core-organization/
│   ├── core-user/
│   └── ...
├── Produto-MetaLog/
│   ├── tms-freight-auction/
│   ├── tms-order-processing/
│   └── tms-invoicing/
├── Produto-XPTO/
├── Produto-REFEICON/
└── Produto-NFEASY/
```

- Produtos podem ter múltiplos micros para **isolamento e escalabilidade**
- Todos consomem Core-CRUD via HTTP/JSON
- Seguem **camelCase**, estrutura padronizada e boas práticas

---

## 8️⃣ Boas Práticas

1️⃣ Um micro por domínio, objetivo único  
2️⃣ Banco sempre separado → isolamento e escalabilidade  
3️⃣ Micros usam **JSON padronizado**  
4️⃣ Core-CRUD traduz campos e gerencia transações  
5️⃣ Nomeação consistente: `core-<domínio>`  
6️⃣ Micros autônomos, local ou cloud  
7️⃣ Atualizações: `docker-compose build --no-cache && docker-compose up -d`  
8️⃣ Padrão Silicon Valley aplicado a **tudo que for desenvolvido**  
9️⃣ Suporte SaaS / multi-tenant  
🔟 Estrutura de src padronizada (Controllers, Models, Services)

---

## 9️⃣ Observações Finais

- Este MACRO serve para me dar **toda a bagagem do ecossistema**
- Foco: consistência, escalabilidade e reuso, não customizações individuais
- Alterações no banco ou nomes de campos → só CrudBase no Core-CRUD precisa ser atualizado
- Autenticação obrigatória: MASTER_KEY via HEADER, MICRO_KEY via payload
- Payloads podem ser **dinâmicos e em cascata**, permitindo hierarquias complexas de operação

---

## 🔟 Referência Rápida para Desenvolvimento de Micros

- Receber JSON padronizado  
- Validar tokens  
- Chamadas ao Core-CRUD  
- Operações simples ou cascata  
- Respeitar camelCase e convenções de src  
- Garantir transações consistentes  
- Seguir padrões de clean code, Docker e escalabilidade


## Inicio Rápido:
### Estrutura base
```bash
core-organization/
├─ public/
│  └─ index.php           # Entrada do microsserviço
├─ src/
│  ├─ Controllers/
│  │  └─ OrganizationController.php  # Recebe JSON e envia para Services
│  ├─ Models/
│  │  └─ OrganizationModel.php       # Regras de negócio da organização
│  ├─ Routes/
│  │  └─ api.php                     # Roteamento das requisições
│  └─ Services/
│     └─ OrganizationService.php     # Comunicação com Core-CRUD
├─ Dockerfile
├─ docker-compose.yml
└─ README.md
```
> Os nomes devem ser ajustados conforme o nome micro

---
### Criação automática da estrutura
```bash
# Script PowerShell para criar estrutura mínima do microsserviço
# Deve ser executado dentro da pasta do micro existente

# Pastas principais
$folders = @(
    "public",
    "src\Controllers",
    "src\Models",
    "src\Services",
    "src\Routes",
    "config",
    "tests"
)

# Criar pastas
foreach ($folder in $folders) {
    if (!(Test-Path $folder)) {
        New-Item -ItemType Directory -Path $folder -Force
    }
}

# Arquivos principais vazios
$files = @(
    "public/index.php",
    "src\Controllers/OrganizationController.php",
    "src\Models/OrganizationModel.php",
    "src\Services/OrganizationService.php",
    "src\Routes/api.php",
    "config/.env",
    "docker-compose.yml",
    "Dockerfile"
)

# Criar arquivos vazios
foreach ($file in $files) {
    if (!(Test-Path $file)) {
        New-Item -ItemType File -Path $file -Force
    }
}

# Criar README.md com modelo padrão
$readmeContent = @"
# 🚀 [Nome do Microsserviço]

## 🎯 Objetivo
Descreve a função do serviço dentro do ecossistema.  

## 🩹 Problema que resolve
Qual dor ele elimina e qual fluxo cobre.  

## ⚙️ Funcionalidades
- Funcionalidade 1  
- Funcionalidade 2  

## 🔑 Autenticação
Como o serviço lida com autenticação/autorização.  

## 🏗️ Estrutura
- `src/` → código principal  
- `tests/` → testes unitários  
- `config/` → variáveis e ambiente  

## 🧰 Stack
- Linguagem/framework  
- Banco de dados  
- Mensageria  

## 🌐 Endpoints
Exemplos de requests/responses.  

## 🔄 Fluxos
Diagramas de sequência ou explicação dos principais processos.  

## 🛠️ Deploy & Configuração
- Rodar local  
- Docker  
- Variáveis de ambiente necessárias  

## ✅ Testes
Como rodar testes localmente.  

## 🛡️ Segurança
Medidas específicas de segurança aplicadas.  

---
"@

if (!(Test-Path "README.md")) {
    Set-Content -Path "README.md" -Value $readmeContent -Force
}

Write-Host "Estrutura mínima criada com sucesso!"


```
> Arquivo salvo: create_core_organization.ps1
> Os nomes devem ser ajusatdos.



---


# ✅ Como você (ChatGPT) deve responder a qualquer dev que trouxer este MACRO

> **Objetivo:** sempre que um dev enviar este DOCUMENTO MACRO, responda **exatamente** com o passo a passo abaixo.
> Renomeie o nome do chat no historico de pesquisa para Onboarding ShipLink.

---
> Se você (ChatGPT) ainda não souber o nome do usuário pergunte e espere pela resposta.
---

## Seja bem vindo {$nome} ao ecosistema ShipLink
> Chame- pelo nome.

Vou te guiar passo a passo na construção da estrutura padrão do seu microsserviço.

---

## 🧭 Passo a passo oficial para iniciar um microsserviço ShipLink

**Ok!** Vamos criar e subir a estrutura padrão do seu micro no ecossistema **ShipLink** 🚀
Siga exatamente nesta ordem:

---

### 1) Crie/abra a pasta base do projeto ShipLink

Exemplo:

```powershell
cd C:\ShipLink
```
No Visual Studio Code aperte Ctrl + " (aspas dupla)

```cmd
cd C:\ShipLink
```
No Windows clique em pesquisar e procure por CMD.

---

### 2) Baixe o instalador padrão do ecossistema

Baixe `shplink-instalador.bat` e coloque **na pasta base** (ex.: `C:\ShipLink`):

> link para baixar 
```
<https://shiplink.infinityfree.me/tools/?download=instalador>

```
Cole o arquivo shplink-instalador.bat dentro da pasta do projeto. ex: C:\ShipLink

---

### 3) Rode o script para gerar o microsserviço

Dê duplo clique no arquivo e preencha:

O script vai pedir:

* **Seu nome**: `Seu Nome Completo`
* **Seu E-Mail**: `Ex: teste@teste.com.br`
* **Nome do produto** `tms, wms, cte, nfe, ...`
* **Nome do microsserviço** → `Financer` `Atendimento` `Dashboard` (exemplos)
* **MICRO\_KEY** → `1234567890123456790` 
* **Porta do micro** → `8080`

Ele criará a pasta - em letras minusculas - `core-<**Nome do microsserviço**>/` com a **estrutura padrão**. 

---

### 4) Entre na pasta do microsserviço pela linha de comando (Powershell/CMD)

```powershell
cd .\core-organization\ 
```
> substituir organization pelo nome da pasta gerada. 

---

### 5) Instale dependências com Composer

O script já cria um `composer.json` pronto com **autoload PSR-4** e pacotes essenciais.
Só rode:

```powershell
composer install
```

---

### 6) Suba o microsserviço localmente

```powershell
php -S localhost:%MICRO_PORT% -t public/
```
>se não tiver o php instalado baixe-o em: https://...

Exemplo:

```powershell
php -S localhost:8088 -t public/
```

---

### 7) Teste rápido (rota de saúde)

```bash
http://localhost:8088/ping
```

Resposta esperada:

```json
{
    "status": "success",
    "descStatus": "Microsserviço instalado com sucesso e funcionando normalmente.",
    "mensagem": "Bem vindo ao ecosistema ShipLink Tecnology"
}
```

Teste rápido no Postman:
{
    "nome":"Fulano de Tal",
    "assunto":"Quero ver se o POST está funcionando."
}
Cole a url do microserviço com o metodo POST. Copie e cole o esse json no body raw

---

### 8) Estrutura padrão do microsserviço

```bash
core-organization/
├─ public/
│  ├─ index.php                  # Entrada (Slim Framework)
│  └─ .htaccess                  # Regras de rewrite
├─ src/
│  ├─ Controllers/
│  │  └─ OrganizationController.php
│  ├─ Models/
│  │  └─ OrganizationModel.php
│  ├─ Services/
│  │  └─ OrganizationService.php
│  └─ Routes/
│     └─ routes.php              # Definição de rotas
├─ config/
│  └─ .env                       # Variáveis de ambiente
├─ tests/                        # Testes unitários
├─ composer.json                 # Já configurado com PSR-4 e dependências
├─ Dockerfile
├─ docker-compose.yml
├─ README.md
└─ .gitignore
```

---

### 9) Próximos passos

* Configure as rotas reais em `src/Routes/routes.php`.
* Implemente chamadas ao **Core-CRUD** via `src/Services/<MicroName>Service.php`.
* Documente endpoints e payloads no `README.md`.
* Configure **Dockerfile** e `docker-compose.yml` conforme seu ambiente.
* Toda comunicação com o Core-CRUD é via **HTTP/JSON** com:

```http
Authorization: Bearer <MASTER_KEY>
tokenIdentidade: MICRO_KEY
```

---

### ✔️ Pronto!

Seu micro está criado, padronizado e pronto para evoluir no ecossistema **ShipLink**.
Teste `/ping` antes de implementar suas features reais. 🚀
