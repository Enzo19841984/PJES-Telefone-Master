# Sistema Inteligente de Gestão de Telefones do PJES

Sistema Web App do **Poder Judiciário do Estado do Espírito Santo (TJES)** para gestão e consulta da base de telefones (comarcas, setores e contatos), construído sobre **Google Apps Script + Google Sheets**.

## Funcionalidades

- **Consulta pública** de telefones por microrregião, comarca e setor (com mapa do ES, links `tel:`, `wa.me` e `mailto:`)
- **Dashboard** com totais de contatos, setores e tipos
- **Gestão administrativa** (CRUD) da base de telefones, com histórico de alterações
- **Solicitações de acesso**: formulário público na aba *Formulário de Acesso* (nome, comarca, e-mail, perfil solicitado e justificativa) gravado na aba `SOLICITACOES_ACESSO` com status `PENDENTE`; o Gestor do Sistema aprova ou rejeita na aba *Solicitações de Acesso*
- **Notificações por e-mail** (fila em `EMAILS_PENDENTES` processada por gatilho)
- **Perfis**: `GESTOR_SISTEMA`, `GESTOR_CONTEUDO` e `USUARIO_CONSULTA`
- Modo escuro e impressão/PDF da consulta

## Estrutura do projeto

| Arquivo | Finalidade |
|---|---|
| `Main.gs` | Roteamento do Web App (público × administrativo) |
| `01_Config.gs` | Configurações centrais (URLs, abas, perfis, limites) |
| `02_Utils.gs` | Utilitários (normalização, datas, objetos) |
| `03_Database.gs` | Acesso à planilha vinculada |
| `04_TelefoneRepository.gs` | CRUD da base TELEFONES |
| `05_HistoryService.gs` | Histórico de alterações |
| `06_ValidationService.gs` | Validação de telefones |
| `07_IdService.gs` | Geração de IDs |
| `08_CacheService.gs` | Cache de listagens |
| `09_LogService.gs` | Log de erros/operações |
| `10_AuthService.gs` | Autenticação e permissões |
| `11_API.gs` | Funções expostas ao frontend (`google.script.run`) |
| `12_Install.gs` | Instalação/migração das abas |
| `*.html` | Telas e scripts do frontend |

## Implantação

1. Crie um projeto no [Google Apps Script](https://script.google.com) vinculado à planilha do sistema (Extensões → Apps Script).
2. Copie todos os arquivos `.gs` (sem a extensão `.txt` do repositório) e `.html` para o projeto.
3. No editor, execute na ordem:
   - `registrarPlanilhaVinculada()` — vincula a planilha ativa;
   - `instalarSistema()` — cria as abas e cabeçalhos (inclusive `SOLICITACOES_ACESSO` com a coluna `COMARCA`);
   - `autorizar()` — concede as autorizações de planilha e e-mail;
   - `instalarTriggerEmails()` — instala o gatilho que processa a fila de e-mails (a cada minuto).
4. Ajuste `CONFIG.WEB_APP.URL_PUBLICA` e `URL_ADMIN` para as URLs reais das implantações.
5. Publique: **Implantar → Nova implantação → Aplicativo da web**, com:
   - *Executar como*: **Eu (conta do deployer)**;
   - *Quem pode acessar*: **Qualquer pessoa** (público) — a área administrativa é protegida por domínio institucional (`tjes.jus.br`) na URL administrativa.
6. Em instalações antigas, a coluna `COMARCA` em `SOLICITACOES_ACESSO` pode ser adicionada executando apenas `atualizarAbaSolicitacoesAcesso()` (sem precisar reinstalar tudo).

## Abas da planilha

- `TELEFONES` — ID, MICRORREGIAO, COMARCA, SETOR, TIPO, TELEFONE, RAMAL, WHATSAPP, E-MAIL, ENDERECO, STATUS, OBSERVACAO, DATA_CRIACAO, DATA_ATUALIZACAO
- `USUARIOS` — EMAIL, NOME, PERFIL, ATIVO
- `SOLICITACOES_ACESSO` — ID, EMAIL, NOME, COMARCA, PERFIL_SOLICITADO, JUSTIFICATIVA, STATUS, DATA_SOLICITACAO, APROVADOR, DATA_APROVACAO
- `CONFIGURACAO` — CHAVE, VALOR (ex.: `EMAIL_GESTOR`)
- `HISTORICO`, `LOG`, `EMAILS_PENDENTES`

## Segurança

- O Web App público permite apenas **consulta** (`VISUALIZAR`/`PESQUISAR`).
- Edição, exclusão, histórico e solicitações exigem login com e-mail institucional `@tjes.jus.br` e perfil adequado.
- Todas as escritas usam `LockService` para evitar concorrência.
