# Escopo Detalhado - Super Checklist

## 1. Módulo de Autenticação

### 1.1 Login de Colaboradores
- **Endpoint**: `POST /api/auth/login`
- **Parâmetros**: email, senha
- **Resposta**: JWT token, dados do usuário, perfil
- **Perfis**: 
  - Admin (acesso total)
  - Gestor (acesso gerencial)
  - Operador (acesso operacional)
  - Recepcionista (acesso limitado)

### 1.2 Logout
- **Endpoint**: `POST /api/auth/logout`
- **Autenticação**: JWT token

### 1.3 Recuperação de Senha
- **Endpoint**: `POST /api/auth/forgot-password`
- **Reset**: `POST /api/auth/reset-password`

---

## 2. Módulo de Checklist por Turno

### 2.1 Criação de Checklist
- **Endpoint**: `POST /api/checklist`
- **Dados**:
  - Turno (MANHA, TARDE, NOITE)
  - Data
  - Colaborador responsável
  - Items (lista de verificação)
  - Status

### 2.2 Atualização de Checklist
- **Endpoint**: `PUT /api/checklist/{id}`
- **Dados**: Status, itens concluídos, observações

### 2.3 Listar Checklists
- **Endpoint**: `GET /api/checklist?turno=MANHA&data=2026-06-07`
- **Filtros**: Turno, data, status, colaborador

---

## 3. Módulo de Rondas nos Andares

### 3.1 Ronda Manhã (10h)
- **Endpoint**: `POST /api/round/morning`
- **Dados**:
  - Turno: MANHA
  - Hora agendada: 10:00
  - Responsável
  - Andares a verificar
  - Checklist de verificação

- **Checklist de Verificação**:
  - Limpeza geral dos andares
  - Estado dos corredores
  - Presença de moadores
  - Problemas identificados
  - Foto evidência

- **Status**: Pendente → Em Progresso → Concluído

### 3.2 Ronda Tarde (17h)
- **Endpoint**: `POST /api/round/afternoon`
- **Dados**:
  - Turno: TARDE
  - Hora agendada: 17:00
  - Responsável
  - Andares a verificar
  - Checklist de verificação
  - **Designação de apartamentos para dia seguinte**

- **Checklist de Verificação**:
  - Inspeção completa de cada unidade
  - Verificação de manutenção necessária
  - Status dos moradores
  - Problemas reportados

- **Designação de Apartamentos**:
  - Apartamento
  - Tipo de serviço (limpeza, manutenção, etc.)
  - Responsável designado
  - Data/hora agendada
  - Prioridade

- **Status**: Pendente → Em Progresso → Concluído

### 3.3 Listar Rondas
- **Endpoint**: `GET /api/round?turno=MANHA&data=2026-06-07`
- **Filtros**: Turno, data, status, responsável

### 3.4 Detalhes da Ronda
- **Endpoint**: `GET /api/round/{id}`
- **Resposta**: Informações completas, fotos, observações

---

## 4. Módulo de Limpeza de Cadex

### 4.1 Registro de Limpeza
- **Endpoint**: `POST /api/cadex-cleaning`
- **Dados**:
  - Data/hora início
  - Responsável
  - Itens para limpar (checklist)
  - Observações
  - Status: Pendente, Em Progresso, Concluído

### 4.2 Foto de Antes e Depois
- **Endpoint**: `POST /api/cadex-cleaning/{id}/photos`
- **Dados**: 
  - Foto antes (base64 ou upload)
  - Foto depois (base64 ou upload)
  - Descrição

### 4.3 Atualizar Limpeza
- **Endpoint**: `PUT /api/cadex-cleaning/{id}`
- **Dados**: Status, data/hora fim, observações

### 4.4 Listar Limpezas
- **Endpoint**: `GET /api/cadex-cleaning?data=2026-06-07`
- **Filtros**: Data, status, responsável

---

## 5. Módulo de Designação de Apartamentos

### 5.1 Designar Apartamento
- **Endpoint**: `POST /api/apartment-designation`
- **Dados**:
  - Apartamento
  - Tipo de serviço
  - Responsável
  - Data/hora agendada
  - Prioridade (alta, média, baixa)
  - Descrição do trabalho
  - Origem: Ronda tarde / Manual

### 5.2 Atualizar Designação
- **Endpoint**: `PUT /api/apartment-designation/{id}`
- **Dados**: Status (Pendente, Em Progresso, Concluído), observações

### 5.3 Listar Designações
- **Endpoint**: `GET /api/apartment-designation?data=2026-06-07`
- **Filtros**: Data, status, responsável, tipo de serviço

### 5.4 Designações do Próximo Dia
- **Endpoint**: `GET /api/apartment-designation/next-day`
- **Resposta**: Lista de apartamentos designados para o dia seguinte

