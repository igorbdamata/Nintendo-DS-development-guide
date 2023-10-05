# Como configurar o modo de vídeo

O modo de vídeo no Nintendo DS define o uso das quatro layers de background. Existem 7 modos, nomeados como Mode 0, 1, 2, 3, 4, 5 e 6. Cada um deles faz um uso diferente das layers. 

Para escolher o modo, você precisa saber como vai usar cada uma das quatro layers. Caso você queira algo mais simples, da para usar o Mode 5, que possui duas layers estáticas e duas "affine estendidas". Se você quiser algo mais específico, recomendo checar o texto: https://www.copetti.org/writings/consoles/nintendo-ds/ na parte de modos de background.

Para configurar o modo de vídeo no código é necessário usar os métodos `videoSetMode` e `videoSetModeSub`, para as telas de cima e de baixo, respectivamente. Ao chamar esses métodos, é necessário passar o modo desejado como parâmetro. O nome desse parâmetro é `MODE_X_2D`, onde `X` é o número do modo (indo de 0-6, inclusivo)

## Fontes:
https://www.copetti.org/writings/consoles/nintendo-ds/