# 14 - Menu

# Criando Menu Inicial em Raylib (C)

```c
#include <stdio.h>
#include "raylib/include/raylib.h"

// Função Botao corrigida para usar cores nativas sem erro
void Botao(Rectangle area, const char* texto, Color cor){
    DrawRectangleRec(area, cor); // Constrói o fundo do botão
    DrawRectangleLinesEx(area, 4, BLACK); // Borda preta simples e funcional
    DrawText(texto, area.x + 25, area.y + 20, 28, WHITE); // Texto branco para contraste
};

typedef enum GameState{
    MENU,
    PAUSA,
    INSTRUCT,
    JOGO,
    GAME_OVER
} GameState;

int main(void)
{
    InitWindow(1000, 1000, "Apenas um Retangulo");

    SetTargetFPS(60); 

    GameState estado = MENU; 
    Vector2 playerPos = {500,500};
    float vel = 10.0f;

    // Definição das áreas dos botões do menu
    Rectangle botaoJogar = { 350, 420, 300, 70 };
    Rectangle botaoInstrucoes = { 350, 520, 300, 70 };

    while (!WindowShouldClose())
    {
        BeginDrawing();
            // Fundo cinza escuro nativo - elegante e seguro contra erros
            ClearBackground(DARKGRAY);
            
            if(estado == MENU){
                // Painel central usando MAROON (um tom de vinho/vermelho escuro)
                DrawRectangle(200, 200, 600, 600, MAROON);
                DrawRectangleLinesEx((Rectangle){200, 200, 600, 600}, 4, WHITE);
                
                // Título alternando entre duas cores nativas seguras (GOLD e ORANGE)
                Color corTitulo = ((int)(GetTime() * 2) % 2 == 0) ? GOLD : ORANGE;
                DrawText("RETÂNGULO ADVENTURE", 240, 260, 45, corTitulo);
                
                // Renderizando os botões com cores padrões da Raylib
                Botao(botaoJogar, "1. JOGAR", LIME);      // Verde limão padrão
                Botao(botaoInstrucoes, "2. REGRAS", BLUE); // Azul padrão
                
                DrawText("Dica: Use o mouse para clicar ou o teclado!", 270, 720, 20, LIGHTGRAY);

                // Cliques e Teclado
                if((IsMouseButtonPressed(MOUSE_BUTTON_LEFT) && CheckCollisionPointRec(GetMousePosition(), botaoJogar)) || IsKeyPressed(KEY_ENTER)){
                    estado = JOGO;
                }
                else if((IsMouseButtonPressed(MOUSE_BUTTON_LEFT) && CheckCollisionPointRec(GetMousePosition(), botaoInstrucoes)) || IsKeyPressed(KEY_I)){
                    estado = INSTRUCT;
                }
            }
            else if(estado == PAUSA){
                DrawRectangle(200, 200, 600, 600, MAROON);
                DrawRectangleLinesEx((Rectangle){200, 200, 600, 600}, 4, ORANGE);
                
                DrawText("PAUSA", 435, 260, 50, ORANGE);
                DrawText("PRESSIONE ENTER PARA VOLTAR", 310, 500, 24, WHITE);
                if(IsKeyPressed(KEY_ENTER)){
                    estado = JOGO;
                }
            }
            else if(estado == JOGO){
                if(IsKeyPressed(KEY_SPACE)){
                    estado = PAUSA;
                }
                if(IsKeyPressed(KEY_M)){
                    estado = MENU;
                }

                // O jogador agora é um quadrado SKYBLUE (Azul Claro) com borda branca
                DrawRectangleV(playerPos, (Vector2){ 40, 40 }, SKYBLUE);
                DrawRectangleLinesEx((Rectangle){playerPos.x, playerPos.y, 40, 40}, 2, WHITE);
                
                if (IsKeyDown(KEY_RIGHT)) playerPos.x += vel;
                if (IsKeyDown(KEY_LEFT))  playerPos.x -= vel;
                if (IsKeyDown(KEY_UP))    playerPos.y -= vel;
                if (IsKeyDown(KEY_DOWN))  playerPos.y += vel;
            }
            else if(estado == INSTRUCT){
                DrawRectangle(200, 200, 600, 600, MAROON);
                DrawRectangleLinesEx((Rectangle){200, 200, 600, 600}, 4, SKYBLUE);
                
                DrawText("INSTRUCOES", 360, 260, 50, SKYBLUE);
                DrawText("Movimente o personagem com as teclas: SETAS", 245, 480, 22, WHITE);
                DrawText("Pressione M para voltar ao MENU", 320, 540, 22, LIGHTGRAY);
                
                if(IsKeyPressed(KEY_M)){
                    estado = MENU;
                }
                if(IsKeyPressed(KEY_ENTER)){
                    estado = JOGO;
                }
            }
            else if(estado == GAME_OVER){
                DrawRectangle(200, 200, 600, 600, MAROON);
                DrawRectangleLinesEx((Rectangle){200, 200, 600, 600}, 4, RED);
                
                DrawText("GAME OVER", 370, 260, 50, RED);
                DrawText("CLIQUE G PARA REVIVER", 360, 500, 24, WHITE);
                if(IsKeyPressed(KEY_G)){
                    playerPos = (Vector2){500,500};
                    estado = JOGO;
                }
            }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

---

# Objetivo da Aula

Nesta aula vamos aprender a criar um **menu inicial** para o jogo.

Até agora, nossos programas começavam direto na tela do jogo.

Agora vamos criar uma estrutura mais profissional com:

- tela de menu
- tela de instruções
- tela do jogo
- tela de game over
- troca entre telas
- controle de estados

---

# O que é um menu em jogos?

Um menu é uma tela inicial onde o jogador pode escolher o que fazer.

Exemplos:

```text
Iniciar Jogo
Instruções
Opções
Sair
```

Em jogos reais, quase sempre existe alguma tela antes do jogo começar.

---

# Conceito principal da aula

O conceito mais importante desta aula é:

```text
estado do jogo
```

Um jogo pode estar em vários estados:

| Estado | Significado |
|---|---|
| MENU | jogador está no menu |
| GAME | jogo está rodando |
| INSTRUCTIONS | tela de instruções |
| GAME_OVER | jogo acabou |

---

# Criando os estados do jogo

```c
typedef enum GameState
{
    MENU,
    GAME,
    INSTRUCTIONS,
    GAME_OVER
} GameState;
```

---

# O que é enum?

`enum` é uma forma de criar um conjunto de valores com nomes.

Em vez de usar números como:

```c
0
1
2
3
```

usamos nomes mais claros:

```c
MENU
GAME
INSTRUCTIONS
GAME_OVER
```

Isso deixa o código muito mais fácil de entender.

---

# Criando o estado inicial

```c
GameState estadoAtual = MENU;
```

Essa linha significa:

```text
o jogo começa no menu
```

Ou seja, quando o programa abre, a primeira tela mostrada será o menu inicial.

---

# Por que usar estado?

Sem estado, o jogo não sabe em que tela está.

Com estado, podemos dizer:

```text
se estiver no MENU, desenhe o menu
se estiver no GAME, rode o jogo
se estiver no GAME_OVER, mostre fim de jogo
```

---

# Criando o jogador

```c
Vector2 jogador = { 400, 225 };
float velocidade = 4.0f;
```

Aqui criamos:

| Variável | Função |
|---|---|
| jogador | posição do jogador |
| velocidade | velocidade de movimento |

---

# Loop principal

```c
while (!WindowShouldClose())
```

Mesmo com menu, o jogo continua usando o mesmo **game loop**.

Dentro dele fazemos:

```text
1. update
2. draw
```

Ou seja:

```text
primeiro atualiza a lógica
depois desenha a tela
```

---

# Parte 1 — Update

A parte de update decide:

- o que o jogador apertou
- qual tela deve aparecer
- se o jogador se move
- se o jogo muda de estado

---

# Estado MENU

```c
if (estadoAtual == MENU)
```

Esse bloco só roda quando estamos no menu.

---

## Entrar no jogo

```c
if (IsKeyPressed(KEY_ENTER))
{
    estadoAtual = GAME;
}
```

Se o jogador apertar ENTER:

```text
o estado muda de MENU para GAME
```

Ou seja, o jogo começa.

---

## Ir para instruções

```c
if (IsKeyPressed(KEY_I))
{
    estadoAtual = INSTRUCTIONS;
}
```

Se apertar `I`:

```text
vai para tela de instruções
```

---

## Sair do jogo

```c
if (IsKeyPressed(KEY_ESCAPE))
{
    break;
}
```

O `break` interrompe o loop principal.

Quando o loop termina, o programa chega em:

```c
CloseWindow();
```

e fecha corretamente.

---

# Estado INSTRUCTIONS

```c
else if (estadoAtual == INSTRUCTIONS)
```

Essa parte roda apenas na tela de instruções.

---

## Voltar ao menu

```c
if (IsKeyPressed(KEY_BACKSPACE))
{
    estadoAtual = MENU;
}
```

Se o jogador apertar BACKSPACE:

```text
volta para o menu inicial
```

---

# Estado GAME

```c
else if (estadoAtual == GAME)
```

Aqui fica a lógica principal do jogo.

---

## Movimento do jogador

```c
if (IsKeyDown(KEY_RIGHT)) jogador.x += velocidade;
if (IsKeyDown(KEY_LEFT))  jogador.x -= velocidade;
if (IsKeyDown(KEY_UP))    jogador.y -= velocidade;
if (IsKeyDown(KEY_DOWN))  jogador.y += velocidade;
```

Essas linhas movem o jogador usando as setas.

---

# Diferença entre IsKeyPressed e IsKeyDown

| Função | Quando usar |
|---|---|
| IsKeyPressed | quando quer detectar um clique único |
| IsKeyDown | quando quer detectar tecla segurada |

Exemplo:

```c
IsKeyPressed(KEY_ENTER)
```

serve para menu.

```c
IsKeyDown(KEY_RIGHT)
```

serve para movimento contínuo.

---

## Simular Game Over

```c
if (IsKeyPressed(KEY_G))
{
    estadoAtual = GAME_OVER;
}
```

Nesta aula usamos a tecla `G` para simular o fim do jogo.

Mais tarde, o `GAME_OVER` pode acontecer quando:

- vida acabar
- inimigo encostar no jogador
- tempo terminar
- jogador perder

---

# Estado GAME_OVER

```c
else if (estadoAtual == GAME_OVER)
```

Essa parte roda quando o jogo acabou.

---

## Reiniciar jogo

```c
if (IsKeyPressed(KEY_R))
{
    jogador = (Vector2){ 400, 225 };
    estadoAtual = GAME;
}
```

Se apertar `R`:

- jogador volta para posição inicial
- jogo recomeça

---

## Voltar ao menu

```c
if (IsKeyPressed(KEY_M))
{
    jogador = (Vector2){ 400, 225 };
    estadoAtual = MENU;
}
```

Se apertar `M`:

- jogador volta para posição inicial
- tela volta ao menu

---

# Parte 2 — Draw

Depois de atualizar a lógica, desenhamos a tela.

```c
BeginDrawing();
ClearBackground(RAYWHITE);
```

Essas funções:

- começam o desenho
- limpam o fundo

---

# Desenhando o MENU

```c
if (estadoAtual == MENU)
```

Se o estado for `MENU`, desenhamos:

```text
título
opção de jogar
opção de instruções
opção de sair
```

---

# Desenhando as INSTRUÇÕES

```c
else if (estadoAtual == INSTRUCTIONS)
```

Se o estado for `INSTRUCTIONS`, desenhamos:

- como mover
- como simular game over
- como voltar

---

# Desenhando o GAME

```c
else if (estadoAtual == GAME)
```

Se o estado for `GAME`, desenhamos:

- informações do jogo
- jogador azul

---

# Desenhando GAME OVER

```c
else if (estadoAtual == GAME_OVER)
```

Se o estado for `GAME_OVER`, desenhamos:

- mensagem GAME OVER
- instrução para reiniciar
- instrução para voltar ao menu

---

# Fluxo visual do programa

```text
MENU
 ├── ENTER → GAME
 ├── I     → INSTRUCTIONS
 └── ESC   → SAIR

