1. Visão geral do sistema
O PraCima é um sistema de gestão de tarefas e projetos corporativos, voltado para equipes internas de empresas que precisam planejar, atribuir, executar e acompanhar trabalho no dia a dia — com uma camada adicional de gestão estratégica (Balanced Scorecard) e engajamento (gamificação).

Para quem é: equipes internas de empresas (o próprio material de produto cita como exemplo real "Maria Pitanga Açaiteria"), com dois perfis principais de uso: gestores, que montam listas/quadros de trabalho e definem indicadores estratégicos (BSC); e colaboradores, que recebem tarefas, etapas e prazos e executam o trabalho do dia a dia.
Contexto de uso: ferramenta de trabalho, usada durante o expediente, lado a lado com outras ferramentas corporativas, tanto em desktop quanto em mobile.
Personalidade de marca: profissional, séria e corporativa — adequada para gestão de equipes e indicadores de desempenho, com identidade visual roxa (#540562/#6B23FF) usada com intenção (não decorativa).
Ponto de entrada principal do produto: o modal de "Cadastro de Tarefa" — nome, etapas, responsável, convidados, prazo, recorrência e vínculo com indicadores BSC. A filosofia de sucesso do produto é permitir criar uma tarefa completa e bem configurada rapidamente, sem fricção.
Arquitetura: monorepo com backend (NestJS + PostgreSQL + S3, deployado no Railway) e frontend (React 19 + Vite + TailwindCSS + PrimeReact), com suíte de testes end-to-end via Playwright.
2. Módulos e funcionalidades (linguagem de negócio)
Gestão de tarefas e etapas (tarefas)
Núcleo do sistema: permite criar, atribuir e acompanhar tarefas com responsável, convidados, prazos e etapas (subtarefas com agenda/horário próprios). Suporta recorrência (tarefas que se repetem automaticamente — rotinas), cópia de tarefas entre listas, e restrições de agenda/horário de bloqueio nas etapas para controlar quando um passo de trabalho pode ser executado.

Listas/quadros de trabalho (listas)
Organiza tarefas em quadros (visão Kanban), agrupando trabalho por projeto, time ou contexto. Cada lista pode ter regras próprias (ex.: vínculo obrigatório com uma empresa) e visão de calendário/planejamento.

Balanced Scorecard — BSC (bsc)
Módulo de gestão estratégica: permite cadastrar indicadores de desempenho organizados por dimensões (financeira, processos, clientes, aprendizado, etc.), acompanhar metas e mensuração ao longo do tempo, e vincular tarefas do dia a dia diretamente aos indicadores estratégicos — conectando execução operacional a objetivos de negócio.

Gamificação (gamificacao)
Camada de engajamento que premia colaboradores por cumprir tarefas, prazos e rotinas, com pontuação e rankings — visa aumentar a adesão ao processo e reconhecer o desempenho individual/de equipe.

Relatórios (relatorios)
Consolida dados operacionais (tarefas, prazos, produtividade) e estratégicos (indicadores BSC) em relatórios gerenciais, dando visibilidade a gestores sobre o andamento do trabalho e o cumprimento de metas.

Notificações via WhatsApp (whatsapp)
Envia lembretes e alertas diretamente no WhatsApp dos usuários (ex.: prazos se aproximando, tarefas atribuídas), usando a API oficial da Meta. Por padrão os envios reais ficam desligados em ambientes de desenvolvimento/teste (todo envio é logado como no-op), evitando disparos indevidos fora de produção. Inclui configuração de horário comercial e lista de permissão (whitelist) de destinatários.

Fila de processamento assíncrono (fila)
Infraestrutura de fila de jobs baseada em banco de dados (tabela fila), segura para múltiplas instâncias rodando em paralelo (usa SELECT FOR UPDATE SKIP LOCKED do PostgreSQL para garantir que cada job seja processado uma única vez). Suporta agendamento de disponibilidade, novas tentativas com backoff e recuperação de jobs travados — é a espinha dorsal técnica que sustenta o envio confiável das notificações do WhatsApp e pode ser reaproveitada para outros processos assíncronos futuros.

Usuários e permissões (usuarios, auth)
Cadastro de usuários com autenticação segura (senhas com hash bcrypt), perfis de acesso diferenciados (permissões por papel/função) e contadores de acesso — permitindo controlar quem pode ver e editar o quê dentro do sistema.

Empresas e unidades (empresas, unidades)
Suporte a múltiplas empresas/filiais dentro do mesmo sistema, permitindo segmentar listas, tarefas e usuários por organização ou unidade de negócio — relevante para operações com mais de uma unidade/franquia.

Auditoria (auditoria)
Registro de ações realizadas no sistema, dando rastreabilidade sobre quem fez o quê e quando — importante para governança e prestação de contas em ambientes corporativos.

Filtros personalizados (filtros, filtros-geral)
Permite ao usuário salvar buscas/filtros personalizados (por status, responsável, lista, indicador, período, etc.) vinculados ao seu perfil, acessíveis em qualquer dispositivo/sessão via um menu dedicado "Filtros salvos" — elimina a necessidade de reconfigurar filtros toda vez que muda de computador ou navegador.

Armazenamento de arquivos (storage)
Upload e download de arquivos anexos (ex.: documentos de apoio a tarefas) usando um bucket compatível com S3 hospedado no Railway.

E-mail (email)
Disparo de comunicações por e-mail (ex.: recuperação de senha, notificações), complementando o canal de WhatsApp.

3. Funcionalidades de negócio identificadas nas especificações (specs)
Tarefas rotineiras (recorrência): permite configurar tarefas que se repetem automaticamente em uma cadência definida, eliminando recriação manual de trabalho repetitivo (rotinas operacionais, checklists periódicos).
Atribuição de tarefas a convidados: permite incluir pessoas além do responsável principal em uma tarefa, viabilizando colaboração e visibilidade compartilhada.
Cópia de tarefas entre listas: agiliza a reutilização de tarefas/modelos de trabalho entre diferentes quadros/projetos.
Agendamento de etapas (task-step-scheduling): cada etapa de uma tarefa pode ter horário/janela de execução e regras de bloqueio, dando controle fino sobre quando cada passo do processo pode/deve ser feito.
Permissões de usuário: controle de acesso por perfil, garantindo que cada colaborador veja e edite apenas o que é pertinente ao seu papel.
Vínculo obrigatório lista-empresa: reforça a organização multiempresa, evitando listas "soltas" sem dono organizacional.
Filtros salvos por contexto de navegação (Planejado, Atribuído a Mim, Minha Semana, Importantes, Rotinas, Lista específica): cada usuário pode salvar, nomear e reaplicar filtros de busca complexos com um clique, agora persistidos em banco (migrado de localStorage) para uso em qualquer dispositivo.
Página de política de privacidade: conformidade/transparência sobre uso de dados dos usuários.
4. Frontend — principais telas (visão do usuário final)
Menu lateral: Planejado, Atribuído a Mim, Minha Semana, Importantes, Rotinas, Perfil (e, mais recentemente, Filtros Salvos).
Páginas: Login, Cadastro, Cadastro de Tarefas (com recorrência), Etapas, Kanban, Calendário, Listas, Gamificação, Balanced Scorecard, Relatórios, Admin.
Padrão de UX diferenciado: cada tela tem uma versão desktop e uma versão mobile como componentes distintos (não é responsividade só via CSS) — garante experiência otimizada em qualquer dispositivo.
Três temas visuais: light, dark (amber) e purple, todos com o mesmo padrão de contraste e acessibilidade (WCAG AA).
Gráficos e exportação: uso de Recharts para visualização de dados (relatórios, BSC, gamificação) e suporte a exportação em Excel (xlsx).
5. Diferenciais técnicos relevantes para o marketing
Fila de jobs própria, resiliente e multi-instância: garante que notificações e processos assíncronos sejam entregues de forma confiável (at-least-once), com nova tentativa automática em caso de falha — sem depender de serviços de fila externos pagos.
Notificações via WhatsApp nativas, com salvaguardas de ambiente (nunca dispara em dev/teste) e configuração de horário comercial — comunicação onde o time realmente está, sem gerar spam fora do expediente.
Conexão entre execução operacional e estratégia: tarefas do dia a dia podem ser vinculadas a indicadores de Balanced Scorecard, conectando o trabalho tático às metas estratégicas da empresa — um diferencial forte para o discurso de "gestão orientada a resultado".
Gamificação integrada para aumentar engajamento e adesão a prazos e rotinas.
Multiempresa/multiunidade nativo, auditoria de ações e controle de permissões — adequado para empresas com múltiplas filiais ou necessidade de compliance interno.
Arquitetura moderna e responsiva de verdade: componentes desktop/mobile dedicados, três temas visuais, acessibilidade como padrão (WCAG AA).
Deploy simples: hospedado inteiramente no Railway (backend, banco, storage e frontend), sem necessidade de infraestrutura própria complexa.
6. Proposta de valor / público-alvo (conforme documentado no produto)
O PraCima se posiciona como uma ferramenta de gestão de tarefas e projetos com visão estratégica: não é apenas um Kanban genérico, mas um sistema que une planejamento operacional (tarefas, etapas, calendário, rotinas), engajamento de equipe (gamificação) e gestão de desempenho estratégico (BSC), com relatórios que dão visibilidade completa a gestores. O público-alvo são equipes internas de empresas (o material de produto usa como referência real uma empresa do ramo alimentício — açaiteria — sugerindo adequação também a redes de varejo/franquias com múltiplas unidades), com dois perfis centrais: gestores que estruturam o trabalho e definem indicadores, e colaboradores que executam e são acompanhados/reconhecidos por isso. A proposta de valor central é permitir criar e gerenciar trabalho "sem fricção", com confiança de que as informações certas foram capturadas, e trazer a comunicação (WhatsApp) e a estratégia (BSC) para dentro do mesmo fluxo em que o trabalho é executado — profissional, confiável e visualmente identificável pela paleta roxa da marca.

Com base no levantamento completo do sistema, aqui está o conteúdo de marketing pronto para você colocar no site:

PraCima — Gestão de Tarefas com Visão Estratégica
Transforme a rotina da sua equipe em resultado.

O PraCima é a plataforma que une gestão de tarefas do dia a dia com gestão estratégica de indicadores (BSC), gamificação e comunicação inteligente — tudo em um só lugar, para que sua empresa saiba exatamente o que está sendo feito e se isso está levando aos resultados certos.

Por que o PraCima?
Ferramentas de tarefas comuns (Kanban genérico) mostram o que está sendo feito, mas não mostram se isso importa para a estratégia da empresa. O PraCima resolve isso conectando cada tarefa do time diretamente aos indicadores estratégicos do negócio — do operacional ao estratégico, em um único fluxo.

Funcionalidades principais
📋 Gestão de tarefas e listas
Crie, atribua e acompanhe tarefas com responsável, convidados, prazos e etapas. Organize tudo em listas e quadros (visão Kanban) por projeto, time ou unidade da empresa. Configure rotinas que se repetem automaticamente, sem precisar recriar tarefas manualmente.

📅 Etapas com agenda inteligente
Cada tarefa pode ser dividida em etapas com horário e janela de execução próprios, dando controle fino sobre quando cada passo do processo deve acontecer.

🎯 Balanced Scorecard (BSC) integrado
Cadastre indicadores estratégicos por dimensão (financeira, processos, clientes, aprendizado), acompanhe metas ao longo do tempo e vincule as tarefas do dia a dia diretamente a esses indicadores — conectando execução operacional aos objetivos estratégicos da empresa.

🏆 Gamificação
Motive sua equipe com pontuação e rankings por cumprimento de prazos e rotinas. Reconhecimento que aumenta engajamento e reduz atraso.

📊 Relatórios gerenciais
Visibilidade completa para gestores: produtividade, cumprimento de prazos e desempenho de indicadores estratégicos, tudo consolidado em relatórios com gráficos e exportação para Excel.

💬 Notificações via WhatsApp
Lembretes de prazos e tarefas atribuídas chegam direto no WhatsApp da equipe — no canal onde as pessoas realmente estão, respeitando horário comercial e sem gerar spam fora do expediente.

🏢 Multiempresa e multiunidade
Ideal para redes, franquias e empresas com mais de uma unidade: segmente listas, tarefas e usuários por organização, com controle de permissões por perfil e trilha de auditoria completa de quem fez o quê.

🔍 Filtros salvos
Cada usuário pode salvar e reaplicar filtros de busca personalizados (Planejado, Atribuído a Mim, Minha Semana, Rotinas, etc.) — acessíveis em qualquer dispositivo.

Diferenciais técnicos
Confiabilidade de entrega: infraestrutura própria de fila de processamento garante que notificações e automações sejam entregues mesmo em caso de falha, com novas tentativas automáticas.
Experiência real em qualquer dispositivo: telas desenhadas especificamente para desktop e mobile, com três temas visuais e acessibilidade como padrão.
Segurança: autenticação com senhas criptografadas, controle de permissões por papel e auditoria de ações.
Deploy ágil e escalável, sem depender de infraestrutura própria complexa.
Para quem é
O PraCima foi feito para equipes internas de empresas que precisam de mais do que um quadro de tarefas — que precisam de clareza sobre o que fazer, disciplina para cumprir prazos, e visibilidade se o trabalho do dia a dia está de fato caminhando para as metas estratégicas do negócio. É especialmente indicado para redes e franquias com múltiplas unidades, mas serve qualquer empresa que queira unir execução operacional e gestão estratégica em uma única ferramenta.

uncionalidades principais
📋 Gestão de tarefas, listas e Kanban
Crie, atribua e acompanhe tarefas com responsável, convidados, prazos, anexos e etapas com horário próprio. Organize o trabalho em listas (com membros e permissões de admin/membro) e visualize tudo em um quadro Kanban com arrastar-e-soltar, ou em formato de calendário. Menus dedicados agrupam sua rotina em Planejado, Atribuído a Mim, Minha Semana, Importantes e Rotinas.

🔁 Rotinas automáticas
Configure tarefas recorrentes (diárias, semanais, mensais, em qualquer intervalo) uma única vez — o sistema recria automaticamente, sem retrabalho manual. Duplique tarefas existentes com um clique quando precisar reaproveitar um modelo.

🎯 Balanced Scorecard (BSC) completo
As 4 perspectivas clássicas do BSC — Financeira, Operação, Clientes e Pessoas — com indicadores acompanhados mês a mês ao longo do ano inteiro, e importação/exportação via Excel. Cada indicador pode ser vinculado às tarefas do time, conectando a execução do dia a dia às metas estratégicas da empresa.

🏆 Gamificação com rankings
Rankings por Prazo, Meta, Impacto e Atrasadas dão visibilidade ao desempenho individual e da equipe, transformando cumprimento de tarefas em reconhecimento e competição saudável.

📊 Relatórios visuais e multiempresa
Gráficos (pizza e outros), filtros dedicados e seletor de empresa — visão gerencial completa sobre produtividade e desempenho, inclusive para operações com múltiplas empresas/unidades.

💬 Notificações via WhatsApp
Lembretes de prazos e tarefas direto no WhatsApp da equipe, com configuração de horário comercial e lista de contatos autorizados — comunicação no canal certo, na hora certa, sem spam fora do expediente.

🔍 Filtros salvos
Salve e reaplique filtros de busca personalizados em qualquer tela de tarefas, com um clique, em qualquer dispositivo.

🔐 Administração, permissões e auditoria
Perfis de acesso (membro, admin, super admin), controle por empresa e trilha completa de auditoria de todas as ações do sistema — segurança e governança para ambientes corporativos.

Telas de gestão de tarefas (menu principal)
Planejado: lista central de tarefas com detalhamento em painel lateral (sidebar de detalhe da tarefa), histórico de alterações da tarefa e organização por seções de data.
Atribuído a Mim: filtro dedicado às tarefas designadas ao usuário logado.
Minha Semana: visão semanal das tarefas do usuário.
Importantes: lista separada de tarefas marcadas como prioritárias/importantes.
Rotinas: gestão de tarefas recorrentes/rotineiras.
Copiar Tarefas / Duplicar Tarefa: funcionalidade para replicar tarefas já cadastradas, evitando recriar do zero.
Cadastro de Tarefa: formulário de criação/edição de tarefa, com um recurso destacado de configuração de recorrência ("Nunca repetir" vs. "Repetir tarefa", com frequência e intervalo configuráveis — ex.: repetir a cada X dias/semanas/meses).
Etapas: subtarefas/passos dentro de uma tarefa, com agenda e controle de horários (funcionalidade recente, ligada ao spec de agendamento de etapas).
Convidados (componente reutilizável): permite adicionar pessoas/convidados a uma tarefa ou lista.
Anexos (componente reutilizável): upload/anexação de arquivos às tarefas.
Notificações: componente de notificações do usuário.