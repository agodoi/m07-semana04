# ARQUITETURA CORPORATIVA

Nessa instrução, vamos explorar a arquitetura corporativa (apresentada na imagem a seguir), que suporta aplicações Web escaláveis de forma a atender milhares a milhões de usuários contemplando também a segurança destas aplicações utilizando um único ponto de contato (load balancer), e como fazer o gerenciamento de fluxos de trabalho escaláveis (load balancer).

## Ganho para o Projeto

Nessa instrução, vamos entender como o projeto da venda de ingressos pode ser arquitetado de modo profissional e real, e que permita a disponibilidade robusta do banco de dados do estoque de telefones entre as milhares de lojas, permitindo a rápida consulta no momento das vendas.

A seguir, você vai entender porque criar cada elemento da arquitetura corporativa no seu projeto.

## Arquitetura e Elementos 


### Infográfico de Fluxo de Serviços AWS
<img src="https://github.com/agodoi/ArquiteturaCorp/blob/main/imgs/Slide1.PNG" width="1000">


### Diagrama de Implantação UML (disponível em [UMLetino](https://umletino.com/umletino.html))

<img src="https://github.com/agodoi/m07-semana04/blob/main/imgs/m07-semana04.png" width="800">


* Route 53 é o conversor do link S3 ou EC2 interno AWS (com vários caracteres) para um DNS amigável que você compra na Internet.

* CloudFront serve para deixar sua aplicação em *cache* ao redor do mundo.

* O S3 vai se comportar como um servidor, mas será estático (HTML e JS), isto é, e no EC2, teremos a parte dinâmica do site (caso exista). Contudo, o S3 na arquitetura corporativa se posiciona tanto um serviço de armazenamento de dados quanto um site estático, porém, não adote o mesmo S3 para ambas finalidades. Adote mais de um S3.
   
* O ALB (Amazon Load Balance), ou ELB (Elastic Load Balance) é um distribuidor de cargas e acessos.
  
* O Bastion Host atua como um único ponto de acesso público para administrar a sua rede privada com os EC2s. Você acessa o Bastion host e depois você abre uma conexão SSH com o respectivo EC2 dentro da AWS. Portanto, o Bastion Host se posiciona na sub-rede pública para ter acesso à sub-rede privada. 
  
* O RDS é o banco de dados.


## Por que criar uma VPC no seu Projeto?

### 1. Rede Virtual Segura:

A VPC é uma rede virtual definida por você dentro da infra da AWS que permite executar, praticamente, todos os recursos disponíveis na AWS. Contudo, por exemplo, Amazon Route 53, Amazon CloudFront e AWS Direct Connect geralmente ficam de fora da VPC porque são serviços que se conectam para fora da AWS.

### 2. Disponibilidade:

Uma VPC pode abranger várias zonas de disponibilidade. Cuidado! Cada VPC precisa estar numa zona diferente. As zonas são indicadas por letras a, b, c, d ... etc

### 3. Conectividade:

Dentro da VPC, vamos criar um IGW (Internet Gateway), que permite a comunicação entre instâncias na VPC e a Internet externa. Depois de criar a VPC, vamos adicionar sub-redes dentro de uma mesma zona de disponibilidade. Se o tráfego de uma sub-rede for roteado para um gateway da Internet, a sub-rede será chamada de *sub-rede pública*. Se a sub-rede não tiver uma rota para o gateway da Internet, ele será chamada de *sub-rede privada*.

**O assistente também criará um gateway NAT (Network Address Translation ou Conversão de Endereços de Rede) que é usado para fornecer conectividade com a Internet para instâncias do EC2 nas sub-redes privadas, mas a Internet não consegue acessar a instância EC2.**

Outro ponto importante é que a *VPC default* criada pelo AWS quando você instancia um EC2 pela primeira vez, possui até 6 sub-redes e todas são públicas. Nenhuma é privada. Isso significa que todos os itens que você colocar nessa VPC default estará visível na Internet. Por que 6 subnets? Porque temos o AZ-Multi em operação oferecendo alta disponibilidade na AWS e no caso do Norte da Virgínia, são criadas 6 zonas: a, b, c, d, e, f. Na região de São Paulo, são criadas 3 zonas: a, b, c.

O que é um AZ? É um data center isolado ou separado física e geograficamente por X quilômetros, 100km por exemplo (esse dado é sigiloso, não sabemos). Caso falhe um data center, você terá disponibilidade em outro.

Caso você queira ter mais de uma VPC, a regra básica é que o range de IPs entre elas não se sobreponham.

Caso queira excluir sua VPC, primeiro exclua a respectiva Interface de Internet (IGW). Você deve buscar por **IGW**, fazer a desassociação e a sua exclusão, e depois, excluir e VPC desejada.

## Por que criar um AWS Route 53 no seu Projeto?

É um **serviço** com 3 importantes funções:

### 1. Registra nomes de domínio
Seu site/banco de dados precisa de um nome, como example.com. O Route 53 permite que você registre um nome para seu site ou aplicação Web, conhecido como um nome de domínio.

### 2. Roteia tráfego de Internet para os recursos do seu domínio
Quando um usuário abre um navegador da Web e informa seu nome de domínio (example.com) ou nome de subdomínio (acme.example.com) na barra de endereços, o Route 53 ajuda a conectar o navegador com o site ou a aplicação Web.

### 3. Verifica a integridade de seus recursos
O Route 53 envia solicitações automáticas através da Internet a um recurso, como um servidor Web, para verificar se está acessível, disponível e funcional. Você também pode optar por receber notificações quando um recurso se tornar indisponível e optar por desviar o tráfego da Internet dos recursos não íntegros.

## Por que criar uma instância S3 no seu Projeto?

A Amazon Simple Storage Service (S3) é um serviço de armazenamento em nuvem oferecido pela AWS que fornece várias vantagens para empresas e desenvolvedores que desejam armazenar e gerenciar dados de forma eficiente e escalável. Nessa prática hoje, vamos armazenar um site estático chamado **index.html** e criar outros objetos, como pasta, subpastas e armazenar um arquivo qualquer. Mas além disso, o S3 oferece:

### 1. Escalabilidade e Disponibilidade: 

O Amazon S3 é altamente escalável e foi projetado para lidar com cargas de trabalho de qualquer tamanho. Ele fornece alta disponibilidade e durabilidade dos dados, replicando automaticamente os objetos em várias zonas de disponibilidade.

### 2. Durabilidade: 

O S3 oferece uma durabilidade excepcional dos dados, com múltiplas cópias dos objetos armazenados em diferentes dispositivos e locais físicos. Isso ajuda a proteger seus dados contra falhas de hardware e erros.

### 3. Segurança: 

