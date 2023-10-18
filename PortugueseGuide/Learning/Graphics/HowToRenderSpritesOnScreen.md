# Como renderizar sprites na tela
## Passos
1. Inicialização do DS
	1. Configurar modo de vídeo
	2. Configurar bancos de memória
	3. Inicializar OAM
2. Inicialização do sprite
	1. Copiar tiles na memória
	2. Copiar paleta na memória
3. Colocar sprite na tela
4. Limpar tela no inicio de cada frame

## Inicialização do DS
O primeiro passo para renderizar um sprite na tela e configurar o modo de vídeo. Como não faremos nada de muito diferente, usaremos o Mode 5. 
```C++
videoSetMode(MODE_5_2D);
videoSetModeSub(MODE_5_2D);
```

Após isso, configuraremos os bancos de memória. Pretendo renderizar dois sprites na tela de cima, e dois na tela debaixo. Para isso, usarei os banco A para sprites da tela de cima, e D para tela debaixo:
```C++
vramSetBankA(VRAM_A_MAIN_SPRITE);
vramSetBankD(VRAM_D_SUB_SPRITE);
```

Após isso é necessário inicializar o OAM, para isso chamaremos a função `oamInit()`. Ela leva três argumentos, o primeiro é o endereço de qual OAM queremos inicializar (`oamMain` para tela de cima e `oamSub` para debaixo). 

O segundo é o modo que os sprites são mapeados. Existem dois modos para fazer isso: o 2D e o 1D. O 2D só pode ser usado para bitmaps. Como não estamos usando eles, eu escolhi mapear em 1D, por isso usarei `SpriteMapping_1D_128`. O 128 indica quantos bytes tem até o offset (deslocamento).

E o terceiro parâmetro é se o OAM usará paletas extendidas, a qual não usaremos por agora. Portanto, o código ficará assim: 
```C++
oamInit(&oamMain, SpriteMapping_1D_128, false);
oamInit(&oamSub, SpriteMapping_1D_128, false);
```
Depois disso, terminamos a inicialização do DS. O código ficou assim:
```C++
videoSetMode(MODE_5_2D);
videoSetModeSub(MODE_5_2D);

vramSetBankA(VRAM_A_MAIN_SPRITE);
vramSetBankD(VRAM_D_SUB_SPRITE);

oamInit(&oamMain, SpriteMapping_1D_128, false);
oamInit(&oamSub, SpriteMapping_1D_128, false);
```


## Inicialização do sprite
Primeiramente precisamos carregar o sprite na memória. Para isso, precisamos incluir o header file gerado pelo GRIT, o qual contem as informações sobre os tiles e também sobre a paleta. Os sprites que incluiremos são:
```C++
#include"PlaceHolderSprite32x32.h";
#include"PlaceHolderSprite64x64.h";
```

Antes de carregar o sprite, nós precisamos saber exatamente "onde" carregá-los. Para isso nós usamos a função `oamAllocateGfx()`. Esse método leva três argumentos, o primeiro é o endereço de qual `oam` carregar. Novamente, pode ser `&oamMain` ou `&oamSub`.

O segundo argumento é o tamanho do sprite. Para configurar esse tamanho nós usamos o enum `SpriteSize` que vem com o libnds. Esse tamanho representa a dimensão do sprite (por exemplo 32x32, 64x64, ...). 

E o terceiro argumento é o formato de cores. Para configurá-lo usamos o enum `SpriteColorFormat`. Nós podemos configurá-la como 16 cores, 256 cores ou bitmap. Como estamos usando sprites de 16 cores, usaremos a primeira opção. Desse modo, o código ficará assim:
```C++
void* spriteAddressMain1 = oamAllocateGfx(&oamMain, SpriteSize_32x32, SpriteColorFormat_16Color);
void* spriteAddressMain2 = oamAllocateGfx(&oamMain, SpriteSize_64x64, SpriteColorFormat_16Color);

void* spriteAddressSub1 = oamAllocateGfx(&oamSub, SpriteSize_32x32, SpriteColorFormat_16Color);
void* spriteAddressSub2 = oamAllocateGfx(&oamSub, SpriteSize_64x64, SpriteColorFormat_16Color);
```

No código acima, nós conseguimos o endereço para carregar dois sprites na tela debaixo e dois na de cima. Agora nós já podemos carregar o sprite na memória, e esse é um processo bem simples, basta usar o método `dmaCopy()`. Ele leva três parâmetros. O primeiro deles é o que queremos copiar (no caso, os tiles que a gente acessa pelos includes), o segundo é onde copiar (o endereço que o OAM nos deu) e o terceiro é o tamanho do que será copiado (como estamos copiando um sprite, usaremos o enum do `SpriteSize`).

