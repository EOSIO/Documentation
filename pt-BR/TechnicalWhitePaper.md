# EOS.IO White Paper Técnico

**26 de Junho de 2017**

**Resumo:** O software EOS.IO introduz uma nova arquitectura de blockchain projetada para permitir escalabilidade vertical e horizontal de aplicativos descentralizados. Isto é conseguido através da criação de um estrutura similar a um sistema operacional sobre o qual aplicações podem ser construídas. O software fornece conta, autenticação, banco de dados, comunicação assíncrona e o agendamento da execução de aplicações em centenas de núcleos de CPU ou clusters. A tecnologia resultante é uma arquitectura de blockchain que escala a milhões de transações por segundo, elimina taxas de usuário e permite a implantação rápida e simples de aplicações decentralizadas.

**ATENÇÃO: TOKENS CRUPTOGRÁFICOS REFERIDOS NESTE DOCUMENTO SE REFEREM A TOKENS CRIPTOGRÁFICOS DE UM BLOCKCHAIN LANÇADO QUE ADOTA O SOFTWARE EOS.IO. ELES NÃO SE REFEREM AOS TOKENS COMPATÍVEIS COM ERC-20 QUE ESTÃO SENDO DISTRIBUÍDOS NO BLOCKCHAIN ETHEREUM EM CONEXÃO COM A DISTRIBUIÇÃO DE TOKENS EOS.**

Copyright © 2017 block.one

Sem permissão, qualquer um pode usar, reproduzir ou distribuir qualquer material em este documento para usos não comerciais e educacionais (ex. sem taxa ou fins comerciais) desde que a fonte original e o aviso de direitos autorais sejam citados.

**AVISO LEGAL:** Este White Paper Técnico do EOS.IO é apenas para fins informativos. block.one não garante a precisão das conclusões neste white paper, e este white paper é fornecido "como está". block.one não faz e expressamente renuncia todas as representações e garantias, expressas, implícitas, estatuária ou de outra forma, de qualquer maneira, incluindo, mas não limitado a: (i) garantia de comercialização, adequação para algum propósito em particular, adequação, uso, título ou não infração; (ii) que o conteúdo deste Whitepaper estão livres de erro; e (iii) que tal conteúdo não irá infringir direitos de terceiros. block.one e seus afiliados não terão responsabilidade por danos de qualquer espécie resultado do uso, referencia à, ou dependência nesse Whitepaper ou qualquer um dos conteúdos contidos no mesmo, mesmo se advertido da possibilidade de tais danos. Em nenhum evento block.one ou seus afiliados será responsável à qualquer pessoa ou entidade por qualquer dano, perda, responsabilidades, custos ou gastos de qualquer espécie, direta ou indireta, consequentes, compensatória, incidental, atual, exemplar, punitivo ou especial para o uso de, referente à, ou confiança nesse Whitepaper ou qualquer conteúdo contidos no mesmo, incluindo, sem limitações, qualquer perda de negócios, receitas, lucros, informações, uso, boa vontade, ou outra perdas intangíveis.

