# DeathtrapDungeon

* [Protótipo do Jogo](#1)
* [Demonstração do Projeto](#2)
* [Recursos Gráficos](#3)
* [Implementação do Código](#4)
* [Observações Importantes](#5)
* [Discussão Técnica](#6)
* [Referências](#7)

----------

<h1 id="1">Protótipo do Jogo</h1>

**DeathtrapDungeon** é um jogo de aventura em masmorras no estilo **2D-Roguelike**. Empunhe sua lâmina, extermine monstros e sobreviva na perigosa masmorra. Mas cuidado, os inimigos também não são fáceis de lidar — manter-se alerta, tomar decisões com determinação e ser engenhoso são as chaves para a vitória.


<h1 id="2">Demonstração do Projeto</h1>

<img width="696" height="516" alt="68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313638383730342f3230313931302f313638383730342d32303139313031343039323533303531302d3337373538303830352e676966" src="https://github.com/user-attachments/assets/179a9a48-e052-4bd7-b864-dda1119b91c4" />
<img width="924" height="456" alt="68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313638383730342f3230313931302f313638383730342d32303139313032323232303133313533322d3738363036323239362e676966" src="https://github.com/user-attachments/assets/1c92d107-d127-4896-8960-0f3d829ecb87" />
<img width="696" height="516" alt="68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313638383730342f3230313931302f313638383730342d32303139313031343039323730383636392d313536373735383133342e676966" src="https://github.com/user-attachments/assets/42e0d0ad-5036-41b4-acd0-69145a917102" />



--------------------

<h1 id="4">Implementação do Código</h1>

**Sistema de Controle Geral:**
- GameManager.cs: padrão Singleton, gerencia de forma unificada as instâncias das diversas classes

**Classes Base:**
- Classe de vida: Fighter.cs (sistema de pontos de vida e dano)
- Classe de movimento: Mover.cs (sistema de movimentação)
- Classe de interação: Collectable.cs (detecta se o colisor é o Player)

**Player**:
- Player.cs: sistema de fúria (Rage); troca de skin, renascimento após a morte, etc.
- Weapon: sistema de ataque de armas, sistema de habilidade de fúria (Rage)

**Enemy**:
- EnemyHitBox.cs: transmite dano ao jogador
- Enemy.cs: classe da maioria dos inimigos, inclui pontos de experiência, sistema de perseguição e ataque, sistema de morte e reaparecimento, etc.
- Enemy_Chest.cs: monstro-baú (mimic)
- Trap.cs: armadilha
- Boss0.cs: chefe final

**GUI**:
- UIManager.cs: gerenciamento de UI
- CharacterMenu.cs: sistema de barra de menu
- CharacterHUD.cs: exibição de vida, experiência e fúria do jogador, tela de morte
- SCUI.cs: tela de carregamento para troca assíncrona de cena
- FloatingTextManager/FloatingText.cs: sistema de exibição de texto flutuante

**Cena**:
- CameraFollow.cs: sistema de câmera que segue o jogador
- SceneTranslate.csa: sistema de carregamento assíncrono de cena
- Portal.cs: portal de teleporte entre cenas diferentes
- Portal_Door.cs: portal de teleporte dentro da cena atual

**Objetos Interativos**:
- NPCTextPerson.cs: interação com NPC
- Chest.cs: baú
- Crate.cs: objeto destrutível
- HealingFountain: fonte de cura
- Door.cs: porta (abrir/fechar)

--------------------

<h1 id="5">Discussão Técnica</h1>

- **Sobre a animação de ataque com o balanço da arma**:
	1. Recomenda-se que a duração da animação de Swing (balanço) da arma seja maior que 0,3s; caso contrário, mesmo acertando o inimigo a cada vez, o intervalo curto não será suficiente para empurrá-lo eficazmente, deixando o jogador vulnerável a ataques.
	2. Recomenda-se que o golpe da arma atinja a posição horizontal na primeira metade do tempo da animação, pois é nesse momento que ocorre o dano por colisão da arma; com a arma na horizontal, a área do colisor é maior, o que ajuda a manter distância do inimigo e ainda assim causar dano.
	3. É melhor simular o movimento de corte como na realidade; eu o dividi nas seguintes partes:
		- p1: a arma é levantada e recuada; velocidade moderada nessa etapa, com o collider desativado
		- p2: movimento principal de corte, de 90° a 0°. Na primeira metade dessa etapa a velocidade é rápida, e na segunda metade é extremamente rápida; collider ativado
		- p3: de 0° a -20°, movimento final do corte. Aqui o importante não é a velocidade, mas manter o número de frames (tempo)
		- p4: a arma é recolhida rapidamente; collider desativado

- **Desenho do Tilemap**:
	1. É melhor definir as informações de pixel do desenho geral e o estilo da cena já ao escolher os recursos de arte; se elementos diferentes forem adicionados depois com estilos distintos, isso ficará destoante.
	2. O tipo de Tile também deve ser definido com antecedência, usando tiles normais + objetos com animação, ou RuleTile para implementar as diversas funcionalidades.
	3. A ordem de renderização dos tiles é definida pela ordem das camadas do Tilemap (por exemplo, a camada superior da parede "wall" e a camada inferior "ScreenWall"; a camada superior pode ser sobreposta pelo player, e a inferior pode sobrepor o player).
	4. A ordem de renderização dos objetos que não são tiles é definida pelo SortingLayer e OrderLayer do SpriteRenderer; configure adequadamente as relações de sobreposição.
- **Problemas de Colisão 2D**:
	- É melhor que todos os objetos estejam no mesmo plano z=0; caso contrário, algumas detecções de colisão podem apresentar problemas.
- **Otimização de Draw Call**:
	- Mapa Tilemap: consome mais de 5 Draw Calls, principalmente na camada de paredes. Como este projeto utiliza dois tipos de recursos de arte e contém tanto Tiles normais quanto RuleTiles, isso gera mais Draw Calls. Portanto, ao desenhar Tilemaps no futuro, deve-se atentar à uniformidade dos recursos de arte e dos tipos de Tile; além disso, objetos estáticos devem ser marcados como static.
	- Menu de equipamento: consome mais de 9 Draw Calls, devido à grande quantidade de informações no menu (troca de Sprites de Weapon e Player, informações numéricas do jogo, etc.). Os diversos componentes com Sprites diferentes, botões com funções variadas e fontes de texto distintas geram muitos Draw Calls. Além de unificar as fontes e reduzir o redimensionamento dos Painéis, ainda não há uma solução melhor.
	- Painel de valores HUD: consome mais de 4 Draw Calls; sua função é exibir as informações numéricas no canto superior esquerdo da tela, e também possui componentes com Sprites diferentes; ainda não há uma solução melhor.
	- Objetos com Animator: para objetos com componente Animator, é possível definir o CullingMode como "Cull Completely", ou seja, parar de funcionar quando fora do campo de visão, o que reduz efetivamente os Draw Calls.
- **Otimização de GC (Coleta de Lixo)**:
	- Use o Profiler para analisar as alocações de GC (GC Alloc) nos scripts, identifique os scripts correspondentes e otimize o código (reduza a criação de novos objetos dentro de loops, reduza o uso de foreach, etc.); além disso, oculte objetos da cena que não estejam em uso no momento.
- **Otimização de Memória**:
	- Por se tratar de um jogo 2D, neste projeto até o momento apenas a música de fundo (BGM) foi otimizada, alterando seu LoadType para reprodução em Streaming, o que reduziu significativamente o uso de memória durante a execução.
	- Outros métodos de otimização: empacotamento com AssetBundle, redução do tamanho das texturas, uso de MipMap, etc.

--------
<img width="696" height="516" alt="68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313638383730342f3230313931302f313638383730342d32303139313031343039323434393931382d313836333032353232332e676966" src="https://github.com/user-attachments/assets/7f926d1f-c9a9-4a1a-b2da-5d28e7b093de" />

<h1 id="6">*Observações Importantes</h1>

Os mecanismos centrais do jogo, como combate, troca de cenas e interação com objetos, **já estão finalizados**, e o Tilemap está completo (camada de solo, camadas superior e inferior de paredes, camada de objetos de superfície, camada de colisão). Além dos Tiles estáticos básicos, também há alguns AnimatedTiles (fonte de água, lava, etc.) e o pincel ChestBrush, que facilita o posicionamento de baús, entre outras funcionalidades. Caso deseje **personalizar suas próprias fases**, é totalmente possível, com base neste sistema de scripts, redesenhar o Tilemap, ajustar os valores do jogo, posicionar objetos de cena, editar falas de NPCs, entre outros.