Assim, nosso código de copiar os sprites ficará desse jeito:
```C++
dmaCopy(PlaceHolderSprite32x32Tiles, spriteAddressMain1, SpriteSize_32x32);
dmaCopy(PlaceHolderSprite64x64Tiles, spriteAddressMain2, SpriteSize_64x64);

dmaCopy(PlaceHolderSprite32x32Tiles, spriteAddressSub1, SpriteSize_32x32);
dmaCopy(PlaceHolderSprite64x64Tiles, spriteAddressSub2, SpriteSize_64x64);
```

Depois de copiar os tiles, nós precisamos copiar a paleta de cores. Hardwares mais antigos, como o Nintendo DS, usavam uma paleta para saber quais cores cada objeto tem. Isso serve para deixar mais leve na renderização. 

Antes de iniciar a cópia, é importante saber que cada paleta de cor possui 16 cores - por conta das configurações que fizemos -. Nós precisaremos usar esse número algumas vezes, por isso, é recomendável que você crie uma constante do tipo int que tenha esse valor. A minha se chamará `COLORS_PER_PALETTE`. 

Para carregar a paleta nós também usaremos o `dmaCopy`. Se você se lembra de como usamos esse método para copiar os sprites, você sabe que o primeiro argumento é o que copiaremos. No caso, é a paleta gerada pelo GRIT, que está no mesmo arquivo dos tiles.

Já o segundo é onde copiaremos. E o libnds nos fornece mais ou menos esse valor. Eu digo mais ou menos porque ele fornece o endereço apenas da primeira paleta. Mas mesmo assim, a partir desse endereço, podemos acessar as outras. 

Existem duas definições para os endereços da paleta: `SPRITE_PALETTE` e `SPRITE_PALETTE_SUB`, que servem respectivamente para as primeiras paletas da tela de cima e debaixo. 

Para a gente acessar qualquer uma das dezesseis paletas nós precisaremos usar a fórmula: `endereçoDaPrimeiraPaleta + tamanhoDaPaleta*paletaQueQueremos`. O `endereçoDaPrimeiraPaleta` são aqueles endereços que o libnds nos da. O `tamanhoDaPaleta` é o número de cores que ela possui, no caso, aquela constante que definimos. E o `paletaQueQueremos` é um número inteiro que indica qual paleta queremos (0 até 15 inclusivo). 

Para aplicar essa fórmula, nós escreveremos o seguinte método:
```C++

const int COLORS_PER_PALETTE = 16;

uint16* GetPalette(uint16* paletteAddress, int paletteID)
{
	return paletteAddress + COLORS_PER_PALETTE * paletteID;
}
```
Com esse método, nós já temos o segundo parâmetro. 

O terceiro parâmetro do `dmaCopy` é o tamanho do que copiaremos, que é aquela constante de cores por paleta que definimos.

Portanto, para copiar as paletas, nós devemos fazer o seguinte:
```C++
spriteMain1Palette =  dmaCopy(PlaceHolderSprite32x32Pal,
							  GetPalette(SPRITE_PALETTE, 0),
							  COLORS_PER_PALETTE);
spriteMain2Palette = dmaCopy(PlaceHolderSprite64x64Pal, 
							 GetPalette(SPRITE_PALETTE, 1),
							 COLORS_PER_PALETTE);

spriteSub1Palette =  dmaCopy(PlaceHolderSprite32x32Pal,
							 GetPalette(SPRITE_PALETTE_SUB, 0),
							 COLORS_PER_PALETTE);
spriteSub2Palette = dmaCopy(PlaceHolderSprite64x64Pal,
							GetPalette(SPRITE_PALETTE_SUB, 1),
							COLORS_PER_PALETTE);
```
Com isso, nós finalizamos o processo de inicialização

## Colocar sprite na tela
Para colocar um sprite na tela, nós precisamos chamar o método `oamSet` no loop infinito. Esse método possui vários argumentos, então cheque a tabela abaixo:

