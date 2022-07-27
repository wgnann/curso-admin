## Virtualização e o serviço mais importante de um admin

### O que é uma máquina virtual?
Uma máquina virtual é uma abstração de um computador realizada a partir de um conjunto de programas que possibilitam emular um computador dentro de outro.

Uma máquina virtual pode se apresentar em pelo menos três sabores: virtualização completa, virtualização assistida por hardware e containers.

Virtualizar completamente significa que todos os componentes do computador virtual são programas de computador que emulam tais componentes. De certa forma, podemos dizer que é o modelo mais garantido do ponto de vista de todas as instruções serem realizadas pelos programas responsáveis pela virtualização, contudo, é justamente esse fato que leva a virtualização completa a ser a mais custosa. O mais usual é o Qemu, de Quick emulator.

Uma alternativa à virtualização completa é a virtualização assistida por hardware. A grande diferença é que, agora, existe um nível a mais de execução no processador que ajuda a isolar o que é rodado na máquina virtual do que é rodado no sistema da máquina de fato. Isso permite rodar as instruções diretamente no processador, evitando a necessidade de emulação completa em vários casos. O custo, claro, é que a troca de contexto fica enorme. Contudo, trata-se de um custo bem menor ao de emular tudo. As tecnologias abertas mais usuais nessa direção são o KVM (Kernel-based Virtual Machine) e o Xen.

Outra alternativa é virtualizar no nível do sistema operacional, separando os contextos dos processos "virtualizados" do contexto dos processos do sistema de fato. Como se trata meramente de uma estratégia de isolamento de processos, o kernel terá de ser o mesmo do sistema de fato. Isso não impede a possibilidade de limitar os recursos concedidos ao contexto da máquina virtual. Entretanto, aumenta a superfície de exposição da máquina de fato a ataques, pois, apesar do isolamento, o kernel continua sendo o mesmo para ambas. Naturalmente, aos olhos da máquina de fato, esse é o modelo menos custoso porque nem emula um processador, tampouco impõe pesadas trocas de contexto. Um sistema de containers bastante comum em distribuições de Linux é o LXC (Linux Containers).

### Por que virtualizar?
Num mundo pregresso, os servidores eram computadores de proporções monstruosas onde todos os programas rodavam em conjunto. A fim de usar o espaço de forma eficiente, as distribuições de Linux comumente adotam um sistema de bibliotecas compartilhadas, isto é, dois programas que compartilham um dado conjunto de bibliotecas ocuparão menos espaço do que se fossem instalados em separado. No entanto, essa economia de espaço acompanha uma debacle potencial: e se as bibliotecas exigidas por ambos os programas se tornarem incompatíveis entre si?

O compartilhamento de recursos, nesse caso, não tem como problema somente o conflito entre dependências. Uma vulnerabilidade em um dos sistemas pode comprometer todos os sistemas do monólito onde os programas foram instalados. Via-de-regra esse altíssimo grau de acoplamento não costuma ser desejável e é uma fonte perpétua de dívida técnica.

Como resolver esse problema num mundo sem virtualização? Simples(?), basta aumentar o número de máquinas, deixando cada sistema numa máquina distinta... É suficiente ler a frase anterior em voz alta para verificar a cilada em construção. Mais máquinas implica, mecanicamente, mais custo de energia, mais custo de equipamento e mais espaço necessário para acomodá-las. Mas, bem pior que os problemas de natureza mecânica, mais máquinas implica em mais computadores para fazer manutenção e, principalmente, mais computadores para quebrar. Hardware não é a coisa mais graciosa que existe no mundo da computação, portanto minimizar o hardware necessário para desempenhar uma atividade não parece um princípio de um todo ruim. Por fim, vale ressaltar, ainda, o tamanho da capacidade ociosa oriunda da existência de vários computadores para realizarem tarefas triviais.

Com o advento da tecnologia de virtualização, tornou-se possível tanto desacoplar os sistemas quanto usar o hardware de forma mais eficiente. A ideia é meramente distribuir os sistemas em máquinas virtuais.

### Considerações de segurança
Adotar máquinas virtuais inerentemente é menos seguro do que separar as aplicações em instâncias físicas distintas. Como temos diversas modalidades de virtualização vale discutir o que cada uma tem de problema.