---

## 6. Módulo de Acompanhamento AskSuite

### 6.1 Sincronizar com AskSuite
- **Endpoint**: `POST /api/asksuite/sync`
- **Dados**: Token de integração
- **Funcionalidade**: Buscar tickets abertos, em andamento e concluídos

### 6.2 Registrar Acompanhamento
- **Endpoint**: `POST /api/asksuite-tracking`
- **Dados**:
  - Turno (MANHA, TARDE, NOITE)
  - Data
  - Tickets verificados
  - Status de cada ticket
  - Observações
  - Responsável

- **Checklist de Acompanhamento**:
  - ✓ Tickets abertos
  - ✓ Tickets em atendimento
  - ✓ Tickets aguardando resposta
  - ✓ Tickets concluídos
  - ✓ Tickets com problemas

### 6.3 Atualizar Status de Ticket
- **Endpoint**: `PUT /api/asksuite-tracking/{id}/ticket/{ticketId}`
- **Dados**: Novo status, observações

### 6.4 Listar Acompanhamentos
- **Endpoint**: `GET /api/asksuite-tracking?turno=MANHA&data=2026-06-07`
- **Filtros**: Turno, data, responsável

### 6.5 Relatório de Tickets por Turno
- **Endpoint**: `GET /api/asksuite-tracking/report?data=2026-06-07`
- **Dados**: Resumo de tickets por turno, tickets pendentes, críticos

---

## 7. Módulo de Passagem de Plantão

### 7.1 Criar Passagem de Plantão
- **Endpoint**: `POST /api/handover`
- **Dados**:
  - Turno anterior
  - Turno atual
  - Data
  - Responsável anterior
  - Responsável atual
  - Observações gerais
  - Problemas pendentes
  - Apartamentos designados para o dia
  - Status do AskSuite
  - Prioridades do turno

### 7.2 Atualizar Passagem
- **Endpoint**: `PUT /api/handover/{id}`
- **Dados**: Confirmação, assinatura digital

### 7.3 Listar Passagens
- **Endpoint**: `GET /api/handover?data=2026-06-07`
- **Filtros**: Data, turno, responsável

### 7.4 Histórico de Plantões
- **Endpoint**: `GET /api/handover/history?dias=7`
- **Resposta**: Últimas passagens de plantão

---

## 8. Módulo de Ocorrências

### 8.1 Registrar Ocorrência
- **Endpoint**: `POST /api/occurrence`
- **Dados**:
  - Tipo (problema, sugestão, elogio)
  - Categoria
  - Prioridade (baixa, média, alta, crítica)
  - Descrição
  - Local (andar, apartamento)
  - Responsável
  - Data/hora
  - Origem: Ronda, Checklist, AskSuite, Manual

### 8.2 Atualizar Ocorrência
- **Endpoint**: `PUT /api/occurrence/{id}`
- **Dados**: Status, resolução, data de resolução

### 8.3 Listar Ocorrências
- **Endpoint**: `GET /api/occurrence?status=aberta`
- **Filtros**: Status, categoria, prioridade, data, responsável

---

## 9. Módulo de Controle de Caixa

### 9.1 Abertura de Caixa
- **Endpoint**: `POST /api/cash-control/open`
- **Dados**:
  - Data/hora
  - Responsável
  - Valor inicial
  - Observações

### 9.2 Movimentação de Caixa
- **Endpoint**: `POST /api/cash-control/movement`
- **Dados**:
  - Tipo (entrada, saída)
  - Valor
  - Descrição
  - Responsável

### 9.3 Fechamento de Caixa
- **Endpoint**: `POST /api/cash-control/close`
- **Dados**:
  - Valor final
  - Observações
  - Reconciliação

### 9.4 Histórico de Caixa
- **Endpoint**: `GET /api/cash-control/history?data=2026-06-07`
- **Resposta**: Movimentações do dia

---

## 10. Módulo de Dashboard Gerencial

### 10.1 Dashboard Principal
- **Endpoint**: `GET /api/dashboard`
- **Dados**:
  - Resumo do dia
  - Checklists concluídos/pendentes
  - Rondas realizadas
  - Ocorrências abertas
  - Tickets AskSuite pendentes
  - Status de caixa

### 10.2 Filtros do Dashboard
- Período (dia, semana, mês)
- Turno
- Responsável
- Status

### 10.3 Gráficos e Indicadores
- Taxa de conclusão de checklists
- Rondas por turno
- Ocorrências por categoria
- Tickets AskSuite por status
- Distribuição de carga de trabalho

---

## 11. Módulo de Exportação Excel

