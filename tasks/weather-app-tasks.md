# Tarefas: Weather App

Fonte: [`plans/weather-app-plan.md`](../plans/weather-app-plan.md), derivado de [`specs/weather-app-spec.md`](../specs/weather-app-spec.md). As tarefas estão ordenadas por entrega e dependência; concluir os critérios de aceite antes de iniciar as tarefas dependentes.

## Entrega 1 — Contratos e dados

### T-01 — Alinhar tipos de domínio

- **ID:** T-01
- **Título:** Alinhar tipos de domínio
- **Descrição:** Definir os contratos compartilhados de cidade, condições atuais, previsão, agregado meteorológico, unidade e erro conforme a spec e o plano.
- **Critérios de aceite:**
  - `Unit` é `'celsius' | 'fahrenheit'`.
  - Métricas que podem faltar são `number | null`; campos geográficos opcionais permanecem opcionais.
  - Temperaturas no domínio são Celsius; `WeatherData` contém cidade, fuso, condições atuais e previsão.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/types/weather.ts`
- **Tipo:** Data

### T-02 — Implementar transporte resiliente

- **ID:** T-02
- **Título:** Implementar transporte resiliente
- **Descrição:** Centralizar chamadas `fetch`, timeout e classificação de falhas para os serviços Open-Meteo.
- **Critérios de aceite:**
  - Cada requisição é cancelada após 10 segundos e o temporizador é limpo ao terminar.
  - HTTP não-2xx, falha de rede, timeout e resposta JSON inválida produzem erros tipados.
  - O transporte não repete requisições automaticamente.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/services/weatherService.ts`, `src/types/weather.ts`
- **Tipo:** Data

### T-03 — Implementar busca de cidades

- **ID:** T-03
- **Título:** Implementar busca de cidades
- **Descrição:** Integrar o endpoint de geocoding e normalizar seus resultados para `City`.
- **Critérios de aceite:**
  - Busca não vazia usa `name` codificado, `count=5`, `language=pt` e `format=json`.
  - Busca vazia não envia request; lista vazia retorna zero cidades sem erro.
  - No máximo cinco cidades são retornadas; país e estado/região ausentes não são inventados.
- **Dependências:** T-01, T-02.
- **Arquivos prováveis:** `src/services/weatherService.ts`, `src/types/weather.ts`
- **Tipo:** Data

### T-04 — Implementar consulta de previsão

- **ID:** T-04
- **Título:** Implementar consulta de previsão
- **Descrição:** Consultar e normalizar condições atuais e cinco dias locais a partir das coordenadas selecionadas.
- **Critérios de aceite:**
  - A request inclui campos `current` e `daily` definidos no plano, `forecast_days=5`, `timezone=auto` e unidades °C, km/h e mm.
  - `current`, timezone e exatamente cinco datas são validados; resposta estrutural inválida gera erro tipado.
  - Campos numéricos individuais `null`/ausentes são normalizados para `null`, nunca para zero; arrays diários são mapeados pelo mesmo índice.
- **Dependências:** T-01, T-02.
- **Arquivos prováveis:** `src/services/weatherService.ts`, `src/types/weather.ts`
- **Tipo:** Data

### T-05 — Completar funções puras

