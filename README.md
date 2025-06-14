# ShipLink – Arquitetura Global de Microserviços

A **ShipLink** é a empresa de tecnologia responsável pela criação e manutenção de uma arquitetura de microserviços **escalável, modular e desacoplada**, voltada ao desenvolvimento de soluções SaaS para **negócios de qualquer natureza** — desde logística e gestão empresarial até setores como saúde, RH, e-commerce, marketplace, pagamentos e muito mais.

Nosso objetivo é consolidar a ShipLink como uma **empresa referência em tecnologia SaaS**, com produtos robustos e independentes, prontos para escalar com segurança, performance e clareza arquitetural.

---

## 📁 Estrutura de Diretórios (máquina local)

```bash
C:\Projetos\
└── ShipLink\
    ├── Core\               ← Microsserviços reutilizáveis e centrais (ex: CRUD, Auth)
    │   ├── core-crud
    │   └── core-user
    ├── TMS\                ← Sistema de Gestão de Transporte
    │   ├── tms-dashboard
    │   ├── tms-ticket
    │   └── ...
    ├── CRM\                ← Gestão de Relacionamento com Clientes e Vendas
    │   ├── crm-lead
    │   └── ...
    ├── CTE\                ← Serviços fiscais de emissão de CTe
    │   ├── cte-issue
    │   ├── cte-cancel
    │   └── ...
    └── ... (ERP, WMS, etc)
```

---

## 🌐 Organização de Repositórios (GitHub)

Cada microserviço é versionado **em um repositório separado**, com nomes padronizados:

```
core-*
tms-*
crm-*
cte-*
wms-*
```

Exemplos:

* [cU+2060oU+2060rU+2060e-crud](https://github.com/shiplink-tech/core-crud)
* [tms-ticket](https://github.com/shiplink-tech/tms-ticket)
* [crm-lead](https://github.com/shiplink-tech/crm-lead)

---

## 🧠 Princípios Arquiteturais

1. **100% HTTP (REST)**
   Toda comunicação entre microserviços ocorre via **API externa HTTP**, inclusive durante o desenvolvimento. Nada é acoplado localmente.

2. **Banco por microserviço**
   Cada serviço possui seu **banco de dados exclusivo**, com tabelas específicas. Isolamento total por responsabilidade.

3. **Banco de dados agnóstico (multi-driver)**
   Cada microserviço pode operar com o banco de dados mais apropriado para seu contexto:
   * MySQL (CRUDs e aplicações transacionais)
   * PostgreSQL (consultas analíticas e geográficas)
   * MongoDB (documentos dinâmicos ou não relacionais)
   * Oracle, SQL Server (sistemas legados ou clientes específicos)
   Os drivers de conexão ficam desacoplados dentro de /drivers, e a Engine.php é responsável por rotear dinamicamente conforme o ambiente e o projeto.

4. **Padrão único de resposta JSON**
   Todos os endpoints seguem o modelo:

   ```json
   {
     "status": 1,
     "descStatus": "OK",
     "response": { ... }
   }
   ```

5. **Versionamento por path (v1, v2...)**
   URLs versionadas, permitindo múltiplas versões simultâneas:

   ```
   https://api.shiplink.com.br/v1/tms-ticket/...
   ```

6. **Autenticação obrigatória**
   Toda API exige **token JWT ou Bearer**. Mesmo ambientes sandbox requerem autenticação.

7. **Separação entre produtos**
   Cada linha de produto (TMS, CRM, WMS, CTE, etc.) é **isolada em microserviços independentes**.

8. **Subdomínios por produto**
   Padrão DNS separado por produto e função:

   * `api.tms.com.br`
   * `painel.tms.com.br`
   * `app.tms.com.br`

   E o mesmo para `crm`, `cte`, etc. Permitindo escalabilidade, cache e balanceamento separados.

9. **Repositórios separados (no monorepo!)**
   Cada microserviço tem **repositório próprio**, com CI/CD, documentação, controle de acesso e versionamento independentes.

10. **Padrões técnicos consolidados**

   * PSR-4 obrigatório
   * `src/` como base dos códigos
   * Estrutura clara por domínio de negócio
   * Pastas `drivers/`, `Engine.php`, `Router.php` ou equivalentes
   * Contratos de comunicação unificados

11. **Documentação viva e distribuída**

    * Cada microserviço possui seu `README.md` técnico interno
    * Um **portal central de documentação** será construído com Swagger ou Redoc
    * Todos os serviços seguem o mesmo contrato de resposta

---

## 🧱 Fundamentos de Expansão

* Estrutura preparada para produtos futuros:

  * ERP
  * WMS
  * App Mobile
  * Marketplace B2B
  * RH, Fiscal, Pagamentos, Saúde, Robótica, IA...
* Produtos serão lançados sob domínios próprios (`airlogexpress.com.br`, etc), mantendo a arquitetura e APIs intactas
* Escalabilidade horizontal com balanceamento por DNS reverso e segmentação por microserviço

---

--- 

## 🧊 Estratégia de Armazenamento HOT / WARM / ICE
A arquitetura ShipLink adota uma abordagem moderna e escalável para gestão de dados, inspirada em grandes players globais:

|   Camada  |   Objetivo    |   Frequência de uso   |   Infraestrutura recomendada  |
| --------------------------------------------------------------------------------- |
|   HOT |   Dados recentes e críticos	|    Muito alta (tempo real)    |   Instância dedicada (banco primário) |
|   WARM    |	Dados intermediários (+30 dias) |   Média   |   Instância otimizada (menos recursos)    |
|   ICE |   Históricos e auditorias (+90 dias)  |   Baixa   |   Storage barato e alta retenção (ex: Azure Blob, S3) |

Cada camada pode residir em bancos diferentes, com integração via API e controle transparente.

---

## 💡 Visão

> Criar a maior e mais confiável plataforma SaaS de gestão e automação de negócios do Brasil — e futuramente do mundo — com base em **microserviços inteligentes, APIs bem definidas, independência total de sistemas e uma arquitetura de referência global**.

---

Este documento é a **base oficial da arquitetura da ShipLink**. Toda nova decisão técnica deverá seguir este padrão ou ter justificativa clara e documentada como exceção.