### 11.1 Exportar Checklist
- **Endpoint**: `GET /api/export/checklist?data=2026-06-07`
- **Formato**: Excel (.xlsx)
- **Campos**: Data, turno, responsável, itens, status

### 11.2 Exportar Rondas
- **Endpoint**: `GET /api/export/round?data=2026-06-07`
- **Formato**: Excel (.xlsx)
- **Campos**: Data, turno, hora, responsável, andares, problemas

### 11.3 Exportar Designações
- **Endpoint**: `GET /api/export/apartment-designation?data=2026-06-07`
- **Formato**: Excel (.xlsx)
- **Campos**: Apartamento, tipo, responsável, data agendada, status

### 11.4 Exportar AskSuite
- **Endpoint**: `GET /api/export/asksuite-tracking?data=2026-06-07`
- **Formato**: Excel (.xlsx)
- **Campos**: Turno, tickets, status, observações

### 11.5 Exportar Relatório Completo
- **Endpoint**: `GET /api/export/full-report?inicio=2026-06-01&fim=2026-06-07`
- **Formato**: Excel (.xlsx)
- **Contém**: Todos os dados do período

---

## 12. Módulo de Auditoria

### 12.1 Log de Ações
- **Endpoint**: `GET /api/audit/log`
- **Dados registrados**:
  - Ação realizada
  - Usuário
  - Data/hora
  - IP
  - Tipo de modificação (CREATE, UPDATE, DELETE)
  - Dados anteriores e novos

### 12.2 Rastreamento de Modificações
- **Endpoint**: `GET /api/audit/trail/{entityType}/{entityId}`
- **Resposta**: Histórico completo de modificações

### 12.3 Auditoria por Usuário
- **Endpoint**: `GET /api/audit/user/{userId}?periodo=7`
- **Resposta**: Ações realizadas pelo usuário

### 12.4 Relatório de Auditoria
- **Endpoint**: `GET /api/audit/report?inicio=2026-06-01&fim=2026-06-07`
- **Resposta**: Relatório completo de auditoria

---

## 13. Modelo de Dados

Ver documento [MODELOS_DADOS.md](MODELOS_DADOS.md)

---

## 14. Endpoints Resumidos

```
AUTH
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/forgot-password
POST   /api/auth/reset-password

CHECKLIST
POST   /api/checklist
GET    /api/checklist
GET    /api/checklist/{id}
PUT    /api/checklist/{id}

RONDAS
POST   /api/round/morning
POST   /api/round/afternoon
GET    /api/round
GET    /api/round/{id}
PUT    /api/round/{id}

CADEX
POST   /api/cadex-cleaning
GET    /api/cadex-cleaning
GET    /api/cadex-cleaning/{id}
PUT    /api/cadex-cleaning/{id}
POST   /api/cadex-cleaning/{id}/photos

APARTAMENTOS
POST   /api/apartment-designation
GET    /api/apartment-designation
GET    /api/apartment-designation/{id}
PUT    /api/apartment-designation/{id}
GET    /api/apartment-designation/next-day

ASKSUITE
POST   /api/asksuite/sync
POST   /api/asksuite-tracking
GET    /api/asksuite-tracking
GET    /api/asksuite-tracking/{id}
PUT    /api/asksuite-tracking/{id}/ticket/{ticketId}
GET    /api/asksuite-tracking/report

PLANTÃO
POST   /api/handover
GET    /api/handover
GET    /api/handover/{id}
PUT    /api/handover/{id}
GET    /api/handover/history

OCORRÊNCIAS
POST   /api/occurrence
GET    /api/occurrence
GET    /api/occurrence/{id}
PUT    /api/occurrence/{id}

CAIXA
POST   /api/cash-control/open
POST   /api/cash-control/movement
POST   /api/cash-control/close
GET    /api/cash-control/history

DASHBOARD
GET    /api/dashboard
GET    /api/dashboard/filters

EXPORTAÇÃO
GET    /api/export/checklist
GET    /api/export/round
GET    /api/export/apartment-designation
GET    /api/export/asksuite-tracking
GET    /api/export/full-report

AUDITORIA
GET    /api/audit/log
GET    /api/audit/trail/{entityType}/{entityId}
GET    /api/audit/user/{userId}
GET    /api/audit/report
```

---

## 15. Funcionalidades Especiais

### 15.1 Notificações
- Email para tarefas designadas
- Alertas de AskSuite críticos
- Lembretes de rondas

### 15.2 Relatórios Automáticos
- Relatório diário enviado ao final do turno noturno
- Relatório semanal na segunda-feira
- Relatório mensal no primeiro dia do mês

### 15.3 Backup de Dados
- Backup automático diário
- Exportação semanal de dados

---

**Versão**: 1.0
**Data**: 2026-06-07
**Autor**: @rheynansantos-beep
