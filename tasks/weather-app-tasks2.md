# Tarefas: Weather App

Fonte: [`plans/weather-app-plan.md`](../plans/weather-app-plan.md), derivado de [`specs/weather-app-spec.md`](../specs/weather-app-spec.md). Tarefas ordenadas por entrega e dependência. Cada tarefa altera no máximo dois arquivos prováveis e mantém uma única responsabilidade.

## Prioridade e tamanho

Prioridade: **P0** bloqueia a primeira versão funcional e demonstrável; **P1** completa a verificação/hardening requerido antes do release; **P2** é melhoria não bloqueante. Não há P2 neste backlog porque todas as tarefas cobrem escopo da v1. Tamanho relativo: **P** pequeno/localizado; **M** envolve vários casos ou uma integração limitada; **G** cruza etapas e exige coordenação/validação ampla.

| ID | Prioridade | Tamanho |
| --- | --- | --- |
| T-01 | P0 | M |
| T-02 | P0 | P |
| T-03 | P0 | P |
| T-04 | P0 | P |
| T-05 | P0 | M |
| T-06 | P0 | M |
| T-07 | P0 | M |
| T-08 | P0 | G |
| T-09 | P0 | M |
| T-10 | P0 | M |
| T-11 | P0 | M |
| T-12 | P0 | P |
| T-13 | P0 | P |
| T-14 | P0 | P |
| T-15 | P0 | P |
| T-16 | P0 | G |
| T-17 | P1 | P |
| T-18 | P1 | P |
| T-19 | P1 | P |
| T-20 | P1 | P |
| T-21 | P1 | M |
| T-22 | P1 | M |
| T-23 | P1 | P |
| T-24 | P1 | P |
| T-25 | P1 | P |
| T-26 | P1 | P |
| T-27 | P1 | P |
| T-28 | P1 | P |
| T-29 | P1 | M |
| T-30 | P1 | G |
| T-31 | P1 | M |

## Sequência de fatias verticais

1. **Busca e clima atual visíveis:** concluir T-01, T-02, T-04–T-10, T-12–T-14 e iniciar T-16 para integrar input → geocoding → seleção → clima atual. O primeiro checkpoint visual pode ser integrado antes de a previsão estar completa; T-16 só fica concluída após todas as suas condições de aceite.
2. **Previsão e unidade:** concluir T-03, T-11 e T-15 e finalizar T-16, entregando os cinco dias e alternância C/F.
3. **Testes por fatia:** executar T-18 após T-02, T-19 após T-03, T-20 após T-04, T-21 após T-05–T-07, T-22 após T-08 e cada teste de componente (T-23–T-29) logo após o componente correspondente. Concluir o fluxo Playwright em T-30.
4. **Hardening de release:** concluir T-17 e T-31 para axe, matriz de navegadores e gates finais.

## Entrega 1 — Tipos

### T-01 — Alinhar tipos de domínio

- **ID:** T-01
- **Título:** Alinhar tipos de domínio
- **Descrição:** Definir contratos compartilhados usados pela UI e pelos serviços.
- **Critérios de aceite:** `Unit` é `'celsius' | 'fahrenheit'`; `City` contém `id`, `name`, `latitude`, `longitude` e metadados geográficos opcionais; métricas individuais aceitam `null`; `WeatherData` contém cidade, fuso, condições atuais e previsão de cinco dias.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/types/weather.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-01–FR-05; AC-02.2, AC-03.3, AC-04.1–AC-04.4, AC-05.2.

## Entrega 2 — Funções puras

### T-02 — Implementar temperatura

- **ID:** T-02
- **Título:** Converter e formatar temperaturas
- **Descrição:** Manter conversão C/F pura e arredondamento somente na apresentação.
- **Critérios de aceite:** `20 °C` converte para `68 °F`; a conversão não arredonda; formatação arredonda para inteiro; alternâncias repetidas partem do valor Celsius original.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/temperature.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-04; AC-04.1–AC-04.3.

### T-03 — Implementar datas locais

- **ID:** T-03
- **Título:** Formatar datas da previsão
- **Descrição:** Produzir rótulos e datas curtas para os dias recebidos no fuso local.
- **Critérios de aceite:** índice 0 retorna `Hoje`; índice 1 retorna `Amanhã`; `2026-03-05` é formatada como `5 Mar`; parsing da data não aplica deslocamento UTC.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/format.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-03; AC-03.1–AC-03.2; NFR-01.