#### Virtualização
No limite, a máquina virtual é uma peça de software que roda numa máquina real - vamos chamá-la de *host* - e, em geral, sobre um sistema operacional já instalado.

Do ponto de vista do *host* a máquina virtual oriunda da virtualização completa é nada mais que mais ou programa rodando, então a superfície de ataque disponível é a mesma de qualquer outro processo comprometido. Já na virtualização assistida por hardware também é possível atacar a infraestrutura que possibilita a própria virtualização. Como se trata de um conjunto extra de instruções, há um conjunto maior de coisas com as quais se deve tomar cuidado.

Como se trata de um programa rodando no *host*, a compromissão de uma máquina virtual dispõe dos mesmos problemas da compromissão de um programa qualquer: uma vez comprometido, o atacante terá no sistema operacional o mesmo nível de privilégio que o programa atacado. Nesse sentido é recomendado não rodar as máquinas virtuais com permissão de `root`.

#### Containers
Um container também é um programa rodando no *host*, mas, ao contrário da máquina virtual de fato, ele **compartilha o kernel** com o *host*. Isso significa que o cuidado com um container, se comparado à uma máquina virtual, deve ser redobrado. Deve-se limitar bastante o que um container deve fazer, por exemplo, utilizando controles de acesso mandatórios. Em linhas gerais, o controle mandatório funciona como um filtro de chamadas de sistema: o processo solicita ao sistema operacional uma operação (e.g. acessar um determinado arquivo) e o sistema de controle verifica se esse processo está autorizado. Segue como corolário que se torna mais difícil para um programa realizar operações que não foram permitidas, reduzindo a superfície de ataque.

Existem duas possibilidades para se rodar um container: como `root` ou como usuário comum. O primeiro modelo é conhecido por *container privilegiado* enquanto o segundo, por *container desprivilegiado*. Rodar o container sem ser como `root`, por construção, diminui a superfície de ataque dado que, aos olhos do *host*, o container será somente mais um processo comum. Como o kernel é compartilhado.

#### Regra de bolso
Sempre que possível, vale a pena tentar usar containers desprivilegiados e garantindo, ainda, que os recursos liberados sejam os mínimos possíveis.

Se houver a necessidade de uma separação maior (e.g. servidor de SSH), pode ser interessante usar o modelo de máquina virtual.

Por fim, se houver a necessidade de se rodar um kernel distinto ou mesmo um sistema numa arquietura completamente distinta, o que resta é usar uma máquina virtual.

### Algumas tecnologias utilizadas
Para a presente aula, utilizaremos diversas tecnologias. Algumas delas serão descritas adiante.

#### AppArmor
Sistema de controle de acesso mandatório utilizado em algumas distribuições de Linux. Confina os processos a um conjunto pré-determinado de recursos, limitando a superfície de um eventual ataque.

#### libvirt
libvirt é uma API que facilita as rotinas de virtualização. Possibilita sem muito esforço a criação de instâncias de máquina virtual tanto usando Qemu quanto usando o KVM. Ela pode ser utilizada por outras ferramentas de virtualização como o Vagrant.

#### Vagrant
O Vagrant fornece uma camada adicional de abstração para a criação de máquinas virtuais, disponibilizando a instalação de uma máquina virtual em cerca de segundos. È bastante utilizado em ambientes de desenvolvimento dada sua praticidade.

#### CUPS
Sistema de filas de impressão para Unix.

### Atividades

1) Instalar um debian com um usuário chamado admir.

2) Instalar o libvirt e o Vagrant.

```bash
apt install libvirt-daemon-system libvirt-clients vagrant virt-viewer
usermod -G libvirt-qemu,libvirt admir
```

3) Criar uma máquina de Ubuntu no Vagrant.

```bash
mkdir -p vagrant/ubuntu
cd vagrant/ubuntu
vagrant init generic/ubuntu2004
vagrant up
```

É possível encontrar imagens de VMs para o Vagrant aqui: https://app.vagrantup.com/boxes/search.

4) Acessar a instância Ubuntu por SSH e instalar o CUPS.

```bash
vagrant ssh
sudo apt update
sudo apt install cups
```

5) Na instância Ubuntu, configurar o CUPS para permitir acesso à interface administrativa.

```bash
sudo editor /etc/cups/cupsd.conf
```

Trocar `Listen localhost:631` por `Port 631`.

