# Como usar GRIT
## Como o grit funciona?
Como foi dito na seção de conceitos, o GRIT é uma sigla para GBA Raster Image Transmogrifier ou Game Raster Image Transmogrifier.  A função dele é basicamente converter formatos de arquivo modernos para arquivos binários que o Nintendo DS consiga ler. 

O GRIT funciona com linhas de comando, similar a códigos. Caso queira detalhes de cada comando, você pode checar a documentação disponível [aqui](https://www.coranac.com/man/grit/html/grit.htm). Mas caso queira apenas algo mais direto, pode checar essa seção.

Para usar o GRIT é necessário criar um arquivo .grit com o mesmo nome do arquivo que você deseja alterar. E caso queira, você pode adicionar comentários no seu arquivo GRIT usando hashtags (#).

## Comandos gerais importantes
### Comando -s
Significa "symbol". Ele define o nome do arquivo que será gerado. Para usá-lo basta colocar `-s Nome`.

### Comandos -W
Define a quantidade de informação que o GRIT coloca no console. Os possíveis usos são:

|**Comando**|**O que mostra**|
|---|---|
|W1 ou We|Mostra apenas mensagens de erro|
|W2 ou Ws|Mostra mensagens de erro e warning|
|W3 ou Ww|Mostra mensagens de erro, warning e status|

### Bit depth
O comando `-gBX` é usado para definir a densidade de bits por pixels, ou seja, quantos bits são necessários para representar cada cor de pixel. O número X pode ser 1, 2, 4, 8 ou 16. Para sprites com 16 cores, usa-se X=4, ou seja `-gB4`. Caso estejamos usando uma imagem sem paleta, devemos colocar X=16, ou seja `-gB16`.

## Comandos importantes para gerar sprites
### Comando -p
- Serve para gerar e incluir a paleta no arquivo final
- Ele já é chamado por padrão. Por isso caso não queira a paleta no arquivo final basta colocar o comando `-p!`
- Por padrão a primeira cor da paleta será transparente. 

### Comando -gt
O DS lê as imagens em tiles, que são pedaços de 8x8 pixels da imagem total. E o comando `-gt` serve justamente para gerar esses tiles.

### Comandos de metamapping
É comum usarmos sprite sheets para animações. Para isso, o GRIT requer um comando específico, que são os comandos de metamapping. Os principais são: 

|**Comando**|**Função**|
|---|---|
|MhX|É o metatile da altura (height). O X representa a altura de um frame do sprite sheet dividido por 8. Então se o frame do sprite sheet for 32x32, por exemplo, o comando deverá ser Mh4, pois 32/8=4|
|MwX|É o metatile da largura (width). O X representa a largura de um frame do sprite sheet dividido por 8. Então se o frame do sprite sheet  for 32x32, por exemplo, o comando deverá ser Mw4, pois 32/8=4|
Ps: Os comandos precisam ser usados juntos


## Comandos importantes para gerar backgrounds
###  Comando  -gTRRGGBB
O comando `-gT` serve para definir a cor de transparência no background. O valor RR define o valor vermelho, o GG define o verde e o BB define o azul (ps: todos os três valores devem estar em hexadecimal).
Um exemplo de uso é `-gT000000`, que define a cor `#000000` como transparente

### Comando -gb
O comando `-gb` serve para criar um bitmap do background. Como backgrounds são imagens maiores, é melhor trabalhar com eles como bitmaps.

## Conclusão
Com base nisso, seguem alguns modelos de arquivo GRIT
- Para sprites normais:
```GRIT
# Define nome do arquivo que será gerado
-s NomeDoSprite

# Configura o nível de detalhes para erros e warnings
-W2

# Inclui paleta no arquivo que será gerado
-p

# Cria os tiles
-gt

# Configura a densidade de bits por pixel para 4
-gB4
```
- Para sprite sheets
```GRIT
# Define nome do arquivo que será gerado
-s NomeDoSprite

# Configura o nível de detalhes para erros e warnings
-W2

# Inclui paleta no arquivo que será gerado
-p

# Cria os tiles
-gt

# Configura o metamapping (os valores são 8 porque cada frame do sprite sheet é 64x64. E 64 pixeis são divididos em 8 tiles (já que 64/8=8))
-Mh8
-Mw8

# Configura a densidade de bits por pixel para 4
-gB4
```
- Para backgrounds
```GRIT
# Define nome do arquivo que será gerado
-s NomeDoBackground

# Configura o nível de detalhes para erros e warnings
-W2

# Define #000000 (em hexadecimal) como cor transparente
-gT000000

# Cria o bitmap do background
-gb

# Configura a densidade de bits por pixel para 16, que é o usado para backgrounds
-gB16

```

## Fontes
https://www.coranac.com/man/grit/html/grit.htm