O S3 oferece diversas opções de segurança, como controle de acesso baseado em políticas (IAM e políticas de bucket), criptografia de dados em repouso (por meio de chaves de servidor ou gerenciadas pelo cliente) e opções de proteção de dados em trânsito.

### 4. Economia de Custos: 

O S3 oferece diferentes classes de armazenamento, como "Standard", "Infrequent Access" e "Glacier", permitindo que você escolha a opção mais adequada para o seu caso de uso, o que pode resultar em economia de custos significativa.

### 5. Recursos de Gerenciamento: 

O S3 oferece recursos robustos de gerenciamento, como versionamento de objetos, logs de acesso, eventos de notificação (através do Amazon S3 Event Notifications), políticas de ciclo de vida para automatizar a transição de objetos entre classes de armazenamento e até mesmo a expiração automática de objetos.

### 6. Integração com Outros Serviços: 

O S3 é amplamente integrado com outros serviços da AWS, o que permite criar soluções complexas e altamente eficientes. Por exemplo, você pode usar o Amazon S3 como origem de dados para serviços de análise, como o Amazon Redshift ou o Amazon Athena.

### 7. Desempenho: 

O Amazon S3 é otimizado para oferecer um bom desempenho de leitura e gravação, mesmo em grandes volumes de dados.

### 8. Acesso Global: 

Os dados armazenados no S3 podem ser acessados de qualquer lugar do mundo por meio de URLs únicos e seguros, o que é útil para distribuição de conteúdo estático, como imagens e vídeos.

### 9. Backup e Recuperação de Desastres: 

O S3 pode ser usado para criar backups de dados importantes e atuar como uma solução de recuperação de desastres, garantindo a disponibilidade e a integridade dos dados mesmo em situações adversas.

### 10. Fácil Integração: 

A AWS oferece uma ampla gama de SDKs e ferramentas para facilitar a integração do S3 em aplicativos e sistemas existentes.

## Por que criar um Amazon Load Balance no seu Projeto?

### 1. Distribuição de Tráfego: 

Um balanceador de carga distribui o tráfego de entrada entre várias instâncias (ou recursos) em diferentes zonas de disponibilidade. Isso ajuda a evitar a sobrecarga de um único recurso e garante que o tráfego seja distribuído de forma equilibrada para garantir melhor desempenho.

### 2. Alta Disponibilidade: 

Os balanceadores de carga da AWS são projetados para alta disponibilidade. Se uma instância ou zona de disponibilidade falhar, o balanceador de carga redireciona automaticamente o tráfego para instâncias saudáveis, garantindo que a aplicação permaneça acessível.

### 3. Escalabilidade: 

Os balanceadores de carga facilitam a escalabilidade horizontal, permitindo adicionar ou remover instâncias conforme necessário para lidar com variações de carga. Isso ajuda a manter o desempenho da aplicação mesmo durante picos de tráfego.

### 4. Detecção de Saúde: 

Os balanceadores de carga monitoram constantemente a saúde das instâncias que estão sendo balanceadas. Caso uma instância falhe ou se torne inacessível, o balanceador de carga deixa de enviar tráfego para essa instância até que ela esteja novamente saudável.

### 5. SSL/TLS Termination: 

Os balanceadores de carga podem lidar com o termino de SSL/TLS, descarregando o trabalho de criptografia e descriptografia das instâncias de back-end. Isso ajuda a melhorar o desempenho das instâncias e simplifica a implantação de certificados SSL/TLS.

### 6. Distribuição Geográfica: 

Alguns balanceadores de carga permitem a distribuição geográfica do tráfego, redirecionando os usuários para instâncias próximas a sua localização geográfica. Isso ajuda a melhorar o desempenho e a latência para os usuários em diferentes regiões.

### 7. Redirecionamento de Portas: 

Os balanceadores de carga podem redirecionar o tráfego de diferentes portas para instâncias específicas, permitindo que você execute várias aplicações ou serviços em portas diferentes nas mesmas instâncias.

### 8. Logs e Métricas: 

Os balanceadores de carga registram informações sobre o tráfego, permitindo monitorar e analisar padrões de acesso, além de fornecer métricas para otimização de desempenho e escalabilidade.

### 9. Integração com Serviços AWS: 

Os balanceadores de carga podem ser integrados com outros serviços da AWS, como Auto Scaling e Amazon ECS, para criar arquiteturas altamente escaláveis e dinâmicas.

### 10. Segurança: 

Os balanceadores de carga podem ajudar a proteger sua infraestrutura, pois atuam como um ponto de entrada único e ocultam as instâncias de back-end, reduzindo a exposição a ameaças.

## Por que criar um Bastion Host no seu Projeto?

É um servidor designado para fornecer acesso seguro a outras instâncias dentro de uma rede privada. Ele atua como um ponto de entrada controlado e seguro para administradores e desenvolvedores acessarem as instâncias em uma VPC ou ambiente de nuvem privada. Vantagens:

### 1. Acesso Seguro: 

O bastion host fornece um ponto de entrada seguro para acessar instâncias em uma rede privada. Isso é importante para proteger suas instâncias de possíveis ataques ou acessos não autorizados, já que o acesso direto a instâncias em uma rede privada é restrito.

### 2. Controle de Acesso: 

O bastion host permite implementar um controle rigoroso sobre quem pode acessar as instâncias em sua rede privada. Você pode configurar as políticas de autenticação e autorização para garantir que apenas usuários autorizados tenham acesso ao bastion host e, consequentemente, às instâncias internas.

### 3. Redução de Superfície de Ataque: 

Ao usar um bastion host, você reduz a exposição das instâncias internas à Internet. Isso limita os pontos de entrada potenciais para sua rede e minimiza os riscos associados a vulnerabilidades de segurança.

### 4. Monitoramento Centralizado: 

O bastion host pode ser configurado para registrar todas as atividades de acesso e uso. Isso facilita o monitoramento e a auditoria de quem está acessando suas instâncias e quais ações estão sendo realizadas.

### 5. Acesso a Recursos Internos: 

O bastion host pode ser configurado para redirecionar o tráfego para as instâncias internas na rede privada, permitindo a administração, manutenção e depuração de sistemas sem expor diretamente essas instâncias à Internet.

### 6. Implementação de Políticas de Segurança: 

O bastion host pode ser configurado com políticas de segurança específicas, como a exigência de autenticação de dois fatores (2FA), certificados SSH ou outras medidas de segurança personalizadas.

### 7. Isolamento de Chaves Privadas: 

A utilização do bastion host pode ajudar a proteger as chaves privadas usadas para autenticação nas instâncias, uma vez que os administradores e desenvolvedores não precisam manter essas chaves em suas máquinas locais.