- **ID:** T-05
- **Título:** Completar funções puras de apresentação
- **Descrição:** Implementar ou alinhar conversão/formatação de temperatura, rótulos meteorológicos e datas locais sem efeitos colaterais.
- **Critérios de aceite:**
  - Conversão C/F usa valores originais sem arredondar; formatação arredonda apenas para o inteiro mais próximo.
  - Conversões repetidas não acumulam arredondamento.
  - Códigos desconhecidos e campos ausentes têm representação segura; datas usam o dia local recebido.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/temperature.ts`, `src/lib/format.ts`, `src/lib/weatherCodes.ts`
- **Tipo:** Data

## Entrega 2 — Estado e interface

### T-06 — Orquestrar estado meteorológico

- **ID:** T-06
- **Título:** Orquestrar busca e consulta
- **Descrição:** Implementar no hook estados, seleção explícita, retry e proteção contra respostas obsoletas.
- **Critérios de aceite:**
  - O hook distingue idle, loading de busca, resultados, loading de clima, sucesso, vazio e erro.
  - Não seleciona automaticamente o primeiro resultado; selecionar uma cidade consulta suas coordenadas.
  - Retry repete a operação que falhou; uma resposta antiga não altera a cidade/dados atuais.
  - Nova busca limpa seleção e dados meteorológicos anteriores.
- **Dependências:** T-03, T-04.
- **Arquivos prováveis:** `src/hooks/useWeather.ts`, `src/types/weather.ts`
- **Tipo:** UI

### T-07 — Implementar busca e seleção

- **ID:** T-07
- **Título:** Implementar busca e seleção de cidade
- **Descrição:** Construir o controle de busca e a lista de resultados com seleção explícita e acessível.
- **Critérios de aceite:**
  - Input vazio não inicia busca; submissão válida chama a ação do hook.
  - Cada resultado mostra cidade e contexto geográfico disponível; itens homônimos permanecem distinguíveis.
  - Busca, resultados e seleção são operáveis por teclado e têm nomes acessíveis.
- **Dependências:** T-06.
- **Arquivos prováveis:** `src/components/SearchBar.tsx`, `src/hooks/useWeather.ts`
- **Tipo:** UI

### T-08 — Exibir condições atuais

- **ID:** T-08
- **Título:** Exibir condições atuais
- **Descrição:** Renderizar cidade, condição e métricas atuais com unidades e placeholders para dados ausentes.
- **Critérios de aceite:**
  - São exibidos temperatura, condição, umidade (%), vento (km/h), pressão (hPa) e precipitação (mm).
  - Cada campo `null` aparece como `—`; temperatura usa a unidade ativa.
  - Os dados são identificados visualmente como pertencentes à cidade selecionada.
- **Dependências:** T-01, T-05.
- **Arquivos prováveis:** `src/components/CurrentWeather.tsx`
- **Tipo:** UI

### T-09 — Exibir previsão de cinco dias

- **ID:** T-09
- **Título:** Exibir previsão diária
- **Descrição:** Renderizar os cinco dias locais com máximas, mínimas, condição e probabilidade de precipitação.
- **Critérios de aceite:**
  - São exibidos exatamente cinco dias consecutivos, começando por hoje, com rótulos de data locais.
  - Cada dia apresenta mínima, máxima, condição e probabilidade (%); campos `null` aparecem como `—`.
  - Todas as temperaturas usam a unidade ativa.
- **Dependências:** T-01, T-05.
- **Arquivos prováveis:** `src/components/ForecastList.tsx`, `src/components/ForecastCard.tsx`
- **Tipo:** UI

### T-10 — Implementar estados acessíveis

- **ID:** T-10
- **Título:** Implementar estados de consulta
- **Descrição:** Construir apresentações de loading, vazio e erro, incluindo retry, com semântica acessível.
- **Critérios de aceite:**
  - Loading usa região `role="status"`; erro usa `role="alert"`.
  - Vazio informa que nenhuma cidade foi encontrada; erro em pt-BR oferece botão “Tentar novamente”.
  - Retry pode ser acionado por teclado.
- **Dependências:** T-06.
- **Arquivos prováveis:** `src/components/states/LoadingState.tsx`, `src/components/states/EmptyState.tsx`, `src/components/states/ErrorState.tsx`
- **Tipo:** UI

### T-11 — Integrar página e unidade

- **ID:** T-11
- **Título:** Integrar página Weather App
- **Descrição:** Compor busca, estados, dados atuais, previsão e alternância de unidade na aplicação.
- **Critérios de aceite:**
  - A página renderiza o estado correspondente do hook e a cidade selecionada.
  - Unidade inicia em Celsius, alterna para Fahrenheit e afeta todas as temperaturas sem novo request.
  - Layout preserva tema dark glassmorphism e permanece utilizável a partir de 320 px.
- **Dependências:** T-07, T-08, T-09, T-10.
- **Arquivos prováveis:** `src/App.tsx`, `src/components/UnitToggle.tsx`, `src/styles/index.css`
- **Tipo:** UI

## Entrega 3 — Verificação e qualidade

### T-12 — Configurar acessibilidade e navegadores

- **ID:** T-12
- **Título:** Configurar verificações E2E
- **Descrição:** Preparar a checagem automatizada de acessibilidade e os projetos de navegador/viewport previstos no plano.
- **Critérios de aceite:**
  - Playwright consegue executar os fluxos nos engines suportados pelo ambiente CI; a matriz ou smoke adicional cobre os navegadores definidos em NFR-07.
  - A verificação automatizada não reporta violações críticas/graves nos fluxos principais.
  - A checagem não substitui verificação explícita de teclado e contraste.
- **Dependências:** T-11.
- **Arquivos prováveis:** `playwright.config.ts`, `package.json`, `pnpm-lock.yaml`, `tests/e2e/weather.spec.ts`
- **Tipo:** Infra

### T-13 — Testar funções puras

- **ID:** T-13
- **Título:** Testar funções puras
- **Descrição:** Cobrir conversão, arredondamento, formatação de datas e códigos meteorológicos com Vitest.
- **Critérios de aceite:**
  - Testes cobrem C/F, valores negativos, valores de referência, conversões repetidas e arredondamento apenas na apresentação.
  - Testes cobrem datas locais, códigos conhecidos/desconhecidos e ausência de valores.
  - Testes não usam rede nem montam componentes React.
- **Dependências:** T-05.
- **Arquivos prováveis:** `tests/unit/temperature.test.ts`, `tests/unit/format.test.ts`, `tests/unit/weatherCodes.test.ts`
- **Tipo:** Test

### T-14 — Testar serviços Open-Meteo

- **ID:** T-14
- **Título:** Testar serviços de dados
- **Descrição:** Validar parâmetros, mapeamento e falhas dos serviços com `fetch` mockado.
- **Critérios de aceite:**
  - Geocoding cobre termo codificado, limite de resultados, lista vazia e campos geográficos opcionais.
  - Forecast cobre parâmetros/unidades, mapeamento de campos, `null`, exatamente cinco datas e resposta estruturalmente inválida.
  - HTTP, rede, timeout e JSON inválido produzem erro tipado; timeout cancela o request.
- **Dependências:** T-03, T-04.
- **Arquivos prováveis:** `tests/unit/weatherService.test.ts`
- **Tipo:** Test

### T-15 — Testar hook e componentes

- **ID:** T-15
- **Título:** Testar estado e componentes
- **Descrição:** Cobrir transições do hook e apresentação dos estados/fluxos de UI com Testing Library.
- **Critérios de aceite:**
  - Testes cobrem busca vazia sem chamada, seleção explícita, retry correto e resposta obsoleta ignorada.
  - Componentes cobrem loading, erro, vazio e sucesso, labels/roles e interação por teclado.
  - Alternar unidade não chama serviço; clima e previsão mostram cidade e unidades corretas.
- **Dependências:** T-06, T-07, T-08, T-09, T-10, T-11.
- **Arquivos prováveis:** `tests/unit/useWeather.test.ts`, `tests/unit/App.test.tsx`, `tests/unit/SearchBar.test.tsx`, `tests/unit/UnitToggle.test.tsx`
- **Tipo:** Test

### T-16 — Validar fluxos E2E

- **ID:** T-16
- **Título:** Validar fluxos E2E e responsividade
- **Descrição:** Cobrir jornadas principais e recuperação de falha no navegador com APIs interceptadas.
- **Critérios de aceite:**
  - E2E cobre busca, seleção, clima atual, cinco dias, troca de unidade, vazio e retry.
  - Em viewport de 320 px, fluxo principal é operável e `scrollWidth <= clientWidth`; há smoke em desktop.
  - Resposta atrasada de uma cidade não substitui a seleção atual; suite usa fixtures determinísticas.
  - `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e` passam.
- **Dependências:** T-12, T-13, T-14, T-15.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`, `playwright.config.ts`
- **Tipo:** Test
