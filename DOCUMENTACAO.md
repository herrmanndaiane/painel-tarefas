# Documentação Técnica do Painel de Gestão

Esta documentação traz o detalhamento técnico da arquitetura, do fluxo de dados e das configurações do Painel de Gestão e Monitoramento, permitindo a manutenção futura da aplicação.

---

## 1. Visão Geral da Arquitetura

A aplicação é uma SPA (Single Page Application) estática desenvolvida em HTML5, Tailwind CSS e JavaScript puro (ES6+), hospedada no GitHub Pages. 

Para eliminar os limites de armazenamento do JSONBin e evitar a exposição de credenciais privadas no lado do cliente (browser), o painel utiliza uma arquitetura Serverless desacoplada:

[ Navegador / GitHub Pages ] ──(HTTP)──> [ Cloudflare Worker ] ──(GitHub API)──> [ dados.json no GitHub ]

---

## 2. Estrutura dos Componentes e Módulos

O sistema é dividido em 4 abas principais:

### Aba 1: Atividades e Cronograma
- Visão Macro: Lista de atividades de nível superior com responsáveis, prazos, prioridades, status e progresso percentual recalculado dinamicamente em tempo real.
- Visão Micro (Detalhada): Subtarefas/Etapas vinculadas a cada atividade macro com datas de início/fim e bloco de observações/alinhamentos.
- Edição Direta: Atualização do prazo direto na tabela principal.

### Aba 2: Domínios e Monitoramento CNPJ
- Consulta Automática RDAP/WHOIS: Busca automática de datas de expiração via proxy interno para domínios .br (Registro.br) e domínios internacionais (.com, .net, etc.).
- Filtros Combinados: Filtragem cruzada por Status de Vencimento (Críticos, Atenção, Em Dia) e Status de Migração do CNPJ (Não iniciado, Em andamento, Finalizado com sucesso).
- Importação em Lote: Caixa para inserção massiva de domínios (um por linha) com busca automática em lote.
- Re-checagem em Segundo Plano: Botão "Atualizar Datas" para sincronização com os servidores WHOIS oficiais.

### Aba 3: Matriz RACI
- Responsabilidades Corporativas: Mapeamento de atribuições por atividade (Responsible, Accountable, Consulted, Informed).
- Visualização Livre: Acessível para leitura pública por qualquer visitante (sem exigência de login).
- Edição Dinâmica: Adição/remoção de colunas (Papéis/Equipes) e linhas (Tarefas) dinamicamente pelo administrador.

### Aba 4: Notas de Reunião (Estilo Google Keep)
- Cartões Coloridos: Criação de notas com títulos, descrições formatadas e paleta de cores organizacionais.
- Edição Inline: Alteração de texto diretamente nos cartões de notas com salvamento ao perder o foco (onchange).
- Proteção de Leitura: Bloqueada para alteração em modo leitor público.

---

## 3. Modelo de Dados (dados.json)

A persistência do sistema é unificada em um único arquivo JSON no repositório GitHub. Abaixo está a estrutura completa do modelo de dados:

{
  "activities": [
    {
      "id": 1724000000000,
      "title": "Migração de Servidores PR/SC",
      "protocol": "#2026-89412",
      "owner": "Daiane Herrmann",
      "deadline": "2026-10-30",
      "status": "Em Andamento",
      "priority": "Alta",
      "notes": "Observações sobre o alinhamento da migração...",
      "subtasks": [
        {
          "id": 1724000000001,
          "title": "Levantamento de servidores virtuais",
          "owner": "Rafael",
          "start": "2026-08-03",
          "end": "2026-08-07",
          "status": "Concluída",
          "priority": "Média"
        }
      ]
    }
  ],
  "domains": [
    {
      "id": 1724000000002,
      "name": "minhaempresa.com.br",
      "expiration": "2027-05-15",
      "cnpjStatus": "Em andamento",
      "provider": "Registro.br",
      "notes": "Renovação automática no cartão"
    }
  ],
  "keepNotes": [
    {
      "id": 1724000000003,
      "title": "Alinhamento com Diretoria",
      "content": "Definição de prazos de entrega do projeto...",
      "color": "bg-amber-50 border-amber-200",
      "date": "20/10/2026 14:30"
    }
  ],
  "raci": {
    "roles": ["Redator", "Designer", "Programador", "Coordenador", "Clientes"],
    "tasks": ["Escrever Roteiro", "Criar Identidade Visual", "Fazer o Planejamento"],
    "matrix": {
      "0_0": "R",
      "0_1": "I",
      "0_3": "A"
    }
  }
}

---

## 4. Configuração da Camada de Segurança (Cloudflare Worker)

A integração com a API do GitHub e a validação de acesso administrativo são geridas pelo Cloudflare Worker icy-wildflower-ec24.

### Endpoints do Proxy Serverless
- GET /get-data: Baixa o arquivo dados.json do repositório para acesso público.
- POST /verify-auth: Valida a senha digitada no modal de login com o segredo salvo (Cabeçalho: X-Admin-Password).
- POST /save-data: Atualiza o arquivo dados.json no GitHub enviando o payload codificado em Base64 com suporte a UTF-8 e SHA.

### Variáveis de Ambiente Seguras (Cloudflare Secrets)
- GITHUB_USER: herrmanndaiane
- GITHUB_REPO: painel-tarefas
- GITHUB_TOKEN: Personal Access Token (Classic) com escopo repo.
- ADMIN_PASSWORD: Senha de acesso para habilitar o modo de edição.

---

## 5. Procedimento para Manutenção Futura

### Alterar a senha do Administrador:
1. Vá até o Cloudflare Worker em Settings -> Variables and Secrets.
2. Edite o segredo ADMIN_PASSWORD para a nova senha e clique em Deploy.

### Atualizar/Renovar o Token do GitHub:
1. No GitHub, vá para Settings -> Developer Settings -> Personal Access Tokens (classic).
2. Gere um novo token marcando a opção repo.
3. No Cloudflare Worker, acesse Settings -> Variables and Secrets.
4. Atualize o segredo GITHUB_TOKEN com o novo valor ghp_... e clique em Deploy.