### 8. Gerenciamento Centralizado: 

Um bastion host pode ser configurado para centralizar a administração e as ferramentas de gerenciamento. Isso pode facilitar as tarefas de manutenção, atualização e implementação de patches em várias instâncias internas.

### 9. Acessibilidade a Instâncias Privadas: 

O bastion host permite que você acesse instâncias internas sem expor diretamente seus endereços IP privados à Internet, o que pode ser útil para cenários em que a visibilidade da rede é limitada.

# Por que criar um EC2 no seu Projeto?

Amazon Elastic Compute Cloud (EC2) é um serviço de computação em nuvem oferecido pela Amazon Web Services (AWS) que fornece capacidade de processamento escalável na nuvem. Há várias vantagens em usar o EC2 na AWS, tornando-o uma escolha popular para hospedar aplicativos e executar cargas de trabalho em nuvem. Algumas das principais vantagens incluem:

### 1. Escalabilidade Sob Demanda: 

O EC2 permite escalar verticalmente (aumentar ou diminuir recursos em uma única instância) e horizontalmente (adicionar ou remover instâncias) de acordo com as demandas da sua aplicação. Isso proporciona flexibilidade para lidar com picos de tráfego e variações de carga de trabalho.

### 2. Configuração Personalizável: 

O EC2 oferece uma variedade de tipos de instância com diferentes quantidades de CPU, memória, armazenamento e capacidades de rede. Isso permite escolher a configuração de hardware mais adequada para atender aos requisitos da sua aplicação.

### 3. Diversidade de SO e Software: 

Você pode escolher entre uma variedade de sistemas operacionais (Windows, Linux, etc.) e instalar o software necessário em suas instâncias EC2. Isso permite executar uma ampla gama de aplicativos e serviços.

### 4. Pagamento por Uso: 

O modelo de preços do EC2 é baseado no pagamento por uso. Você paga apenas pelas instâncias que provisiona e pelo tempo em que elas estão em execução. Isso permite reduzir custos operacionais, já que você não precisa investir em hardware dedicado.

### 5. Acesso Administrativo Completo: 

Você tem controle total sobre suas instâncias EC2, o que permite personalizar a configuração, instalar software, realizar atualizações e configurar a segurança de acordo com suas necessidades.

### 6. Integração com Outros Serviços AWS: 

O EC2 é facilmente integrado a outros serviços da AWS, como o Amazon RDS (banco de dados relacional), Amazon S3 (armazenamento em nuvem), Amazon VPC (rede virtual privada) e muitos outros. Isso permite construir soluções completas e integradas na nuvem.

### 7. Alta Disponibilidade e Redundância: 

O EC2 permite criar instâncias em várias zonas de disponibilidade, proporcionando resiliência contra falhas de hardware ou problemas em data centers específicos.

### 8. Autoscaling: 

O EC2 permite configurar políticas de escalonamento automático para adicionar ou remover instâncias com base nas métricas de desempenho. Isso ajuda a manter o desempenho da aplicação durante flutuações de carga.

### 9. Backup e Recuperação: 

Você pode criar imagens (AMIs) das suas instâncias EC2, o que facilita a criação de backups e a replicação de ambientes para recuperação de desastres.

### 10. Desenvolvimento e Testes: 

O EC2 é frequentemente usado para ambientes de desenvolvimento e testes, pois permite criar rapidamente ambientes sob medida e replicáveis para testar código e recursos sem afetar a infraestrutura de produção.

# Por que criar um RDS no seu Projeto?

Amazon RDS (Relational Database Service) é um serviço gerenciado de bancos de dados relacionais. Vantagens:

### 1. Facilidade de Gerenciamento: 

O RDS gerencia tarefas operacionais complexas, como provisionamento de hardware, instalação de software, patches, backups e recuperação de dados. Isso permite que você se concentre mais no desenvolvimento de aplicativos e menos na administração do banco de dados.

### 2. Escalabilidade Vertical e Horizontal: 

O RDS permite dimensionar verticalmente (aumentar recursos da instância) e horizontalmente (adicionar réplicas de leitura) para lidar com o aumento da carga de trabalho, garantindo o desempenho e a disponibilidade do banco de dados.

### 3. Backup e Recuperação Automatizados: 

O RDS oferece backups automáticos diários e permite a retenção de backups por um período específico. Além disso, você pode criar snapshots manuais para momentos específicos. Isso facilita a recuperação de dados em caso de falhas.

### 4. Alta Disponibilidade e Failover: 

O RDS permite criar réplicas de leitura para distribuir a carga de leitura e fornecer redundância. Além disso, em bancos de dados multi-AZ, o RDS automaticamente realiza failover para uma instância de backup em caso de falhas na instância primária.

### 5. Atualizações de Software Gerenciadas: 

O RDS gerencia automaticamente as atualizações de software e patches do banco de dados, permitindo que você mantenha seus bancos de dados atualizados sem interrupções significativas.

### 6. Segurança Avançada: 

O RDS oferece várias camadas de segurança, incluindo a possibilidade de executar bancos de dados em uma VPC (Virtual Private Cloud), criptografia de dados em repouso e em trânsito, e integração com a AWS Identity and Access Management (IAM).

### 7. Monitoramento e Métricas: 

O RDS fornece métricas de desempenho e permite configurar alertas para monitorar a saúde e o desempenho do banco de dados. Também é possível usar serviços como o Amazon CloudWatch para obter insights detalhados sobre o desempenho.

### 8. Compatibilidade com Diferentes Bancos de Dados: 

O RDS oferece suporte a vários bancos de dados relacionais populares, como MySQL, PostgreSQL, Oracle, SQL Server e MariaDB. Isso permite que você escolha a tecnologia que melhor se adapta às suas necessidades.

### 9. Gerenciamento Simplificado de Multi-AZ: 

Ao usar o RDS em uma configuração de várias zonas de disponibilidade (Multi-AZ), o sistema gerencia automaticamente a replicação, failover e recuperação em caso de falhas.

### 10. Compatibilidade com Ferramentas e Aplicativos: 

O RDS é compatível com muitas ferramentas e aplicativos que são usados com bancos de dados relacionais, facilitando a migração de aplicações existentes para a nuvem.

## Sugestão de fluxo, porém pago e não obrigatório:

