# Plano Técnico: Weather App

Fonte da verdade: [`specs/weather-app-spec.md`](../specs/weather-app-spec.md). Este plano define arquitetura e contratos para a v1; não contém implementação final.

## Architecture

Aplicação cliente estática, sem backend e sem autenticação. Manter a separação já adotada no projeto:

- `components/`: apresentação e interação acessível.
- `hooks/useWeather.ts`: orquestra busca, seleção, consulta, retry e estados da UI.
- `services/weatherService.ts`: integração HTTP, validação e normalização de respostas Open-Meteo.
- `lib/`: funções puras de conversão, formatação e mapeamento de códigos meteorológicos.
- `types/weather.ts`: contratos de domínio compartilhados.

Não adicionar biblioteca de estado global. A unidade selecionada é estado local da aplicação; dados meteorológicos permanecem em Celsius no domínio e são formatados somente na apresentação.

## Tech Stack

- TypeScript strict, React 19 e Vite 8.
- Tailwind CSS para estilos, conforme o projeto.
- pnpm como gerenciador; Node.js 22 ou superior.
- Vitest e Testing Library para testes unitários/de componentes; Playwright para E2E.
- Biome para lint e formatação.
- `fetch` nativo e `AbortController` para chamadas HTTP e timeout. Sem chave de API ou dependência de backend.
- Para o requisito WCAG, usar uma verificação automatizada de acessibilidade no fluxo E2E; adicionar `@axe-core/playwright` como dependência de desenvolvimento se ainda não estiver disponível.

## Project Structure

Reutilizar os arquivos existentes; adicionar novos somente se uma implementação específica exigir.

```text
src/
  App.tsx                         # composição da página e unidade ativa
  components/
    SearchBar.tsx                 # entrada e submissão da busca
    CurrentWeather.tsx            # condições atuais
    ForecastList.tsx              # previsão de cinco dias
    ForecastCard.tsx              # resumo de um dia
    UnitToggle.tsx                # controle Celsius/Fahrenheit
    states/                       # vazio, loading e erro acessíveis
  hooks/
    useWeather.ts                 # estado e operações de consulta
  lib/
    temperature.ts                # conversão e formatação
    format.ts                     # rótulos e datas locais
    weatherCodes.ts               # código Open-Meteo para condição pt-BR
  services/
    weatherService.ts             # geocoding, forecast, validação e mapping
  types/
    weather.ts                    # modelos e erros de domínio
  styles/index.css

tests/
  unit/                           # serviços, lógica pura e componentes
  e2e/                            # fluxos principais e responsividade
```

## Data Model

Contratos normalizados para a UI. Campos meteorológicos individuais podem estar ausentes; o serviço os normaliza para `null`. Respostas sem os objetos principais ou sem cinco datas são inválidas.

```ts
type Unit = 'celsius' | 'fahrenheit';

type NullableNumber = number | null;

interface City {
  id: number;
  name: string;
  country?: string;
  admin1?: string;
  latitude: number;
  longitude: number;
}

interface CurrentWeather {
  time: string;
  temperatureC: NullableNumber;
  weatherCode: number | null;
  humidityPercent: NullableNumber;
  windSpeedKmh: NullableNumber;
  pressureHpa: NullableNumber;
  precipitationMm: NullableNumber;
}

interface ForecastDay {
  date: string; // YYYY-MM-DD no fuso local da cidade
  weatherCode: number | null;
  minTemperatureC: NullableNumber;
  maxTemperatureC: NullableNumber;
  precipitationProbabilityPercent: NullableNumber;
}

interface WeatherData {
  city: City;
  timezone: string;
  current: CurrentWeather;
  forecast: ForecastDay[]; // pós-condição do serviço: exatamente 5 itens
}

type WeatherOperation = 'geocoding' | 'forecast';
type WeatherErrorKind = 'timeout' | 'network' | 'http' | 'invalid-response';

interface WeatherError {
  operation: WeatherOperation;
  kind: WeatherErrorKind;
  message: string; // mensagem pt-BR para apresentação
}
```

Invariantes:

- Geocoding retorna zero a cinco cidades; uma cidade só fica selecionada após ação explícita da pessoa usuária.
- Temperaturas no domínio são sempre Celsius, sem arredondamento.
- A previsão normalizada contém exatamente cinco datas consecutivas locais.
- Campo JSON individual ausente/nulo vira `null`; valor numérico inválido ou estrutura principal inválida gera `invalid-response`.
- A unidade não é persistida; cada carregamento da aplicação inicia em Celsius.

## Data Flow

1. A pessoa submete a busca. A UI remove espaços nas extremidades; entrada vazia não faz requisição.
2. `useWeather` inicia busca e `weatherService.searchCities` codifica o nome via `URLSearchParams` e solicita até cinco resultados.
3. Lista vazia vira estado `empty`; não há chamada de previsão. Resultados são apresentados sem seleção automática.
4. Ao iniciar uma nova busca, limpar a cidade selecionada e os dados meteorológicos anteriores. A seleção explícita de um resultado define a nova cidade, inicia loading e chama `getWeather` com suas coordenadas.
5. O serviço valida e normaliza a resposta. Só publica resultado se a requisição ainda corresponder à seleção atual; respostas obsoletas são ignoradas.
6. A UI renderiza dados, placeholders `—` e unidades. Alternar unidade recalcula a exibição dos valores originais sem requisição externa.
7. Retry repete a última operação que falhou: busca com o mesmo termo ou previsão para a mesma cidade.

