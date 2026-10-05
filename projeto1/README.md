# 🤖 Painel TV Senac · Placar Casa Aberta

![Modo Torneio na TV](docs/screenshots/Tela2_modo_torneio_aberta.png)

Na Casa Aberta Senac, robôs se enfrentam numa arena de sumô e quem assiste, muitas vezes em pé, a alguns metros de distância, precisa entender o jogo em segundos: quem está ganhando, quanto tempo falta, quem venceu. Este projeto nasceu para resolver isso: um placar eletrônico em que o juiz marca os pontos pelo celular e a TV do público mostra tudo na mesma hora.

## O desafio

Um placar improvisado costuma falhar justamente no momento em que o evento está mais cheio. Eu precisava de uma solução com quatro qualidades: **legível de longe**, **sem atraso** entre o juiz e a TV, **simples de operar sob pressão** e capaz de **organizar o fluxo de visitantes** que querem testar seus robôs. A TV também precisava ser confiável: se alguém recarregasse a página no meio de uma partida, o placar não podia sumir.

## A solução

O sistema tem duas telas que conversam em tempo real.

Na **TV**, o público vê o placar em letras enormes, com o cronômetro no centro, onde o olhar cai primeiro. O azul e o vermelho identificam cada robô, e a tipografia de impacto (Anton e Montserrat) mantém a leitura fácil mesmo de longe. Quando há um vencedor, a mensagem de vitória aparece no próprio painel central.

![TV em espera](docs/screenshots/Tela1_casa_aberta.png)

No **Painel de Controle**, pensado para o celular e protegido por senha, o juiz escolhe o modo, inicia a partida e marca os pontos com botões grandes, um por cor, cada um com seu "Corrigir" ao lado. A ação mais perigosa, "Zerar partida", é tracejada e vermelha para que ninguém a toque sem querer.

![Painel de controle](docs/screenshots/Tela2_controle_modos_casa_aberta.png)

Existem dois modos de jogo:

- **Torneio:** uma partida entre o Robô Azul e o Robô Vermelho, com cronômetro, pontuação, pausa, retomada e cancelamento.
- **Freestyle:** cada participante tenta um desempenho cronometrado, com tempo máximo configurável, e os melhores tempos entram num ranking. Uma fila de participantes permite mandar cada nome direto para o robô azul ou vermelho, e o operador escolhe o que a TV mostra: placar e ranking, só o placar ou só o ranking.

![Modo Freestyle](docs/screenshots/ModoFreestyle_ranking.png)

## Como funciona por dentro

O coração do projeto é um servidor **Node.js** com **Express** e **Socket.IO**. Ele guarda o estado da competição (placar, cronômetro, modo, fila) e é a única fonte de verdade: o painel envia comandos, o servidor atualiza o estado e transmite o resultado a todas as telas conectadas. Por isso a TV nunca depende do celular do juiz, e quem abre a tela depois já recebe o placar atual.

As configurações, o ranking do Freestyle e a fila de espera ficam gravados em `ranking.json`, então sobrevivem a um reinício do servidor. O front-end foi escrito em **HTML, CSS e JavaScript puros**, sem framework, para ter controle total do layout e das animações. Há também um som de início de partida, carregado antes para tocar sem atraso.

```
 Painel de Controle  ⇄  Servidor Node.js (Express + Socket.IO)  ⇄  TV
 (celular do juiz)        estado da competição + ranking.json      (público)
```

## Resultado

Pontos, cronômetro e vitória chegam à TV assim que o juiz toca no celular. A fila e o ranking Freestyle substituem papel e planilha, e os dados continuam lá depois de um reinício do servidor.

<!-- EDITE: conte aqui como foi no evento (partidas realizadas, participantes, reação do público e dos juízes). -->

## Para onde vai

- Exportar logs das partidas (CSV/JSON)
- Migrar do `ranking.json` para um banco de dados relacional
- Tirar a senha do código e usar variável de ambiente
- Criar chaveamento automático do torneio e uma tela de pódio para o Freestyle

## Rodando o projeto

Você precisa do [Node.js](https://nodejs.org/) (versão LTS).

```bash
git clone https://github.com/iluanmendes/painel_tv_senac.git
cd painel_tv_senac
npm install
node server.js
```

O servidor sobe na porta 3000. Abra a TV em `http://localhost:3000/` e o painel em `http://localhost:3000/controle.html`. Num evento, deixe o computador da TV e o celular do juiz na mesma rede e, no celular, use `http://IP-DO-COMPUTADOR:3000/controle.html`.

> 🔐 A senha do painel é a constante `SENHA_MESTRE` em `server.js`. Troque-a antes de usar em um evento.

Se o celular não abrir o painel, confira se os dois aparelhos estão na mesma rede e se o firewall libera a porta 3000. Se o som não tocar, o navegador provavelmente exige um clique na página antes de liberar o áudio.

<details>
<summary>Detalhes técnicos: eventos e estrutura de pastas</summary>

**Eventos Socket.IO:** `autenticar_controle`, `comando_controle`, `salvar_configuracoes` (painel → servidor); `carregar_dados`, `estado_competicao`, `erro_autenticacao`, `erro_comando` (servidor → telas).

**Ações de `comando_controle`:** `MUDAR_MODO`, `PREPARAR_TORNEIO`, `PONTUAR_TORNEIO`, `DESFAZER_PONTO_TORNEIO`, `PAUSAR_TORNEIO`, `RETOMAR_TORNEIO`, `CANCELAR_TORNEIO`, `PREPARAR_FREESTYLE`, `INICIAR_FREESTYLE`, `REGISTRAR_FREESTYLE`, `CANCELAR_FREESTYLE`, `DEFINIR_EXIBICAO_FREESTYLE`, `FILA_ADICIONAR`, `FILA_REMOVER`, `FILA_MOVER_TOPO`, `FILA_SUBIR`, `RESETAR_COMPETICAO`.

```
painel_tv_senac/
├── server.js            # servidor, regras da competição e eventos
├── ranking.json         # configurações, ranking e fila
└── public/
    ├── index.html       # tela da TV
    ├── controle.html    # painel de controle
    ├── tela.js · controle.js · sons.js · animacoes.js
    ├── style.css · animacoes.css
    ├── fonts/           # Montserrat e Anton
    └── sons/            # áudios
```
</details>

## Autor

**Luan Mendes**, instrutor de tecnologia e desenvolvedor. [GitHub](https://github.com/iluanmendes)
