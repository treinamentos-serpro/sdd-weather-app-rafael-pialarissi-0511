# Discovery: Aplicação de Previsão do Tempo

## Contexto

A empresa solicitou uma aplicação para que usuários consultem informações meteorológicas de cidades. O produto deve apresentar o clima atual e a previsão de hoje mais os quatro dias seguintes, permitir alternância entre Celsius e Fahrenheit e funcionar em dispositivos móveis. A interface será em pt-BR e os dados virão do Open-Meteo. O público-alvo e alguns detalhes de interação ainda precisam ser definidos.

## Requisitos Funcionais

- **RF-01 — Buscar cidades:** permitir que o usuário pesquise uma cidade e selecione o local desejado para consultar seus dados meteorológicos.
- **RF-02 — Consultar clima atual:** apresentar as condições meteorológicas atuais da cidade selecionada.
- **RF-03 — Consultar previsão:** apresentar a previsão do tempo para hoje e os quatro dias seguintes.
- **RF-04 — Alternar unidades:** permitir alternar entre Celsius e Fahrenheit e apresentar as temperaturas na unidade escolhida.
- **RF-05 — Usar em dispositivos móveis:** permitir a consulta e a interação com as funcionalidades principais em dispositivos móveis.

## Requisitos Não-Funcionais

- **RNF-01 — Responsividade:** a interface deve se adaptar a telas móveis sem impedir a busca ou a leitura das informações meteorológicas.
- **RNF-02 — Usabilidade:** busca, seleção da cidade e alternância de unidade devem ser compreensíveis e operáveis em telas pequenas.
- **RNF-03 — Clareza dos dados:** temperaturas devem indicar claramente a unidade selecionada; informações devem estar associadas à cidade consultada.
- **RNF-04 — Dependência de dados externos:** a solução dependerá de uma fonte de dados meteorológicos; disponibilidade, atualização e cobertura dessa fonte devem ser avaliadas durante o planejamento.

## Riscos

- Cidades com nomes iguais ou semelhantes podem levar à seleção de uma localização incorreta.
- Indisponibilidade, atraso ou cobertura limitada da fonte de dados pode impedir ou prejudicar a consulta.
- O briefing não define a granularidade da previsão (por exemplo, diária ou por horário), o que pode gerar expectativas diferentes.
- A apresentação em dispositivos móveis pode ficar difícil de ler se houver muitos dados ou telas estreitas.
- A alternância de unidade pode exibir valores inconsistentes se não for aplicada de forma uniforme ao clima atual e à previsão.

## Perguntas em Aberto

- Qual é o público-alvo e há requisitos de região além do idioma pt-BR para a primeira versão?
- A busca deve aceitar somente nomes de cidades ou também países, estados, códigos postais e localização atual do usuário?
- Como diferenciar resultados com o mesmo nome (por exemplo, cidade, estado e país)?
- Quais condições e métricas devem ser exibidas no clima atual e na previsão?
- A previsão de hoje mais quatro dias deve ser resumida por dia ou detalhada por horário?
- A unidade selecionada pelo usuário deve permanecer após recarregar ou reabrir a aplicação? Se sim, pode ser armazenada localmente no dispositivo?
- Há requisitos de acessibilidade, navegadores/dispositivos suportados ou conexão mínima?
- Quais são as expectativas de atualização dos dados, tempo de resposta e comportamento quando o Open-Meteo estiver indisponível?

## Suposições

- A primeira versão permite consultar uma cidade por vez.
- A pesquisa e a seleção da cidade precedem a exibição dos dados correspondentes.
- Celsius e Fahrenheit se aplicam a todas as temperaturas exibidas na aplicação, com Celsius como unidade inicial.
- “Previsão de 5 dias” significa hoje mais os quatro dias seguintes; o detalhamento diário ou por horário ainda precisa ser confirmado.
- A experiência móvel é parte da mesma aplicação, e não um produto separado.
- Requisitos quantitativos de desempenho, disponibilidade, acessibilidade e suporte de dispositivos serão definidos após alinhamento com as partes interessadas.

## Decisões

- **Fonte de dados: Open-Meteo, sem API key.** Justificativa: adotar uma fonte sem chave simplifica a integração e evita gestão de credenciais para esse serviço. Resolve a escolha inicial do provedor de dados; cobertura, atualização, limites de uso e comportamento em indisponibilidade continuam sujeitos a validação.
- **Definição de “5 dias”: hoje mais os quatro dias seguintes.** Justificativa: fixa o período da previsão em cinco dias-calendário, incluindo o dia atual. Resolve a ambiguidade sobre quais dias compõem a previsão; a apresentação diária ou por horário permanece em aberto.
- **Unidade padrão: Celsius.** Justificativa: define um valor inicial consistente com a experiência em pt-BR. Resolve a unidade exibida na primeira consulta; a preferência deve continuar alternável para Fahrenheit. Persistência da escolha após recarregar ainda precisa ser decidida.
- **Sem autenticação e sem persistência de servidor.** Justificativa: mantém a primeira versão sem contas de usuário e sem armazenamento de dados de usuário no servidor. Resolve as dúvidas sobre login e persistência no servidor; não determina se preferências serão salvas localmente no dispositivo.
- **Idioma da interface: pt-BR.** Justificativa: define o idioma e as convenções linguísticas da primeira versão. Resolve a dúvida sobre o idioma da UI; outros requisitos de regionalização, como abrangência geográfica e formatos específicos de dados, ainda podem ser esclarecidos.
