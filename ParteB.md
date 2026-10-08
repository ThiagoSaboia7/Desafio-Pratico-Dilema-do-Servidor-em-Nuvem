### Parte B — Diagnosticando o Escalonador

#### B.1 Por que o FCFS "congela" a interface?

No **FCFS (*First-Come, First-Served*)**, a CPU é entregue ao primeiro processo da fila de prontos. Quando um processo *batch* com uma rajada longa de CPU (como o processamento dos dados financeiros) chega primeiro, as requisições web, que necessitam de apenas poucos milissegundos, ficam retidas atrás dele.

Esse fenômeno é conhecido como **Efeito Comboio (*Convoy Effect*)**: processos curtos ficam presos aguardando a finalização de um processo longo.

##### Exemplo Numérico (Ilustrativo)

| Processo | Tempo de Chegada | Tempo de CPU (Burst) |
| :--- | :--- | :--- |
| **Relatório (*batch*)** | 0 s | 8 s |
| **Requisição web 1** | 0,1 s | 0,02 s |
| **Requisição web 2** | 0,2 s | 0,02 s |

No FCFS, a *Requisição web 1* só inicia sua execução em $t = 8\text{ s}$, gerando um **tempo de espera de $\approx 7,9\text{ s}$** para uma tarefa que leva apenas $20\text{ ms}$. O usuário percebe esse atraso como um "congelamento" da interface.

> **Nuance importante:** Quando o relatório executa uma instrução `read()` no disco, ele entra em estado de bloqueio e libera a CPU, permitindo que as requisições web assumam o processamento. O congelamento ocorre durante as **rajadas de CPU entre as leituras** (cálculo ostensivo sobre os dados), que são longas e não sofrem interrupções. Havendo múltiplos relatórios simultâneos, esse efeito se acumula progressivamente.

---

#### B.2 O que significa "Não Preemptivo" e como afeta a CPU aqui?

**Preempção** é a capacidade do Sistema Operacional de retirar a CPU de um processo à força antes que ele termine ou bloqueie voluntariamente — normalmente motivado por uma interrupção de *timer* (fim do *quantum*) ou pela chegada de um processo de maior prioridade.

Um algoritmo **não preemptivo** só retoma a CPU quando o processo atual:

1. Termina sua execução;
2. Bloqueia (por exemplo, ao emitir uma *syscall* de E/S);
3. Cede voluntariamente a CPU (*yield*).

##### Efeito no Cenário

Em um ambiente com apenas um núcleo de processamento, enquanto o relatório está em etapa de cálculo, ele detém a posse exclusiva da CPU. O escalonador fica impossibilitado de interferir, mesmo que existam dezenas de requisições web aguardando na fila de prontos.

* **Resultado:** Tempo de resposta elevado e imprevisível para processos interativos, resultando em uma CPU "monopolizada" por ordem de chegada, em vez de atender a critérios de urgência ou tempo de resposta.
