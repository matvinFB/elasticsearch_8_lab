# Laboratório de Instalação do Elasticsearch 8 no Ubuntu Server 24.04

Este laboratório guia o processo de instalação e configuração do Elasticsearch 8 em uma máquina virtual Ubuntu Server, utilizando o VirtualBox. O objetivo é aprender mais sobre a ferramenta de maneira prática e no processo criar um ambiente de busca e análise de dados eficiente e totalmente funcional. Uma instalação via Docker também é possível, mas não será abordada.

Esse laboratório se baseia fundamentalmente nestas duas fontes:

- [Elasticsearch 8 and the Elastic Stack: In Depth and Hands On](https://www.udemy.com/course/elasticsearch-7-and-elastic-stack)
- [How To Install and Configure Elasticsearch on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-22-04)

Entretanto varios outros sites foram consultados e em sua maioria são citados dentro do próprio texto.

## Pré-requisitos

Antes de começar, alguns itens são fundamentais:
- VirtualBox instalado em seu sistema, pra isso acesse a seção de donwloads do site oficial do [Virtualbox](https://www.virtualbox.org/wiki/Downloads).
- Imagem ISO do Ubuntu Server, nesse laboratório foi usada a versão 24.04, porém as versões 22 e 20 devem funcionar de maneira similar. Você pode obter a isso no site odicial do [Ubuntu](https://ubuntu.com/download/server).
- Imagem VDI do Ubuntu server, que pode ser baixada no [OSBoxes](https://www.osboxes.org/virtualbox-images/). (Opcional)
- Requisitos mínimos de hardware, nesse laboratório será utilizada uma máquina virtual, que apesa de não ter interface gráfica ainda tem requisitos minimos, por tanto sua máquina hospedeira deve ter recursos o bastante para sustentar tanto o sistema hospedeiro quando o convidado. É recomendado: 
    - 2 GB de RAM
    - 2 núcleos de processamento
    - 25 GB de armazenamento

## Configuração do Ambiente

Antes de começar saiba que existe um outro caminho para conseguir sua máquina virtual sem instalar o Ubuntu server manualmente do zero. O item opcional na lista de requisitos se trata de uma máquina virtual já pronta que pode ser importada no VirtualBox. Essa opção poupa bastante tempo e é razoavelmente intuitiva, entretando para esse laboratório a máquina virtual foi criada e configurada manualmente.

1. **Criando a Máquina Virtual:**
    - Já com o VirtualBox instalado vá em "Novo"
    - Dê um nome a sua máquina virtual
    - Selecione o diretório onde ela será armazenada (esse passo é importante já que os possíveis logs de erro estarão aqui). 
    - Selecione o tipo como Linux e por fim em version busque a distro especifica que deseja instalar ou selecione "outro" que provavelmente estará no fim da lista de versões. Avance para a próxima etapa.
    <center>
    <img src="img/summary_vm_creation.png" alt="Configurando o nome do servidor" width="600"/>
    </center>
    
    - Nesta etapa selecione a memória RAM e o número de núcleos de processamento que sua máquina virtual terá acesso. As recomendações para o Ubuntu Server é de 1GB de RAM. Um único processador também deve ser suficiente, entretanto nesse laborátorio a máquine tem 2GB de RAM  e 2 núcleos. Avance para a próxima etapa.
    - Seleciona a quantidade de memória que sua máquina terá a disposição, utilizaremos 25GB.
    - Selecione sua máquina virtual recém criada e a inicie. 
    - O passo anterior resultará em um popup de erro (é possivel evitar esse caminho adicionando a imagem ISO à máquina antes fazer o primeiro boot) com um campo de seleção, clique no botão de seta para baixo e em "outro" para navegar até a imagem do Ubuntu server. Clique na opção de montar e reiniciar.
    <center>
    <img src="img/iso_error_first_vm_boot.png" alt="Configurando o nome do servidor" width="600"/>
    </center>
    - A máquina irá reiniciar e você irá se deparar com o GRUB, selecione "Try or install Ubuntu Server"
    - A partir daqui tudo é intuitivo e é necessário atenção apenas na segunda tela do processo de intalação onde você deve informar o layout do seu teclado. 
    - Outro ponto de atenção virá algumas telas depois onde você deve decidir se deseja ou não instalar um servidor SSH, essa escolha fica ao seu critério, porém é bastante mais fácil logar na sua máquina via SSH e copiar e colar comandos no terminal hospedeiro do que configurar o VirtualBox para aceitar tranferencia entre as máquinas convidada e hospedeira.


2. **Configuração Inicial da VM:**
   - Se você optou por usar SSH para se conectar à sua máquina faça isso nesse passo. Para isso:
        - Vá as configurações da sua máquinada, na aba rede mude o adaptador para Bridge ou leia [isso](https://www.baeldung.com/linux/virtualbox-ping-guest-machines) e [isso](https://www.virtualbox.org/manual/UserManual.html#natforward) e vá pelo seu próprio caminho
        - No seu ubuntu server rode `ip addr` e descubra seu IP.
        - Em seguida vá ao terminal do máquina hospedeira (Linux ou Mac, caso você use Windows utilize o [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/) ou algo similar) faça o login com `ssh {NOME DO SEU USUÁRIO}@{IP DA MÁQUINA VIRTUAL}`, digite "yes" para a pergunta que surgirá na sua tela em seguinda entre com a senha e você estará logado.
   - Apesar de você ter recém instalado seu Ubuntu Server é primordial rodor o comando `sudo apt update && sudo apt upgrade` isso garantirá que o sistema está devidamente atualizado e ajuda a evitar erros e conflitos, além de manter seu sistema mais seguro.
   - O Elasticsearch roda em cima de uma JVM logo o próximo passo é instalar o Java. Nesse laboratório será utilizado Java 17. Para isso insira o comando `sudo apt install openjdk-17-jdk -y`.
   - Verifique a instalação com `java -version`

3. **Instalando o Elasticsearch 8**
    - Como o Elasticsearch tem o próprio repositório o primeiro passo é baixar e instalar a chave pública deles:
    
     &emsp;&emsp;`curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg`
    
    - Adicione o repositório do Elasticsearch à sua APT source list:

     &emsp;&emsp;`echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`

    - Faça o update do sistema mais uma vez:
    
    &emsp;&emsp;`sudo apt update`

    - Instale o Elasticsearch:

    &emsp;&emsp;`sudo apt install elasticsearch`

4. **Configurando o Elasticsearch**
    
    - Acesse o arquivo de configuração com o seu editor favorito (I use VIM BTW 😂), esse arquivo se encontra em `/etc/elasticsearch/elasticsearch.yml`. Segue o exemplo com o VIM:

    &emsp;&emsp;`sudo vim /etc/elasticsearch/elasticsearch.yml`

    - Primeiramente vamos definir o nome do nosso servidor, busque por `node.name` no vim isso pode ser feito utilizando `/`. Esse nome deve ser amigável para humanos já que facilmente identifica seu servidor. O Elasticsearch irá por padrão utilizar o nome da máquina onde ele está instalado, porém, para customizar a mesma altere a linha em questão para algo parecido com isso:

    <center>
    <img src="img/node.name_settup.png" alt="Configurando o nome do servidor" width="600"/>
    </center>

    - A seguir iremos alterar a linha que contém `network.host` esse campo determina a quais interfaces de rede o Elasticsearch ira servir. Por padrão o Elasticsearch vem configurado para responder apenas requisições do localhost, entretanto para esse laboratório queremos que ele responda a todos os IPs locais (o que inclui nossa máquina hospedeira), em produção essa escolha deve ser avaliada com mais cuidado, [esse é um artigo interessante para usar de referência](https://opster.com/guides/elasticsearch/glossary/elasticsearch-network-host-configuration/).
   
    <center>
    <img src="img/network.host.png" alt="Configurando a(s) interfaces às quais o servidor irá servir" width="600"/>
    </center>

    - O próximo passo é definir a lista de servidores de descoberta. Em caso de formação um cluster os nós contidos nessa lista servirão para informar quem é o nó mestre e quem são os outros nós, basicamente estes são os pontos de contato para o nosso nó se situar no cluster. No caso desse laboratório como teremos um unico nó, também haverá apenas o endereço de loopback na lista. Vá a linha que contém `discovery.seed_hosts` e altere-a da seguinte forma:
    
    <center>
    <img src="img/seed_hosts_settup.png" alt="Configurando os nós de descoberta" width="600"/>
    </center>

    - O útimo passo vem acompanhado de um **ALERTA** essa configuração não é válida para produção e **NÃO** deve ser repetida fora de um ambiente de desenvolvimento ou teste. Iremos alterar as configurações de segurança, desativando-as. A segurança do ES server para realizar controle de acesso, criptografia da comunicação, logs de auditoria e gerenciamento de usuário. Em uma futura versão esse laboratório sera atualizado para habilitar essa opção. Segue a alteração na linha contendo `xpack.security.enabled`:

    <center>
    <img src="img/security_settup.png" alt="Configurando os nós de descoberta" width="600"/>
    </center>

    - Salve o arquivo e saia do editor.

    - Inicie o serviço com `sudo systemctl start elasticsearch`

    - Verifique o estado do serviço com `sudo systemctl status elasticsearch`

    <center>
    <img src="img/ES_service_active.png" alt="Configurando os nós de descoberta" width="600"/>
    </center>

    - Habilite o serviço para ser iniciado junto com a inicialização do sistema. Para isso rode `sudo systemctl enable elasticsearch`

    - Para testar o servidor execute o comando `curl -X GET 'http://localhost:9200'` ou `curl -X GET 'http://{IP DA VM}:9200'` caso você tenha seguido o passo a passo a risca e deseje usar o sistema hospedeiro para testar seu servidor Elasticsearch.

    <center>
    <img src="img/es_first_response.png" alt="Configurando os nós de descoberta" width="600"/>
    </center>


## **Parabéns você concluiu a instalação e configuração do Elasticsearch 8** 
    
    
    