Adicionar nas seções `<Location />` e `<Location /admin>`, abaixo de `Order`: `Allow @LOCAL`.

```bash
sudo systemctl restart cups
```

A fim de verificar se tudo correu como o esperado, basta obter o endereço IP da máquina, abrir um navegador e tentar acessá-lo:

```bash
ip address
```

O comando listará todas as suas interfaces de rede. No contexto do exercício, serão dois:
  - `lo`
  - `eth0`

O nome `lo` é dado à interface de *loopback*, cujo objetivo é permitir a comunicação do computador com ele mesmo. O endereço IP associado à interface vem justamente depois da palavra `inet`.

Com o endereço obtido no passo anterior, suporemos que é o `192.168.31.41`, abrir o navegador e acessar: `192.168.31.41:631`. Deverá aparecer uma página com a palavra **CUPS**.

6) Criar uma máquina Debian no Vagrant.
Basta seguir o trecho de Ubuntu, mas usando como fonte de imagem `generic/debian11`.

7) Acessar a máquina Debian, instalar e configurar o cups-ipp-utils. Ele fornecerá uma impressora virtual que fala IPP.

```bash
vagrant ssh
sudo apt update
sudo apt install cups-ipp-utils
```

Obteremos, agora, o endereço IP da máquina Debian a partir do comando `ip` tal qual fizemos anteriormente.

```bash
ip address
```

Anotar o endereço em algum lugar, pois ele será utilizado para instalar a impressora.

Agora vamos rodar um programa que faz o papel da impressora. Ele assume como parâmetros:
  - `-k`: guarda os arquivos impressos;
  - `-f`: determina com qual tipo de arquivo a impressora trabalha;
  - `-r off`: desabilita anúncio via broadcast.

```bash
sudo ippeveprinter -p 631 -k -f application/pdf -r off nomedasuaimpressora
```

8) Instalaremos no CUPS a impressora. No navegador, e supondo que o endereço do passo anterior é `192.168.27.18`:
  - Voltar à página do **CUPS**;
  - Clicar em **Administration** e, depois, em **Add Printer**. Aparecerá uma página escrevendo **Upgrade Required** e voltará para a página anterior depois de uns instantes;
  - Clicar novamente em **Add Printer**;
  - Na nova página, selecionar `Internet Printing Protocol (ipp)` e clicar em **Continue**;
  - Preencheremos, no campo **Connection**, o endereço obtido no passo anterior acompanhado do protocolo utilizado e, depois, clicar em **Continue**. O conteúdo completo ficará:

```bash
ipp://192.168.27.18/ipp/print
```

  - Basta escolher um nome e marcar para compartilhar (*Share This Printer*) e continuar;
  - Falta escolher o *driver*. Escolheremos `Generic`, depois clicaremos em **Continue** para escolhermos o modelo. Infelizmente o CUPS não reconheceu o modelo, então ele apresentará o modelo como `{current_make_and_model}`. Basta clicar em **Add Printer**;
  - Para finalizar, o CUPS dá uma lista de opções padrão para a impressora. Por exemplo, frente-e-verso é uma opção possível. Clicar em **Set Default Options**.

9) Vamos imprimir alguma coisa para testar. Talvez seja necessário instalar o `evince`.

```bash
/bin/su -
apt install evince
exit
```

Baixar um PDF. Exemplo:

```bash
wget www.linux.ime.usp.br/~gnann/100m.pdf
```

É possível configurar momentaneamente um servidor de impressão a partir da variável `CUPS_SERVER`. Vamos usar o endereço do Ubuntu, que é onde roda o CUPS.

```bash
export CUPS_SERVER=192.168.31.41
evince 100m.pdf
```

Imprimir como o usual, escolhendo a impressora recém instalada.

**OBS:** se não definirmos a variável, o padrão é usar o *hostname* da máquina.

10) Resta-nos verificar se algo foi impresso.

Na máquina Debian, que ainda deve estar rodando o `ippeveprinter`, fechá-lo com um `CTRL+C`.

Depois, verificar se algum documento foi impresso em `/tmp/ippeveprinter.NUMERO`.

```bash
ls /tmp/ipp*
```

### Referências
  - manual (comando `man`)
  - README.Debian.gz
  - https://www.admin-magazine.com/Articles/Hardware-assisted-Virtualization
  - https://debian-handbook.info/browse/pt-BR/stable/sect.apparmor.html