### T-04 — Mapear códigos meteorológicos

- **ID:** T-04
- **Título:** Mapear códigos WMO
- **Descrição:** Fornecer ícone e rótulo pt-BR para códigos de condições atuais e previsão.
- **Critérios de aceite:** cada código WMO mapeado retorna rótulo não vazio e ícone; código desconhecido retorna `Condição desconhecida` e ícone neutro, sem lançar exceção.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/weatherCodes.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-02–FR-03; AC-02.1, AC-03.1.

## Entrega 3 — Services

### T-05 — Implementar transporte HTTP

- **ID:** T-05
- **Título:** Implementar transporte HTTP resiliente
- **Descrição:** Centralizar timeout, cancelamento e classificação de erros do `fetch`.
- **Critérios de aceite:** timeout em 10 s aborta o request; temporizador é limpo em sucesso, erro e timeout; cada falha retorna `WeatherError` com `operation`, `kind` e `message`; nenhum erro inicia retry automático.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/services/weatherService.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-05; AC-05.2–AC-05.4; NFR-06.

### T-06 — Integrar geocoding

- **ID:** T-06
- **Título:** Buscar e normalizar cidades
- **Descrição:** Implementar a consulta de geocoding e mapear resultados para `City`.
- **Critérios de aceite:** URL contém termo aparado/codificado, `count=5`, `language=pt` e `format=json`; termo vazio não chama `fetch`; lista vazia retorna `[]`; resposta mapeia IDs, nome, coordenadas e metadados presentes sem inventar os ausentes.
- **Dependências:** T-01, T-05.
- **Arquivos prováveis:** `src/services/weatherService.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-01; AC-01.1–AC-01.3, AC-01.5–AC-01.6; NFR-05, NFR-06.

### T-07 — Integrar forecast

- **ID:** T-07
- **Título:** Normalizar condições e previsão
- **Descrição:** Consultar e validar dados atuais e previsão diária de cinco dias.
- **Critérios de aceite:** URL envia `current`/`daily`, `forecast_days=5`, `timezone=auto`, Celsius, km/h e mm; `current`, timezone e cinco datas são obrigatórios; campo individual ausente/nulo vira `null`; valores diários são mapeados por índice.
- **Dependências:** T-01, T-05.
- **Arquivos prováveis:** `src/services/weatherService.ts`
- **Tipo:** Data
- **Rastreabilidade:** FR-02–FR-03; AC-02.1–AC-02.3, AC-03.1–AC-03.4; NFR-04–NFR-06.

## Entrega 4 — Hook

### T-08 — Orquestrar o estado

- **ID:** T-08
- **Título:** Implementar o hook meteorológico
- **Descrição:** Orquestrar busca, seleção, estados, retry e descarte de respostas obsoletas.
- **Critérios de aceite:** estado exposto é um dos estados definidos no plano; resultado de geocoding não seleciona cidade; `selectCity` consulta as coordenadas selecionadas; `retry` repete a última busca ou forecast; nova busca limpa seleção/dados; resposta iniciada antes da seleção atual não altera o estado atual.
- **Dependências:** T-06, T-07.
- **Arquivos prováveis:** `src/hooks/useWeather.ts`
- **Tipo:** UI
- **Rastreabilidade:** FR-01, FR-05; AC-01.3–AC-01.5, AC-05.3, AC-05.5; NFR-04–NFR-06.

## Entrega 5 — Componentes

### T-09 — Implementar busca e resultados

- **ID:** T-09
- **Título:** Criar busca de cidades
- **Descrição:** Apresentar input, submissão e resultados selecionáveis.
- **Critérios de aceite:** submeter input vazio não chama a ação de busca; cada opção mostra nome e campos geográficos presentes; nenhuma opção fica selecionada automaticamente; clique, Enter e Espaço selecionam a opção focada e chamam o handler uma vez.
- **Dependências:** T-08.
- **Arquivos prováveis:** `src/components/SearchBar.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-01; AC-01.1–AC-01.4, AC-01.6; NFR-01–NFR-03, NFR-07.

### T-10 — Exibir condições atuais

