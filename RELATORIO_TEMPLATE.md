# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
O metodo write gera uma chama de sistema a cada escrita, ja o printf escreve o texto primeiro em um buffer e so chama o sistema quando o buffer fica cheio.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O metodo mais previsivel é o write, pois cada chamada dele gera uma chamada de sistema, diferentemente do printf que depende do tamanho do buffer.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor usado foi o 3. Nao começou em 0, 1 ou 2, pois esses sao reservados pelo sistema então a abertura de um arquivo extra usa o 3.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
A função read() retornou a quantidade de bytes igual ao tamanho do conteúdo do arquivo, mostrando que não há mais dados a serem lidos.  
```

**3. Por que verificar retorno de cada syscall?**

```
Cada retorno de syscall pode falhar, por isso verificamos seus retornos para tratar de possiveis erros e garantindo o funcionamento esperado do programa.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.001524 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |        82       |  0.000115 |
| 64          |        21       |  0.001524 |
| 256         |         6       |  0.000093 |
| 1024        |         2       |  0.000087 |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o buffer menor é o numero de chamadas de sistema, pois cada syscall le mais bytes por vez.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não, apenas a chamada inicial e intermediarias, pois a ultima chamada de read() pode retornar menos bytes dependo de quantos ainda faltam até o final do arquivo
```

**3. Qual é a relação entre syscalls e performance?**

```
Cada syscall envolve uma troca de dados entre o kernel e o usuario e isso gasta um tempo, por isso que quanto maior o numero de syscall's (buffers menores) menor vai ser a performance/eficiência do programa.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000204 segundos
- Throughput: 6529.56 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para poder garantir que todos os bytes foram copiados e gravados no destinos, pois é possivel que a função write() grave menos bytes do que o solicitado por conta de algum eventual erro.
```

**2. Que flags são essenciais no open() do destino?**

```
Os flags sao: O_WRONLY, O_CREAT e O_TRUNC. Elas abrem o arquivo para leitura, cria um arquivo se nao existir e sobrescreve o arquivo se ele ja existir, respectivamente.
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, pois estamos copiando um arquivo para outro, ou seja, para cada read no arquivo original temos um write no arquivo copia
```

**4. Como você saberia se o disco ficou cheio?**

```
O write retornaria um numero menor se comparado com bytes_lidos, indicando que nao há mais espaço em disco e que a escrita não conseguiu gravar os dados.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Se os arquivos continuarem abertos eles continuaram a ocupar os recursos do sistema, isso pode levar a vazamentos de recursos, alem disso os dados podem ficar no buffer do kernel e nao serem gravados no disco corretamenta. 
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls são pedidos do programa para o sistema operacional para executar tarefas que exigem privilégios, com isso o processador muda de modo usuario para modo kernel e executa as terefas pedidas pelo programa, apos isso retorna para o modo usuario.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors são importantes, pois eles permitem que o kernel gerencie os recursos de forma otimizada, independente de serem arquivos ou dispositivos, permitindo um acesso seguro sem que o programa precisa lidar com os detalhes do hardware ou do sistema de arquivos.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o buffer mais dados por chamada de sistema ele transfere reduzindo a quantidade de syscalls e o tempo de conexao entre o usuario e o kernel, ou seja, se tivermos um buffer grande o programa terminara de rodar mais rapido fazendo com que sua performance seja alta.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** Meu programa

**Por que você acha que foi mais rápido?**

```
Acho que meu programa foi mais rapido pois tem menos operaçoes de entrada e saida e tambem tem um buffer grande, reduzindo as comunicaçoes com o sistema e consequentemente o tempo de execução.
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
