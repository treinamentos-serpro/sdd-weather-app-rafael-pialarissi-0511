
## Overview

Aplicação web responsiva, em português do Brasil, para consultar o clima de uma
cidade. A pessoa usuária pesquisa e seleciona uma localização, consulta as
condições atuais e a previsão para hoje e os quatro dias seguintes, e alterna a
unidade de temperatura entre Celsius e Fahrenheit. Os dados são obtidos do
Open-Meteo, sem autenticação nem chave de API. A primeira versão atende uma
cidade selecionada por vez.

## Functional Requirements

- **FR-01 — Buscar e selecionar cidade:** aceitar busca por nome de cidade após
  remover espaços nas extremidades e apresentar até cinco resultados. Cada
  resultado deve mostrar o nome e, quando fornecidos, estado/região e país. A
  pessoa deve selecionar um resultado; não selecionar automaticamente.
- **FR-02 — Consultar condições atuais:** mostrar temperatura, condição,
  umidade, vento, pressão e precipitação da cidade selecionada. Usar °C ou °F
  para temperatura, %, km/h, hPa e mm para as demais métricas,
  respectivamente. Valores individuais indisponíveis devem ser identificados
  como `—`.
- **FR-03 — Consultar previsão:** mostrar cinco dias locais consecutivos,
  começando por hoje, com mínima, máxima, condição e probabilidade máxima de
  precipitação por dia. O serviço deve fornecer cinco datas; uma resposta com
  menos datas é erro de consulta. Campos individuais ausentes são exibidos como
  `—`.
- **FR-04 — Alternar unidade de temperatura:** oferecer Celsius e Fahrenheit,
  com Celsius como padrão. Aplicar a escolha à temperatura atual e a todas as
  mínimas e máximas. Converter os valores originais sem arredondamento e
  arredondar apenas para exibição ao inteiro mais próximo.
- **FR-05 — Comunicar estados da consulta:** distinguir carregamento, ausência
  de resultados e falha. Falhas de geocoding, previsão ou rede devem encerrar o
  carregamento, apresentar erro em pt-BR e permitir nova tentativa manual.

## User Stories

- **US-01 (FR-01):** Como usuário da aplicação, quero buscar e selecionar uma
  cidade para consultar o clima da localização correta.
- **US-02 (FR-02):** Como usuário da aplicação, quero consultar as condições
  meteorológicas atuais da cidade selecionada para saber como está o tempo
  nesse local.
- **US-03 (FR-03):** Como usuário da aplicação, quero consultar a previsão de
  hoje e dos quatro dias seguintes para planejar minhas atividades.
- **US-04 (FR-04):** Como usuário da aplicação, quero alternar entre Celsius e
  Fahrenheit para compreender as temperaturas na unidade que prefiro.
- **US-05 (FR-05):** Como usuário da aplicação, quero receber informações sobre
  carregamento e falhas e poder tentar novamente para entender e recuperar uma
  consulta que não foi concluída.

## Acceptance Criteria

**FR-01 — Buscar e selecionar cidade**

- **AC-01.1:** Given uma busca não vazia que retorna cidades. When a busca
  termina. Then a interface exibe no máximo cinco resultados, cada um com nome
  e os campos de estado/região e país presentes na resposta.
- **AC-01.2:** Given dois resultados com o mesmo nome. When a lista é exibida.
  Then cada resultado mantém seu próprio contexto geográfico e coordenadas.
- **AC-01.3:** Given uma lista de resultados. When a pessoa seleciona um deles.
  Then a consulta meteorológica usa as coordenadas desse resultado selecionado.
- **AC-01.4:** Given um termo composto somente por espaços. When a pessoa
  submete a busca. Then nenhum pedido de geocoding é enviado e a interface
  solicita um nome de cidade.
- **AC-01.5:** Given um termo válido sem correspondências. When o geocoding
  termina. Then a interface exibe “Nenhuma cidade encontrada” e não inicia uma
  consulta de previsão.
- **AC-01.6:** Given o termo `São José-dos-Campos`. When a busca é enviada.
  Then o parâmetro de busca é codificado corretamente e o serviço recebe o
  mesmo termo após decodificação.

**FR-02 — Consultar condições atuais**

- **AC-02.1:** Given uma cidade selecionada e uma resposta atual completa. When
  a consulta termina. Then a interface exibe temperatura, condição, umidade
  (%), vento (km/h), pressão (hPa) e precipitação (mm), associados à cidade.
- **AC-02.2:** Given uma resposta atual com um ou mais campos individuais
  ausentes. When os dados são exibidos. Then cada campo ausente mostra `—` e
  cada campo presente mostra seu valor e unidade definidos em FR-02.
- **AC-02.3:** Given uma resposta sem o objeto de condições atuais. When a
  validação da resposta termina. Then a consulta é tratada como falha, a
  interface mostra erro e oferece nova tentativa, em vez de exibir valores
  inválidos.

**FR-03 — Consultar previsão**