- **ID:** T-10
- **Título:** Criar painel de clima atual
- **Descrição:** Apresentar cidade, condição e métricas atuais com unidades.
- **Critérios de aceite:** renderiza os seis campos definidos em FR-02 com °C/°F, %, km/h, hPa e mm; cada campo `null` mostra `—`; nome da cidade e métricas vêm do mesmo objeto `WeatherData` recebido.
- **Dependências:** T-01, T-02, T-04.
- **Arquivos prováveis:** `src/components/CurrentWeather.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-02; AC-02.1–AC-02.3; NFR-01–NFR-04, NFR-07.

### T-11 — Exibir previsão diária

- **ID:** T-11
- **Título:** Criar previsão de cinco dias
- **Descrição:** Renderizar cards diários com datas, condições e métricas.
- **Critérios de aceite:** para uma previsão válida, renderiza cinco itens na ordem recebida; cada item mostra data, mínima, máxima, condição e probabilidade (%); `null` mostra `—`; temperaturas recebem a unidade ativa.
- **Dependências:** T-01, T-02, T-03, T-04.
- **Arquivos prováveis:** `src/components/ForecastList.tsx`, `src/components/ForecastCard.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-03–FR-04; AC-03.1–AC-03.3, AC-04.1–AC-04.3; NFR-01–NFR-04, NFR-07.

### T-12 — Exibir carregamento

- **ID:** T-12
- **Título:** Criar estado de carregamento
- **Descrição:** Apresentar loading acessível enquanto busca ou forecast está pendente.
- **Critérios de aceite:** enquanto uma operação está pendente, o componente renderiza `role="status"`, `aria-live="polite"` e o texto “Carregando o clima…”.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/components/states/LoadingState.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-05; AC-05.1; NFR-01, NFR-03.

### T-13 — Exibir estado vazio

- **ID:** T-13
- **Título:** Criar estado sem resultados
- **Descrição:** Comunicar busca concluída sem cidades correspondentes.
- **Critérios de aceite:** componente renderiza exatamente o texto “Nenhuma cidade encontrada”; não recebe nem exibe dados meteorológicos da busca sem correspondência.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/components/states/EmptyState.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-01, FR-05; AC-01.5, AC-05.1; NFR-01–NFR-03.

### T-14 — Exibir falha e retry

- **ID:** T-14
- **Título:** Criar estado de erro
- **Descrição:** Apresentar erro acessível e ação para repetir a operação.
- **Critérios de aceite:** componente renderiza `role="alert"`, a mensagem recebida e um botão “Tentar novamente”; Enter/Space no botão chama o callback exatamente uma vez.
- **Dependências:** T-08.
- **Arquivos prováveis:** `src/components/states/ErrorState.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-05; AC-05.2–AC-05.3; NFR-01, NFR-03, NFR-06.

### T-15 — Alternar unidade

- **ID:** T-15
- **Título:** Criar controle de unidade
- **Descrição:** Expor a seleção entre Celsius e Fahrenheit.
- **Critérios de aceite:** controle recebe Celsius como valor inicial; selecionar Fahrenheit/Celsius chama o handler com a unidade correspondente; o controle tem nome acessível e pode ser operado por teclado.
- **Dependências:** T-01, T-02.
- **Arquivos prováveis:** `src/components/UnitToggle.tsx`
- **Tipo:** UI
- **Rastreabilidade:** FR-04; AC-04.1–AC-04.4; NFR-02–NFR-04, NFR-07.

## Entrega 6 — Integração

### T-16 — Compor a aplicação

- **ID:** T-16
- **Título:** Integrar página e layout
- **Descrição:** Conectar hook e componentes na tela principal, preservando o tema do projeto.
- **Critérios de aceite:** cada estado do hook renderiza o componente correspondente; unidade inicia em Celsius e chega aos componentes de temperatura; a página não tem rolagem horizontal em viewport de 320 px; aplicação usa as classes/tokens Tailwind existentes para o tema dark glassmorphism.
- **Dependências:** T-09, T-10, T-11, T-12, T-13, T-14, T-15.
- **Arquivos prováveis:** `src/App.tsx`, `src/styles/index.css`
- **Tipo:** UI
- **Rastreabilidade:** FR-01–FR-05; AC-01.1–AC-05.5; NFR-01–NFR-04, NFR-07.

## Entrega 7 — Testes

### T-17 — Adicionar verificação de acessibilidade

- **ID:** T-17
- **Título:** Configurar ferramenta de acessibilidade
- **Descrição:** Adicionar `@axe-core/playwright` como dependência de desenvolvimento se necessário.
- **Critérios de aceite:** `@axe-core/playwright` consta em `devDependencies` e no lockfile; um teste E2E importa a ferramenta e executa uma análise na página sem incluir a dependência no bundle de produção.
- **Dependências:** T-16.
- **Arquivos prováveis:** `package.json`, `pnpm-lock.yaml`
- **Tipo:** Infra
- **Rastreabilidade:** NFR-03.

### T-18 — Testar conversão de temperatura

- **ID:** T-18
- **Título:** Testar temperatura
- **Descrição:** Cobrir conversão e formatação com Vitest.
- **Critérios de aceite:** `20 °C` converte para `68 °F`; `-10 °C` converte para `14 °F`; conversão não arredonda; formatar `20.4 °C` retorna `20°`; teste não monta React nem acessa rede.
- **Dependências:** T-02.
- **Arquivos prováveis:** `tests/unit/temperature.test.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-04; AC-04.1–AC-04.3.