- [Background](#background)
- [Requisitos para Aplicações Blockchain](#requirements-for-blockchain-applications) 
  - [Suporte a Milhões de Usuários](#support-millions-of-users)
  - [Uso Gratuito](#free-usage)
  - [Simplicidade nas Atualizações e Recuperação de Falhas](#easy-upgrades-and-bug-recovery)
  - [Baixa Latência](#low-latency)
  - [Desempenho Sequencial](#sequential-performance)
  - [Desempenho Paralelo](#parallel-performance)
- [Algoritmo de Consenso (DPOS)](#consensus-algorithm-dpos) 
  - [Confirmação da Transação](#transaction-confirmation)
  - [Transação como Prova de Participação (TaPoS)](#transaction-as-proof-of-stake-tapos)
- [Contas](#accounts) 
  - [Mensagens & Handlers](#messages--handlers)
  - [Gerenciamento de Permissões Baseadas em Papeis](#role-based-permission-management) 
    - [Níveis de Permissão Nomeados](#named-permission-levels)
    - [Grupos de Handlers de Mensagens Nomeados](#named-message-handler-groups)
    - [Mapeamento de Permissões](#permission-mapping)
    - [Avaliação das Permissões](#evaluating-permissions) 
      - [Grupos de Permissão Padrão](#default-permission-groups)
      - [Avaliação Paralela de Permissões](#parallel-evaluation-of-permissions)
  - [Mensagens com Atraso Obrigatório](#messages-with-mandatory-delay)
  - [Recuperação de Chaves Roubadas](#recovery-from-stolen-keys)
- [Execução Paralela Determinística de Aplicações](#deterministic-parallel-execution-of-applications) 
  - [Minimizando a Latência de Comunicação](#minimizing-communication-latency)
  - [Handlers de Mensagens Somente de Leitura](#read-only-message-handlers)
  - [Transações Atômicas com Múltiplas Contas](#atomic-transactions-with-multiple-accounts)
  - [Avaliação Parcial do Estado do Blockchain](#partial-evaluation-of-blockchain-state)
  - [Agendamento Subjetivo baseado em Melhor Esforço](#subjective-best-effort-scheduling)
- [Modelo de Token e Uso de Recursos](#token-model-and-resource-usage) 
  - [Medições Objetivas e Subjetivas](#objective-and-subjective-measurements)
  - [Receptor Paga](#receiver-pays)
  - [Delegando Capacidade](#delegating-capacity)
  - [Separando os Custos de Transação do Valor do Token](#separating-transaction-costs-from-token-value)
  - [Custos de Armazenamento do Estado](#state-storage-costs)
  - [Recompensas de Bloco](#block-rewards)
  - [Aplicações de Benefício da Comunidade](#community-benefit-applications)
- [Governança](#governance) 
  - [Congelamento de Contas](#freezing-accounts)
  - [Alterando o Código da Conta](#changing-account-code)
  - [Constituição](#constitution)
  - [Atualizando o Protocolo & Constituição](#upgrading-the-protocol--constitution) 
    - [Mudanças de Emergência](#emergency-changes)
- [Scripts & Máquinas Virtuais](#scripts--virtual-machines) 
  - [Mensagens Definidas por Esquema](#schema-defined-messages)
  - [Banco de Dados Definido por Esquema](#schema-defined-database)
  - [Separando a Autenticação da Aplicação](#separating-authentication-from-application)
  - [Arquitetura Independente de Máquina Virtual](#virtual-machine-independent-architecture) 
    - [Web Assembly (WASM)](#web-assembly-wasm)
    - [Máquina Virtual de Ethereum (EVM)](#ethereum-virtual-machine-evm)
- [Comunicação Inter Blockchain](#inter-blockchain-communication) 
  - [Provas de Merkle para Validação Leve de Cliente (LCV)](#merkle-proofs-for-light-client-validation-lcv)
  - [Latência de Comunicação Interchain](#latency-of-interchain-communication)
  - [Proof of Completeness](#proof-of-completeness)
- [Conclusão](#conclusion)

# Contexto

A tecnologia Blockchain foi introduzida em 2008 com o lançamento da moeda bitcoin e desde então empreendedores e desenvolvedores tem intentado generalizar a tecnologia para oferecer suporte a uma mais ampla gama de aplicações num único blockchain.

Enquanto varias plataformas de blockchain têm tido dificuldades para suportar aplicações descentralizadas em funcionamento, blockchains especializados por aplicação como o exchange descentralizado BitShares (2014) e a plataforma de mídia social Steem (2016) tornaram-se blockchains intensamente utilizados com dezenas de milhares de usuários ativos diariamente. Eles conseguiram isto aumentando o desempenho para milhares de transações por segundo, reduzindo a latência para 1,5 segundos, eliminando taxas, e proporcionando uma experiência de usuário semelhante aos actualmente prestados pelos serviços centralizados existentes.

Plataformas de blockchain existentes, estão sobrecarregadas por grandes taxas e capacidade computacional limitada que impede a adoção generalizada do blockchain.

# Requisitos para Aplicações Blockchain

Para ser adotadas amplamente, aplicações que rodam no blockchain exigem uma plataforma que é suficientemente flexível para atender aos seguintes requisitos:

## Suporte a Milhões de Usuários

Disromper empresas como Ebay, Uber, AirBnB e Facebook, requer uma tecnologia blockchain capaz de lidar com dezenas de milhões de usuários ativos diariamente. Em certos casos, os aplicativos podem não funcionar a menos que uma massa crítica de usuários é atingida e, portanto, uma plataforma que pode lidar com uma quantidade massiva de usuários é fundamental.

## Uso Gratuito

Os desenvolvedores de aplicativos precisam da flexibilidade para oferecer aos usuários serviços gratuitos; usuários não devem ter que pagar para usar a plataforma ou se beneficiar de seus serviços. Uma plataforma de blockchain que é gratuita para ser usada pelos usuários provavelmente ganhará mais ampla adoção. Desenvolvedores e empresas podem criar estratégias de monetização eficaz.

## Atualizações fáceis e Recuperação de Falhas

Empresas construindo aplicações baseadas em blockchain precisam de flexibilidade para melhorar suas aplicações com novas funcionalidades.

Todos os softwares mais complexos são sujeitos a errores, mesmo com a verificação formal mais rigorosa. A plataforma deve ser robusta o suficiente para permitir a correção de erros quando eles ocorrerem inevitavelmente.

## Baixa Latência

Uma boa experiência do usuário exige feedback confiável com atraso de não mais que alguns segundos. Atrasos maiores frustram os usuários e fazem que as aplicações construídas sobre um blockchain sejam menos competitivas com alternativas não baseadas no blockchain.

## Desempenho Sequencial

Existem alguns aplicativos que não podem ser implementados com algoritmos paralelos devido a passos sequencialmente dependentes. Aplicações tais como exchanges precisam de bastante desempenho sequencial para lidar com grandes volumes e, portanto, uma plataforma com desempenho rápido sequencial é necessária.

## Desempenho Paralelo

Aplicações de grande escala precisam dividir a carga de trabalho entre vários processadores e computadores.

# Algoritmo de Consenso (DPOS)

O software EOS.IO utiliza o único algoritmo de consenso descentralizado capaz de atender aos requisitos de desempenho de aplicativos sobre o blockchain, [Prova de Participação Delegada (DPOS)](https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper). Neste algoritmo, quem detêm tokens de um blockchain adotando o software EOS pode selecionar produtores de bloco através de um sistema de votação de aprovação continua e qualquer um pode optar por participar na produção de blocos e será dada a oportunidade de produzir blocos proporcionais aos votos totais recebidos em relação a todos os outros produtores. Para blockchains privados os gestores poderiam usar os tokens para adicionar e remover o pessoal de TI.

O software EOS.IO permite que blocos sejam produzidos exatamente a cada 3 segundos e exatamente um único produtor está autorizado a produzir um bloco em qualquer ponto no tempo. Se o bloco não é produzido no horário agendado, então o bloco para aquele slot de tempo é ignorado. Quando um ou mais blocos são ignorados, há uma lacuna de 6 ou mais segundos no blockchain.

Usando o software EOS.IO blocos são produzidos em rodadas de 21. No início de cada rodada 21 produtores de blocos únicos são escolhidos. Os top 20 pela aprovação total são automaticamente escolhidos a cada rodada e o último produtor é escolhido proporcional ao seu número de votos em relação a outros produtores. Os produtores selecionados são embaralhados usando um número pseudo-aleatório derivado da hora do bloco. Esse embaralhamento é feito para garantir que todos os produtores mantenham conectividade equilibrada para todos os outros produtores.

Se um produtor perde um bloco e não produziu qualquer bloco nas últimas 24 horas, eles são removidos da consideração até eles notificarem a blockchain da sua intenção de começar a produzir blocos novamente. Isso garante que a rede opera sem problemas, minimizando o número de blocos que falhou por não agendar aqueles que se provou que não são confiáveis.

Em condições normais um blockchain DPOS não experimenta qualquer fork porque os produtores do bloco cooperarem para produzir blocos, ao invés de competir. No caso há uma bifurcação, consenso mudará automaticamente para a cadeia mais longa. Esta métrica funciona porque a taxa em que blocos são adicionados a um fork de cadeia blockchain é diretamente correlacionada com a porcentagem de produtores do bloco que compartilham o mesmo consenso. Em outras palavras, um fork de blockchain com mais produtores nele irá crescer em comprimento mais rápido do que um com poucos produtores. Além disso, nenhum produtor de bloco deve produzir blocos em dois forks ao mesmo tempo. Se um produtor de bloco é pego fazendo isto então o produtor de bloco será provavelmente eliminado. Evidencias criptográficas dessa produção dupla podem também ser usadas para remover automaticamente os abusadores.

## Confirmação da Transação

Os blockchains DPOS típicos tem 100% de participação dos produtores de blocos. Uma transação pode ser considerada confirmada com 99,9% de certeza, após uma média de 1,5 segundos de tempo de transmissão.

Existem alguns casos extraordinários onde um bug de software, congestionamento da Internet ou um produtor de bloco malicioso irá criar dois ou mais forks. Para ter absoluta certeza que uma transação é irreversível, um nó pode optar por esperar a confirmação de 15 dos 21 produtores de blocos. Baseado em uma configuração típica do software EOS.IO, isto levará a uma média de 45 segundos em circunstâncias normais. Por padrão, todos os nós irão considerar um bloco confirmado por 15 dos 21 produtores irreversível e não vão mudar para um fork que exclui aquele bloco independentemente do comprimento.

É possível para um nó avisar os usuários que há uma alta probabilidade de que eles estão em um fork minoritário dentro de 9 segundos do início de um fork. Depois de 2 blocos consecutivos perdidos há uma probabilidade de 95%, que um nó é um fork minoritário. Com 3 blocos perdidos consecutivos há um 99% de certeza de ser um fork minoritário. É possível gerar um modelo preditivo robusto que irá utilizar a informação sobre quais nós perdeu, recentes taxas de participação e de outros fatores que rapidamente avisar os operadores que algo está errado.

A resposta a tal aviso depende inteiramente da natureza das transações de negócio, mas a resposta mais simples é esperar por 15/21 confirmações até as paradas de advertência.

## Transação como Prova de Participação (TaPoS)

O software EOS.IO requer que cada transação inclua o hash de um cabeçalho de bloco recente. Este hash serve dois propósitos:

1. impede que uma repetição de uma transação num fork que não incluei o bloco referenciado; e
2. sinaliza a rede que um usuário específico e sua participação estão num fork específico.

Ao longo do tempo, todos os usuários acabam diretamente confirmando o blockchain, o que torna difícil forjar falsas correntes já que o falsificador não será capaz de migrar as operações da corrente legítima.

# Contas

O software EOS.IO permite que todas as contas sejam referenciadas por um nome humanamente legível exclusivo de 2 a 32 caracteres de comprimento. O nome é escolhido pelo criador da conta. Todas as contas devem ser financiadas com o saldo mínimo no momento que elas são criadas para cobrir o custo de armazenamento dos dados da conta. Os nomes de conta também oferecem suporte a namespaces tal que o proprietário da conta @domain é o único que pode criar a conta @user.domain.

Em um contexto descentralizado, os desenvolvedores de aplicativos vão pagar o custo nominal de criação da conta para inscrever um novo usuário. As empresas tradicionais já gastam quantias significativas de dinheiro por cliente que eles adquirem na forma de serviços de publicidade, enciclopédia, etc. O custo do financiamento de uma nova conta no blockchain deve ser insignificante em comparação. Felizmente, não há nenhuma necessidade de criar contas de usuários que já se inscreveram por outro aplicativo.

## Mensagens & Handlers

Cada conta pode enviar mensagens estruturadas para outras contas e pode definir scripts para manipular mensagens quando elas são recebidas. O software EOS.IO dá para cada conta seu próprio banco de dados particular, que só pode ser acessado por seus próprios handlers de mensagens. Scripts de manipulação de mensagens também podem enviar mensagens para outras contas. É a combinação de mensagens e manipuladores de mensagens automatizadas como EOS.IO define contratos inteligentes.

## Gerenciamento de Permissões Baseadas em Papeis

Gestão de permissão envolve determinar se ou não uma mensagem esta devidamente autorizada. A forma mais simples de gerenciamento de permissão é verificar que uma transação tem as assinaturas necessárias, mas isto implica que as assinaturas requeridas já são conhecidas. Geralmente autoridade é vinculada a indivíduos ou grupos de indivíduos e muitas vezes é compartimentada. O software EOS.IO fornece um sistema de gestão de permissão declarativa que da para as contas um alto nível controle com boa granularidade sobre quem pode fazer o quê e quando.

É crítico que o gerenciamento de autenticação e autorização seja padronizado e separado da lógica de negócios do aplicativo. Isso permite que ferramentas a serem desenvolvidas para gerenciar permissões de forma geral e também oferecem oportunidades significativas para otimização de desempenho.

Cada conta pode ser controlada por qualquer combinação ponderada de outras contas e chaves privadas. Isto cria uma estrutura de autoridade hierárquica que reflete como as permissões são organizadas na realidade e facilita mais do que nunca o controle por vários usuários sobre os fundos. Controle multi-usuário é o fator único com maior contribuição para a segurança, e, quando usado corretamente, pode eliminar consideravelmente o risco de roubo devido a hacking.

O software EOS.IO permite que as contas definam qual a combinação de chaves e/ou contas podem enviar um tipo específico de mensagem para outra conta. Por exemplo, é possível ter uma chave para uma conta de usuário mídia social e outro para acesso ao exchange. É até possível dar outras contas permissão para agir em nome de uma conta de usuário sem atribuir-lhes as chaves.

### Níveis de Permissão Nomeados

<img align="right" src="http://eos.io/wpimg/diagram3.png" width="228.395px" height="300px" />

Usando o software EOS.IO, contas podem definir níveis de permissões nomeadas cada um dos quais pode ser derivado de permissões nomeada de um nível mais alto. Cada nível de permissões nomeadas define uma autoridade; uma autoridade é uma verificação de múltiplas assinatura consistindo de chaves e/ou níveis de permissão nomeados de outras contas. Por exemplo, o nível de permissão de "Amigo" de uma conta pode ser definido para a conta para ser controlado igualitariamente por qualquer um dos amigos da conta.

Outro exemplo é o blockchain Steem que tem marretados três níveis de permissão nomeados: proprietário, ativo e postar. A permissão de postar só pode realizar ações sociais, tais como votação e postar, enquanto a permissão ativa pode fazer tudo, exceto a mudança do proprietário. A permissão de proprietário é para armazenamento frio e é capaz de fazer tudo. O software EOS.IO generaliza este conceito, permitindo que cada titular de conta defina sua própria hierarquia, bem como o agrupamento de ações.

### Grupos de Handlers de Mensagens Nomeados

O software EOS.IO permite que cada conta organize os seus próprios manipuladores de mensagens em grupos nomeados e aninhados. Estes grupos de manipuladores de mensagem nomeados podem ser referenciados por outras contas quando eles configuram seus níveis de permissão.

O grupo de manipulador de mensagem de nível mais alto é o nome da conta e o nível mais baixo é o tipo de mensagem individual, sendo recebido pela conta. Esses grupos podem ser referenciados da seguinte forma: **@accountname.groupa.subgroupb.MessageType**.

Sob este modelo, é possível para um contrato de câmbio agrupar a criação da ordem e o seu cancelamento separadamente de depositar e retirar. Este agrupamento pelo contrato de câmbio é uma conveniência para os usuários do exchange.

### Mapeamento de Permissões

O software EOS.IO permite que cada conta defina um mapeamento entre um Grupo Nomeado de Manipuladores de Mensagens de qualquer conta e seu Nível de Permissão Nomeado. Por exemplo, um titular de conta poderia mapear a aplicação de mídia social do titular da conta para o grupo de permissão do titular da conta "Amigo". Com esse mapeamento, qualquer amigo poderia postar como titular da conta na mídia social do titular da conta. Mesmo que eles postariam como titular da conta, ainda usariam suas próprias chaves para assinar a mensagem. Isto significa que sempre é possível identificar quais amigos usaram a conta e de que maneira.

### Avaliando Permissões

Quando se esta entregando uma mensagem do tipo "**Action**", de **@alice** para **@bob** o software EOS.IO primeiro verificará se **@alice** definiu um mapeamento de permissão para **@bob.groupa.subgroup.Action**. Se nada for encontrado então será verificado se existem mapeamentos para **@bob.groupa.subgroup**, depois **@bob.groupa** e, por último, **@bob**. Se não for encontrada nenhuma correspondência, então será assumido o mapeamento do grupo de permissão nomeada **@alice.active**.

Depois de um mapeamento ser identificado em seguida é validada a autoridade da assinatura usando o processo de múltiplas assinaturas e autoridade associada com a permissão nomeada. Se isso falhar, então ele percorre a permissão do pai e, finalmente, a permissão do proprietário, **@alice.owner**.

<img align="center" src="http://eos.io/wpimg/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### Grupos de Permissão Padrão

A tecnologia EOS.IO também permite que todas as contas tenham um "owner" que pode fazer tudo e um grupo "active" que pode fazer tudo, exceto alterar o grupo proprietário. Todos os outros grupos de permissão são derivados de "active".

#### Avaliação Paralela de Permissões

O processo de avaliação de permissão é "somente leitura" e as alterações de permissões feitas por transações não terão efeito até ao final de um bloco. Isto significa que todas as chaves e avaliações de permissão para todas as transações podem ser executadas em paralelo. Além disso, isto significa que uma rápida validação da permissão é possível sem iniciar a custosa lógica do aplicativo que teria de ser revertida. Por fim, significa que as permissões de transação podem ser avaliadas enquanto as transações pendentes são recebidas e não precisam ser re-avaliadas quando elas são aplicadas.

Quando consideramos todas as coisas, a verificação de permissão representa uma percentagem significativa do poder computacional necessário para validar as transações. Fazer disto um processo solo leitura e trivialmente paralelizável permite um aumento dramático no desempenho.

Quando se esta re-processando o blockchain para regenerar o estado determinístico do log de mensagens não há nenhuma necessidade de avaliar as permissões novamente. O fato de que uma transação estar incluída em um bloco conhecidamente bom é suficiente para ignorar esta etapa. Isto reduz drasticamente a carga computacional associada a re-processamento de um blockchain sempre em crescimento.

## Mensagens com Atraso Obrigatório

Tempo é um componente crítico de segurança. Na maioria dos casos, não é possível saber se uma chave privada foi roubada até que ela tenha sido usada. Segurança baseada em tempo é ainda mais crítica quando as pessoas têm aplicações que requerem que as chaves sejam mantidas em computadores conectados à internet para o seu uso diário. O software EOS.IO permite que os desenvolvedores de aplicativos indiquem que certas mensagens devam esperar um período mínimo de tempo depois de ser incluídas em um bloco antes que elas possam ser aplicadas. Durante este tempo pode ser canceladas.

Os usuários podem então receber avisos através de e-mail ou mensagem de texto quando uma dessas mensagens é transmitida. Se eles não autorizaram isso, então eles podem usar o processo de recuperação de conta para recuperar sua conta e cancelar a mensagem.

O atraso necessário depende de quão sensível é uma operação. Pagar por um café não pode ter nenhum atraso e ser irreversível em segundos, enquanto que comprar uma casa pode exigir um período de compensação de 72 horas. Transferir uma conta inteira para novo controle pode demorar até 30 dias. Os atrasos exatos escolhidos devem ser definidos pelos desenvolvedores de aplicativos e os usuários.

## Recuperação de Chaves Roubadas

O software EOS.IO fornece aos usuários uma maneira de restaurar o controle da sua conta quando as chaves são roubadas. Um proprietário de conta pode usar qualquer chave de proprietário que era ativo nos últimos 30 dias juntamente com a aprovação de seu parceiro de recuperação de conta designado para redefinir a chave do proprietário na sua conta. O parceiro de recuperação de conta não pode redefinir o controle da conta sem a ajuda do proprietário.

Não há nada para um hacker ganhar ao tentar passar pelo processo de recuperação, porque eles já "controlam" a conta. Além disso, se eles passarem pelo processo, o parceiro de recuperação demandaria identificação e autenticação de vários fatores (telefone e e-mail). Isso provavelmente iria comprometer o hacker ou nada ganharia no processo.

Este processo também é muito diferente de um simples arranjo de múltiplas assinaturas. Numa transação múlti-assinaturas, há outra empresa, que é a parte em cada transação que é executada, mas com o processo de recuperação, o agente é apenas uma parte para o processo de recuperação e não tem poder sobre as transações do dia a dia. Isto drasticamente reduz os custos e obrigações legais para todos os envolvidos.

# Execução Paralela Determinística de Aplicações

O consenso do Blockchain depende do comportamento determinístico (reprodutível). Isso significa que toda execução paralela deve ser livre da utilização de mutexes ou outros primitivos de locking. Sem os locks, deve haver alguma maneira de garantir que todas as contas só podem ler e escrever seu próprio banco de dados particular. Significa também que cada conta processa mensagens sequencialmente e que paralelismo será no nível das contas.

Em um blockchain baseado no EOS.IO, é o trabalho do produtor bloco organizar a entrega de mensagens em threads independentes para que elas possam ser avaliadas em paralelo. O estado de cada conta depende somente das mensagens entregadas a ela. A agenda é a saída de um produtor de bloco e será deterministicamente executada, mas o processo para gerar a agenda não precisa ser determinístico. Isto significa que os produtores do bloco podem utilizar algoritmos paralelos para agendar transações.

Parte de execução em paralelo significa que quando um script gera uma nova mensagem, ela não é entregue imediatamente, em vez disso é agendada para ser entregue no próximo ciclo. A razão pela qual que ela não pode ser entregue imediatamente é porque o receptor pode estar ativamente modificando seu próprio estado em outra thread.

## Minimizando a Latência de Comunicação

Latência é o tempo que leva para uma conta enviar uma mensagem para outra conta e então receber uma resposta. O objetivo é permitir que duas contas troquem mensagens entre elas dentro de um único bloco, sem ter que esperar 3 segundos entre cada mensagem. Para permitir isto, o software EOS.IO divide cada bloco em ciclos. Cada ciclo é dividido em threads e cada thread contém uma lista de transações. Cada transação contém um conjunto de mensagens a ser entregues. Essa estrutura pode ser visualizada como uma árvore onde camadas alternadas são processadas sequencialmente e em paralelo.

        Bloco
    
          Ciclos (sequencial) 
    
             Threads (paralelo)
    
                Transações (sequencial)
    
                   Mensagens (sequencial)
    
                      Receptor e Contas Notificadas (paralelo)
    

Transações geradas em um ciclo podem ser entregue em qualquer ciclo subsequente ou bloco. Os produtores do bloco vão continuar adicionando ciclos para um bloco até que passe o tempo de relógio máximo ou não exista nenhuma nova transação gerada para entregar.

É possível usar a análise estática de um bloco para verificar se, dentro de um determinado ciclo não existem duas threads que contenham transações que modificam a mesma conta. Desde que esse invariante é mantida, um bloco pode ser processado por todos os threads de execução em paralelo.

## Handlers de Mensagens Somente de Leitura

Algumas contas podem ser capazes de processar uma mensagem na base de aprovação/reprovação sem modificar seu estado interno. Se este for o caso esses manipuladores podem ser executados em paralelo enquanto apenas manipuladores de mensagens somente leitura de uma determinada conta estão incluídos em um ou mais threads dentro de um determinado ciclo.

## Transações Atômicas com Múltiplas Contas

Às vezes é desejável assegurar que as mensagens são entregues e aceites pela várias contas atomicamente. Neste caso, ambas as mensagens são colocadas em uma única transação, e ambas as contas serão atribuídas o mesmo thread e as mensagens serão aplicadas sequencialmente. Esta situação não é ideal para desempenho e quando se trata de "faturamento" usuários para o uso, eles serão cobrados pelo número de contas únicas referenciado por uma transação.

Por motivos de desempenho e custo é melhor minimizar as operações atômicas envolvendo duas ou mais contas muito utilizadas.

## Avaliação Parcial do Estado do Blockchain

Para escalar a tecnologia de blockchains exige que os componentes sejam modulares. Nem todos deveriam ter que rodar tudo, especialmente se eles só precisam usar um pequeno subconjunto dos aplicativos.

Um desenvolvedor de aplicativos de exchange roda um nó completo com a finalidade de exibir o estado da troca para seus usuários. Este aplicativo do exchange não tem necessidade do estado associado com aplicações de mídia social. O software EOS.IO permite que qualquer nó completo escolha qualquer subconjunto de aplicativos para serem executados. Mensagens entregues para outras aplicações são ignoradas porque o estado do aplicativo é derivado inteiramente das mensagens que são entregues a ele.

Isto tem algumas implicações significativas na comunicação com outras contas. Mais significativamente, ele não pode ser considerado que o estado de outra conta é acessível na mesma máquina. Significa também que, embora seja tentador habilitar "locks" que permitam que uma conta chame de forma síncrona outra conta, esse padrão de design falha se a outra conta não esta residente na memória.

Todas as comunicações do estado entre as contas devem ser passada através de mensagens incluídas no blockchain.

## Agendamento Subjetivo baseado em Melhor Esforço

O software EOS.IO não pode obrigar os produtores do bloco a entregar qualquer mensagem para qualquer conta. Cada produtor de bloco avalia sua própria medida subjetiva da complexidade computacional e tempo necessário para processar uma transação. Isso se aplica se uma transação é gerada por um usuário ou automaticamente por um script.

Em um blockchain lançado que adota o software EOS.IO, no nível da rede todas as transações são cobradas a um custo fixo de largura de banda computacional independentemente se tomou .01ms ou um total de 10 ms para executá-lo. No entanto, cada produtor de bloco individual usando o software pode calcular usando seu próprio algoritmo e medições o seu uso de recursos. Quando um produtor de bloco conclui que uma conta ou transação consumiu uma quantidade desproporcional da capacidade computacional podem simplesmente rejeitar a transação ao produzir o seu próprio bloco; no entanto, eles ainda irão processar a transação se outros produtores do bloco consideram ela válida.

Em geral, enquanto um único produtor de bloco considera uma transação como válida e abaixo do limite de uso de recursos, então todos os outros produtores de bloco também irão aceitá-la, porem pode demorar até 1 minuto para a transação encontrar aquele produtor.

Em alguns casos, um produtor pode criar um bloco que inclui transações que estão uma ordem de magnitude fora dos intervalos aceitáveis. Neste caso o próximo produtor de bloco pode optar por rejeitar o bloco e a ligação vai ser quebrada pelo terceiro produtor. Isto não é diferente do que aconteceria se um grande bloco causasse atrasos na propagação da rede. A comunidade iria notar um padrão de abuso e eventualmente removeria os votos dos produtores desonestos.

Esta avaliação subjetiva de custo computacional libera a blockchain de ter que precisamente e deterministicamente medir quanto tempo leva para algo executar. Com este design não existe necessidade de contar com precisão as instruções o que drasticamente aumenta as oportunidades de otimização sem quebra de consenso.

# Modelo de Token e Uso de Recursos

**POR FAVOR NOTE: OS TOKENS CRIPTOGRÁFICOS REFERIDOS NESTE WHITE PAPER SE REFEREM A TOKENS CRIPTOGRÁFICOS EM UM BLOCKCHAIN LANÇADO QUE ADOTA O SOFTWARE EOS.IO. ELES NÃO SE REFEREM AOS TOKENS COMPATÍVEIS COM ERC-20 QUE ESTÃO SENDO DISTRIBUÍDOS NO BLOCKCHAIN ETHEREUM EM CONEXÃO COM A DISTRIBUIÇÃO DE TOKENS EOS.**

Todos os blockchains são recursos limitados e necessitam de um sistema para prevenir abusos. Com um blockchain que usa o software EOS.IO, existem três grandes classes de recursos que são consumidos por aplicações:

1. Largura de Banda e Armazenamento de Log (Disco);
2. Poder Computacional e Backlog Computacional (CPU); e
3. Armazenamento de Estado (RAM).

Largura de banda e poder computacional têm dois componentes, o uso instantâneo e o uso a longo prazo. Um blockchain mantém um log de todas as mensagens e este log é finalmente armazenado e baixável por todos os nós completos. Com o log de mensagens é possível reconstruir o estado de todas as aplicações.

A dívida computacional são os cálculos que devem ser executados para regenerar o estado a partir do log de mensagens. Se a dívida computacional cresce muito então torna-se necessário tirar fotos (snapshots) do estado do blockchain e descartar a história da blockchain. Se a dívida computacional cresce muito rápidamente então pode levar até 6 meses re-processar 1 ano de transações. É fundamental, portanto, que a dívida computacional seja gerida com cuidado.

Armazenamento do estado de Blockchain é informação acessível a partir da lógica do aplicativo. Inclui informações tais como livros de ordens e os saldos das contas. Se o estado nunca é lido pelo aplicativo, então não deve ser armazenado. Por exemplo, comentários e conteúdo do post de blog não são lidos pela lógica do aplicativo então eles não devem ser armazenados no estado de blockchain. Entretanto a existência de um post/comentário, o número de votos e outras propriedades ficam armazenados como parte do estado do blockchain.

Produtores de bloco publicam sua capacidade disponível para largura de banda, computação e estado. O software EOS.IO permite que cada conta consuma um percentual da capacidade disponível proporcional à quantidade de tokens, guardados em um contrato de 3 dias (3-day stacking contract). Por exemplo, se um blockchain baseado no software EOS.IO é lançado e se uma conta detém 1% dos tokens totais distribuídos nos termos de aquele blockchain, então essa conta tem o potencial para utilizar 1% da capacidade de armazenamento de estado.

Adotando o software EOS.IO em um blockchain lançado significa a largura de banda e capacidade computacional são alocados em uma base de reservas fracionárias, porque eles são transitórios (capacidade não utilizada não pode ser guardada para uso futuro). O algoritmo usado pelo software EOS.IO é semelhante ao algoritmo usado pelo Steem para calcular a taxa-limite do uso de largura de banda.

## Medições Objetivas e Subjetivas

Como discutido anteriormente, instrumentar o uso computacional tem um impacto significativo no desempenho e otimização; Portanto, todas as restrições de uso de recursos são, em última análise subjetivas, e a aplicação é feita pelos produtores do bloco de acordo com seus algoritmos individuais e estimativas.

Dito isso, há certas coisas que são triviais para medir objetivamente. O número de mensagens entregues e o tamanho dos dados armazenados no banco de dados interno são baratos para medir objetivamente. O software EOS.IO permite aos produtores de bloco aplicar o mesmo algoritmo sobre estas medidas objetivas, mas pode optar por aplicar algoritmos subjetivos mais rigorosos sobre medições subjetivas.

## Receptor Paga

Tradicionalmente, é o negócio que paga o espaço do escritório, poder computacional e outros custos necessários para gerir o negócio. O cliente compra produtos específicos do negócio e as receitas provenientes das vendas do produto são usadas para cobrir os custos da operação do negócio. Da mesma forma, nenhum site obriga seus visitantes a fazer micro-pagamentos para visitar seu site para cobrir os seus custos de hospedagem. Portanto, aplicativos descentralizados não devem forçar seus clientes a pagar o blockchain diretamente pelo uso do blockchain.

Um blockchain lançado que usa o software EOS.IO não exige que seus usuários paguem o blockchain diretamente para o seu uso e, portanto, não restringem ou impedem um negócio de determinar sua própria estratégia de monetização para os seus produtos.

## Delegando Capacidade

Um titular de tokens em um blockchain lançado que adota o software EOS.IO, que pode não ter uma necessidade imediata para consumir toda ou parte da largura de faixa disponível, pode dar ou alugar essa largura de banda não consumida a outros; os produtores do bloco executando o software EOS.IO em tal blockchain vão reconhecer esta delegação de capacidade e alocar largura de banda em conformidade.

## Separando os Custos de Transação do Valor do Token

Um dos principais benefícios do software EOS.IO é que a quantidade de largura de banda disponível para uma aplicação é inteiramente independente de qualquer preço token. Se um proprietário de uma aplicação possui um número relevante de tokens em um blockchain adotando EOS.IO, então a aplicação pode rodar indefinidamente dentro de um uso de estado e de largura de banda fixo. Neste caso, os desenvolvedores e os usuários são afetados de qualquer volatilidade de preços no mercado de token e, portanto, não dependente de preço (price feed). Em outras palavras, um blockchain que adota o EOS.IO permite aos produtores de bloco a naturalmente aumentar a largura de banda, computação e armazenamento disponível por token independente do valor do token.

Um blockchain usando o software EOS.IO também recompensa os produtores de bloco com tokens cada vez que produzem um bloco. O valor dos tokens terá impacto sobre a quantidade de largura de banda, armazenamento e computação, que um produtor pode ter recursos para comprar; Este modelo naturalmente alavanca o valor crescente dos tokens para aumentar o desempenho da rede.

## Custos de Armazenamento do Estado

Enquanto podem ser delegadas a largura de banda e de computação, o armazenamento do estado da aplicação exigirá a um desenvolvedor de aplicações possuir tokens até esse estado seja excluído. Se o estado nunca é eliminado os tokens são efetivamente retirados de circulação.

Cada conta de usuário requer uma certa quantidade de armazenamento; Portanto, todas as contas devem manter um saldo mínimo. Conforme a capacidade de armazenamento da rede aumenta esse saldo mínimo necessário ira cair.

## Recompensas de Bloco

Um blockchain que adota o EOS.IO recompensará com novos tokens a um produtor de bloco, cada vez que um bloco é produzido. Nestas circunstâncias, o número de tokens criados é determinado pela mediana do pagamento desejado, publicado por todos os produtores do bloco. O EOS.IO pode ser configurado para impor um topo nas recompensas dos produtores, tal que o aumento anual total de oferta de token não exceda 5%.

## Aplicações de Benefício da Comunidade

Além de eleger os produtores de blocos, em conformidade com um blockchain baseado no EOS.IO, os usuários podem eleger 3 aplicações de benefício comunitário também conhecidas como contratos inteligentes. Estas 3 aplicações receberão tokens até uma porcentagem configurado do oferta do token por ano, menos os tokens que foram pagos para os produtores de blocos. Estes contratos inteligentes receberão tokens proporcionais aos votos que cada aplicativo tem recebido dos titulares dos token. As aplicações eleitas ou contratos inteligentes podem ser substituídos por outras aplicações recém-eleitas ou contratos inteligentes pelos detentores de token.

# Governança

Governança é o processo no qual pessoas chegam um consenso sobre quesões subjetivas as quais não podem ser totalmente capturados por algoritmos de software. Um blockchain baseado no software EOS.IO implementa um processo de governança que direciona eficientemente a influência existente dos produtores do bloco. Sem um processo de governança definidos, blockchains anteriores dependiam de um processo de governança ad-hoc, informal e frequentemente controverso que resultam em resultados imprevisíveis.

Um blockchain baseado no software EOS.IO reconhece que o poder se origina com os titulares dos tokens que delegam esse poder para os produtores de blocos. Aos produtores de blocos é dada autoridade limitada e verificada para congelar contas, atualizar aplicativos defeituosos e propor alterações no protocolo subjacente que resultem em hard forks.

A eleição dos produtores de blocos é incorporada no software EOS.IO. Antes que qualquer mudança possa ser feita ao blockchain esses produtores de blocos devem aprová-lo. Se os produtores de blocos se recusam a fazer as mudanças desejadas pelos titulares token então pode ser votado contra a continuidade deles. Se os produtores de blocos fazem alterações sem a permissão dos titulares de tokens então todos os outros nodos completos validadores que não produzem blocos (exchanges, etc.) irão rejeitar a mudança.

## Congelamento de contas

Às vezes um contato inteligente se comporta de forma imprevisível ou aberrante e falha a funcionar conforme o esperado; outras vezes, um aplicativo ou conta pode descobrir uma falha de segurança que permita que sejam consumidos uma quantidade não razoável de recursos. Quando tais questões inevitavelmente acontecerem, os produtores de blocos tem o poder de corrigir tais situações.

Os produtores de blocos em todos os blockchains têm o poder de selecionar quais operações estão incluídas em blocos o que lhes dá a capacidade de congelar a contas. Um blockchain usando o software EOS.IO formaliza esta autoridade, sujeitando o processo de congelamento de uma conta a 17/21 dos votos dos produtores ativos. Se os produtores abusam o poder eles podem ser eliminados e uma conta será descongelada.

## Alterando o Código da Conta

Quando tudo mais falhar, e um aplicativo "imparável" atua de forma imprevisível, um blockchain usando o software EOS.IO permite aos produtores de blocos substituir o código da conta sem ser necessário fazer um hard fork do blockchain inteiro. Semelhante ao processo de congelamento de uma conta, esta substituição do código exige uma votação de 17/21 dos produtores de blocos eleitos.

## Constituição

O software EOS.IO permite que blockchains estabeleçam um termo de acordo de serviço entre pares (peer-to-peer) ou um contrato vinculativo entre aqueles usuários que assiná-lo, referido como uma "Constituição". O conteúdo desta Constituição define obrigações entre os usuários que não podem ser aplicadas inteiramente pelo código e facilita a resolução de litígios, estabelecendo a jurisdição e a escolha da lei, juntamente com outras regras mutuamente aceitas. Todas as transações na rede de transmissão devem incorporar o hash da constituição como parte da assinatura e desse modo explicitamente vincula o signatário do contrato.

A Constituição também define a intenção legível do protocolo de código fonte. Esta intenção é usada para identificar a diferença entre um defeito de software e uma funcionalidade, quando ocorrerem erros e guia a comunidade na decisão sobre quais correções podem ser adequadas ou inadequadas para esse caso.

## Atualizando o Protocolo & Constituição

O software EOS.IO define um processo pelo qual o protocolo conforme definido pelo código fonte canônico e sua constituição, podem ser atualizados usando os seguintes passos:

1. Produtores de blocos propõem uma alteração da Constituição e obtém aprovação de 17/21.
2. Produtores de blocos mantêm uma aprovação de 17/21, durante 30 dias consecutivos.
3. É requerido que todos os usuários passem a assinar transações usando o hash da nova constituição.
4. Produtores de blocos adoptam as alterações ao código-fonte para refletir a mudança na Constituição e propôem ele para o blockchain usando o hash de um commit do git.
5. Produtores de blocos mantêm uma aprovação de 17/21, durante 30 dias consecutivos.
6. Alterações no código entram em vigor 7 dias depois, dando a todos os nós completo 1 semana para atualizar após a ratificação do código-fonte.
7. Todos os nós que não atualizarem para o novo código são desligados automaticamente.

Pela configuração padrão do software EOS.IO, o processo de atualizar o blockchain para adicionar novas funcionalidades leva 2 a 3 meses, enquanto as atualizações para corrigir bugs não-críticos que não necessitam de alterações à Constituição podem levar de 1 a 2 meses.

### Mudanças de Emergência

Os produtores de blocos podem acelerar o processo, se uma mudança de software é necessária para corrigir uma falha de segurança ou um defeito prejudicial que está ativamente prejudicando os usuários. De um modo geral pode ser contra a Constituição fazer atualizações aceleradas ou introduzir novas funcionalidades ou corrigir bugs inofensivos.

# Scripts & Virtual Machines

The EOS.IO software will be first and foremost a platform for coordinating the delivery of authenticated messages to accounts. The details of scripting language and virtual machine are implementation specific details that are mostly independent from the design of the EOS.IO technology. Any language or virtual machine that is deterministic and properly sandboxed with sufficient performance can be integrated with the EOS.IO software API.

## Schema Defined Messages

All messages sent between accounts are defined by a schema which is part of the blockchain consensus state. This schema allows seamless conversion between binary and JSON representation of the messages.

## Schema Defined Database

Database state is also defined using a similar schema. This ensures that all data stored by all applications is in a format that can be interpreted as human readable JSON but stored and manipulated with the efficiency of binary.

## Separating Authentication from Application

To maximize parallelization opportunities and minimize the computational debt associated with regenerating application state from the transaction log, EOS.IO software separates validation logic into three sections:

1. Validating that a message is internally consistent;
2. Validating that all preconditions are valid; and
3. Modifying the application state.

Validating the internal consistency of a message is read-only and requires no access to blockchain state. This means that it can be performed with maximum parallelism. Validating preconditions, such as required balance, is read-only and therefore can also benefit from parallelism. Only modification of application state requires write access and must be processed sequentially for each application.

Authentication is the read-only process of verifying that a message can be applied. Application is actually doing the work. In real time both calculations are required to be performed, however once a transaction is included in the blockchain it is no longer necessary to perform the authentication operations.

## Virtual Machine Independent Architecture

It is the intention of the EOS.IO software-based blockchain that multiple virtual machines can be supported and new virtual machines added over time as necessary. For this reason, this paper will not discuss the details of any particular language or virtual machine. That said, there are two virtual machines that are currently being evaluated for use with an EOS.IO software-based blockchain.

### Web Assembly (WASM)

Web Assembly is an emerging web standard for building high performance web applications. With a few small modifications Web Assembly can be made deterministic and sandboxed. The benefit of Web Assembly is the widespread support from industry and that it enables contracts to be developed in familiar languages such as C or C++.

Ethereum developers have already begun modifying Web Assembly to provide suitable sandboxing and determinism in with their [Ethereum flavored Web Assembly (WASM)](https://github.com/ewasm/design). This approach can be easily adapted and integrated with EOS.IO software.

### Ethereum Virtual Machine (EVM)

This virtual machine has been used for most existing smart contracts and could be adapted to work within an EOS.IO blockchain. It is conceivable that EVM contracts could be run within their own sandbox inside an EOS.IO software-based blockchain and that with some adaptation EVM contracts could communicate with other EOS.IO software blockchain applications.

# Inter Blockchain Communication

EOS.IO software is designed to facilitate inter-blockchain communication. This is achieved by making it easy to generate proof of message existence and proof of message sequence. These proofs combined with an application architecture designed around message passing enables the details of inter-blockchain communication and proof validation to be hidden from application developers.

<img align="right" src="http://eos.io/wpimg/Diagram1.jpg" width="362.84px" height="500px" />

## Merkle Proofs for Light Client Validation (LCV)

Integrating with other blockchains is much easier if clients do not need to process all transactions. After all, an exchange only cares about transfers in and out of the exchange and nothing more. It would also be ideal if the exchange chain could utilize lightweight merkle proofs of deposit rather than having to trust its own block producers entirely. At the very least a chain's block producers would like to maintain the smallest possible overhead when synchronizing with another blockchain.

The goal of LCV is to enable the generation of relatively light-weight proof of existence that can be validated by anyone tracking a relatively light-weight data set. In this case the objective is to prove that a particular transaction was included in a particular block and that the block is included in the verified history of a particular blockchain.

Bitcoin supports validation of transactions assuming all nodes have access to the full history of block headers which amounts to 4MB of block headers per year. At 10 transactions per second, a valid proof requires about 512 bytes. This works well for a blockchain with a 10 minute block interval, but is no longer "light" for blockchains with a 3 second block interval.

The EOS.IO software enables lightweight proofs for anyone who has any irreversible block header after the point in which the transaction was included. Using the hash-linked structure shown below it is possible to prove the existence of any transaction with a proof less than 1024 bytes in size. If it is assumed that validating nodes are keeping up with all block headers in the past day (2 MB of data), then proving these transactions will only require proofs 200 bytes long.

There is little incremental overhead associated with producing blocks with the proper hash-linking to enable these proofs which means there is no reason not to generate blocks this way.

When it comes time to validate proofs on other chains there are a wide variety of time/ space/ bandwidth optimizations that can be made. Tracking all block headers (420 MB/year) will keep proof sizes small. Tracking only recent headers can offer a trade off between minimal long-term storage and proof size. Alternatively, a blockchain can use a lazy evaluation approach where it remembers intermediate hashes of past proofs. New proofs only have to include links to the known sparse tree. The exact approach used will necessarily depend upon the percentage of foreign blocks that include transactions referenced by merkle proof.

After a certain density of interconnectedness it becomes more efficient to simply have one chain contain the entire block history of another chain and eliminate the need for proofs all together. For performance reasons, it is ideal to minimize the frequency of inter-chain proofs.

## Latency of Interchain Communication

When communicating with another outside blockchain, block producers must wait until there is 100% certainty that a transaction has been irreversibly confirmed by the other blockchain before accepting it as a valid input. Using an EOS.IO software-based blockchain and DPOS with 3 second blocks and 21 producers, this takes approximately 45 seconds. If a chain's block producers do not wait for irreversibility it would be like an exchange accepting a deposit that was later reversed and could impact the validity of the blockchain's consensus.

## Proof of Completeness

When using merkle proofs from outside blockchains, there is a significant difference between knowing that all transactions processed are valid and knowing that no transactions have been skipped or omitted. While it is impossible to prove that all of the most recent transactions are known, it is possible to prove that there have been no gaps in the transaction history. The EOS.IO software facilitates this by assigning a sequence number to every message delivered to every account. A user can use these sequence numbers to prove that all messages intended for a particular account have been processed and that they were processed in order.

# Conclusion

The EOS.IO software is designed from experience with proven concepts and best practices, and represents fundamental advancements in blockchain technology. The software is part of a holistic blueprint for a globally scalable blockchain society in which decentralised applications can be easily deployed and governed.