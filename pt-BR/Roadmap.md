## EOS.IO Roadmap do Software

Este documento descreve o plano de desenvolvimento de alto nível e será atualizado conforme é feito progresso em direção a versão 1.0. Este roadmap se aplica unicamente ao software do blockchain e não as outras ferramentas e utilitários como carteiras e exploradores de blocos que que terão os seus próprios times e roadmaps assim que a Fase 1 seja finalizada.

***Todo o conteúdo deste documento é rascunho e sujeito a alterações a qualquer momento e é publicado apenas para fins informativos. block.one não garante a exatidão das informações contidas neste roadmap e a informação é oferecida sem nenhum tipo de garantias, expressas ou implícitas.***

# Fase 1 - Ambiente de Teste Minimamente Viável - Verão 2017

O objetivo desta fase é estabelecer as APIs que desenvolvedores irão precisar para começar a construir e testar aplicações no EOS.IO. Para que os desenvolvedores possam começar a testar as suas aplicações eles precisarão implementado o seguinte:

### Nó Autônomo (Dan & Nathan)

Um nó autônomo opera um blockchain de testes e produz blocos e expõe uma API. Este nó não precisa preocupar-se com qualquer código da rede P2P.

### Contratos nativos (Nathan)

O software EOS.IO tem vários contratos nativos. Estes são os contratos que gerenciam as operações principais do blockchain e existem fora da interface Web Assembly. Estes contratos incluem:

1. @eos - gerencia as transferências de token EOS
2. @stake - gerencia EOS bloqueado, votação e Eleição do Produtor
3. @system - gerencia permissões, mensagens e atualizações de código

### API da Máquina Virtual (Dan)

Contratos são compilados para WebAssembly (WASM) e WASM deve interagir com o blockchain através de uma API definida. Esta API é o que os desenvolvedores dependem para construir aplicações e deve estar relativamente estável antes que os desenvolvedores começem a construir em cima do EOS.

### Interface RPC (Arhag, Nathan)

Uma interface simples usando JSON RPC sobre HTTP será fornecida que permite que os desenvolvedores transmitam transações e consultem o estado do aplicativo. Isto é critico para publicação e interação com as aplicações de teste.

### Ferramentas de linha de comando (Arhag)

As ferramentas de linha de comando facilitam a integração entre a interface RPC com o ambientes de desenvolvimento dos desenvolvedores.

### Documentação Basica para Desenvolvedores (Josh)

Documentos que ensinam os desenvolvedores como começar a construir acima dos blockchains do EOS.IO. Isto inclui documentação API WASM, Interface RPC e ferramentas de linha de comando.

# Fase 2 - Rede de Testes Minimamente Viável - Outono 2017

Tudo na Fase 1 pressupõe um ambiente confiável que apenas executa código do desenvolvedor. Antes que uma rede de teste possa ser implantada vários recursos adicionais precisam ser implementados e testados.

### Código da rede P2P (Phil)

Este é um plugin que é responsável por sincronizar o estado do blockchain entre dois nós autônomos.

### Saneamento do WASM & Sandboxing da CPU (Brian)

O código WASM precisa ser higienizado para verificar o comportamento não-determinístico, como operações de ponto flutuante e loops infinitos.

### Rastreamento do uso de Recursos & Limitação de Taxa (Arhag)

Para evitar o abuso a monitoração de recursos e o rastreamento da taxa de uso limita os usuários de acordo com as políticas do EOS.

### Teste da Importação Inicial (DappHub)

As ferramentas precisam ser desenvolvidas para exportar os dados do estado de Distribuição de Tokens EOS e criar um arquivo de configuração inicial. Isto permitira que qualquer que esta participando na distribuição do Token possa adquirir alguns Tokens EOS de teste (TEOS).

### Comunicação Interblockchain (Nathan)

Esta característica envolve verificar se o hash de Merkle das transações é adequado.

# Fase 3 - Testes & Auditorias de Segurança - Inverno 2017, Primavera 2018

Durante esta fase, a plataforma passará por testes pesados com foco na busca de bugs e problemas de segurança. No final da fase 3 a versão 1.0 será taggeada.

### Desenvolver Aplicações de Exemplo

As aplicações de exemplo são criticas para demostrar que a plataforma fornece os recursos necessários por desenvolvedores reais.

### Prémios para atacar com sucesso a rede

O processo de atacar a rede com spam, exploits da máquina virtual e crashes causados por bugs, e comportamento não determinístico será intensivo, mas necessário para garantir que a versão 1.0 seja estável.

### Suporte de Linguagens

Adicionar suporte para linguagens adicionais para que possam ser compiladas para o WASM: C++, Rust, etc.

### Documentação & Tutoriais

# Fase 4 - Otimização Paralela - Verão / Outono 2018

Depois de conseguir liberar um produto estável 1.0, vamos passar a otimizar o código para execução paralela.

# Fase 5 - Implementação do Cluster O Futuro