### T-19 — Testar datas locais

- **ID:** T-19
- **Título:** Testar formatação de datas
- **Descrição:** Cobrir parsing e rótulos de previsão.
- **Critérios de aceite:** índice 0 retorna `Hoje`; índice 1 retorna `Amanhã`; `getShortDate('2026-03-05')` retorna `5 Mar`; os resultados são iguais sob timezones UTC e America/Sao_Paulo.
- **Dependências:** T-03.
- **Arquivos prováveis:** `tests/unit/format.test.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-03; AC-03.1–AC-03.2; NFR-01.

### T-20 — Testar códigos WMO

- **ID:** T-20
- **Título:** Testar rótulos meteorológicos
- **Descrição:** Cobrir códigos WMO mapeados e fallback.
- **Critérios de aceite:** códigos usados nas fixtures retornam os rótulos/ícones configurados; código fora do mapa retorna `Condição desconhecida` e ícone neutro, sem exceção.
- **Dependências:** T-04.
- **Arquivos prováveis:** `tests/unit/weatherCodes.test.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-02–FR-03; AC-02.1, AC-03.1.

### T-21 — Testar serviços de dados

- **ID:** T-21
- **Título:** Testar serviços Open-Meteo
- **Descrição:** Validar requests, normalização e falhas com `fetch` mockado.
- **Critérios de aceite:** testes verificam URLs/parâmetros exatos e mapping para `City`/`WeatherData`; cada campo ausente/nulo vira `null`; geocoding vazio retorna `[]`; HTTP não-2xx, rejeição de rede, abort após 10 s, JSON inválido, ausência de objeto obrigatório e forecast com menos de cinco datas rejeitam com o `WeatherError.kind` esperado.
- **Dependências:** T-05, T-06, T-07.
- **Arquivos prováveis:** `tests/unit/weatherService.test.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-01–FR-03, FR-05; AC-01.1, AC-01.5–AC-01.6, AC-02.1–AC-02.3, AC-03.1–AC-03.4, AC-05.2, AC-05.4; NFR-05–NFR-06.

### T-22 — Testar orquestração do hook
- **ID:** T-22
- **Título:** Testar estados do hook
- **Descrição:** Verificar transições e coordenação dos serviços.
- **Critérios de aceite:** serviço mock não é chamado para termo vazio; testes observam idle/loading/results/empty/success/error; seleção chama forecast com a cidade escolhida; retry repete endpoint e argumentos da operação falha; nova busca zera cidade/dados; resolução atrasada não altera o estado mais recente.
- **Dependências:** T-08.
- **Arquivos prováveis:** `tests/unit/useWeather.test.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-01, FR-05; AC-01.3–AC-01.5, AC-05.3, AC-05.5; NFR-04–NFR-06.

### T-23 — Testar busca e seleção UI

- **ID:** T-23
- **Título:** Testar busca de cidade
- **Descrição:** Testar input, resultados e seleção com Testing Library.
- **Critérios de aceite:** submeter vazio chama handler zero vezes; resultados renderizam os campos geográficos fornecidos; nenhuma opção começa selecionada; click, Enter e Space chamam handler uma vez com o `City` correto.
- **Dependências:** T-09.
- **Arquivos prováveis:** `tests/unit/SearchBar.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-01; AC-01.1–AC-01.4, AC-01.6; NFR-03.

### T-24 — Testar painel atual

- **ID:** T-24
- **Título:** Testar clima atual
- **Descrição:** Verificar conteúdo e placeholders do painel atual.
- **Critérios de aceite:** fixture renderiza nome e seis campos; rótulos incluem %, km/h, hPa e mm; campo `null` renderiza `—`; outra cidade não aparece no painel.
- **Dependências:** T-10.
- **Arquivos prováveis:** `tests/unit/CurrentWeather.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-02; AC-02.1–AC-02.2; NFR-04.

