# Como configurar a memória de vídeo (VRAM)

## Como a memória de vídeo funciona no Nintendo DS
A memória de vídeo, ou VRAM,  é dividida em bancos de memória. Existem nove bancos, nomeados com as letras do alfabeto: A, B, C, D, E, F, G, H e I. Esses bancos servem para nós podermos atribuir alguma responsabilidade à eles. Por exemplo, nós podemos usar o banco A para guardar sprites da tela de cima.

Quando nós atribuimos uma função à um banco, estamos na verdade dizendo qual endereço de memória ele ficará responsável. Por exemplo, ao atribuir o banco A aos sprites da tela de cima, nós estamos dizendo que ele ficará com o endereço 0x06400000 ou 0x06420000 (não se assuste com esses números).

Contudo, nós não temos total liberdade de definir o uso de cada banco, por exemplo, o banco B não pode ser usado para sprites da tela debaixo. Para não ser necessário decorar as configurações possíveis, existe esse site: [https://mtheall.com/banks.html](https://mtheall.com/banks.html). Ele nos permite escolher o que fazer com cada banco, sem criar configurações impossíveis ou conflitos - como usar dois bancos para uma mesma função, já que eles teriam o mesmo endereço -. 

![](VRAMSitePrint.png)

Para usar é bem simples, basta escolher um uso possível para o banco e selecioná-lo ao apertar o circulo. Só uma coisa, o programa chama a memória dedicada aos sprites de “OBJ VRAM”

Outra coisa, no canto superior direito, o programa também nos da o código que configura os bancos. Caso queira, pode só copiar e colar ele. Apesar disso, nós iremos programá-lo por conta própria para entendê-los.
![](PrintWithVRAMCodeSelected.jpg)
## Como configurar a memória de video
Para escolher uma configuração de bancos, é necessário saber como precisaremos usar a memória. Vamos supor que nosso objetivo é renderizar um sprite e um background em cada tela. Para isso, precisaremos escolher um banco para cada uma dessas coisas. 

Com base no site acima, nós podemos fazer o seguinte uso de bancos:

|**Banco**|**Função**|
|---|---|
|A|Sprites da tela de cima|
|B|Background da tela de cima|
|C|Background da tela debaixo|
|D|Sprites da tela debaixo|

Após escolher os bancos, é necessário configurá-los no código. Para isso, o libnds nos permite usar os métodos `vramSetBankX(VRAM_X_TYPE)` sendo X o nome do banco e `VRAM_X_TYPE` o modo que usaremos o banco. 

Para configurar o banco A, usaremos `vramSetBankA()`. Como queremos usá-lo para sprites, o modo deve ser `VRAM_A_MAIN_SPRITE` , com isso, o código ficará `vramSetBankA(VRAM_A_MAIN_SPRITE);`. Nós podemos seguir a mesma lógica com o banco D, ficando `vramSetBankD(VRAM_D_SUB_SPRITE);`. Agora para os backgrounds, nós usaremos `vramSetBankB(VRAM_B_MAIN_BG);` e `vramSetBankC(VRAM_C_SUB_BG);`. Sendo assim, o código ficará:
```C++
vramSetBankA(VRAM_A_MAIN_SPRITE);
vramSetBankB(VRAM_B_MAIN_BG);
vramSetBankC(VRAM_C_SUB_BG);
vramSetBankD(VRAM_D_SUB_SPRITE);
```
Só uma observação, como esse código é uma inicialização, ele deve ser chamado antes do loop infinito do jogo.

Com aquele código, nós temos os bancos configurados para o que nós queríamos fazer. Mas tem alguns detalhes que é bom ressaltar. Tem funções quem podem precisar de mais de um banco, como backgrounds. Por exemplo, podemos configurar o background da tela de cima para usar os bancos A e B. Mas se usarmos o VRAM_A_MAIN_BG e VRAM_B_MAIN_BG ocorrerá um conflito, pois dois bancos estarão sendo usados para exatamente a mesma coisa. Para resolver isso, existem “sub-bancos” (esse não é um nome oficial até onde sabemos). Eles funcionam igual os bancos normais, mas tem um tamanho menor.

Para configurar esses “sub-bancos” é necessário especificar o início do endereço de memória que você quer para o banco após o nome normal. Por exemplo: `VRAM_A_MAIN_BG_0x06000000`. 

Só que isso é um pouco complicado, e por isso eu recomendo configurar esses bancos apenas por meio do  site que citei, já que ele já fornece os endereços direto. No exemplo acima, o código ficaria:
```C++
vramSetBankA(VRAM_A_MAIN_BG_0x06000000);  
vramSetBankB(VRAM_B_MAIN_BG_0x06020000);
```