1) Criar um domínio (exemplo, goDaddy ou [RegistroBr](https://registro.br/))

2) Registrar o domínio recém criado no Zonas de Hospedagem do Route 53 (custo de U$0,50/mês). Para isso, cria-se um Route 53 (essa etapa não poderá ser executada no Leaner Lab). Cada domínio criado no AWS gera um custo real de U$0,50/mês (valor de 2023) e por isso, não está disponível no Leaner Lab. Já no domínio particular da AWS, sim é possível criar um Route 53, mas precisa ter um cartão de crédito cadastrado e assumir o custo. Usando o site [goDaddy](https://www.godaddy.com/) (que é um site criador de domínios pagos), o professor criou o domínio **aulaarquitetura.com** para testes. Mais detalhes sobre isso, fale com ele.

3) Criar um certificado https (essa etapa não poderá ser executada no Leaner Lab) para o domínio goDaddy.

3.1) Digite **certificate manager** na lupa do console da AWS

3.2) No menu vertical esquerdo, clique em **Solicitar certificado** e depois clique em **Solicitar um certificado público**

3.3) No campo **Nome de domínio totalmente qualificado**, você vai colar o domínio que criaria na etapa Passo-02, que nesse exemplo foi **aulaarquitetura.com**

4) Regitrar os nomes dos servidores Route 53 no GoDaddy

5) Seguir os passos abaixo.


# Passo-01: Criando a VPC

**1.1)** Busque por VPC no console da AWS;

**1.2)** Clique no botão laranja CRIAR;

**1.3)** Selecione **Somente VPC**.

**1.4)** No campo **Tag de nome** digite **VPC_Arquitetura_Corp**.

**1.5)** Bloco CIDR IPV4 digite **192.168.0.0/22**

**1.6)** As demais opções, você não precisa mexer e basta confirmar no botão laranja.

# Passo-02: Criando as sub-redes
## sub-rede pública

**2.1)** No menu vertical da VPC, clique em **sub-redes** e então, clique no botão laranja **Criar sub-redes** e aponte para a VPC corporativa que acabou de criar **VPC_Arquitetura_Corp**.

**2.2)** No campo **Nome da sub-rede** coloque **Sub_Publica_a**.

**2.3)** Em **Zona de disponibilidade** deixe **us-east-1a**.

**2.4)** Em **Bloco CIDR IPV4** coloque um IP que esteja dentro da faixa da rede da VPC que você criou, então, **digite 192.168.0.0/24**. Essa faixa está dentro da faixa maior 192.168.0.0/22. Vamos discutir o mapa de endereçamento numa instrução futura. Aguente firme! Clique no botão laranja.

## sub-rede privada

**2.5)** Repetindo os passos para criar a Rede Privada, no menu vertical da VPC, clique em **sub-redes** e então, aponte para a VPC corporativa que acabou de criar **VPC_Arquitetura_Corp**.

**2.6)** No campo **Nome da sub-rede** coloque **Sub_Privada_b**. Note que você está apontado para uma zona diferente da sua sub-rede pública. É uma estratégia para 
[alta disponibilidade](https://github.com/agodoi/VocabularioAWS).

**2.7)** Em **Zona de disponibilidade** deixe **us-east-1b**.

**2.8)** Em **Bloco CIDR IPV4** coloque um IP que esteja dentro da faixa da rede da VPC que você criou, então, **digite 192.168.1.0/24**. Essa faixa está dentro da faixa maior 192.168.0.0/22. Novamente, vamos discutir o mapa de endereçamento numa instrução futura.

# Passo-03: Criando Tabelas de Rotas

A sua sub-rede pública ainda não sabe como chegar na Internet. Para isso, precisamos de um **IGW (Internet Gateway)**, certo? E nem a privada sabe chegar na Internet. Para isso precisamos de quem? Se você pensou em **NAT**, acertou!. IGW resolve a Internet para a pública e a NAT resolve para Internet para a privada.

**3.1)** No menu vertical da VPC, procure por **Tabela de rotas**. Daí você vai observar que já existe uma Tabela de Rotas sem nome. Essa tabela sem nome é da sua VPC recém criada. Isso é normal, pois todas as vezes que você cria uma VPC nova, já carrega uma Tabela de Rotas **default** e sem nome. Só para organizar melhor, passe o mouse na coluna **Name** dessa Tabela de Rotas sem nome, veja um lápis e daí você digita **TabRota_Default_VPC_ArqCorp**.

E agora, vamos criar duas novas Tabelas de Rotas, sendo uma para a subnet pública e outra para a subnet privada. Chique?

**3.2)** Clique em **Criar tabela de rotas**, e em **nome** coloque **TabRota_Publica_ArqCorp** e selecione a VPC recém criada e confirme no botão laranja.

**3.3)** Faça o mesmo para a sua subnet privada. Clique em **Criar tabela de rotas**, e em **nome** coloque **TabRota_Privada_ArqCorp** e selecione a VPC recém criada e confirme no botão laranja.

Até agora, você só criou os nomes das Tabelas de Rotas que não sabem o que fazer ainda. Elas apenas estão dentro da sua VPC recém criada **VPC_Arquitetura_Corp**.

**3.4)** Vamos agora associar as Tabelas de Rotas com as sub-redes propriamente ditas.

**3.4.1)** Clique no link azul da tabela de rotas privada **TabRota_Privada_ArqCorp**, vá na aba **Associação de sub-rede**, clique no botão **Editar associações de sub-rede** e selecione a sub-rede privada **Sub_Privada_b** e confirme no botão laranja.

**3.4.2)** Faça o mesmo para a tabela de rotas pública **TabRota_Publica_ArqCorp**, clicando em seu link azul, depois indo na aba **Associação de sub-rede**, clicando no botão **Editar associações de sub-rede** e selecione a sub-rede privada **Sub_Publica_a** e confirme no botão laranja.


Portanto, agora suas Tabelas de Rotas já possuem as devidas sub-redes.

# Passo-04: Criando o IGW

Esse elemento de rede resolve como sua rede pública vai encontrar a Internet.

**4.1)** Para criar uma saída para Internet da sub-rede pública, vá no menu vertical esquerdo da VPC, clique em **Gateways da Internet**, depois **Criar gateway da Internet** e em **Tag name** digite **IGW_ArqCorp** e confirme no botão laranja. Cuidado agora! Você precisa associar o seu IGW à VPC_Arquitetura_Corp. Então clique no botão verde que vai aparecer na barra superior ou volte no menu vertical esquerdo, liste o seu **Gateways da Internet**, vá no botão **Ações**, selecione **Associar à VPC** e escolha a VPC recém criada e confirma no botão laranja.

Agora, vamos atualizar as rotas de entrada e saída ou regras de entrada e saída.

**4.2)** Volte na tabela de rotas **TabRota_Publica_ArqCorp** para indicar as regras de entrada e saída da sua VPC. Então, vá no menu esquerdo vertical, clique em **Tabela de Rotas** e escolha a **TabRota_Publica_ArqCorp**, e depois, vá na aba **Rotas**. Já existe uma rota padrão interna 192.168.0.0/22 mas isso não dá acesso externo à sua VPC e sim, somente acesso interno. Clique em **Editar rota**, depois **Adicionar rota** e selecione em **destino** 0.0.0.0/0 (que significa qualquer lugar) e em **alvo** você seleciona **Gateway da Internet** e daí vai aparecer a sua o **IGW_ArqCorp**, daí vc o seleciona e coloque para salvar no botão laranja.

