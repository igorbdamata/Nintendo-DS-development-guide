# Controles

## Lendo inputs

O uso de inputs no libnds é bem parecido com o de ferramentas modernas e é bem simples. Basicamente, é necessário chamar a função `scanKeys()` para atualizar os inputs. Depois, dentro de um `if` comparar os métodos `keysDown()` (para quando acaba de apertar um botão), `keysHeld()` (para quando está segurando um botão) ou `keysUp()` (para quando solta um botão) com um botão. Os botões são:

`KEY_A`: Checa o botão A
`KEY_B`: Checa o botão B
`KEY_Y`: Checa o botão Y
`KEY_X:` Checa o botão X
`KEY_START`: Checa o botão start
`KEY_SELECT`: Checa o botão select
`KEY_LEFT`: Checa a setinha para esquerda
`KEY_UP`: Checa a setinha para cima
`KEY_DOWN`: Checa a setinha para baixo
`KEY_RIGHT`: Checa a setinha da direita
`KEY_L:` Checa o botão L
`KEY_R`: Checa o botão R
`KEY_TOUCH`: Checa se tocou a tela
`KEY_LID:` Checa se o DS está fechado

Para comparar é possível usar os operadores lógicos e binários. Os mais usados é o and binário (`&`).  Caso você esteja em dúvida da diferença entre `&` e `&&,` em prática, o `&&` faz a comparação ser `true` sempre que algum botão é apertado. Por exemplo:
`if(keysHeld() && KEY_A)//Retorna true se tiver apertando qualquer botão, mesmo que não seja A `
Já o operador binário (`&`) retorna true apenas se o botão comparado é apertado. Por exemplo:
`if(keysHeld() & KEY_A)//Retorna true se tiver apertando A`
Por conta disso, o `&` é mais comum.

Alguns exemplos de usos de inputs são:
````C++
scanKeys();
if(keysHeld() & KEY_LEFT)
{
	//Movimenta jogador para esquerda
}
else if(keysHeld() & KEY_RIGHT)
{
	//Movimenta jogador para direita
}

if(keysDown() & KEY_A)
{
	//Faz jogador pular
}

if(keysUp() & KEY_TOUCH)
{
	//Usa power up guardado na touch screen se jogador tiver tocado nele
}
````

## Usando a tela de toque
Usar a tela de toque também é bastante simples. Primeiramente, é necessário declarar uma variável do tipo `touchPosition`. Depois é necessário chamar a função `touchRead()` e passar o endereço da variável como parâmetro. Desse modo, o código fica:
```C++
touchPosition touch;
touchRead(&touch);
```

Depois, é possível acessar os valores `px` e `py` de `touch` para saber as coordenadas horizontais e verticais do toque na tela. Por exemplo:
```C++
touchPosition touch;
touchRead(&touch);

if(touch.px > SCREEN_WIDTH/2)
{
	//O toque está no lado direito da tela
}

if(touch.py > SCREEN_HEIGHT/2)
{
	//O toque está na metade de baixo da tela
}

```
O código acima checa em qual metade da tela de baixo o toque está.