- **AC-03.1:** Given uma cidade selecionada e uma resposta com cinco datas.
  When a consulta termina. Then a previsão exibe exatamente cinco dias locais
  consecutivos, começando por hoje, cada um com mínima, máxima, condição e
  probabilidade de precipitação (%).
- **AC-03.2:** Given uma resposta que inclui o fuso horário da cidade. When as
  datas são exibidas. Then a primeira data corresponde ao dia atual nesse fuso e
  as quatro seguintes são os dias locais consecutivos.
- **AC-03.3:** Given uma resposta com um campo de um dia ausente. When a
  previsão é exibida. Then o campo ausente mostra `—` e os demais valores
  recebidos continuam visíveis.
- **AC-03.4:** Given uma resposta com menos de cinco datas ou sem o objeto de
  previsão diária. When a validação da resposta termina. Then a previsão não é
  apresentada como completa e a interface mostra erro com opção de nova
  tentativa.

**FR-04 — Alternar unidade de temperatura**

- **AC-04.1:** Given todas as temperaturas carregadas em Celsius, incluindo
  $20\,°C$. When a pessoa seleciona Fahrenheit. Then todas as temperaturas
  atuais e previstas usam $F = (C \times 9/5) + 32$, são arredondadas ao
  inteiro mais próximo e $20\,°C$ é exibido como $68\,°F$.
- **AC-04.2:** Given temperaturas exibidas em Fahrenheit a partir de dados
  originais em Celsius. When a pessoa seleciona Celsius. Then todas as
  temperaturas atuais e previstas voltam aos valores originais arredondados e a
  unidade ativa é °C.
- **AC-04.3:** Given dados meteorológicos já carregados. When a pessoa alterna
  a unidade uma ou mais vezes. Then nenhuma nova requisição é enviada e cada
  exibição é calculada a partir do valor original, sem acumular arredondamento.
- **AC-04.4:** Given uma nova sessão sem preferência persistida. When a
  aplicação é carregada. Then Celsius está selecionado.

**FR-05 — Comunicar estados da consulta**

- **AC-05.1:** Given uma busca ou consulta iniciada e ainda pendente. When a
  resposta não foi recebida. Then a interface exibe “Carregando o clima…” em
  uma região de status acessível.
- **AC-05.2:** Given uma requisição que falha por erro HTTP, rede ou resposta
  inválida. When a falha é recebida. Then o carregamento termina, a interface
  anuncia uma mensagem de erro em pt-BR e oferece o botão “Tentar novamente”.
- **AC-05.3:** Given um erro com a ação “Tentar novamente”. When a pessoa aciona
  o botão. Then a operação que falhou é repetida e o estado de carregamento é
  exibido até nova resposta ou timeout.
- **AC-05.4:** Given uma requisição sem resposta. When se completam 10 segundos.
  Then a requisição é cancelada, o carregamento termina e a interface exibe erro
  com opção de nova tentativa.
- **AC-05.5:** Given a pessoa selecionou outra cidade enquanto uma requisição
  anterior está pendente. When a resposta anterior chega depois. Then ela não
  substitui nem altera os dados da cidade atualmente selecionada.

## Non-Functional Requirements

- **NFR-01 — Idioma:** textos da interface e mensagens de estado devem estar em
  português do Brasil; datas devem seguir a localização pt-BR.
- **NFR-02 — Responsividade:** busca, seleção, unidade e dados meteorológicos
  permanecer utilizáveis entre 320 px e 1920 px, sem rolagem horizontal da
  página no fluxo principal.
- **NFR-03 — Acessibilidade:** atender WCAG 2.2 nível AA. Busca, seleção,
  alternância, loading e erro devem ter nomes/semântica acessíveis, operação por
  teclado e foco visível; informação não pode depender somente de cor. Os
  fluxos de busca, seleção, alternância e nova tentativa devem ser concluíveis
  por teclado, e a verificação automatizada de acessibilidade não deve reportar
  violações críticas ou graves.
- **NFR-04 — Clareza dos dados:** mostrar a unidade ativa junto ao controle de
  unidade e usar unidades consistentes em todos os dados. Associar resultados à
  cidade selecionada e não exibir respostas obsoletas.
- **NFR-05 — Integração externa:** usar Open-Meteo sem exigir conta, chave ou
  credenciais da pessoa usuária. Pesquisar uma cidade ou selecionar outra deve
  iniciar a consulta correspondente; não há atualização automática na v1.
- **NFR-06 — Tempo limite e recuperação:** cada requisição externa deve terminar
  com resposta ou timeout em até 10 segundos. Timeout, erro HTTP, falha de rede
  e resposta estruturalmente inválida devem liberar a interface para nova
  tentativa manual.
- **NFR-07 — Navegadores:** suportar as duas versões estáveis mais recentes de
  Chrome, Edge, Firefox e Safari, incluindo Safari em iOS e Chrome em Android.

## Traceability