| Argumento |Valor esperado| O que ele é |
| --- |---| --- |
|Oam que será usado|`OamState*`| Endereço de qual engine o sprite deve ser renderizado. Ou `&oamMain` ou `&oamSub`|
|OamID|`int`|Qual entidade do OAM será renderizada. Pode ser qualquer número de 0-127 (inclusivo). Esse número não pode se repetir em uma mesma tela. É recomendado que use o menor disponível na medida que vai renderizando (0, depois 1, depois 2, ...)|
|Posição x|`int`| O número que define a posição horizontal do sprite que será renderizado|
|Posição y|`int`|O número que define a posição vertical do sprite que será renderizado|
|Prioridade|`ObjPriority`| Define a sobreposição de sprites. Por exemplo, se dois sprites forem renderizados, e um ficar em cima do outro, esse valor vai definir qual sprite deve ficar por cima. Ele deve ser configurado usando o enum `OBJPRIORITY`. Quanto menor o número da prioridade, mais em cima o sprite vai ficar (0 é a maior prioridade)|
|ID da paleta|`int`|Qual das 16 paletas o sprite vai usar (0-15 inclusivo)|
|SpriteSize|`SpriteSize`|Qual o tamanho do sprite. Usa o enum `SpriteSize`|
|SpriteColorFormat|`SpriteColorFormat`|Qual o formato da cor (16, 256 ou bitmap). Usa o enum `SpriteColorFormat`|
|Endereço do sprite|`const void*`|Endereço fornecido pelo `oamAllocateGfx`, no qual os tiles foram copiados.|
|Affine index|`int`|Isso só é usado quando um sprite é rotacionável. Por enquanto, essa documentação não terá explicações sobre isso. Caso o sprite não se rotacione, você pode colocar qualquer número menor que zero ou maior que 31 (como -1)|
|Tamanho dobrado|`bool`|Esse valor também só é usado quando o sprite é rotacionável. Quando esse valor é `true`, o sprite size é dobrado para rotação. Caso o sprite não seja rotacionável, basta usar `false`|
|Esconder|`bool`|Caso não queira que seu sprite apareça -Como quando ele está fora da tela-, você pode deixar o valor `true`|
|Flip horizontal|`bool`|Define se o sprite será girado horizontalmente (Olhando para direita ou esquerda)|
|Flip vertical|`bool`|Define se o sprite será girado  verticalmente (de cabeça para baixo)|
|Mosaico|`bool`|É um efeito gráfico aplicado ao sprite. Caso não queira, basta colocar `false`|
Com base nisso, dentro do loop infinito, nós usaremos o seguinte código para mostrar nossos sprites:
```C++
oamSet(&oamMain, 0, 100, 50, OBJPRIORITY_0, 0, SpriteSize_32x32, SpriteColorFormat_16Color, spriteAddressMain1, -1, false, false, false, false, false);

oamSet(&oamMain, 1, 300, 100, OBJPRIORITY_0, 1, SpriteSize_64x64, SpriteColorFormat_16Color, spriteAddressMain2, -1, false, false, true, false, false);

oamSet(&oamSub, 0, 200, 100, OBJPRIORITY_0, 0, SpriteSize_32x32, SpriteColorFormat_16Color, spriteAddressSub1, -1, false, false, false, true, false);

oamSet(&oamSub, 1, 400, 150, OBJPRIORITY_0, 1, SpriteSize_64x64, SpriteColorFormat_16Color, spriteAddressSub2, -1, false, false, false, false, false);
```
Coloquei um flip vertical no sprite 64x64 da tela de cima, e um horizontal no 32x32 da tela debaixo. Além disso, no OAM ID (segundo argumento) eu coloquei 0 e 1. 

Como esse é um projeto simples para mostrar a base, eu deixarei manual a seleção do ID do OAM, mas caso você vá fazer um jogo, é recomendável que você crie um código que pegue automaticamente os IDs disponíveis. Isso também vale para escolher uma paleta disponível.

Com isso já conseguimos renderizar sprites na tela. Mas vamos tentar mover o primeiro sprite
```C++
float sprite1PositionX = 100;
oamSet(&oamMain, 0, sprite1PositionX, 50, OBJPRIORITY_0, 0, SpriteSize_32x32, SpriteColorFormat_16Color, spriteAddressMain1, -1, false, false, false, false, false);
sprite1PositionX += 0.1f;
```
Se você rodar o código acima, perceberá que o sprite movido cria uma "linha" muito esquisita. Isso ocorre porque o código não está apagando os sprites já desenhados. Para resolver isso, precisamos limpar a tela a cada frame. Para isso, no inicio do loop infinito, precisamos chamar o método `oamClear`. 

Esse método também leva três parâmetros, o primeiro é o endereço de qual oam - `&oamMain` ou `&oamSub`- será limpado. O segundo é em qual OAM ID deve começar a limpeza da tela. E o terceiro é quantos OAM IDs devem ser limpados.  Caso queira limpar todos - como é o nosso caso - , o libnds nos permite colocar 0 nos dois últimos parâmetros, desse modo o código para limpar as duas telas fica:
```C++
oamClear(&oamSub, 0, 0);
oamClear(&oamMain, 0, 0);
```
É bom lembrar que esse método apenas apaga a tela, os sprites e paletas continuam carregados nos mesmos endereços que a gente tinha salvo para renderizar.

E com isso, finalizamos a parte de renderizar sprites na tela.