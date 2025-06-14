# S&#8288;h&#8288;i&#8288;p&#8288;L&#8288;i&#8288;n&#8288;k – Arquitetura Global de Microserviços

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

* [c&#8288;o&#8288;r&#8288;e&#8288;-&#8288;c&#8288;r&#8288;u&#8288;d](https://github.com/shiplink-tech/core-crud)
* [t&#8288;m&#8288;s&#8288;-&#8288;t&#8288;i&#8288;c&#8288;k&#8288;e&#8288;t](https://github.com/shiplink-tech/tms-ticket)
* [&#8288;c&#8288;r&#8288;m&#8288;-&#8288;l&#8288;e&#8288;a&#8288;d&#8288;](https://github.com/shiplink-tech/crm-lead)

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
   * MongoDB (documentos d&#8288;i&#8288;n&#8288;â&#8288;m&#8288;i&#8288;c&#8288;o&#8288;s ou não relacionais)
   * Oracle, SQL Server (sistemas legados ou clientes específicos)

   Os drivers de conexão ficam desacoplados dentro de /drivers, e a Engine.php é responsável por r&#8288;o&#8288;t&#8288;e&#8288;ar dinamicamente conforme o ambiente e o projeto.

4. **Padrão único de resposta JSON**

   Todos os endpoints seguem o modelo:

   ```json
   {
     "status": 1,
     "descStatus": "OK",
     "response": { ... }
   }
   ```

5. **Versionamento por &#8288;p&#8288;a&#8288;t&#8288;h (v1, v2...)**

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
  * M&#8288;a&#8288;r&#8288;k&#8288;e&#8288;t&#8288;p&#8288;l&#8288;a&#8288;c&#8288;e B2B
  * RH, Fiscal, Pagamentos, Saúde, Robótica, IA...
* Produtos serão lançados sob domínios próprios (`airlogexpress.com.br`, etc), mantendo a arquitetura e APIs intactas
* Escalabilidade horizontal com balanceamento por DNS reverso e segmentação por microserviço

---

--- 

## 🧊 Estratégia de Armazenamento HOT / WARM / ICE

A arquitetura ShipLink adota uma abordagem moderna e escalável para gestão de dados, inspirada em grandes players globais:



|Camada |Objetivo                           |Frequência de uso          |Infraestrutura recomendada                             |
|-------|-----------------------------------|---------------------------|-------------------------------------------------------|
|H&#8288;O&#8288;T    |Dados recentes e críticos	        |Muito alta (tempo real)    |Instância dedicada (banco primário)                    |
|W&#8288;A&#8288;R&#8288;M   |Dados intermediários (+30 dias)    |M&#8288;é&#8288;d&#8288;i&#8288;a                      |Instância otimizada (menos recursos)                   |
|I&#8288;C&#8288;E    |Históricos e a&#8288;u&#8288;ditorias (+90 dias) |Baixa                      |S&#8288;t&#8288;o&#8288;r&#8288;a&#8288;g&#8288;e barato e alta retenção (ex: Azure Blob, S3)    |

Cada camada pode residir em bancos diferentes, com integração via API e controle transparente.

---

## 💡 Visão

> Criar a maior e mais confiável plataforma SaaS de gestão e automação de negócios do Brasil — e futuramente do mundo — com base em **microserviços inteligentes, APIs bem definidas, independência total de sistemas e uma arquitetura de referência global**.

---

Este documento é a **base oficial da arquitetura da ShipLink**. Toda nova decisão técnica deverá seguir este padrão ou ter justificativa clara e documentada como exceção.