| User Story | Acceptance Criteria | Requisitos não funcionais relevantes |
| --- | --- | --- |
| US-01 — Buscar e selecionar cidade (FR-01) | AC-01.1–AC-01.6 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, NFR-07 |
| US-02 — Consultar condições atuais (FR-02) | AC-02.1–AC-02.3 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, NFR-07 |
| US-03 — Consultar previsão (FR-03) | AC-03.1–AC-03.4 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, NFR-07 |
| US-04 — Alternar unidade (FR-04) | AC-04.1–AC-04.4 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-07 |
| US-05 — Estados e recuperação de consulta (FR-05) | AC-05.1–AC-05.5 | NFR-01, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, NFR-07 |

## Edge Cases

- **Busca vazia:** não chamar geocoding; solicitar o nome da cidade.
- **Nenhum resultado:** exibir “Nenhuma cidade encontrada” e não consultar a
  previsão.
- **Nomes iguais, acentos ou caracteres especiais:** codificar o termo da busca;
  exibir o contexto geográfico retornado e não selecionar localidade
  automaticamente.
- **Timeout:** após 10 segundos sem resposta, cancelar a requisição e oferecer
  nova tentativa.
- **Erro HTTP, rede, limite de uso ou JSON inválido:** encerrar loading,
  comunicar erro em pt-BR e oferecer nova tentativa manual.
- **Campo individual ausente:** exibir `—` no campo, sem inventar valor nem
  ocultar os demais campos válidos.
- **Objeto atual/diário ausente ou menos de cinco datas:** tratar a resposta
  como inválida, não apresentar previsão incompleta como completa e permitir
  nova tentativa.
- **Resposta atrasada de outra cidade:** ignorá-la para manter cidade e dados
  selecionados consistentes.
- **Alternância repetida de unidade:** sempre converter a partir dos valores
  originais, não dos valores arredondados exibidos.

## Assumptions

- A aplicação atende uma cidade por vez e a pessoa usuária seleciona a cidade
  antes de consultar seus dados meteorológicos.
- A busca da v1 aceita nomes de cidades e exibe até cinco resultados; não aceita
  CEP, endereço ou geolocalização.
- “Previsão de cinco dias” significa hoje mais os quatro dias seguintes.
- A previsão da v1 é diária, usa o fuso horário local da cidade e mostra mínima,
  máxima, condição e probabilidade máxima de precipitação.
- O Open-Meteo fornece temperaturas em Celsius; conversão e arredondamento são
  responsabilidades da apresentação.
- Celsius é o padrão a cada carregamento da aplicação; a escolha não persiste
  após recarregar.
- A frequência e a disponibilidade dos dados dependem do Open-Meteo; a v1 não
  atualiza dados automaticamente.
- Não há autenticação nem persistência de dados em servidor.
- Interface, mensagens e formatos de data usam pt-BR.

## Risks

- **Indisponibilidade, limites de uso ou cobertura do provedor:** podem impedir
  ou reduzir a completude das consultas; mitigar com estados de erro, nova
  tentativa e tratamento de campos ausentes.
- **Seleção de localidade incorreta:** nomes iguais ou semelhantes podem levar a
  dados de outra cidade; mitigar exibindo contexto geográfico nos resultados.
- **Dados incompletos ou atrasados:** podem reduzir a utilidade da consulta;
  mostrar `—` para campos ausentes, rejeitar respostas estruturais inválidas e
  não substituir dados da cidade atual por respostas atrasadas.
- **Inconsistência de unidades:** conversão aplicada apenas a parte das
  temperaturas pode confundir; os critérios de FR-04 verificam unidade e
  conversão em todas as temperaturas exibidas.
- **Limites, disponibilidade e termos do provedor:** podem afetar consultas ou
  publicação; validar cotas, disponibilidade e condições de atribuição do
  Open-Meteo antes do lançamento.
- **Datas e fuso horário:** usar o fuso incorreto pode rotular a previsão no dia
  errado; usar as datas locais fornecidas para a localização quando disponíveis.

## Out of Scope

- Autenticação, contas de usuário ou dados persistidos em servidor.
- Favoritos, histórico de consultas ou sincronização entre dispositivos.
- Busca por CEP, endereço, estado/região ou país sem nome de cidade; localização
  automática por GPS ou endereço IP.
- Exibição ou comparação simultânea de várias cidades.
- Previsão horária ou para além dos cinco dias definidos.
- Atualização automática periódica ou controle dedicado para atualizar dados; a
  nova consulta ocorre ao selecionar uma cidade, e a repetição manual limita-se
  a recuperar falhas.
- Provedor meteorológico alternativo ou fallback automático em caso de
  indisponibilidade do Open-Meteo.
- Alertas meteorológicos, notificações push ou recomendações de atividades.
- Mapas, radar, dados meteorológicos em tempo real de terceiros ou comparação
  com dados de outros provedores.
- Aplicativos nativos para iOS/Android ou instalação como PWA.
- Interface em idiomas diferentes de pt-BR na primeira versão.
- Garantia de funcionamento offline ou armazenamento em cache como recurso do
  produto.

## Open Questions

- Antes do lançamento, quais cotas, metas de disponibilidade e condições de
  atribuição do Open-Meteo se aplicam ao ambiente de produção?