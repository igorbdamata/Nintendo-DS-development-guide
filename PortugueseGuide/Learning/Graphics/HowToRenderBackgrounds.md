# Como renderizar backgrounds
## O que são backgrounds
Basicamente, backgrounds são o fundo da tela. Eles podem ser usados tanto para colocar uma imagem de fundo na tela de título, quando para uma logo que não pode ser um sprite e até mesmo para fazer as fases do jogo.

## Passos para renderiza-los
1. Inicialização do DS
	1. Configurar modo de vídeo
	2. Configurar bancos de memória
	3. Inicializar backgrounds
3. Copiar dados na tela

## Como renderiza-los
### Inicializaçao
O primeiro passo para renderizar os backgrounds na tela é configurar o modo de vídeo. Assim como nos sprites, não irei fazer nada de muito complexo, por isso usarei o mode 5.
``` C++
videoSetMode(MODE_5_2D);
videoSetModeSub(MODE_5_2D);
```

Depois de configurar o modo de vídeo, nós precisamos configurar os bancos de memória. Como irei colocar dois BGs na tela de cima e um na tela debaixo, usarei os bancos B (para de cima) e C (para debaixo):
``` C++
vramSetBankB(VRAM_B_MAIN_BG);  
vramSetBankC(VRAM_C_SUB_BG);
```

Depois de configurar o modo de vídeo e bancos de memória, nós precisamos inicializar os backgrounds. Para isso, o libnds nos fornece o método `bgInit` e `bgInitSub` para as telas de cima e debaixo, respectivamente. Esses métodos retornam o ID do background (como `int`) e levam os mesmos 5 parâmetros:

|**Parâmetro**|**Tipo**|**O que é**|
|---|---|---|
|Layer|`int`|Qual das layers você deseja inicializar. Pode ser qualquer número de 0 a 3 (inclusivo)|
|Tipo do background|`BgType`|O tipo do background é determinado pelo enum `BgType`. Ele pode ser `BgType_Bmp16` ou `BgType_Bmp8` (para backgrounds que são bitmaps), `BgType_ExRotation` ou `BgType_Rotation` (para backgrounds rotacionaveis) e `BgType_Text4bpp` ou `BgType_Text8bpp` (para backgrounds de texto)| 
|Tamanho do background|`BgSize`|O BgSize indica as dimensões da imagem. Pode ser 256x256, 128x128, etc.|
|Map base do background|`int`|O map base é basicamente onde você quer que o background comece na memória. Esse é um tópico um pouco maior, por isso será explicado na seção abaixo|
|Tile base do background|`int`|Esse parâmetro não é usado por backgrounds bitmap, por isso você deve deixar 0.|

--------------------------------------------------------------------------
#### Map base
Imagine que a memória seja uma estante. Em cada prateleira cabe um background. O que definiria essa prateleira é o map base.

Porém, alguns backgrounds podem gastar mais que uma prateleira (map base). Por conta disso, escolher o map base não é simplesmente seguir os números 0, 1, 2.... 

Por isso, para escolher um map base, eu recomendo usar a ferramenta: https://mtheall.com/vram.html#. Ela permite que você configure o que fazer com cada uma das 4 layers, incluindo o tamanho e map base. Caso você configure algo errado (como tentar copiar dois backgrounds em um mesmo lugar), o site te avisará. Há um exemplo de como usar essa ferramenta abaixo.

----------------

Voltando para a inicialização do background, nós usaremos a ferramenta para facilitar a escolha do map base.

![](PrintMapBaseError.PNG)
Na imagem acima, eu tentei carregar dois background (nas layers 2 e 3), de tamanho 256x256 cada, no mesmo map base. Isso fez com que ocorresse um erro.
![](PrintMapBaseWorking.PNG)
Já na imagem acima, ao mudar o map base, o conflito se resolveu.

Para a tela debaixo, como vamos renderizar só um background, nós não precisaremos nos preocupar com o map base. Com base nisso, o código ficará assim:

```C++
int background3ID = bgInit(3, BgType_Bmp16, BgSize_B16_256x256, 0, 0);
int background2ID = bgInit(2, BgType_Bmp16, BgSize_B16_256x256, 8, 0);

int backgroundSub3ID = bgInitSub(3, BgType_Bmp16, BgSize_B16_256x256, 0, 0);
```

Eu guardei os IDs dos backgrounds porque eles podem ser úteis para operações nos backgrounds (como movimentá-los na tela).

Com isso, nós terminamos a parte de inicialização.

### Copiar dados na tela
Para copiar os dados na tela, nós primeiramente precisamos incluir os backgrounds gerados pelo GRIT:
```C++
#include "BackgroundPlaceHolder1.h"
#include "BackgroundPlaceHolder2.h"
```

Depois de inclui-los, nós precisamos copiá-los na tela. Para isso, usaremos o `dmaCopy`. Como você se lembra, o primeiro parâmetro é o que copiar, o segundo é onde copiar e o terceiro é o tamanho do que será copiado. 

Primeiramente, o que copiaremos será `BackgroundPlaceHolder1Bitmap` e `BackgroundPlaceHolder2Bitmap` (ambos estão nos arquivos incluidos). Esses valores são os bitmaps - ou seja, as imagens - do background. 

O segundo valor é onde copiar. Para saber esse valor, o libnds nos fornece as definições `BG_BMP_RAM` e `BG_BMP_RAM_SUB`. Ambas aceitam um parâmetro que indica o map base, por exemplo: `BG_BMP_RAM(2); //Acessa a ram de backgrounds com map base 2`.

E o terceiro valor é o tamanho do background. Esse valor está no arquivo gerado pelo GRIT, e no nosso caso é  `BackgroundPlaceHolder1BitmapLen` e `BackgroundPlaceHolder2BitmapLen`.

Com base nisso, nosso código de copiar o background na tela ficará assim:
```C++
dmaCopy(BackgroundPlaceHolder1Bitmap, BG_BMP_RAM(0), BackgroundPlaceHolder1BitmapLen);
dmaCopy(BackgroundPlaceHolder2Bitmap, BG_BMP_RAM(8), BackgroundPlaceHolder2BitmapLen);

dmaCopy(BackgroundPlaceHolder1Bitmap, BG_BMP_RAM_SUB(0), BackgroundPlaceHolder1BitmapLen);
```

Um detalhe importante, como nós não apagamos o background a cada frame, nós não devemos recopiar os mesmos backgrounds a cada quadro.

Com isso, nós finalizamos a renderização de backgrounds.
## Fontes:
https://libnds.devkitpro.org/background_8h.html