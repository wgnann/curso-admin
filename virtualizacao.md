# Curso para Admin
Material para algumas aulas do curso de admin da Linux.

## Virtualização e o serviço mais importante de um admin

### O que é uma máquina virtual?
Uma máquina virtual é uma abstração de um computador realizada a partir de um conjunto de programas que possibilitam emular um computador dentro de outro.

Uma máquina virtual pode se apresentar em pelo menos três sabores: virtualização completa, virtualização assistida por hardware e containers.

Virtualizar completamente significa que todos os componentes do computador virtual são programas de computador que emulam tais componentes. De certa forma, podemos dizer que é o modelo mais garantido do ponto de vista de todas as instruções serem realizadas pelos programas responsáveis pela virtualização, contudo, é justamente esse fato que leva a virtualização completa a ser a mais custosa. O mais usual é o Qemu, de Quick emulator.

Uma alternativa à virtualização completa é a virtualização assistida por hardware. A grande diferença é que, agora, existe um nível a mais de execução no processador que ajuda a isolar o que é rodado na máquina virtual do que é rodado no sistema da máquina de fato. Isso permite rodar as instruções diretamente no processador, evitando a necessidade de emulação completa em vários casos. O custo, claro, é que a troca de contexto fica enorme. Contudo, trata-se de um custo bem menor ao de emular tudo. As tecnologias abertas mais usuais nessa direção são o KVM (Kernel-based Virtual Machine) e o Xen.

Outra alternativa é virtualizar no nível do sistema operacional, separando os contextos dos processos "virtualizados" do contexto dos processos do sistema de fato. Como se trata meramente de uma estratégia de isolamento de processos, o kernel terá de ser o mesmo do sistema de fato. Isso não impede a possibilidade de limitar os recursos concedidos ao contexto da máquina virtual. Entretanto, expõe aumenta a superfície de exposição da máquina de fato a ataques, pois, apesar do isolamento, o kernel continua sendo o mesmo para ambas. Naturalmente, aos olhos da máquina de fato, esse é o modelo menos custoso porque nem emula um processador, tampouco impõe pesadas trocas de contexto. Um sistema de containers bastante comum em distribuições de Linux é o LXC (Linux Containers).

### Por que virtualizar?
Num mundo pregresso, os servidores eram computadores de proporções monstruosas onde todos os programas rodavam em conjunto. A fim de usar o espaço de forma eficiente, as distribuições de Linux comumente adotam um sistema de bibliotecas compartilhadas, isto é, dois programas que compartilham um dado conjunto de bibliotecas ocuparão menos espaço do que se fossem instalados em separado. No entanto, essa economia de espaço acompanha uma debacle potencial: e se as bibliotecas exigidas por ambos os programas se tornarem incompatíveis entre si?

O compartilhamento de recursos, nesse caso, não tem como problema somente o conflito entre dependências. Uma vulnerabilidade em um dos sistemas pode comprometer todos os sistemas do monólito onde os programas foram instalados. Via-de-regra esse altíssimo grau de acoplamento não costuma ser desejável e é uma fonte perpétua de dívida técnica.

Como resolver esse problema num mundo sem virtualização? Simples(?), basta aumentar o número de máquinas, deixando cada sistema numa máquina distinta... É suficiente ler a frase anterior em voz alta para verificar a cilada em construção. Mais máquinas implica, mecanicamente, mais custo de energia, mais custo de equipamento e mais espaço necessário para acomodá-las. Mas, bem pior que os problemas de natureza mecânica, mais máquinas implica em mais computadores para fazer manutenção e, principalmente, mais computadores para quebrar. Hardware não é a coisa mais graciosa que existe no mundo da computação, portanto minimizar o hardware necessário para desempenhar uma atividade não parece um princípio de um todo ruim. Por fim, vale ressaltar, ainda, o tamanho da capacidade ociosa oriunda da existência de vários computadores para realizarem tarefas triviais.

Com o advento da tecnologia de virtualização, tornou-se possível tanto desacoplar os sistemas quanto usar o hardware de forma mais eficiente. A ideia é meramente distribuir os sistemas em máquinas virtuais.

### Algumas tecnologias utilizadas
Para a presente aula, utilizaremos diversas tecnologias. Algumas delas serão descritas adiante.

#### libvirt
libvirt é uma API que facilita as rotinas de virtualização. Possibilita sem muito esforço a criação de instâncias de máquina virtual tanto usando Qemu quanto usando o KVM. Ela pode ser utilizada por outras ferramentas de virtualização como o Vagrant.

#### Vagrant
O Vagrant fornece uma camada adicional de abstração para a criação de máquinas virtuais, disponibilizando a instalação de uma máquina virtual em cerca de segundos. È bastante utilizado em ambientes de desenvolvimento dada sua praticidade.

#### CUPS
Sistema de filas de impressão para Unix.

### Atividades

1) Instalar um debian com um usuário admir.

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

6) Criar uma máquina Debian no Vagrant.
Basta seguir o trecho de Ubuntu, mas usando como fonte de imagem `generic/debian11`.

### Referências
  - https://www.admin-magazine.com/Articles/Hardware-assisted-Virtualization

## Organização e instalação de um sistema web
