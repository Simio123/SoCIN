# 🖥️ ParIS NoC Router - Logisim Implementation

![Status](https://img.shields.io/badge/Status-Concluído-success)
![Hardware](https://img.shields.io/badge/Hardware-Logisim-red)
![Architecture](https://img.shields.io/badge/Architecture-NoC%20(Network--on--Chip)-blue)

Implementação completa e funcional do Roteador **ParIS** (Packet Routing in SoC) em nível de portas lógicas e diagramas de bloco estruturais utilizando o simulador **Logisim**. Este projeto transpõe a arquitetura acadêmica original escrita em VHDL para um esquemático visual e interativo.

## ⚙️ Características da Arquitetura

O ParIS é um roteador projetado para Redes em Chip (Networks-on-Chip), sendo responsável por gerenciar o tráfego de dados entre diferentes núcleos de processamento (Cores) em um chip multicore.

* **Topologia:** 5 Portas de Comunicação (Local, Norte, Leste, Sul, Oeste).
* **Roteamento:** Algoritmo de Roteamento XY (Determinístico).
* **Arbitragem:** Juiz Round-Robin (`arb_rr`), garantindo justiça sem *starvation* no cruzamento de dados.
* **Bufferização:** FIFOs independentes em cada porta de entrada (`Xin`).
* **Largura de Banda:** Barramentos de dados de 10 bits.
* **Controle de Fluxo:** Sinais de `valid` / `return` (handshake) e permissões internas `req` / `gnt`.

---

## 📂 Estrutura do Projeto

O arquivo `SoCIN.circ` contém toda a hierarquia do chip. Os principais módulos incluem:
* **`ParIS` (Placa-Mãe):** O circuito topo de hierarquia, contendo as 5 portas `Xin` (Recepção) e `Xout` (Transmissão) interligadas pelo barramento Crossbar.
* **`Xin` / `Xout`:** Módulos de entrada e saída contendo FIFOs e lógica de controle.
* **`ic` / `oc`:** Controladores de Entrada e Saída (Máquinas de Estado).
* **`arb_rr`:** O cérebro da arbitragem (Round-Robin).
* **`fifo`:** Memória circular para armazenamento temporário de pacotes.

---

## 📦 Formato do Pacote (Data Packet)

Os dados trafegam em um barramento de **10 bits**. A estrutura básica de um pacote de injeção (Header) utilizada nos testes é:

| Bit 9 | Bit 8 (`bop`) | Bits [7:3] | Bits [2:0] (Destino) |
| :---: | :---: | :---: | :---: |
| - | **1** (Início de Pacote) | Payload/Zero | **Endereço (XY)** |

**Mapa de Destinos (Exemplo):**
* `000` (0) -> Local
* `001` (1) -> Norte
* `010` (2) -> Leste
* `011` (3) -> Sul
* `100` (4) -> Oeste

*Exemplo prático:* Para enviar um pacote para o Sul, o dado de entrada deve ser `01 0000 0011` (em binário).

---

## 🚀 Como Simular (Guia de Testes)

Para rodar a simulação no Logisim, é necessário realizar o "Boot" (limpeza das FIFOs) e depois injetar o tráfego.

### 1. Reset Síncrono (Lavagem de Memória)
Sempre que abrir o projeto ou iniciar um teste novo, limpe as FIFOs:
1. Altere o pino global **`RST` para `1`**.
2. Ligue o Clock automático (Menu: `Simular -> Clock Ativado` ou `Ctrl + K`).
3. Aguarde 2 segundos e retorne o **`RST` para `0`**.
*(Isso limpará os estados de erro `U` e fios vermelhos).*

### 2. Teste Simples (Ponto a Ponto)
*Exemplo: Enviando pacote da porta Local para a porta Leste.*
1. Certifique-se de que o Clock automático está rodando e o `RST` é `0`.
2. No pino `Local_in_data`, digite o valor: **`01 0000 0010`** (Destino Leste).
3. Mude o pino **`Local_in_val`** para **`1`** (A porta receberá o dado).
4. Volte o **`Local_in_val`** para **`0`** rapidamente (para enviar apenas 1 pacote).
5. **Resultado:** O pacote cruzará a placa e aparecerá no pino `East_out_data`, acompanhado da luz verde no `East_out_val`.

### 3. Teste de Tráfego Cruzado (Paralelismo)
Injete pacotes simultâneos em caminhos que não se cruzam para ver o barramento central trabalhando em paralelo:
* Coloque um pacote para o Norte no `Local_in_data`.
* Coloque um pacote para o Sul no `East_in_data`.
* Acione ambos os `in_val` simultaneamente. Ambos sairão em suas respectivas portas ao mesmo tempo.

### 4. Teste de Estresse (Arbitragem Round-Robin)
Force um engarrafamento para testar a justiça do juiz:
* Injete um pacote para o Oeste no `Local_in_data` e outro para o Oeste no `South_in_data`.
* Ative os dois `in_val` ao mesmo tempo.
* **Resultado:** O roteador processará ambos os pedidos (`req`), mas concederá permissão (`gnt`) de forma sequencial (um após o outro), sem colisões.

---

## 🛠️ Requisitos
* [Logisim-evolution](https://github.com/logisim-evolution/logisim-evolution) (Recomendado) ou Logisim original.

## 🎓 Agradecimentos / Notas
Este projeto foi desenvolvido como um exercício profundo de Arquitetura de Computadores e Design de Hardware (RTL), convertendo lógicas descritivas complexas em um esquemático funcional e simulável visualmente.