Portanto, todas as instância com IP público dentro do sub-rede pública terão conexão com a Internet a partir de agora. Caso um EC2 seja criado nessa sub-rede com IP privado, ainda continuará sem conexão externa.

Nos próxmos passos, você terá que associar um EC2 (que será público) à essa sub-rede pública. E assim, esse EC2 específico terá conexão com a Internet. Ele se chamará Bastion Host.


# Passo-05: Criando o NAT
Agora vamos resolver o acesso à Internet da sub-rede privada, porém, acesso de saída. Não de entrada, por enquanto.

**5.1)** No menu vertical da VPC, clique na opção **Gateways NAT**, depois clique no botão **Criar gateway NAT**, e depois, em nome coloque **NAT_ArqCorp** e na opção **sub-rede** você aponta para a **sub-rede pública**. Note que existe uma opção chamada **Tipo de conexão** que já está pré-marcada em **Público** e é isso que garante que sua sub-rede privada poderá acessar à Internet. Existe a opção também de **Alocar IP elástico**, então clique nesse botão **Alocar IP elástico** para gerar um IP elástico e daí você terá a opção como **eipalloc-xxxxxxxx**. Finalmente, clique no botão laranja para confirmar tudo.

**5.2)** Dessa forma, o NAT (uma espécia de proxy de camada 3, roteamento de pacotes) vai controlar o fluxo de acordo com a origem da conexão: conexão de origem interna é permita, conexão de origem externa não é permitida. Mas ainda temos um detalhe para definir que é criar uma rota na sua **TabRota_Privada_ArqCorp**. Então, no meu vertical esquerdo da sua VPC, clique em **Tabela de rotas**, clique no link azul **TabRota_Privada_ArqCorp** e daí, **Editar rotas**, clique no botão **Adicionar rotas**, escolha o **Destino 0.0.0.0/0** (Internet externa) e coloque em **Alvo** como **Gateway NAT** (algo do tipo assim **nat-0f1c0fbcfded07cf8** vai aparecer). **Esse item gasta-se alguns minutos para propagar e começar a funcionar.**

Conclusão: sem essa etapa, o EC2 privado não poderá ser atualizado, isto é, não poderá originalizar conexões para fora da VPC (Internet).

### Pronto! Suas sub-redes públicas e privadas estão ok agora!


# Passo-06: Criando EC2 - Bastion Host (público)

Nesse passo você já deve estar ficando bom, pois já vimos EC2 em outra aula! Vamos criar 2 instâncias EC2, sendo uma na sub-rede pública e outra na sub-rede privada. O EC2 da sub-rede pública vai se comportar como **Bastion Host** e o EC2 da sub-rede privada, será seu servidor dinâmico, por exemplo.

**6.1)** Buscando por EC2 na lupa do console, crie uma instância que será pública nomeando-a como **Bastion_Host_Publica_ArqCorp**, escolha **Ubuntu**, deixe como **Tipo de instância** qualificada para o nível gratuito, gere uma par de chave com o nome **PEM_EC2Publico_ArqCorp**, edite as opções de **Configurações de rede**, aponte para a **VPC_Arquitetura_Corp**, aponte para sub-rede pública recém criada, deixe **Atribuir IP público automaticamente** no **habilitar**, no **Firewall** deixe marcado a opção **Criar grupo de segurança**, coloque um nome no seu **Grupo de segurança** como **GS_EC2Publico** habilite apenas a opção do SSH e confirme no botão laranja.

Mas atenção: esse IP público que você está recebendo agora nessa instância vai mudar se você desligar o EC2.

## Passo-07: Criando outro EC2 - Servidor (privado)

**7.1)** Faça o mesmo para o EC2 privado criando uma nova instância, nomeie-a como **EC2_Privado_ArqCorp**, escolha **Ubuntu**, deixe como **Tipo de instância** qualificada para o nível gratuito, gere uma par de chave com o nome **PEM_EC2Privado_ArqCorp**, edite as opções de **Configurações de rede**, aponte para a **VPC_Arquitetura_Corp**, aponte para sub-rede privada recém criada, deixe **Atribuir IP público automaticamente** no **desabilitar**, no **Firewall** deixe marcado a opção **Criar grupo de segurança**, coloque um nome no seu **Grupo de segurança** como **GS_EC2Privado** habilite as opções o **SSH** e aponte **GS_EC2Publico** (para apontar, na opção **Origem**, deixe **Personalizado** que vai aparecer o grupo de segurança mencionado). Caso precise no futuro (mas não agora para essa aula) adicione **HTTP** e **HTTPS** com **0.0.0.0/0**. Nesse caso, seu EC2 estará disponível para acesso HTTP e HTTPS para fora da sua VPC. Confirme no botão laranja.

## Passo-08 (testes):

Confira se seu EC2 privado (que é um servidor interno) está acessível a partir do Bastion Host. Para isso:

**8.1)** Faça uma conexão **ssh -i** no seu Bastion Host usando o IP público (que você pega no botão **Conectar** das propriedade do EC2 externo chamado Bastion Host;

**8.2)** Crie uma pasta raiz **mkdir** chamada **keys**. 

**8.3)** Dentro dessa pasta, dê um **sudo nano PEM_EC2Privado_ArqCorp.pem** para criar o arquivo PEM, copie o texto da sua chave que deve estar na sua área do seu PC, cole no arquivo, dê um **Ctrl+S** e depois um **Ctrl+X** para salvar e sair.

**8.4)** E dê outro **ssh -i** mas usando o endereço privado do EC2 privado. 

**8.5)** Você deve estar dentro do EC2 interno usando o EC2 externo.

**8.6)** Agora tente acessar o seu EC2 interno como se fosse o EC2 externo, isto é, execute o item **(8.1)** dessa passo mas usando o IP privado do botão **Conectar** do seu EC2 privado. Funcionou? Não! Por que? Seu EC2 interno possui IP público? Ele está com acesso ao IGW do EC2 externo? Não!

Conclusões: seu EC2 público está protegendo o EC2 interno. Você pode ter quantos EC2 internos desejar. Basta armazenar as chaves internas dentro do EC2 público. Mas cuidado com os acessos do EC2 público. Ele está como regra de entrada o 0.0.0.0/0 e isso não é legal. O correto é você colocar o IP fixo externo da sua empresa. Assim, somente os DEVOPS vão acessar esse EC2 externo.

