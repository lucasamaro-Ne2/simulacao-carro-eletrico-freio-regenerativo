# simulacao-carro-elétrico-freio-regenerativo
Sistema de Controle de Frenagem Regenerativa e Gestão de Energia para Veículo Elétrico (VE)
![MATLAB](https://img.shields.io/badge/MATLAB-R2023a-orange.svg)
![Simulink](https://img.shields.io/badge/Simulink-Model-blue.svg)
![Stateflow](https://img.shields.io/badge/Stateflow-Máquina%20de%20Estados-red.svg)
![Setor](https://img.shields.io/badge/Setor-Automotivo%20%2F%20Engenharia-green.svg)

 # Resumo
Este projeto implementa uma **Máquina de Estados Finitos (FSM)** no **Stateflow** integrada a um modelo dinâmico no **Simulink** para simular e validar o **Sistema de Frenagem Regenerativa** de um veículo elétrico.

O objetivo principal é otimizar a recuperação de energia durante as desacelerações, garantindo a segurança do veículo, limites de carga da bateria e transições suaves entre os modos de condução (*Aceleração*, *Inércia/Coasting* e *Frenagem Regenerativa*).

---

# Arquitetura do Sistema e Lógica de Controle

A lógica opera através de estados discretos baseados nos comandos do condutor (`Acelerador`, `Freio`) e parâmetros operacionais do veículo (`Velocidade, V`).
<img width="705" height="408" alt="image" src="https://github.com/user-attachments/assets/92a19e5d-70dd-4de5-bc8f-902d6e141786" />
### Regras de Transição e Condições de Segurança:
1. **Aceleração → Inércia:** Ativada quando [`Acelerador` >= 0.01 **E** `Freio` == 0].
2. **Inércia → Frenagem Regenerativa:** Ativada quando [`Freio` > 0 **E** `V` > 5 km/h].
3. **Frenagem Regenerativa → Inércia (Condição de Segurança):** Retorna automaticamente para o modo de inércia se o pedal do freio for solto [`Freio` == 0] **OU** se a velocidade cair abaixo do limite operacional [`V` <= 5 km/h], evitando instabilidades ou travamentos em baixas velocidades.

---

##  Modelagem Matemática e Física

A potência regenerada pela máquina elétrica durante a frenagem é calculada por:

```math
P_{\text{reg}} = V \cdot \eta \cdot F_{\text{frenagem}}
```


Onde:

- $V$: Velocidade do veículo ($\mathrm{m/s}$).
- $\eta$: Eficiência da regeneração ($\eta = 0{,}75$).
- $F_{\mathrm{frenagem}}$: Força de frenagem aplicada ($\mathrm{N}$).

### Integração da Energia Líquida

A potência líquida que entra ou sai do pacote de baterias é dada por:

```math
P_{\mathrm{liquida}} = P_{\mathrm{tracao}} - P_{\mathrm{reg}}
```

- **$P_{\mathrm{liquida}} > 0$:** Descarga da bateria (energia consumida pelo motor de tração).
- **$P_{\mathrm{liquida}} < 0$:** Carga da bateria (energia regenerada e devolvida às células).

A energia líquida acumulada ao longo do tempo é calculada pela integral:

```math
E(t) = \int_0^t P_{\mathrm{liquida}}(\tau)\,d\tau
```

Durante a frenagem regenerativa, a potência líquida pode se tornar negativa, reduzindo a energia líquida acumulada e contribuindo para o aumento do SoC (*State of Charge* — estado de carga) da bateria.

##  Validação e Resultados de Simulação

### Configuração do Ciclo de Teste:
* **Tempo Total de Simulação:** $3600 \text{ s}$ ($1 \text{ hora}$)
* **Sinal do Freio:** Gerador de Pulsos ($T = 360 \text{ s}$, Duração = $3,6 \text{ s}$)
* **Velocidade do Veículo:** Constante em $100 \text{ km/h}$ ($27,78 \text{ m/s}$)

### Resultados Observados (Análise no Scope):

| Parâmetro | Valor / Comportamento durante o Pulso ($3,6\text{s}$) | Interpretação Física |
| :--- | :--- | :--- |
| **Potência $P_{reg}$** | $+22,5 \text{ kW}$ | Motor atua como gerador |
| **Energia Líquida ($\Delta E$)** | $-60,48 \text{ kJ}$ | Desconto na energia total consumida |
| **Estado de Carga ($\Delta \text{SoC}$)** | Recarga efetiva por pulso | Elevação no percentual da bateria |

> **Observação Chave:** Em $t = 360,1 \text{ s}$, o pulso de $P_{reg}$ dispara instantaneamente. A potência líquida torna-se negativa, criando um degrau de redução na energia gasta ($\Delta E \approx -60,5 \text{ kJ}$), confirmando a sincronização e a precisão do modelo.

---