## External APIs

### Geocoding

- Endpoint: `https://geocoding-api.open-meteo.com/v1/search`.
- Parâmetros: `name` (termo aparado e codificado), `count=5`, `language=pt`, `format=json`.
- Mapear `id`, `name`, `country`, `admin1`, `latitude` e `longitude`; preservar ausência de campos opcionais.
- Resultado sem `results` ou lista vazia é busca sem correspondência, não falha de transporte.

### Forecast

- Endpoint: `https://api.open-meteo.com/v1/forecast`.
- Parâmetros: `latitude`, `longitude`, `current=temperature_2m,weather_code,relative_humidity_2m,wind_speed_10m,surface_pressure,precipitation`, `daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max`, `forecast_days=5`, `timezone=auto`, Celsius e vento em km/h.
- Validar `current`, `daily`, timezone e cinco datas. Correlacionar arrays diários pelo mesmo índice.
- Usar as unidades esperadas pela spec: °C internamente, %, km/h, hPa, mm e probabilidade em %.
- Sem polling, retry automático ou provedor de fallback. Confirmar cotas, disponibilidade e atribuição exigida pelo Open-Meteo antes do lançamento.

## State Management

Manter uma máquina de estados pequena no hook, sem store global:

```ts
type WeatherStatus =
  | 'idle'
  | 'loading-search'
  | 'results'
  | 'loading-weather'
  | 'success'
  | 'empty'
  | 'error';

interface WeatherViewState {
  status: WeatherStatus;
  query: string;
  cities: City[];
  selectedCity: City | null;
  data: WeatherData | null;
  error: WeatherError | null;
}
```

- `App.tsx` mantém `unit: Unit`, inicializado como `celsius`.
- `useWeather` expõe operações `search`, `selectCity` e `retry`; retry guarda a operação e seus argumentos.
- Cada nova busca/seleção invalida a requisição anterior por contador de sequência ou mecanismo equivalente. Não introduzir cache ou persistência.
- Ao iniciar outra busca, não apresentar o clima anterior como pertencente à nova cidade.

## Error Handling

- Timeout por requisição: 10 segundos com `AbortController`; limpar temporizador em sucesso, erro ou cancelamento.
- HTTP não-2xx, falha de rede, JSON inválido, objetos obrigatórios ausentes ou previsão com menos de cinco datas: encerrar loading, emitir `WeatherError` e oferecer retry manual.
- Erros individuais não estruturais: manter o registro e mapear o campo ausente para `null`, renderizado como `—`.
- Geocoding vazio: estado `empty` com “Nenhuma cidade encontrada”; não mostrar como erro nem chamar forecast.
- Erro em região acessível (`role="alert"`); carregamento em região de status (`role="status"`, anúncio polido). Mensagens em pt-BR e retry por botão/teclado.
- Não repetir automaticamente erro HTTP ou timeout; evitar carga duplicada no provedor e manter retry sob controle da pessoa usuária.

## Testing Strategy

- **Unitários:** aparar/codificar busca; conversão C/F e arredondamento apenas na apresentação; rótulos weather code; datas no fuso local; normalização de campos ausentes.
- **Serviço (Vitest + fetch mock):** URLs/parâmetros dos dois endpoints, limite de cinco resultados, unidades, lista vazia, HTTP 429/5xx, rede, timeout de 10 s, JSON inválido, campo individual ausente e resposta com menos de cinco datas.
- **Hook/componentes (Testing Library):** não selecionar cidade automaticamente; input vazio não chama API; selecionar resultado consulta coordenadas corretas; retry repete a operação falha; resposta obsoleta não substitui cidade; loading, vazio e erro acessíveis; alternar unidade não chama API.
- **E2E (Playwright):** busca, seleção explícita, clima atual, cinco dias, alternância, vazio, retry e fluxo mobile a 320 px. Interceptar Open-Meteo para cenários determinísticos; manter ao menos uma verificação de integração real fora do conjunto bloqueante, condicionada à disponibilidade externa.
- **Acessibilidade:** executar análise automatizada sem violações críticas/graves e cobrir busca, seleção, alternância e retry por teclado.
- **Gate do projeto:** `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e`.

## Risks & Trade-offs

- **Limites, disponibilidade e atribuição do Open-Meteo:** verificar condições de uso e cota antes do lançamento. Sem fallback, indisponibilidade impede consultas.
- **Validação estrita de cinco dias:** garante contrato simples e previsível, mas transforma respostas parciais de dias em erro recuperável; campos individuais continuam tolerantes via `null`.
- **Sem atualização automática:** reduz tráfego e complexidade, mas os dados podem envelhecer durante uma sessão.
- **Acessibilidade WCAG 2.2 AA e matriz de navegadores:** aumentam custo de validação; manter verificações automatizadas e fluxos E2E focados.
- **Divergências atuais a resolver durante implementação:** o hook existente seleciona automaticamente o primeiro resultado; os tipos existentes tornam métricas obrigatórias; o serviço atual converte probabilidade diária ausente em `0` e não valida cinco datas; não há proteção completa contra respostas obsoletas. A implementação deve passar a obedecer à spec antes de considerar os critérios concluídos.