**8.7)** Outro teste é o EC2 privado não consegue acessar a Internet para se atualizar. Então o NAT Gateway precisa enchegar esse caminho para fora. Da forma que está agora, não consiguiremos atualizar nada no EC2 privado. **O segredo é assossiar o NAT Gateway à sub-rede pública** que você fez no Passo-05 **5.2**. 

**8.8)** **Tente atualizar o Ubuntu do seu EC2 interno** que vai dar certo [sudo apt-get update]. E se você tentar acessar o EC2 privado usando qualquer IP (público ou privado), não vai funcionar porque o NAT Gateway só permite o fluxo de dentro para fora e não de fora para dentro.

## Conceito Importante sobre imagem do EC2:

Outro ponto importante: para fazer sentido usar o Bastion Host, a imagem do sistema operacional dos demais EC2 internos (que você vai criar no próximo passo) deve ter o que chamamos de **Gold Image**:

Uma **Gold Image** é uma boa prática no ambiente da AWS que se refere a uma imagem de máquina virtual (VM) pré-configurada que serve como uma base para a criação de novas instâncias. Essa imagem contém um sistema operacional e, muitas vezes, software pré-instalado, configurações otimizadas e possivelmente até mesmo dados ou configurações específicas da aplicação.

A ideia por trás de uma **Gold Image** é criar uma imagem de referência padronizada que possa ser usada para provisionar rapidamente novas instâncias. Isso é especialmente útil quando você tem várias instâncias que precisam ter a mesma configuração básica, como ambientes de desenvolvimento, testes ou produção. Algumas vantagens de usar gold images na AWS incluem:

* Consistência: Ao usar uma gold image, você garante que todas as instâncias criadas a partir dela tenham a mesma configuração, o que ajuda a evitar inconsistências e problemas de configuração.

* Eficiência: A criação de novas instâncias a partir de uma gold image economiza tempo, pois você não precisa configurar manualmente cada instância.

* Padronização: Uma gold image permite que você defina um padrão específico para suas instâncias, garantindo que todas sigam as mesmas diretrizes de configuração.

* Rápido Provisionamento: Como a imagem já contém o sistema operacional e as configurações básicas, a criação de novas instâncias é mais rápida do que começar do zero.

* Reprodutibilidade: Se você precisar escalar rapidamente ou substituir instâncias existentes, usar uma *gold image* facilita a recriação de ambientes consistentes.

Para criar uma **Gold Image** na AWS, você geralmente inicia uma instância EC2, configura-a conforme suas necessidades (instalando software, configurando as opções do sistema operacional, etc.) e, em seguida, cria uma imagem personalizada a partir dessa instância. Essa imagem pode ser armazenada no Amazon Machine Image (AMI) e ser usada para lançar novas instâncias com base na configuração definida. Isso são dicas para boas práticas para o futuro dos seus estudos.

Ter uma abordagem de **Gold Image** é especialmente benéfico em ambientes em nuvem, onde a escalabilidade e a automação são fundamentais. Isso ajuda a reduzir o esforço manual, melhorar a consistência do ambiente e agilizar a implantação de recursos.


# Passo-09: Criando um serviço S3
## Por padrão, todo S3 é totalmente bloqueado. Sua função agora é liberar as devidas funções dele.

Esse S3 serve para você adicionar seu site estático e não arquivos corporativos da sua empresa, ou logs de acesso, ou dados de clientes, pois esse S3 estará visível na Internet. Se você precisa de um S3 mais seguro, crie outro dentro da VPC. Deixar um S3 visível na Internet é o principal motivo de vazamento de dados sensíveis ou pessoal.

**9.1)** Digite S3 na lupa do console.

**9.2)** Clique em **Criar bucket** (botão laranja).

**9.3)** No campo **Nome do bucket** digite algor parecido com **s3_arqcoporativa** (os nomes de buckets são exclusivos mundialmente porque possuem URL exclusivas, logo você terá que criar o seu nome exlusivo). Mas atenção: não pode ter letras maiúsculas e nem começar o nome com número.

**9.4)** Em região, aponte para o **us-east-1**.

**9.5)** Deixa todas as demais configurações básicas como estão e clique no botão laranja **Criar bucket**.

**9.6)** Retorne em suas instâncias de buckets e clique no bucket que você acabou de criar (link azul) para configurá-los. Note que ele está como **Bucket e objetos não públicos**.

**9.6.1)** Dentro das configurações do Bucket, procure pela aba **Propriedades**, vá até o final da página e em **Hospedagem de site estático**, clique em **Editar** e depois, **ativar**.

**9.6.2)** Em **Tipo de hospedagem** deixe em **Hospedar um site estático**.

**9.6.3)** Em **Documento de índice** coloque o nome do arquivo-fonte do seu site, que nesse caso, pode ser **index.html**. Você deve salvar esse código abaixo em arquivo texto tipo *html* e vai usar numa etapa futura, não agora. Então, apenas salve esse código num arquivo **index.html** aí no seu HD local.

```
<!doctype html>
<html>
  <head>
    <title>This is the title of the webpage!</title>
  </head>
  <body>
    <p>This is an example paragraph. Anything in the <strong>body</strong> tag will appear on the page, just like this <strong>p</strong> tag and its contents.</p>
  </body>
</html>

```

**9.6.4)** No campo **Documento de erro - opcional** você pode deixar em branco ou elaborar uma página personalizada para quando der um erro em seu site ou até usar a mesma arquivo **index.html**. Para hoje, deixe esse campo em branco

**9.6.5)** Clique no botão laranja no final da página para confirmar.

**9.6.6)** Se você retornar em **Propriedades** após a confirmação, você terá o link do seu S3 instanciado, algo assim: **http://arquiteturacorp.s3-website-us-east-1.amazonaws.com/**, e se você clicar nesse link, constará **erro 403 Forbidden, Code: AccessDenied**.

**9.7)** Você pode carregar um arquivo qualquer no seu S3, clicando em **Objetos**. E ainda pode criar uma pasta qualquer chamada **Teste**. Então, suba um arquivo qualquer e crie uma pasta qualquer em seu S3. Veja a figura a seguir que demonstra como fica o seu HD virtual lá na AWS.

**9.8)** Você precisa liberar acesso ao público do S3. Vá no botão **Permissões**, depois em **Bloquear acesso público (configurações do bucket)** clique em **Editar**, depois desmarque **Bloquear todo o acesso público** e **Salvar alterações** e digite **confirmar**.


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/objetos.png">
   <img alt="Objetos" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/objetos.png)">
</picture>