INSTRUCTIONS
 └── BACKSPACE → MENU

GAME
 ├── SETAS → move jogador
 └── G     → GAME_OVER

GAME_OVER
 ├── R → reinicia jogo
 └── M → volta ao menu
```

---

# Por que isso é importante?

Esse padrão é usado em praticamente todo jogo.

Mesmo jogos grandes usam alguma forma de:

```text
estado atual do jogo
```

Exemplos:

```text
MENU
LOADING
CUTSCENE
PLAYING
PAUSED
GAME_OVER
CREDITS
```

---

# Conceitos aprendidos

| Conceito | Explicação |
|---|---|
| Menu | tela inicial do jogo |
| Estado | situação atual do jogo |
| enum | conjunto de valores nomeados |
| GameState | tipo criado para estados |
| Transição | troca de uma tela para outra |
| IsKeyPressed | tecla pressionada uma vez |
| IsKeyDown | tecla segurada |

---

# Resultado esperado

Ao executar o programa:

1. aparece o menu inicial
2. ENTER inicia o jogo
3. I abre instruções
4. BACKSPACE volta ao menu
5. G mostra game over
6. R reinicia
7. M volta ao menu
8. ESC sai do programa

---

# Atividade da Aula

## Exercício 1 — Menu Simples

Crie uma tela de menu com:

- título do jogo
- opção para iniciar
- opção para sair

---

## Exercício 2 — Tela de Instruções

Adicione uma tela explicando:

- controles
- objetivo do jogo
- como voltar ao menu

---

## Exercício 3 — Game Over

Crie uma tela de fim de jogo com:

- mensagem GAME OVER
- opção para reiniciar
- opção para voltar ao menu

---

# Desafio Extra

Adicione uma tela de pausa.

Dica:

```c
PAUSE
```

pode ser mais um estado dentro do `enum`.

Exemplo:

```c
typedef enum GameState
{
    MENU,
    GAME,
    PAUSE,
    INSTRUCTIONS,
    GAME_OVER
} GameState;
```

---

# Super Desafio

Crie um menu com seleção usando setas:

```text
> Jogar
  Instruções
  Sair
```

Quando o jogador apertar para baixo, a seleção muda.

---

# Próximo passo

Na próxima aula podemos evoluir para:

```text
15 - HUD
```

onde iremos criar:

- pontuação na tela
- vida
- barra de energia
- informações do jogador
- interface visual do jogo
