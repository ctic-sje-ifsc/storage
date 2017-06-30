# Storage/Armazenamendo FreeNAS

Nesse repositório é descrita a nova forma de armazenamento permanente dos arquivos que estamos implementando. Adotamos a solução [FreeNAS](http://www.freenas.org/) após várias pesquisas e testes. A escolha dessa solução e desse formato centralizado dos arquivos em um storage se deu pela facilidade de manutenção, velicidade e estabilidade implementada por essa ferramenta.

## A estrutura adorada é a seguinte:
![Desenho lógico](docs/storage_armazenamento.png)

Utilizaremos como storage servidores com a seguinte especificações de Hardware:

* 2 x HP ProLiant 360 G6: 
  * 1 x Intel(R) Xeon(R) CPU E5504 @ 2.00GHz
  * 4 x 8 GB (32GB)
  * 2 x Broadcom NetXtreme II BCM5709 Gigabit Ethernet em agregação de link LACP (IEEE802.3ad)
  * 4 x HD SAS 10k  600 GB (Em RaidZ2 = 1.1TB Utilizável)

## O que é o FreeNAS:

"*FreeNAS é um sistema operacional que pode ser instalado em praticamente qualquer plataforma de hardware para compartilhar dados em uma rede. FreeNAS é a maneira mais simples de criar um lugar centralizado e facilmente acessível para seus dados. Use o FreeNAS com o ZFS para proteger, armazenar, fazer backup, todos os seus dados. FreeNAS é usado em todos os lugares, para a casa, pequenos negócios e empresas.*" [Referência](http://www.freenas.org/)

## O que é o ZFS:

"*O FreeNAS é o sistema operacional de armazenamento de código aberto mais popular do mundo, não só por causa de suas características e facilidade de uso, mas também o que está por baixo da superfície: o sistema de arquivos ZFS. Com mais de sete milhões de downloads, a FreeNAS colocou o ZFS em mais sistemas do que qualquer outro produto ou projeto até o momento e é usado em todos os lugares de casas para empresas.
O FreeNAS usa o ZFS porque é um sistema de arquivos de código aberto e gerenciador de volumes prontos para a empresa com flexibilidade sem precedentes e um compromisso intransigente com a integridade dos dados. O ZFS é um sistema de arquivos verdadeiramente de última geração que elimina a maioria, senão todas as deficiências encontradas em sistemas de arquivos legados e dispositivos RAID de hardware. Uma vez que você vai ao ZFS, você nunca vai querer voltar.*" [Referência](http://www.freenas.org/zfs/)


## Testes de possíveis defeitos nos discos do FreeNAS:

Após a implementação e configuração do FreeNAS seguindo a [documentação oficial](http://doc.freenas.org/) fizemos alguns testes :

* Com o sistema operando, retiramos dois HD's aleatóriamente.
  * Continuou funcionando normalmente.
* Colocamos os HD's invertido no local.
  * O sistema reconheceu e sincronizou sozinho os discos.
* Retiramos outro HD(diferente dos dois anteriores) e colocamos outro HD limpo no local.
  * O Sistema reconheceu que foi substituido o HD e perguntou se queriamos associar esse disco ao RaidZ2.
    * Ele começou a sincronizar os arquivos e tudo certo.

Achamos os testes bem satisfatórios, ele se mostrou bem estável quanto a problemas com os discos. Não chegamos a testar a queda do desempenho no momento dos sincronimos dos discos.
    
## Testes de desempenho de escrita de dados no FreeNAS:

Montamos em um máquina um armazenamento do FreeNAS por NFS e fizemos os testes a seguir.

### Velocidade de escrita em NFS

```sh
sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
1024+0 registros de entrada
1024+0 registros de saída
1073741824 bytes (1,1 GB) copiados, 9,82436 s, 109 MB/s
```
### Velocidade de leitura em NFS
```sh
dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 registros de entrada
1024+0 registros de saída
1073741824 bytes (1,1 GB) copiados, 9,19565 s, 117 MB/s
```
Podemos constatar que ele pode, dependendo do tipo de arquivos, chegar ao limite da rede. Escrita 109 MBps = 872Mbps, e  leitura 117 MBps = 936Mbps.

[Referência dos testes](https://serenity-networks.com/simple-method-to-benchmark-read-write-speeds-from-the-linux-command-line/)