**9.8)** Nessa etapa, vamos definir as permissões de acesso ao S3. Por padrão, ele totalmente bloqueado. Clique na aba **Permissões** e perceba que em **Visão geral das permissões** que está como **bucket e objetos não públicos**. Isso significa que o seu S3 ou site não está visível externamente. Então, em **Bloquear acesso público**, você clica em **Editar** e desmarque a opção **Bloquear todo o acesso público** e clique no botão laranja **Salvar alterações**. Note que aparecerá uma tela de confirmação novamente, conforme abaixo, onde você vai ter que digitar **confirmar**. Note que os vazamentos de dados ocorrem devido às configurações erradas e não *sem querer querendo*.


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/confirmacao_desbloqueio.png">
   <img alt="Confirmação" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/confirmacao_desbloqueio.png)">
</picture>

Agora, vá novamente na sua lista de Buckets e veja como está o **Acesso** na frente do seu bucket. Ele deve estar assim: **Os objetos podem ser públicos**, igual ao da imagem abaixo, então o S3 ainda não é público, mas pode ser...

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/podem_ser_publicos.png">
   <img alt="Podem Ser Públicos" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/podem_ser_publicos.png)">
</picture>

**9.10)** Agora, vamos gerar uma **Apólice do bucket**, que vai liberar os acessos ao seu S3. Para isso, clique em **Permissões** do seu bucket e em, **Política do bucket**, clique em **Editar** e depois **Gerador de Apólices**. Daí você vai ver uma tela como a seguir:


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/gerador_politica.png">
   <img alt="Gerador de Política" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/gerador_politica.png)">
</picture>


**9.11)** Em **Step 1 - Select Policy Type**, selecione **S3 Bucket Policy**, em **Step 2 - Add Statements(s)**, marque **Allow** em **Principal**, coloque asterisco para indicar qualquer objeto. Em **Actions** escolha **GetObject** que serve para liberar acesso aos objetos do seu site, para aparecer na sua tela do navegador. Mas note que não será possível deletar, atualizar ou colocar nada (do CRUD, somente o R - Read estará liberado). No campo **ARN (Amazon Resource Name)** você pega lá no seu bucket na aba **Propriedades**. Tem que ser algo do tipo *arn:aws:s3:::arquiteturacorp*. MAS ATENÇÃO! Adicione um /* logo após seu ARN. Então ficaria algo assim:


### arn:aws:s3:::arquiteturacorp/*

Por fim, clique em **Add Statement** e depois, confirme em **Generate Policy** para gerar sua política. O resultado será um arquivo JSON com sua apólice, algo do tipo:
```
{
  "Id": "Policy1692046481274",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1692046458290",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::arquiteturacorp/*",
      "Principal": "*"
    }
  ]
}
```

Agora, dê um Ctrl+C nesse JSON para você levar lá no bucket criado e editar a **Permissões** dele. 

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ArquiteturaCorp/blob/main/imgs/gerador_politica-2.png">
   <img alt="Gerador de Política" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ArquiteturaCorp/blob/main/imgs/gerador_politica-2.png)">
</picture>

Vá na aba **Permissões** do seu Bucket, clique em **Política do bucket** e cole esse JSON lá e confirme no SALVAR. Veja a figura a seguir para entender melhor.

### Observe que seu bucket agora está público, e com um ícone de atenção.

Para fazer um teste, acesse o link do seu S3 e verá que o erro terá mudado agora. Ele está acessível externamente, mas não tem o arquivo **index.html**. Veja a imagem a seguir para entender melhor.


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/erro404-sem-indexHTML.png">
   <img alt="Gerador de Política" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/erro404-sem-indexHTML.png)">
</picture>


**9.12)** Agora, basta você colocar um arquivo index.html dentro do seu S3 como se fosse um objeto. Arraste o seu index.html para dentro e salva. Veja a imagem de como fica no final:


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/ARQUITETURA/blob/main/imgs/index-html-S3.png">
   <img alt="Gerador de Política" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/ARQUITETURA/blob/main/imgs/index-html-S3.png)">
</picture>

Agora você pode atualizar o seu link do S3 que o seu site estático estará no ar.

# Passo-10: Criando uma CloudFront
## Serve para manter dados do site em cache, aumentar velocidade e minimizar latências ao redor do mundo

Sua principal função é melhorar a velocidade e a disponibilidade de entrega de conteúdo, como imagens, vídeos, scripts e outros arquivos, aos usuários finais em diferentes regiões do mundo.

Ao usar o CloudFront, os arquivos são armazenados em pontos de presença globais da AWS, chamados de "edge locations". Quando um usuário solicita um arquivo, o CloudFront direciona essa solicitação para o servidor de *edge location* mais próximo do usuário, o que reduz a latência e melhora a velocidade de carregamento do conteúdo. Além disso, o CloudFront ajuda a reduzir a carga nos servidores de origem, pois ele armazena em cache os arquivos e responde diretamente a solicitações subsequentes que podem ser atendidas a partir desses caches.

**10.1)** Fala uma busca por **CloudFront** na lupa do console.

**10.2)** Clique em **Create Distribution** (botão laranja).

**10.3)** No campo **Origin Domain** você seleciona o bucket criado no Passo-03, e daí o console vai sugerir uma adaptação **Use website endpoint**, daí você pode confirmar no botão. O link resultante deve ser algo do tipo **arquiteturacorp.s3-website-us-east-1.amazonaws.com** e o protocolo marcado será o **HTTP**. Não teremos o **HTTPS** porque teríamos que ter um Certificado sobre o DNS externo que você compraria.

**10.4)** Na opção **Web Application Firewall (WAF)** opte por **Enable security protections**.

**10.5)** Em **Default root object - optional**, aponte para o arquivo **index.html**. Com isso, o CloudFonte vai distribuir o seu pequeno servidor.

**10.6)** Confirme tudo no botão laranja.

As demais configurações, como em **Viewer** e **Viewer protocol policy**, você poderia marcar a opçção **Redirect HTTP to HTTPS** caso você quisesse forçar toda a conexão do seu site para HTTPS, mas para isso, você precisa do certificado validado sobre o seu domínio comprado na goDaddy, por exemplo.

Em **Allowed HTTP methods**, você pode deixar o GET, HEAD, mas se fosse para executar um CRUD, terá que colocar a opção **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**.

Se você tiver um certicado validado, puxe esse certificado na opção **Custom SSL certificate - optional**.

Para testar o CloudFront, teria que ter um cliente fora da região que você instanciou o seu site (Leste dos EUA) para observar a baixa latência de acesso. Você pode fazer isso criando uma VPN para se conectar em outro país para simular você fora do Brasil e naquele computador de outro país, você carrega seu site principal. Você deve notar uma resposta bem parecida com a resposta que você teve usando a sua rede local para acessar o seu site. Procure na Internet um tutorial de como fazer uma VPN para fora do Brasil.


## Passo-11: Criação do ELB

Todas as requisições externas da sua arquitetura vão entrar em um ELB para depois entrar em seus EC2. Dentro do ELB existe um algoritmo que monitora o tempo de resposta de cada EC2 para cada requisição e assim, o ELB distribui de forma equilibrada o número de requisições para cada EC2 seja ele, livre, com problemas, sobrecarregado, seja se sobrecarregando, etc.

**11.1)** Busque por **EC2** na lupa da AWS, e no menu esquerdo vertical, procure por **Load balancers**. Você terá 3 opções no seu console: 

**1) Application Load Balancer** >> quando precisar de um conjunto de recursos flexível para suas aplicações com tráfego HTTP e HTTPS, com API e entregas de conteúdos estáticos.

**2) Network Load Balancer** >> quando precisar de desempenho altíssimo e de baixo nível, como sockets, descarga de TLS em escala, implantação de certificados centralizada, suporte para UDP e endereços IP estáticos para sua aplicação.

**3) Gateway Load Balancer** >> quando precisar implantar e gerenciar uma frota de dispositivos virtuais de terceiros compatíveis com GENEVE. Esses dispositivos permitem que você melhore a segurança, a conformidade e os controles de políticas.

**Escolha a opção (1)**

**11.2)** Adicione um nome para o seu ELB, como **ELB-ArqCorp** [não permite certos caracteres aqui], deixe **Voltado para Internet** (mas você pode criar um ELB interno, onde ele não terá acesso público e sim, somente, entre suas instâncias internas), 

**11.3)** em **Tipo de endereço IP** deixe como **IPV4**.

**11.4)** Em **Mapeamento de rede**, aponte para a sua **VPC_Arquitetura_Corp**.

**11.5)** Em **Mapeamentos**, **aponte todas as zonas que aparecem em sua VPC** e suas respectivas sub-redes Pública e Privada. Isso vai permite que o **Auto Scalling** funcione depois. Dúvidas sobre o que é [Auto Scalling][https://github.com/agodoi/VocabularioAWS]?

**11.6)** En **Grupos de Segurança**, aponte para **GS_EC2Publico** e **GS_EC2Privado** e desmarque **default**.

**11.7)** Em **Listener**, deixe o protocolo HTTP da forma que está, com porta 80. Você só poderá adiconar o HTTPS se você tiver feito um certificado para o seu domínio dentro do Route 53 como mostra o início dessa instrução. E só lembrando, o Route 53 só funciona numa conta particular e não no Learner Lab.

**11.8)** Agora você deve clicar em **Criar grupo de destino**, onde você fará o seguinte:

**11.8.1)** Em **Escolha um tipo de destino** opte por **Instâncias**.

**11.8.2)** Em **Nome do grupo de destino**, você pode digitar **GrupoDestino-ELB** [não funciona com underline] ou outro nome que lhe fizer sentido.

**11.8.3)** Em **Protocolo**, deixe o HTTP porta 80. Lembre-se que se você tiver um domínio já certificado, então pode colocar o HTTPS. E atenção: se você tiver usando outra porta em seu servidor, como porta 8000, esse é o local adequado para alterar a porta de 80 para 8000.

**11.8.4)** Selecione a VPC que estamos tratando, que nesse caso é **VPC_ArquiteturaCorp**.

11.8.5) Em **Versão do Protocolo**, deixe **HTTP1**.

**11.8.6)** Em **Verificação de integridade**, deixe **HTTP**. Essa opção serve para o ELB bater na instância que você criou e verificar sua saúde. O ELB terá uma resposta 200 se fizer um pedido de verificação usando HTTP.

**11.8.7)** Em **Caminho da verificação de integridade**, você define o path onde o ELB precisa bater. Nesse caso, deixe **/** como path raiz padrão, mas se você tiver usando uma API com HTTPS, você precisa apontar qual o path do local da sua aplicação para o ELB encontrar.

