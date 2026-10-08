# Diagrama do Ciclo de Vida do Processo e Chamadas de Sistema

Abaixo encontra-se o fluxo de execução de uma chamada de sistema (*System Call*), com a respetiva transição de modos e estados do processo:

![Diagrama do Ciclo de Vida e E/S](Text%20Container20%Ecosystem-2026-10-08-191035.png)
---

### Descrição dos Módulos e Etapas do Diagrama

#### 1. Resumo do Ciclo de Vida e E/S
O processo inicia em **Modo Usuário**, solicita E/S via **Trap**, passa ao **Modo Kernel** e fica **Bloqueado** aguardando o disco. O Escalonador (*Round-Robin*) alterna a CPU para evitar travamentos.

#### 2. Módulos de Suporte
* **Modo Usuário (Aplicações):**
  1. Chamada `read()` na biblioteca
  2. Parâmetros nos registradores
  3. Execução da instrução *Trap*
* **Modo Kernel (Sistema Operacional):**
  4. Tratador de *Syscall*
  5. Chamada ao Driver do Disco
* **Subsistema de E/S & Escalonador:**
  6. Leitura física no hardware
  7. Interrupção ao concluir
  8. Algoritmo *Round-Robin*

---

#### 3. Sequência do Fluxo de Execução
1. **Início da Requisição | Processo Executando**
2. **Disparo da Syscall** (Transição: Usuário $\rightarrow$ Kernel)
3. **Execução de E/S** (Estado: Bloqueado / Esperando)
4. **Término da Leitura** (Estado: Pronto / Fila)
5. **Seleção do Escalonador** (Transição: Kernel $\rightarrow$ Usuário)
6. **Retorno dos Dados | Conclusão do Processamento**
