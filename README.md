# Pet Prime - Sistema de Gestão para Pet Shop e Clínica Veterinária

Este é o repositório do projeto **Pet Prime**, um sistema completo de gestão para Pet Shops e Clínicas Veterinárias com integração financeira ao gateway de pagamentos **Asaas**.

O projeto é dividido em duas partes principais:
1. **Backend (`/back/Pet-prime`)**: Desenvolvido em Python com FastAPI, SQLAlchemy, PostgreSQL/MySQL e Docker.
2. **Frontend (`/front/project-front/petshop`)**: Desenvolvido em Vue 3, TypeScript, Vite, Tailwind CSS e agora com a integração do **PrimeVue** (Layout Prime / Aura Preset).

---

## 🏗️ Arquitetura do Sistema

O projeto segue um padrão arquitetural baseado em camadas bem definidas e modelagem orientada a domínio (Domain-Driven Design simplificado):

- **Frontend (Vite + Vue 3 Client)**:
  - **Componentes SFC**: Interface de usuário reativa construída com Vue 3, estilizada com Tailwind CSS e componentes da biblioteca PrimeVue.
  - **Services / Axios API**: Módulo de comunicação com o backend FastAPI.
  - **Pinia Stores**: Gerenciamento de estado global para sessão e cache de dados.
- **Backend (FastAPI Server)**:
  - **Routers / Controllers**: Exposição dos endpoints REST e validação sintática via Schemas Pydantic.
  - **Service Layer**: Contém a lógica de orquestração de negócios, regras de transição de estado e chamadas a serviços externos (como Asaas).
  - **Domain Models / SQLAlchemy**: Representação das entidades do negócio contendo as regras intrínsecas e validações do domínio.
  - **Repository Layer**: Abstração de acesso a dados e persistência SQL.
  - **Database**: Banco de dados relacional MySQL rodando em container Docker.

### ⚠️ Gotcha e Detalhe de Design do Backend
* **Retorno de Tuplas nas Rotas**: Os routers do FastAPI retornam dados no formato `return objeto, 200` ou `return objeto, 201`. Para suportar esse padrão de retorno, o Axios no frontend possui um interceptor específico em `src/services/api.ts` que desempacota o array `[objeto, status_code]` quando o backend retorna um array com 2 elementos, garantindo compatibilidade transparente para a interface.

---

## 🛠️ Tecnologias Utilizadas

### Backend
- **FastAPI**: Framework web moderno, rápido (alta performance) para construção de APIs.
- **SQLAlchemy (v2.0)**: ORM poderoso para mapeamento objeto-relacional.
- **Alembic**: Ferramenta para gerenciamento de migrações de banco de dados.
- **Docker & Docker Compose**: Orquestração dos containers de aplicação (`app-pet` FastAPI) e banco de dados (`db-pet` MySQL 8.0).
- **Asaas SDK/API**: Integração financeira para geração de cobranças (Pix, Boleto, Cartão de Crédito) e recebimento via webhooks.

### Frontend
- **Vue 3**: Framework progressivo em JavaScript (usando `<script setup>` e Composition API).
- **TypeScript**: Tipagem estática para robustez no desenvolvimento.
- **Vite**: Build tool extremamente rápida para desenvolvimento frontend.
- **Tailwind CSS (v3)**: Framework CSS utilitário para design personalizado.
- **PrimeVue (v4)**: Biblioteca de componentes de UI premium com o preset **Aura** integrado ao Tailwind CSS por meio do plugin `tailwindcss-primeui` e CSS Layers.

---

## 🗄️ Modelagem de Banco de Dados e Regras de Negócio

As entidades estão mapeadas em `app/models/models.py`. Seguem as principais entidades e suas regras de negócio associadas:

1. **Usuario**:
   - Campos: `id`, `nome`, `email`, `senha`, `ativo`, `admin`.
   - Regras: Validação rigorosa de senha (no mínimo uma letra maiúscula e dois números) e domínio de email.

2. **Customer (Cliente)**:
   - Campos: `id`, `asaas_customer_id`, `cpf`, `name`, `phone`, `email`, `status` (Ativo/Inativo), `description`.
   - Regras: Validação de CPF brasileiro e verificação estrutural de telefone (padrão celular DDD + 9). Integração direta com Asaas para sincronizar cadastros de clientes.

3. **Pet**:
   - Campos: `id`, `name`, `specie`, `race`, `age`, `status` (Ativo/Inativo), `customer_id`.
   - Regras: Validação de espécies permitidas e raças válidas (definidas no enum do sistema). Desativação automática do Pet caso o Cliente seja desativado.

4. **Service (Serviço)**:
   - Campos: `id`, `name`, `price`, `description`.
   - Regras: Preço deve ser um valor estritamente positivo.

5. **Appointment (Agendamento)**:
   - Campos: `id`, `date`, `time`, `status` (Pendente, Agendado, Concluído, Cancelado), `pet_id`.
   - Regras:
     - Não são permitidos agendamentos nos finais de semana.
     - Não são permitidos agendamentos em feriados nacionais (calculados via biblioteca `holidays`).
     - Horários de agendamento devem respeitar o expediente de atendimento.
     - Não são permitidas datas ou horários retroativos.
     - Somente agendamentos nos estados *Pendente* ou *Agendado* podem ser cancelados.

6. **Payment & Charge**:
   - Gerenciamento de cobranças vinculadas a agendamentos ou vendas, sincronizando status (Pending, Confirmed, Received, Overdue, Canceled) via webhook do Asaas.

---

## 🚀 Como Executar o Projeto

### Pré-requisitos
- Docker & Docker Compose
- Node.js (v18+) e npm

### Executando o Backend
1. Navegue até o diretório do backend:
   ```bash
   cd back/Pet-prime
   ```
2. Inicialize os containers via Docker Compose:
   ```bash
   docker-compose up -d
   ```
3. O banco de dados MySQL e o servidor FastAPI estarão ativos.
   - O backend rodará na porta `1000` (e.g. `http://localhost:1000`).
   - A documentação interativa Swagger estará disponível em `http://localhost:1000/docs`.

### Executando o Frontend
1. Navegue até o diretório do frontend:
   ```bash
   cd front/project-front/petshop
   ```
2. Instale as dependências:
   ```bash
   npm install
   ```
3. Execute o servidor de desenvolvimento:
   ```bash
   npm run dev
   ```
4. Acesse a aplicação na URL fornecida pelo Vite (e.g. `http://localhost:5173`).
