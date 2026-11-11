# sistema-colonia-fase-3

Simulação de gerenciamento de energia de uma colônia.

O sistema prevê quanta energia eólica será gerada de acordo com o vento, soma com a energia solar, compara com o consumo e decide automaticamente se a colônia deve entrar em modo de economia.

## Como executar

1. Acesse esse link compartilhado: [Sistema Colônia Fase 3](https://colab.research.google.com/drive/1Qsr4rk2PbeUzBZIWz27Ciivgck4Ez3jP);
2. Faça uma cópia para o seu Drive: Arquivo → Salvar uma cópia no Drive.
3. Execute as células na ordem correta: Ambiente de execução → Executar tudo.
4. A saída irá aparecer abaixo da última célula com código, a célula que chama a função `main()`.

### Alterando os dados de entrada

Os dados da colônia ficam no dicionário `colonia`, na segunda célula.

Altere os valores e execute o notebook de novo para ver como as decisões mudam.

O campo `energia.geracao_eolica` começa em 0 e é preenchido pelo próprio programa com o valor previsto.

| Campo                     | Significado                        | Unidade          |
| ------------------------- | ---------------------------------- | ---------------- |
| `clima.vento`             | Velocidade do vento                | km/h             |
| `energia.consumo`         | Consumo atual da colônia           | kW               |
| `clima.temperatura`       | Temperatura                        | °C               |
| `energia.geracao_solar`   | Geração solar                      | kW               |
| `energia.reserva`         | Energia armazenada                 | kWh              |
| `sistemas.suporte_vida`   | Estado do suporte à vida           | ligado/desligado |
| `sistemas.nao_essenciais` | Estado dos sistemas não essenciais | ligado/desligado |

## Como funciona

O programa segue quatro etapas:

1. Previsão da energia eólica (`prever_energia_eolica`): um modelo de regressão linear (scikit-learn) é treinado com poucos dados históricos de vento e energia produzida. Com a velocidade atual do vento, o modelo estima quanta energia eólica será gerada.
2. Análise da energia (`analisar_energia`): soma a geração solar com a eólica prevista e compara com o consumo. O resultado pode ser um alerta de falta de energia, uma sugestão de armazenar o excedente ou a mensagem de que está equilibrado.
3. Tomada de decisões (`tomar_decisoes`): decide o que fazer com os sistemas da colônia, seguindo as regras abaixo.
4. Relatório (`main`): reúne tudo e imprime os dados.

### Regras de decisão

| Condição                                                | Resultado                        | Sistemas não essenciais |
| ------------------------------------------------------- | -------------------------------- | ----------------------- |
| Reserva abaixo de 20 kWh                                | Alerta crítico: ativar economia  | Desligados              |
| Reserva abaixo de 50 kWh ou consumo maior que a geração | Atenção: ativar modo de economia | Desligados              |
| Nenhuma das anteriores                                  | Operação normal                  | Ligados                 |

O **suporte à vida** nunca é desligado, em nenhuma situação.

## Exemplo de entrada e saída

### Entrada

```python
colonia = {
    "clima": {
        "vento": 11,
        "temperatura": 22
    },
    "energia": {
        "geracao_solar": 40,
        "geracao_eolica": 0,
        "consumo": 70,
        "reserva": 100
    },
    "sistemas": {
        "suporte_vida": "ligado",
        "nao_essenciais": "ligado"
    }
}
```

### Saída

```text
========== SMART COLONY ==========

--- DADOS CLIMÁTICOS ---
Vento: 11 km/h
Temperatura: 22 °C

--- SISTEMA ENERGÉTICO ---
Geração solar: 40 kW
Geração eólica prevista: 27.50 kW
Geração total: 67.50 kW
Consumo: 70 kW
Reserva: 100 kWh

--- ANÁLISE ENERGÉTICA ---
ALERTA: consumo maior que geração. Faltam 2.50 kW.

--- DECISÕES AUTOMÁTICAS ---
ATENÇÃO: ativar modo de economia de energia.

--- STATUS DOS SISTEMAS ---
Suporte à vida: ligado
Sistemas não essenciais: desligado

==================================
```

### Por que essa saída?

- Com vento de 11 km/h, o modelo prevê 27,5 kW de energia eólica.
- A geração total é 40 + 27,5 = 67,5 kW. Que é menor que o consumo de 70 kW (faltam 2,5 kW).
- A reserva (100 kWh) está alta, mas como o consumo supera a geração, o programa entra em modo de economia e desliga os sistemas não essenciais.

## Tecnologias usadas

- NumPy
- Python 3
- Google Colab
- scikit-learn (`LinearRegression`)
