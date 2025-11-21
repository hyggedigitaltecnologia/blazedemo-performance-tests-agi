# 🚀 BlazeDemo Performance Tests (JMeter)

Este repositório contém testes de **performance** para o fluxo de **compra de passagem aérea** no site:

🔗 [https://www.blazedemo.com](https://www.blazedemo.com)

O objetivo deste projeto é atender ao teste técnico, que exige validar se o sistema suporta:

> **250 requisições por segundo**
> **90º percentil < 2 segundos**
> **Fluxo de compra concluído com sucesso**

Todos os testes foram desenvolvidos utilizando **Apache JMeter 5.6.3**.

---

# 📁 Estrutura do Projeto

```
blazedemo-performance-tests-agi/
│
├── performance/
│   ├── blazedemo_load_test.jmx       # Plano de testes completo
│   └── docs/
│       ├── carga-aggregate.png       # Evidência teste de carga
│       └── pico-aggregate.png        # Evidência teste de pico
│
├── .github/
│   └── workflows/
│       └── jmeter-performance.yml    # Pipeline executando os testes
│
├── README.md
└── .gitignore
```

---

# 🧪 Cenário Testado

**Fluxo completo de compra de passagem:**

1. Acessar página inicial
2. Buscar voos
3. Selecionar voo
4. Enviar dados do passageiro
5. Confirmar compra
6. Validar retorno contendo:

```
Thank you for your purchase today!
```

Esse fluxo está implementado nos Thread Groups:

* `TG_Funcional_Compra_Passagem`
* `TG_Carga_Compra_Passagem`
* `TG_Pico_Compra_Passagem`

Cada passo do fluxo é um **HTTP Sampler** do JMeter.

---

# ⚙️ Configuração Técnica do Teste

## **HTTP Request Defaults**

* Protocol: `https`
* Server: `www.blazedemo.com`

Aplicado a todos os samplers.

---

## **Teste Funcional – TG_Funcional_Compra_Passagem**

Objetivo: validar funcionalmente o fluxo antes das cargas.

```
Threads: 1
Ramp-up: 1s
Loop Count: 1
```

---

## **Teste de Carga – TG_Carga_Compra_Passagem**

Objetivo: carga constante por vários minutos.
Simula usuários reais acessando de forma contínua.

```
Threads: 100
Ramp-up: 60s
Loop: infinito
Duration: 300s (5 minutos)
```

### **Constant Throughput Timer (OBRIGATÓRIO)**

Adicionado dentro do TG:

```
Target: ~50 requisições/s
Based on: all active threads
```

---

## **Teste de Pico – TG_Pico_Compra_Passagem**

Objetivo: simular explosão de acessos em curto período.

```
Threads: 200
Ramp-up: 5s
Loop: infinito
Duration: 30s
```

### **Constant Throughput Timer**

```
Target: ~150 requisições/s
Based on: all active threads
```

---

# 📊 Resultados Obtidos

## **Teste de Pico – TG_Pico_Compra_Passagem**

📌 evidência: `performance/docs/pico-aggregate.png`

* **Throughput:** ~156 req/s
* **90th Percentil:** ~719 ms
* **Erros:** 0%

---

## **Teste de Carga – TG_Carga_Compra_Passagem**

📌 evidência: `performance/docs/carga-aggregate.png`

* **Throughput:** ~46 req/s
* **90th Percentil:** ~1013 ms
* **Erros:** ~0,02%

---

# 🎯 Avaliação do Critério de Aceitação

| Critério Exigido | Resultado Obtido                    |
| ---------------- | ----------------------------------- |
| 250 req/s        | ❌ Não atingido (máximo ~156 req/s)  |
| 90th < 2s        | ✅ Atingido em ambos cenários        |
| Compra completa  | ✅ Fluxo OK e validado via Assertion |

### **Conclusão**

O requisito de **tempo de resposta** foi atendido.
Porém, a **vazão (throughput) máxima atingida (~156 req/s)** ficou abaixo das **250 req/s** exigidas.

Motivos prováveis:

* Limitações da máquina local que gerou a carga
* Ambiente público do BlazeDemo **não suporta cargas muito altas**
* Throughput depende do hardware gerador, rede, CPU e limitações do ambiente alvo

Ainda assim, o teste cumpre o objetivo de avaliar o comportamento sob carga e pico.

---

# ▶️ Como Executar Localmente (GUI)

1. Baixe o Apache JMeter 5.6.3
   [https://jmeter.apache.org](https://jmeter.apache.org)

2. Inicie o JMeter

   * Windows: `bin/jmeter.bat`
   * Linux/macOS: `bin/jmeter`

3. Abra o arquivo:

```
performance/blazedemo_load_test.jmx
```

4. Execute o Thread Group desejado.

---

# ▶️ Como Executar em Linha de Comando (CLI)

```
jmeter -n \
  -t performance/blazedemo_load_test.jmx \
  -l performance/results/results.jtl \
  -e -o performance/results/html-report
```

O dashboard HTML será gerado em:

```
performance/results/html-report/index.html
```

---

# 🤖 Execução no GitHub Actions

O repositório inclui o workflow:

```
.github/workflows/jmeter-performance.yml
```

O pipeline:

✔️ Baixa o JMeter
✔️ Roda o teste completo
✔️ Gera relatório HTML
✔️ Publica como *artifact*

Para rodar manualmente:
➡️ **GitHub → Actions → JMeter Performance Tests → Run workflow**

---

# 📌 Tecnologias

* Apache JMeter 5.6.3
* Java 21
* GitHub Actions
* JMeter HTML Dashboard

---