### T-25 — Testar previsão diária

- **ID:** T-25
- **Título:** Testar previsão diária
- **Descrição:** Verificar renderização dos cards da previsão.
- **Critérios de aceite:** fixture de cinco dias produz cinco itens na ordem recebida; cada item mostra data, máxima, mínima, condição e probabilidade; unidade selecionada afeta temperaturas; campo `null` mostra `—`.
- **Dependências:** T-11.
- **Arquivos prováveis:** `tests/unit/ForecastList.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-03–FR-04; AC-03.1–AC-03.3, AC-04.1–AC-04.3; NFR-01, NFR-04.

### T-26 — Testar loading

- **ID:** T-26
- **Título:** Testar estado de carregamento
- **Descrição:** Verificar o componente de loading sem montar a aplicação inteira.
- **Critérios de aceite:** loading expõe `role="status"`, `aria-live="polite"` e o texto “Carregando o clima…”.
- **Dependências:** T-12.
- **Arquivos prováveis:** `tests/unit/LoadingState.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-05; AC-05.1; NFR-01, NFR-03.

### T-27 — Testar estado vazio

- **ID:** T-27
- **Título:** Testar ausência de resultados
- **Descrição:** Verificar o componente vazio de geocoding.
- **Critérios de aceite:** componente exibe exatamente “Nenhuma cidade encontrada” e não renderiza campos meteorológicos.
- **Dependências:** T-13.
- **Arquivos prováveis:** `tests/unit/EmptyState.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-01, FR-05; AC-01.5; NFR-01–NFR-03.

### T-28 — Testar estado de erro

- **ID:** T-28
- **Título:** Testar erro e retry
- **Descrição:** Verificar mensagem e callback do estado de erro.
- **Critérios de aceite:** erro expõe `role="alert"` e a mensagem recebida; click, Enter e Space no botão “Tentar novamente” chamam o callback uma vez por ativação.
- **Dependências:** T-14.
- **Arquivos prováveis:** `tests/unit/ErrorState.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-05; AC-05.2–AC-05.3; NFR-03, NFR-06.

### T-29 — Testar unidade e composição

- **ID:** T-29
- **Título:** Testar unidade e página
- **Descrição:** Cobrir controle de unidade e integração da página.
- **Critérios de aceite:** controle parte de Celsius e expõe unidade selecionada; App propaga a unidade ativa; mocks dos serviços têm zero chamadas causadas pela alternância.
- **Dependências:** T-15, T-16.
- **Arquivos prováveis:** `tests/unit/UnitToggle.test.tsx`, `tests/unit/App.test.tsx`
- **Tipo:** Test
- **Rastreabilidade:** FR-04; AC-04.1–AC-04.4; NFR-03–NFR-04.

### T-30 — Testar jornadas E2E

- **ID:** T-30
- **Título:** Testar fluxo principal e viewport mobile
- **Descrição:** Cobrir jornada completa e recuperação com Playwright e APIs interceptadas.
- **Critérios de aceite:** E2E verifica busca, seleção, clima atual, cinco dias, troca de unidade, vazio e retry; fixtures tornam resultados determinísticos; em viewport de 320 px `scrollWidth <= clientWidth`; resposta atrasada não muda a cidade exibida.
- **Dependências:** T-17–T-29.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`
- **Tipo:** Test
- **Rastreabilidade:** FR-01–FR-05; AC-01.1–AC-05.5; NFR-01–NFR-04.

## Entrega 8 — Hardening

### T-31 — Endurecer navegadores e acessibilidade

- **ID:** T-31
- **Título:** Configurar matriz de navegadores e axe
- **Descrição:** Executar verificações de acessibilidade e compatibilidade da spec após os fluxos E2E.
- **Critérios de aceite:** `playwright.config.ts` executa Chromium, Firefox e WebKit; a análise axe retorna zero violações críticas/graves; o CI e a documentação registram a cobertura complementar de Edge, Safari/iOS e Chrome/Android; `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e` terminam com código 0.
- **Dependências:** T-17, T-30.
- **Arquivos prováveis:** `playwright.config.ts`, `tests/e2e/weather.spec.ts`
- **Tipo:** Infra
- **Rastreabilidade:** NFR-02–NFR-03, NFR-07.
