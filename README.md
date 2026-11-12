
# Smart Home – Programação Socket (Python / TCP)

Sistema cliente/servidor para controlar dispositivos de uma casa inteligente.
Um **servidor central** (multithread) conversa por **TCP** com cinco tipos de
**clientes**: lâmpada, tomada, sensor de presença, ar condicionado e termômetro.

Autores: Thiago Luiz Costa Ferreira Teixeira & Aquillys Rosa Passos

## Arquivos
| Arquivo | Função |
|---|---|
| `protocolo.py` | Código comum: formato das mensagens, envio e leitura |
| `servidor.py` | Central: conexões, regras de automação, registro de consumo |
| `lampada.py` | Cliente lâmpada |
| `ar_condicionado.py` | Cliente ar condicionado |
| `sensor_presenca.py` | Cliente sensor de presença |
| `termometro.py` | Cliente termômetro |
| `tomada.py` | Cliente tomada |
| `consumo.txt` | Arquivo de registro (criado pelo servidor) |

## Protocolo de mensagens
Cada mensagem é um **JSON em uma única linha**, terminada em `\n`
(isso resolve a falta de delimitação de mensagens do TCP).

Campos: `id` (obrigatório), `comando` (obrigatório), `tipo`, `ambiente`, `valor`.

### Cliente → Servidor
| Comando | Quem envia | Exemplo |
|---|---|---|
| `REGISTRO` | todos, ao conectar | `{"id":"lamp01","comando":"REGISTRO","tipo":"lampada","ambiente":"sala"}` |
| `PRESENCA` | sensor de presença | `{"id":"pres01","comando":"PRESENCA","valor":1}` |
| `TEMPERATURA` | termômetro | `{"id":"term01","comando":"TEMPERATURA","valor":30.5}` |
| `CONSUMO` | tomada (a cada 2 min) | `{"id":"tom01","comando":"CONSUMO","valor":4.8}` |

### Servidor → Cliente
| Comando | Destino | Exemplo |
|---|---|---|
| `OK` | todos (confirma o registro) | `{"id":"servidor","comando":"OK"}` |
| `LIGAR` | lâmpada | `{"id":"servidor","comando":"LIGAR"}` |
| `LIGAR` (+ temperatura) | ar condicionado | `{"id":"servidor","comando":"LIGAR","valor":22}` |
| `DESLIGAR` | lâmpada / ar | `{"id":"servidor","comando":"DESLIGAR"}` |

## Regras de automação (no servidor)
1. Presença detectada → liga as lâmpadas **do mesmo ambiente**.
2. Ambiente sem presença por mais de 10 min → desliga as lâmpadas.
3. Temperatura > 28 °C → liga o ar do ambiente, definido em 22 °C.
4. Temperatura ≤ 22 °C → desliga o ar (regra adicional, pois o enunciado não diz quando desligar).

## Minitutorial
Requisitos: Python 3.8+ (apenas bibliotecas padrão). Abra **um terminal por programa**,
dentro da pasta do projeto.

### 1) Executar o servidor (sempre primeiro)
```
python servidor.py
```
### 2) Executar os clientes (cada um em um terminal)
```
python lampada.py --id lamp01 --ambiente sala
python ar_condicionado.py --id ar01 --ambiente sala
python sensor_presenca.py --id pres01 --ambiente sala
python termometro.py --id term01 --ambiente sala
python tomada.py --id tom01 --ambiente sala
```
Opções comuns: `--id`, `--ambiente`, `--host` (padrão 127.0.0.1), `--porta` (padrão 5000).
Dispositivos com o mesmo `--ambiente` interagem entre si.

### 3) Simular eventos
- **Presença:** no terminal do sensor, digite `1`. A lâmpada imprime `ligada`.
- **Temperatura:** no terminal do termômetro, digite `30`. O ar imprime `ligado com temperatura 22 graus`.
  Digite `21` e o ar imprime `desligado`.
- **Sem presença:** após 10 min, a lâmpada imprime `desligada`.

### 4) Visualizar o arquivo de registro
O servidor grava em `consumo.txt`, formato `data hora;id;ambiente;corrente(A)`:
```
cat consumo.txt        # Linux/Mac
type consumo.txt       # Windows
```

### Modo de teste rápido
```
TEMPO_SEM_PRESENCA=10 python servidor.py     # 10 s em vez de 10 min
INTERVALO_TOMADA=5 python tomada.py          # 5 s em vez de 2 min
```
No Windows (PowerShell): `$env:TEMPO_SEM_PRESENCA=10; python servidor.py`
