### Barreira do Sistema (Mudança de Modo e Transição de Estados)

O processo roda em **Modo Usuário** e não tem permissões para executar instruções de E/S diretamente no hardware. Ele precisa solicitar o serviço ao kernel por meio de uma **Chamada de Sistema (*System Call*)**.

O programador chama uma função de biblioteca (que atua como um *wrapper*). Essa função realiza os seguintes passos:

1. **Preparação dos dados:** Coloca o número da *System Call* e os argumentos nos registradores específicos que o kernel espera.
2. **Mudança de modo:** Executa uma instrução especial de interrupção por software (*trap* / *syscall*), alternando a CPU do **Modo Usuário** para o **Modo Kernel**.
3. **Tratamento e Retorno:** Após a execução do serviço pelo SO, o controle retorna ao programa. A função *wrapper* ajusta a variável `errno` em caso de erro e devolve o resultado final ao programa em **Modo Usuário**.

---

### Transição Detalhada: Modo Usuário vs. Modo Kernel

| Etapa | Modo de Acesso | Fluxo de Execução e Estado do Processo |
| :--- | :--- | :--- |
| **1** | **Modo Usuário** | O processo do relatório chama a função de biblioteca `read()`. |
| **2** | **Usuário $\rightarrow$ Kernel** | A instrução *trap* provoca a mudança de modo de acesso. A CPU salta para um ponto de entrada fixo no kernel (o vetor de interrupções), impedindo que o usuário escolha o endereço de execução. |
| **3** | **Modo Kernel** | O kernel salva o contexto do processo, valida os argumentos e as permissões de acesso, e encaminha a requisição ao sistema de arquivos e ao driver do disco. |
| **4** | **Modo Kernel** | Devido à lentidão do disco, o SO altera o estado do processo de **Executando $\rightarrow$ Bloqueado (Espera)**. O escalonador assume o controle e seleciona outro processo da fila de prontos para usar a CPU. |
| **5** | **Modo Kernel** | Ao concluir a leitura, o controlador do disco gera uma interrupção de hardware. O tratador de interrupção acorda o processo, alterando seu estado de **Bloqueado $\rightarrow$ Pronto**. |
| **6** | **Kernel $\rightarrow$ Usuário** | Quando o escalonador escolhe o processo novamente, o kernel restaura o contexto, comuta a CPU do **Modo Kernel de volta para o Modo Usuário** e devolve os dados lidos ao programa. |
