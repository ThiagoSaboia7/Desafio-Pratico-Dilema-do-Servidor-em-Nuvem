### Parte C — Propondo a Solução

#### C.1 Algoritmo mais adequado: Round-Robin (RR)

##### Como funciona:
* **Fila Circular:** Os processos prontos são organizados em uma estrutura de fila circular.
* **Fatia de Tempo (*Quantum*):** Cada processo recebe a CPU por um tempo máximo pré-determinado (*quantum*), com valores típicos variando de **10 ms a 100 ms**.
* **Interrupção por *Timer*:** Um temporizador de hardware gera uma interrupção assim que o *quantum* expira.
* **Preempção e Troca de Contexto:** Se o processo não terminar dentro do *quantum*, o kernel executa a troca de contexto: salva seu estado, insere-o no final da fila de prontos e atribui a CPU ao próximo processo.
* **Liberdade Antecipada:** Se o processo bloqueia (por E/S) ou termina antes do fim do *quantum*, a CPU é liberada imediatamente para o próximo da fila.

##### Por que resolve o problema:
Com $n$ processos na fila de prontos e um *quantum* $q$, nenhum processo aguarda mais do que **$(n - 1) \times q$** para retornar à CPU. 

> **Exemplo Prático:** Com 5 processos e $q = 50\text{ ms}$, o tempo máximo de espera no pior caso é de apenas $4 \times 50\text{ ms} = 200\text{ ms}$ (em vez de longos 8 segundos no FCFS). O relatório continua progredindo, porém fatiado em pequenos intervalos.

---

##### Comparativo de Algoritmos Alternativos

| Algoritmo | Vantagens | Problemas / Desvantagens neste Cenário |
| :--- | :--- | :--- |
| **SJF**<br>*(Shortest Job First - Não Preemptivo)* | Minimiza o tempo médio de espera global. | Exige estimar o tempo da próxima rajada de CPU. Por ser **não preemptivo**, se um relatório já iniciou a execução, ele continua bloqueando a CPU. Risco de *starvation* de processos longos. |
| **SRTN**<br>*(Shortest Remaining Time Next)* | Excelente para tarefas interativas, pois aplica preempção ao chegar um trabalho mais curto. | Depende de estimativas precisas da rajada de CPU, gera risco de *starvation* para relatórios e possui alto custo computacional para manter a fila ordenada. |
| **Round-Robin**<br>*(Solução Recomendada)* | Não exige conhecimento prévio sobre a duração dos processos; garante tempo de resposta previsível, limitado e justo. | Sensível ao tamanho do *quantum*: se for muito grande, **degenera em FCFS**; se for muito pequeno, gera excesso de trocas de contexto (*overhead*). |

> **Aplicações na Prática:** Kernels modernos utilizam variações do Round-Robin com pesos e prioridades dinâmicas. No Linux, o escalonador **CFS/EEVDF** distribui fatias de CPU proporcionais ao peso (*nice*) de cada tarefa, favorecendo naturalmente processos com baixo uso de CPU (como requisições web). Uma alternativa clássica equivalente é a **Fila Multinível com Realimentação (MLFQ)**.

---

#### C.2 Prioridades, *Starvation* e Como Evitá-la

* **Escalonamento por Prioridades:** Atribui a CPU sempre ao processo pronto de maior prioridade. Definir a interface web com prioridade máxima garante respostas ágeis.
* **Inanição (*Starvation*):** Condição na qual um processo pronto para executar nunca (ou apenas após um período inaceitável) obtém acesso à CPU, pois processos de maior prioridade entram continuamente na fila. Nesse cenário, requisições web ininterruptas poderiam reter os relatórios *batch* na fila indefinidamente.

##### Mecanismos para Evitar a *Starvation*:

1. **Envelhecimento (*Aging*):** Aumenta gradualmente a prioridade de um processo à medida que seu tempo na fila de espera cresce. Ao atingir uma prioridade alta o suficiente, o relatório é executado e sua prioridade retorna ao nível original.
2. **Fila Multinível com Realimentação (MLFQ):** Permite a movimentação de processos entre diferentes filas de acordo com seu comportamento histórico. Processos retidos por muito tempo são promovidos a filas de maior prioridade; processos que consomem todo o *quantum* são rebaixados.
3. **Fatia Garantida de CPU (*Fair-Share*):** Reserva uma porcentagem mínima de processamento para a fila de *batch* (por exemplo, 80% dedicado às requisições interativas e 20% aos relatórios).
4. **Hibridismo de Prioridades com Round-Robin:** Aplica o algoritmo Round-Robin dentro de cada nível de prioridade, garantindo o revezamento entre processos de mesma prioridade.
