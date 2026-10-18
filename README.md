# simulaçao-carro-elétrico-freio-regenerativo
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

#  Modelagem Matemática e Física
## Potência de Tração ($P_{\text{tração}}$) e Força Resistiva ($F_r$)

A potência de tração necessária para manter o veículo em velocidade constante é dada por:

A potência de tração é calculada por:

$$
P_{\mathrm{tração}} = \frac{F_r \cdot V}{3{,}6}
$$

A força de frenagem \(F_r\) varia conforme a velocidade do veículo:

```math
F_r =
\begin{cases}
225\ \text{N}, & \text{se } V < 100\ \text{km/h} \\
450\ \text{N}, & \text{se } 100 \leq V < 120\ \text{km/h} \\
700\ \text{N}, & \text{se } V \geq 120\ \text{km/h}
\end{cases}
```

##
 A potência regenerada pela máquina elétrica durante a frenagem é calculada por:
```math
P_{\text{reg}} = V \cdot \eta \cdot F_{\text{frenagem}}
```


Onde:
- $V$: Velocidade do veículo ($27{,}78 \text{ m/s}$, equivalente a $100 \text{ km/h}$).
- $\eta$: Eficiência global da regeneração ($\eta = 0{,}70$).
- $F_{\text{frenagem}}$: Força de frenagem aplicada pelo sistema ($F_{\text{frenagem}} = 1500 \text{ N}$).
Com esses parâmetros, a potência gerada durante cada pulso de frenagem atinge $P_{\text{reg}} = 27{,}78 \cdot 0{,}70 \cdot 1500 \approx 29{,}17 \text{ kW}$.

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

<img width="1361" height="639" alt="image" src="https://github.com/user-attachments/assets/0641cc15-6ea2-4f7e-a341-accea1015ab9" />


| Parâmetro | Valor / Comportamento durante o Pulso ($3,6\text{s}$) | Interpretação Física |
| :--- | :--- | :--- |
| **Potência $P_{reg}$** | $+29,17 \text{ kW}$ | Motor atua como gerador |
| **Energia Líquida ($\Delta E$)** | $-60 \text{ kJ}$ | Desconto na energia total consumida |
| **Estado de Carga ($\Delta \text{SoC}$)** | +0,0333% | Recarga efetiva por pulso | Elevação no percentual da bateria |

> **Observação :** Em $t = 1800,1 \text{ s}$, o pulso de $P_{reg}$ dispara instantaneamente. A potência líquida torna-se negativa, criando um degrau de redução na energia gasta ($\Delta E \approx -60 \text{ kJ}$), confirmando a sincronização e a precisão do modelo.

### Resultados finais da simulação

Após 1 hora de simulação do veículo elétrico, foram obtidos os seguintes resultados:

<img width="1204" height="312" alt="image" src="https://github.com/user-attachments/assets/99580fd3-7d10-4c57-ad7f-db50a1a4016d" />

- **Energia consumida:**

```math
E_{\mathrm{consumida}} = 12{,}2374\ \text{kWh}
```

- **Estado de carga final da bateria (SoC):**

```math
\mathrm{SoC}_{\mathrm{final}} = 75{,}53\%
```