**11.8.9)** Em **Configurações avançadas de verificação de integridade** não precisa mexer agora e clique no botão laranja.

**11.8.10)** Você vai cair numa tela diferente chamada **Registrar destinos** Você deve marcar todas as instâncias que você deseja que o ELB vai distribui a carga. No nosso caso, vamos marcar **Bastion_Host_Publica_ArqCorp** e **EC2_Privado_ArqCorp** e clicar no botão laranja chamado **Criar grupo de destino**.

**11.8.11)** Confirme se seu **Registrar destinos** deu certo, clicando em **Grupo de destino** no menu vertical esquerdo. Clique no apelido dado **GrupoDestino-ELB** e logo abaixo, em **Destinos registrado**, tem que aparecer as duas instâncias EC2 criadas: **Bastion_Host_Publica_ArqCorp** e **EC2_Privado_ArqCorp**. Se não aparecem, clique em **Registrar destinos** e marque as duas instâncias. Daí você pressiona um botão no meio da tela chamado **Incluir como pendente abaixo**, e depois, confirma no botão laranja. Aguarde alguns instantes para iniciar tais conexões entre ELB e instâncias EC2. Contudo, mesmo após aguardar alguns instantes, vai aparecer o status **unhealthy** em vermlho. Isso indica que o seu ELB ainda não conseguiu se conectar com suas instâncias por dois motivos:

   1) Suas instâncias EC2 estão vazias, não possuem nada;
   2) O grupo de segurança **GrupoDestino-ELB** está isolado, isto é, não está conectado uma API que poderia rodar no EC2 privado, por exemplo.
   3) Para resolver, vamos subir uma API no EC2 privado e finalizar essa conexão com o ELB mais tarde.

Seu ELB está praticamente pronto, basta fazer a conexão de saúde com uma aplicação rodando em EC2. Veremos isso numa instrução futura. Caso já queira resolver essa etapa hoje ainda, abra esse [vídeo](https://www.youtube.com/watch?v=8vcrE0FojKY) na posição 15min e assita.

## Passo-12: Criando o RDS

Para criar o RDS, você precisa criar uma VPC com 2 zonas como vimos na aula já explorada: [EC2-RDS](https://github.com/agodoi/EC2-RDS). 

Portanto, para o seu RDS funcionar, essa VPC de hoje não terá suporte, porque criamos apenas 1 zona.

Mas calma! Antes de surtar e descobrir que seu serviço de hoje foi em vão e não suporta o RDS, existe um passo que cria a VPC + IGW + NAT + 2 subRedes Privadas + 2 subRedes Públicas automaticamente. 

Vá em **VPC** e crie uma nova **VPC**, mas clique em **VPC e muito mais** e coloque algumas configurações eventualmente solicitadas iguais às configurações dos Passos 1 e 2. 
