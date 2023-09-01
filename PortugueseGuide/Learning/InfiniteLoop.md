# Loop infinito
Os jogos usam loops infinitos para ser possível a executar ações a qualquer quadro do jogo. Ferramentas modernas abstraem o loop infinito. Geralmente elas possuem algum método que é executado uma vez por frame. Mas usando o libnds, nós precisamos implementá-lo por conta própria. E na verdade isso é extremamente simples. Só é necessário escrever:

~~~C++
while(true)
{
  /*
   *
   *Códigos do jogo
   *
  */
  swiWaitForVBlank();
}
~~~

O que está escrito dentro do while true é executado uma vez por frame. Para entender o que a linha swiWaitForVBlank() faz, é necessário saber brevemente como a tela do Nintendo DS (e as “televisões tubão”) funcionam. Imagine que essa é a tela do DS, e cada quadrado é um pixel: 

Basicamente, um pixel é desenhado por vez. O primeiro pixel é o que está mais no topo e esquerda da tela. 
Depois desse, ele desenha o pixel da direita até que acabe a linha. 


Quando a linha acaba, um mecanismo “anda”até o pixel mais a esquerda, mas da linha debaixo. 


O intervalo de tempo entre o desenho do último pixel de uma linha e o primeiro da seguinte é chamado de Horizontal Blank, ou HBlank. 

Quando todos os pixeis da tela forem desenhados, o mecanismo “anda” até o primeiro pixel da tela novamente. 


E o tempo entre a última linha e a primeira é chamado de Vertical Blank ou VBlank. Esse VBlank é o que determina quanto tempo há entre um quadro e outro.

Com base nisso, a linha swiWaitForVBlank(); faz o loop reiniciar assim que o VBlank começa. Isso é importante para evitar que a imagem seja alterada enquanto a tela está desenhando um quadro, o que evita glitches de imagem.
